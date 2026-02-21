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

ある型のすべてのインスタンス間で、1つの仮想関数テーブル（Vtable）が共有されていることを覚えておいてください。 ただし、各インスタンスにはVポインタと呼ばれる隠れたメンバーが存在し、コンパイラはコンストラクタ実行時にこれを正しいVテーブルを指すように設定します。したがって、仮想関数の呼び出しは単にそのポインタを間接参照し、Vテーブルをインデックス付けして呼び出される正確な仮想関数を見つけます。

![VPointer](./figures/VirtualFunctions-VPointers.png)

仮想関数を実装するためにコンパイラが何をしているのかがわかったので、次に、なぜそれが　GPU　では機能しないのかを見ていきましょう。

Credit: the content of this section is adapted from [this article of Pablo Arias]クレジット：このセクションの内容は[Pablo Ariasのこの記事]を改訂したものです
(https://pabloariasal.github.io/2017/06/10/understanding-virtual-tables/).

## では、なぜ直接的なアプローチがうまくいかないのか？

上記の単純なアプローチが失敗する理由は、仮想関数を持つGPU互換クラスを扱う場合、仮想関数テーブル（Vtable）が1つではなく2つ存在するためです。 最初のものはホスト版の仮想関数を保持し、2つ目はデバイス関数を保持します。

![VTableDevice](./figures/VirtualFunctions-VTablesHostDevice.png)

クラスインスタンスをホスト上で構築するため、その Vpointer はホストの Vtable を指します。

![VPointerToHost](./figures/VirtualFunctions-VPointerToHost.png)

クラスメンバー全体をGPUメモリに忠実にコピーし、ホスト関数を指し示すVポインタも含まれ、その後、デバイス上でこれらの関数を呼び出します。

## 解決策

ここでの問題点は、インスタンスをホスト上で構築していることです。
デバイス上で構築している場合、正しいVポインタを取得できるため、正しい関数が得られます。
この変更により、仮想関数は、デバイス側でのみ呼び出せるようになり、ホスト側では呼び出せなくなります。

したがって、まずデバイス上でメモリを割り当て、次に[*配置型　new*](https://en.cppreference.com/w/cpp/language/new#Placement_new)　と呼ばれる手法を用いてデバイス上で構築します:

```cpp
#include <Kokkos_Core.hpp>

クラスベース {
  パブリック:
  void Foo() {}

  KOKKOS_FUNCTION
  virtual void Bar() {}
};

class Derived : パブリックベース {
  public:
  KOKKOS_FUNCTION
  void Bar() override {}
};

int main(int argc, char *argv[])
{
  Kokkos::initialize(argc, argv);
  {

  // 作成
  void* deviceInstanceMemory = Kokkos::kokkos_malloc(sizeof(Derived)); // デバイス上でメモリを割り当て
  Kokkos::parallel_for("initialize", 1, KOKKOS_LAMBDA (const int i) {
    new (static_cast<Derived*>(deviceInstanceMemory)) Derived(); // デバイス上で初期化
  });
  Base* deviceInstance = static_cast<Derived*>(deviceInstanceMemory); // 本メモリ上で宣言

  // 使用
  Kokkos::parallel_for("myKernel", 10, KOKKOS_LAMBDA (const int i) {
      deviceInstance->Bar();
  });

  // クリーンアップ
  Kokkos::parallel_for("destroy", 1, KOKKOS_LAMBDA (const int i) {
    deviceInstance->~Base(); // destroy on device
  });
  Kokkos::kokkos_free(deviceInstanceMemory); // フリー

  }
  Kokkos::finalize();
}
```

まず、`KOKKOS_FUNCTION` マクロを使用して、カーネルからメソッドを呼び出せるようにします
インスタンスを作成する際、そのインスタンスが使用する　*メモリ*　と、実際にインスタンス化された　*オブジェクト*　との区別を導入することに、注意してください。
オブジェクトインスタンスは、デバイス上で単一反復の`parallel_for`　内で、[配置新規]　(https://en.cppreference.com/w/cpp/language/new#Placement_new)　を使用して構築されます
カーネルには戻り値の型がないため、オブジェクトの型とメモリ割り当てを関連付けるために、静的キャストを使用します。

[破棄可能な]　(https://en.cppreference.com/w/cpp/language/destructor#Trivial_destructor)　オブジェクトの場合、デストラクタはデバイス上で明示的に呼び出さなければなりません。
単一反復の　`parallel_for`　でオブジェクトを破棄した後、`kokkos_free`　でメモリ割り当てを最終的に解放できます。

このコードは非常に見苦しいですが、デバイス上で仮想関数の呼び出しを機能させます。Vpointerは現在、デバイスのVtableを指しています。
それらの仮想関数は、もはやホスト上で呼び出せなくなることに注意してください！

![VPointerToDevice](./figures/VirtualFunctions-VPointerToDevice.png)

完全な動作例については、 [ repo　内の例](https://github.com/kokkos/kokkos/blob/master/example/virtual_functions/main.cpp)　を参照してください。

## ホスト値と連動するセッターが必要な場合はどうするべきですか？

この方法を使う際に最初に直面する問題は、ホストデータに基づいて特定のフィールドを設定したい場合です。オブジェクトインスタンスは、デバイスメモリに存在するため、ホストからアクセスできない可能性があります。　ただし、フィールドはデバイス上の`parallel_for`　内で設定できます。 しかしながら、これには、デバイス上でフィールドを設定するラムダ式またはファンクタがホストデータにアクセスできる必要があります。
これらのケースで私たちが発見した最も効率的な解決策は、オブジェクトインスタンスを　`SharedSpace`　に割り当てることです。これにより、オブジェクトをデバイス上で構築した後、ホスト上でフィールドを設定することが可能になります:

```c++
// 作成
void* deviceInstanceMemory = Kokkos::kokkos_malloc<Kokkos::SharedSpace>(sizeof(Derived)); //　共有スペース上に配置
// ...
deviceInstance->setAField(someHostValue); // ホスト上に設定
```

セッターは依然としてホスト上で呼び出されます。
これは、`SharedSpace`　をサポートするバックエンドでのみ有効であることに注意してください。

"統一された"　`SharedSpace`　を使用しているにもかかわらず、デバイス上で正しい　Vpointer（そしてそれゆえの Vtable ）を得るためには、依然として配置 new に頼らなければならないことに注意してください！

## では、デバイス側で　Vtable が本当に必要ない場合はどうでしょうか？

以下の例を考えてみましょう。これは、派生クラス型のポインタからデバイスに対して仮想関数　`Bar()`　を呼び出しています。
One might think this should work because no Vtable lookup on the device is necessary.

```c++
#include <Kokkos_Core.hpp>

クラスベース {
  パブリック:
  KOKKOS_FUNCTION
  virtual void Bar() const = 0;
};

派生クラス : ベース {
  パブリック:
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

### なぜこれが移植可能ではないのでしょうか？

parallel_for　内で、Bar()　が呼び出されます。 As `Derived` derives from the pure virtual class `Base`, the `Bar()` function is marked `override`.`Derived`　が純粋仮想クラス　`Base`　から派生しているため、`Bar()`　関数は　`override`　としてマークされています。
ROCm（少なくとも6.0まで）では、これによりメモリアクセス違反が発生します。
呼び出しを実行する際、ランタイムは　Vtable　を参照し、デバイス上のホスト関数ポインタを間接参照します。

### しかし、その場合には、なぜNVCCでは動作するのでしょうか？

`parallel_for` は、`Derived` オブジェクトを指す `Base` 型のポインタではなく、`Derived` 型のポインタから呼び出されていることに注意してください。
したがって、呼び出しの文脈から、それが `Derived::Bar()` であることが推測できるため、`Bar()` の Vtable 参照は不要となります。
しかし、ここでの問題はコンパイラがルックアップをどのように処理するかです。NVCCは、呼び出しが`Derived`オブジェクトから来ていることを理解し、次のように考えます: 「ああ、なるほど、`Derived`オブジェクトから呼び出しているのですね。このクラススコープでは`Bar()`になるのは承知しています。代わりに私が対応します。」
一方、ROCm　はこの呼び出しを見て、「ああ、これは仮想メソッドへの呼び出しだ。代わりに調べてあげよう」と考え、ホスト仮想関数テーブル内のホスト関数ポインタの参照解除に失敗します。

### これをどうやって解決するのでしょう？
厳密に言えば、NVCCで観察される動作は、コンテキスト情報を利用して仮想テーブルのルックアップを回避する最適化です。
コンパイラがこの最適化を適用しない場合、追加情報を提供することで様々な方法で支援できます。 残念ながら、これらの戦略はいずれも、すべてのバックエンドに完全に移植できるものではありません。

- `Bar()`　を呼び出す際に、Vtable　内の関数名を検索しないようコンパイラに指示するには、[qualified name lookup](https://en.cppreference.com/w/cpp/language/qualified_lookup)　を使用します。 このため、コンパイラにどの関数を使用したいかを、例：`this->Derived::Bar()`;　等、その関数が存在するクラススコープを明示的に記述することで伝えます。この動作は、C++　標準で規定されています。しかしながら、一部のバックエンドは、標準に完全には準拠していません。
- `Derived`　クラス内の　`Bar()`　メソッドの　`override`　を　`final`　に変更します。 これは、`Bar()`　が変更されていないことをコンパイラに伝えます。 
- ただし、この効果は、C++　標準で規定されていないため、特定のコンパイラでのみ機能する可能性があります。
- 同様に、派生クラス `Derived` 全体を `final` とマークすることも可能です。これも同じ理由でコンパイラ依存となります。

## 問題/フォローアップ

これは、当社ユーザー向けの教育リソースとして提供されるものです。何か不明点や追加の質問がある場合は、[Slack](https://kokkosteam.slack.com) または [GitHub](https://github.com/kokkos/kokkos) 上で知らせていただければ幸いです。
