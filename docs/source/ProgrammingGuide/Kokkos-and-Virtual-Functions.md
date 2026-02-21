# Kokkos　および 仮想関数

```{警告}
並列領域で仮想関数を使用することは、一般的に推奨されません。 パフォーマンスを低下させることが多く、GPU上で正しく実行するには特定のコードが必要であり、すべてのバックエンドで移植可能ではありません。可能な限り別の方法を使用することを推奨します。
```

GPUプログラミングの特殊性により、Kokkosの並列領域における仮想関数の使用は複雑になり得ます。この文書では、直面する可能性のある問題、その原因、および回避策について説明します。

以下のバックエンドにおいて、仮想関数はデバイス上で実行可能であることに、注意してください:

- Cuda; および
- HIP (最後に説明の通りに、制限あり)。

特に、 SYCL 2020 [仮想関数処理不可能](https://registry.khronos.org/SYCL/specs/sycl-2020/html/sycl-2020.html#architecture).

## 問題

GPU　プログラミングにおいて、デバイスからホスト関数を呼び出すバグに遭遇したことがあるかもしれません。 仮想関数を使用するコードでは、微妙な理由で同様の現象が発生する可能性があります。 次のシリアルコードについて考えてください:

```c++
クラスベース {
  パブリック:
  void Foo() {}

  virtual void Bar() {}
};

派生クラス : パブリックベース {
  public:
  void Bar() オーバーライド {}
};


int main(int argc, char *argv[]) {
  // 作成
  ベース* インスタンス = new Derived();

  // 使用
  for (int i = 0; i < 10; i++) {
    instance->Bar();
  }

  // クリーンアップ
  インスタンス削除;
}
```

このコードは、見た目以上に、GPUへ移植するのが難しいです。
単純なアプローチを使用して、関数を　`KOKKOS_FUNCTION`　でアノテーションし、`for`　ループを　`parallel_for`　に置き換え、`instance`　をGPUメモリにコピーします（具体的な方法は現時点では明示しません）。
Then, we would call 次に、　`Bar()` inside the `parallel_for`　を呼び出します。
一見問題ないように見えますが、`instance`がホスト版の`Bar()`を呼び出すため、通常はクラッシュします。
その理由を理解するには、仮想関数がどのように実装されているかを少し理解する必要があります。

## 仮想テーブル、仮想ポインタ、GPUでは非常に厄介

仮想関数により、プログラムは基底クラスへのポインタを通じて派生クラスを扱い、期待通りの動作を可能にします。これを機能させるには、コンパイラが、名目上は基底クラスへのポインタであるものが、実際に基底クラスへのポインタなのか、それとも派生クラスへのポインタなのかを識別する何らかの方法が必要です。 これはVポインタと仮想関数テーブル（Vtable）を通じて実現されます。仮想関数を持つクラスごとに、全てのインスタンス間で共有される1つの　Vtable　が存在し、このテーブルにはそのクラスが実装する全ての仮想関数への関数ポインタが含まれています。 

![VTable](./figures/VirtualFunctions-VTables.png)

それでは、仮想関数テーブル（Vtable）について説明します。クラスが自身の型を認識できれば、正しい関数を呼び出せます。しかし、どうやって認識するのでしょうか？

Remember that we have one Vtable shared amongst all instances of a type. Each instance, however, has a hidden member called the Vpointer, which the compiler points at construction to the correct Vtable. So a call to a virtual function simply dereferences that pointer, and then indexes into the Vtable to find the precise virtual function called.

![VPointer](./figures/VirtualFunctions-VPointers.png)

Now that we know what the compiler is doing to implement virtual functions, we'll look at why it doesn't work with GPU's.

Credit: the content of this section is adapted from [this article of Pablo Arias](https://pabloariasal.github.io/2017/06/10/understanding-virtual-tables/).

## Then why doesn't the straightforward approach work?

The reason why the straightforward approach described above fails is that when dealing with GPU-compatible classes with virtual functions, there isn't one Vtable, but two. The first holds the host version of the virtual functions, while the second holds the device functions.

![VTableDevice](./figures/VirtualFunctions-VTablesHostDevice.png)

Since we construct the class instance on the host, its Vpointer points to the host Vtable.

![VPointerToHost](./figures/VirtualFunctions-VPointerToHost.png)

We faithfully copied all of the members of the class on the GPU memory, including the Vpointer happily pointing at host functions, which we then call on the device.

## Make it work

The problem here is that we are constructing the instance on the host.
If we were constructing it on the device, we'd get the correct Vpointer, and thus the correct functions.
Note that this would allow to call virtual functions on the device only, not on the host anymore.

Therefore, we first allocate memory on the device, then construct on the device using a technique called [*placement new*](https://en.cppreference.com/w/cpp/language/new#Placement_new):

```cpp
#include <Kokkos_Core.hpp>

class Base {
  public:
  void Foo() {}

  KOKKOS_FUNCTION
  virtual void Bar() {}
};

class Derived : public Base {
  public:
  KOKKOS_FUNCTION
  void Bar() override {}
};

int main(int argc, char *argv[])
{
  Kokkos::initialize(argc, argv);
  {

  // create
  void* deviceInstanceMemory = Kokkos::kokkos_malloc(sizeof(Derived)); // allocate memory on device
  Kokkos::parallel_for("initialize", 1, KOKKOS_LAMBDA (const int i) {
    new (static_cast<Derived*>(deviceInstanceMemory)) Derived(); // initialize on device
  });
  Base* deviceInstance = static_cast<Derived*>(deviceInstanceMemory); // declare on this memory

  // use
  Kokkos::parallel_for("myKernel", 10, KOKKOS_LAMBDA (const int i) {
      deviceInstance->Bar();
  });

  // cleanup
  Kokkos::parallel_for("destroy", 1, KOKKOS_LAMBDA (const int i) {
    deviceInstance->~Base(); // destroy on device
  });
  Kokkos::kokkos_free(deviceInstanceMemory); // free

  }
  Kokkos::finalize();
}
```

We first use the `KOKKOS_FUNCTION` macro to make the methods callable from a kernel.
When creating the instance, note that we introduce a distinction between the *memory* that it uses, and the actual instantiated *object*.
The object instance is constructed on the device, within a single-iteration `parallel_for`, using [placement new](https://en.cppreference.com/w/cpp/language/new#Placement_new).
Since the kernel does not have a return type, we use a static cast to associate the object type with the memory allocation.

For not [trivially destructible](https://en.cppreference.com/w/cpp/language/destructor#Trivial_destructor) objects the destructor must explicitly be called on the device.
After destructing the object in a single-iteration `parallel_for`, the memory allocation can be finally release with `kokkos_free`.

This code is extremely ugly, but leads to functional virtual function calls on the device. The Vpointer now points to the device Vtable.
Remember that those virtual functions cannot be called on the host anymore!

![VPointerToDevice](./figures/VirtualFunctions-VPointerToDevice.png)

For a full working example, see [the example in the repo](https://github.com/kokkos/kokkos/blob/master/example/virtual_functions/main.cpp).

## What if I need a setter that works with host values?

The first problem people run into with this is when they want to set some fields based on host data. As the object instance resides in device memory, it might not be accessible by the host. But the fields can be set within a `parallel_for` on the device. Nevertheless, this requires that the lambda or functor that sets the fields on the device must have access to the host data.
The most productive solution we've found in these cases is to allocate the object instance in `SharedSpace`, which allows to have the object constructed on the device, and then to set fields on the host:

```c++
// create
void* deviceInstanceMemory = Kokkos::kokkos_malloc<Kokkos::SharedSpace>(sizeof(Derived)); // allocate on shared space
// ...
deviceInstance->setAField(someHostValue); // set on host
```

The setter is still called on the host.
Beware that this is only valid for backends that support `SharedSpace`.

Keep in mind that, despite using a "unified" `SharedSpace`, you still have to resort to placement new in order to have the correct Vpointer and hence Vtable on the device!

## But what if I do not really need the Vtables on the device side?

Consider the following example which calls the virtual function `Bar()` on the device from a pointer of derived class type.
One might think this should work because no Vtable lookup on the device is necessary.

```c++
#include <Kokkos_Core.hpp>

class Base {
  public:
  KOKKOS_FUNCTION
  virtual void Bar() const = 0;
};

class Derived : Base {
  public:
  KOKKOS_FUNCTION
  void Bar() const override {
    Kokkos::printf("Hello from Derived\n");
  }

  void apply() {
    Kokkos::parallel_for("myLoop", 10,
      KOKKOS_CLASS_LAMBDA (const size_t i) { this->Bar(); }
    );
  }
};

int main(int argc, char *argv[])
{
  Kokkos::initialize(argc, argv);
  {

  auto derivedPtr = std::make_unique<Derived>();
  derivedPtr->apply();
  Kokkos::fence();

  }
  Kokkos::finalize();
}
```

### Why is this not portable?

Inside the `parallel_for`, `Bar()` is called. As `Derived` derives from the pure virtual class `Base`, the `Bar()` function is marked `override`.
On ROCm (at least up to 6.0) this results in a memory access violation.
When executing the `this->Bar()` call, the runtime looks into the Vtable and dereferences a host function pointer on the device.

### But if that is the case, why does it work with NVCC?

Notice that the `parallel_for` is called from a pointer of type `Derived` and not a pointer of type `Base` pointing to an `Derived` object.
Thus, no Vtable lookup for the `Bar()` would be necessary as it can be deduced from the context of the call that it will be `Derived::Bar()`.
But here it comes down to how the compiler handles the lookup. NVCC understands that the call is coming from an `Derived` object and thinks: "Oh, I see, that you are calling from an `Derived` object, I know it will be the `Bar()` in this class scope, I will do this for you".
ROCm, on the other hand, sees the call and thinks "Oh, this is a call to a virtual method, I will look that up for you", failing to dereference the host function pointer in the host virtual function table.

### How to solve this?
Strictly speaking, the observed behavior on NVCC is an optimization that uses the context information to avoid the Vtable lookup.
If the compiler does not apply this optimization, you can help in different ways by providing additional information. Unfortunately, none of these strategies are fully portable to all backends.

- Tell the compiler not to look up any function name in the Vtable when calling `Bar()` by using [qualified name lookup](https://en.cppreference.com/w/cpp/language/qualified_lookup). For this, you tell the compiler which function you want by spelling out the class scope in which the function should be found e.g. `this->Derived::Bar();`. This behavior is specified in the C++ standard. Nevertheless, some backends are not fully compliant to the standard.
- Changing the `override` to `final` on the `Bar()` in the `Derived` class. This tells the compiler `Bar()` is not changing in derived objects. Many compilers do use this in optimization and deduce which function to call without the Vtable. Nevertheless, this might only work with certain compilers, as this effect of adding `final` is not specified in the C++ standard.
- Similarly, the entire derived class `Derived` can be marked `final`. This is compiler dependent too, for the same reasons.

## Questions/Follow-up

This is intended to be an educational resource for our users. If something doesn't make sense, or you have further questions, you'd be doing us a favor by letting us know on [Slack](https://kokkosteam.slack.com) or [GitHub](https://github.com/kokkos/kokkos).
