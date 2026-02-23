ビュー: 多次元配列
============================

本章を読んだ後に、以下を理解する必要があります:

* Kokkos Viewは、ゼロ個以上の次元を持つ配列。
* ビューの最初のテンプレートパラメータを使用して、エントリの型、次元の数、および次元が実行時かコンパイル時に決定されるかを指定する方法
* Kokkos　は配列の割り当て処理の解除を自動的に行います
* Kokkosは、コンピューターアーキテクチャに応じて、最適で全体的なパフォーマンスを実現するため、コンパイル時に配列レイアウトを選択
* 実行領域やレイアウトの低レベル制御についてのビューのオプションテンプレートパラメータの設定方法について、および　Kokkos　が配列要素にアクセスする方法

本章のすべてのコード例において、`Kokkos`　名前空間内のすべてのクラスが、作業名前空間にインポート済みであることを前提としています。


Kokkos が多次元配列を必要とする理由
----------------------------------------

多くの科学技術計算コードでは、データの配列を用いた計算に多くの時間を費やしており、プログラマーはこれらの配列計算を可能な限り高速化するために多大な努力を注いでいます。 この取り組みは、多くの場合に、コンピュータアーキテクチャ、実行環境、言語、プログラミングモデルの詳細と密接に結びついています。例えば、 最適な配列レイアウトは、アーキテクチャによって異なる可能性があり、誤った場合には、大きな整数倍のペナルティが生じます。 ポインタのアライメント、配列のレイアウト、インデックス処理のオーバーヘッド、初期化等、低レベルの問題が、いずれもパフォーマンスに影響を及ぼします。 これは、順次コードにおいてさえも真ですが、スレッド並列処理では、ファーストタッチ割り当てや偽の共有等、さらに多くの落とし穴を生じます。

最高のパフォーマンスを得るためには、コーダーは、配列の管理方法の詳細を、並列コードがそれらの配列にアクセスし管理する方法の詳細と結びつける必要があります。 アーキテクチャ固有のコードを記述するプログラマーは、その後、アーキテクチャとプログラミングモデルの低レベルな特徴を、高レベルのアルゴリズムに組み込む必要があります。　これにより、アーキテクチャ間でコードを移植することが難しくなり、また、アーキテクチャが進化しても良好なパフォーマンスを維持するコードを書くことが難しくなります。

Kokkos　は、特定のアーキテクチャ向けに配列の管理とアクセスを最適化することで、この負担の一部を軽減することを目指しております配列を共有メモリ並列処理に結びつけることで、Kokkos　は、前者を後者に最適化することが可能となります。例えば、Kokkosは配列の初期化に使用できるスレッドを制御できるため、ファーストタッチ割り当てを容易に行うことができます。 Kokkos　のアーキテクチャ認識機能により、適切な配置とパディング割り当てを自動的に選択し、整列を最適化します。 熟練したコーダーによる、Kokkos　の活用により、さらにユーザーフレンドリーな方法で、低レベルあるいはアーキテクチャ固有の最適化にアクセスすることも可能です。 例えば、Kokkosでは様々な配列レイアウトを、簡単に試すことができます。

ビューの作成および使用
-------------------------

.. _Constructing_a_view:

ビュー構築
~~~~~~~~~~~~~~~~~~~

ビューとは、ゼロ個以上の次元からなる配列です。プログラマーは、ビューの型の一部として、コンパイル時にエントリの型と次元の数を両方とも設定します。 例えば、以下は、型が　`int`　であるエントリに対して、4次元のビューを指定し割り当てます:

.. code-block:: c++

  const size_t N0 = ...;
  const size_t N1 = ...;
  const size_t N2 = ...;
  const size_t N3 = ...;
  Kokkos::View<int****> a ("一部のレベル", N0, N1, N2, N3);

文字列引数は、Kokkosがデバッグに使用するラベルです。 異なるビューが同じラベルを持つ場合があります。 省略記号は、実行時に指定される整数次元の値を示しています。 ユーザーは、コンパイル時に一部の次元を設定することも可能です。 例えば、以下のビューには二つの次元がありますが、最初の次元（アスタリスクで示されています）は実行時次元であり、二番目の次元（[3]で示されています）はコンパイル時次元です。　このように、 ビューは、N×3　の　double　型の配列であり、ここで　N　は、ビューのコンストラクタ内で実行時に指定されます。

.. code-block:: c++

  const size_t N = ...;
  Kokkos::View<double*[3]> b ("another label", N);

ビューは、最大で8次元まで持つことができ、これらのうち任意の数が実行時またはコンパイル時に定義される可能性があります。 唯一の制限事項は、実行時次元（存在する場合）を最初に記述し、その後すべてのコンパイル時次元（存在する場合）を記述しなければならないという点です。 例えば、以下のものが、有効な三次元ビュータイプです:

* `View<int***>`  (3 run-time dimensions)
* `View<int**[8]>`  (2 run-time, 1 compile-time)
* `View<int*[3][8]>`  (1 run-time, 2 compile-time)
* `View<int[4][3][8]>`  (3 compile-time)

以下のものは、有効な三次元ビュータイプでは、*ありません*:

* `View<int[4]**>`
* `View<int[4][3]*>`
* `View<int[4]*[8]>`
* `View<int*[3]*>`

この制限は、ビューが　C++　テンプレートを用いて実装されていることに起因します。ビューの最初のテンプレートパラメータは、有効な　C++　型でなければなりません。

なお、上記で使用したコンストラクタは、すべてのビュータイプで必ずしも利用可能とは限りません; 特定のレイアウトやメモリ空間には、より専門的なアロケーターが必要となる場合があります. これについては、後ほど説明します。

もう一つ重要な点は、`ビュー`　ハンドルは、状態を保持するオブジェクトであるということです。 ポインタの型変換によって、生のメモリから`View`ハンドルを作成することは、法的に認められていません。 `ビュー` の演算子（代入演算子を含む）を呼び出すには、事前にそのコンストラクタが呼び出されている必要があります。 生のメモリ領域を、`ビュー`　ハンドルで初期化する必要がある場合、配置newを使用して合法的に行うことが可能です。 上記の内容は、`ビュー`　が参照しているデータとは、一切関係ありません。 管理対象外の`ビュー`　のコンストラクタに型キャストされたポインタを渡すことは、完全に法的に正当なものです。

.. code-block:: c++

  Kokkos::View<int*> *a_ptr = (Kokkos::View<int*>*) malloc(10*sizeof(View<int*);
  a_ptr[0] = Kokkos::View<int*>("A0",1000); // これは違法
  new(&a_ptr[1]) Kokkos::View<int*>("A1",10000); // これは合法

.. _view_types_of_data:

ビューにはどのような種類のデータを含めることができますか？
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

C++　では、ユーザーが、構文上は数値のように見えるかもしれませんが、内部では任意に複雑な処理を実行するデータ型を構築することが可能です。 それらの処理の中には、グローバル状態への保護されていない更新等、スレッドセーフでないものも含まれる可能性があります。 他のものは、並行処理において、例えば細かい動的メモリ割り当て等、うまく機能しない場合があります。　そのため、Kokkos　のビュー内では、シンプルなデータ型のみの使用を強く推奨します。ユーザーは、常にエントリが以下の形式であるビューを構築することが可能です

* `int`　および　`double`　等、組み込みデータ型（"プレーン・オールド・データ"）、あるいは
* 組み込みデータ型の構造体。

原則として、任意のオブジェクトに対して　Kokkos　ビューを作成することは可能ですが、Kokkos　では　`View<T*>`　を構築できる型　`T`　の集合に対して制限を設けています。例えば:

* `T` 、仮想メソッドを保有してはいけません
* `T`のデフォルトコンストラクタおよびデストラクタは、データの割り当てや割り当て解除を行ってはならず、スレッドセーフである必要があります。 
* `T`　の代入演算子ならびにデフォルトコンストラクタおよびデコンストラクタは、`KOKKOS_INLINE_FUNCTION` または `KOKKOS_FUNCTION` マクロでマークする必要があります。

これらの制限はすべて、`View<T*>` がすべての実行環境およびメモリ空間で動作するという要件に、起因しています。 `View<T*>`　のコンストラクタは、単にメモリを割り当てるだけではありません; デフォルトでは、各エントリに対して　`T`　のデフォルト値で割り当てを初期化します。 したがって、`T`　のデフォルトコンストラクタは、`ビュー`　の　`MemorySpace`　に関連付けられた　`ExecutionSpace`　を呼び出すために正しく設定されている必要があります。結果として生成される　`ビュー`　の意味論は、`ビューの 'ビュー'`　のセマンティクスおよび要素型の動作の組み合わせであることに、留意してください。

`T`　のデストラクタがメモリの割り当てを解除するという要件は、技術的に　`T`　が、管理対象のビューであること、または直接的・間接的に管理対象のビューを含む構造体であることを禁止します。極端なケースにおいては、`View<T>` 自体の割り当て解除前に、各 `T` に含まれるビューを安全に解除するための非並列ループが使用されている限りにおいて、ユーザーが型 `T` に対して管理対象ビューを保持することを許可しております。これは、各内包ビューに対して、同じ型のデフォルトコンストラクタで生成されたビューを割り当てることにより、可能となります。`T`内でビューを管理対象とすることは、推奨されません。

最後に、仮想関数は技術的には許可されていますが、それらを呼び出す際にはさらなる制限の対象となる点に、注意してください; デベロッパーの方は、第13章　「Kokkosと仮想関数」（開発中）の議論を参照してください。

Can I make a View of Views?
~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. warning::

  NOVICES: THE ANSWER FOR YOU IS "NO."  PLEASE SKIP THIS SECTION.  

A "View of Views" is a special case of View, where the type of each entry is itself a View. It is possible to make this, but before you try, please see below.

You probably don't want this
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

If you really just want a multidimensional array, please don't do this.  Instead, see :ref:`Constructing_a_view` above for the correct syntax.

If you want to represent an array of arrays, and the inner arrays have fixed length or a fixed upper bound on length, consider instead using a *compressed sparse row* data structure. Kokkos' Containers subpackage has a `StaticCrsGraph` class that you may use for this purpose.

If you want a hash table, Kokkos' Containers subpackage has an `UnorderedMap` class that you may use for this purpose.

One reason you might *actually* want a View of Views is because you need a representation of a "ragged" array of arrays -- where the inner arrays have widely varying length -- and you need to be able to reallocate the inner arrays dynamically.

You might also want a View of some class that itself contains Views. If you want this, first think about how to reorganize your data structures for better efficiency.

What's the problem with a View of Views?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

A View of Views would have an "outer View," with zero or more "inner Views." :ref:`view_types_of_data` above explains how the outer View's constructor would work.  The outer View's constructor does not just allocate memory; it also initializes the allocation with `T`'s default value for each entry. If the View's execution space is `Cuda`, then that means the entry type's default constructor needs to be correct to call on device. That is a problem, because the entry type in this case is itself `View`. `View`'s constructor wants to allocate memory, and thus does not work on device. If the outer `View` does not allow access on Host, one must go through extra mechanisms to allocate the inner `View` (e.g. a host mirror of the outer `View`). Kokkos parallel regions generally forbid memory allocation.

You could create the outer View without initializing, like this:

.. code-block:: c++

  using Kokkos::View;
  using Kokkos::view_alloc;
  using Kokkos::WithoutInitializing;

  // Need an std::string here, because the compiler may get confused
  // if you pass view_alloc a char* as its first argument.
  const std::string label ("v_outer");
  View<View<int*>> v_outer (view_alloc (label, WithoutInitializing));

However, that leaves the inner Views in an undefined state.  You can't legally assign to them or call their destructors.  (Remember that View assignment updates the assignee's reference count.)  You'll need to do more than just this in order to create valid inner Views and ensure their safe deallocation.

You'll have worse problems if the outer View's memory space is `CudaSpace`.  Allocating View construction must run on host in order to allocate memory, but you won't be able to assign the resulting constructed View to any element of the outer View.

Another issue is that View construction in a Kokkos parallel region does not update the View's reference count.  Thus, the inner Views must be created in sequential host code, not inside of a `Kokkos::parallel_*`.

I really want a View of Views; what do I do?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Here is how to create a View of Views, where each inner View has a separate owning allocation:

1. The outer View must have a memory space that is both host and device accessible, such as :cpp:type:`SharedSpace`.
2. Create the outer View using the :cpp:type:`SequentialHostInit` property.
3. Create inner Views in a sequential host loop.  (Prefer creating the inner Views uninitialized.  Creating the inner Views initialized launches one device kernel per inner View.  This is likely much slower than just initializing them all yourself from a single kernel over the outer View.)
4. At this point, you may access the outer and inner Views on device.
5. Get rid of the outer View as you normally would.

Here is an example:

.. code-block:: c++

  using Kokkos::SharedSapce;
  using Kokkos::View;
  using Kokkos::view_alloc;
  using Kokkos::SequentialHostInit;
  using Kokkos::WithoutInitializing;

  using inner_view_type = View<double*>;
  using outer_view_type = View<inner_view_type*, SharedSpace>;

  const int numOuter = 5;
  const int numInner = 4;
  outer_view_type outer (view_alloc (std::string ("Outer"), SequentialHostInit), numOuter);

  // Create inner Views on host, outside of a parallel region, uninitialized
  for (int k = 0; k < numOuter; ++k) {
    const std::string label = std::string ("Inner ") + std::to_string (k);
    outer(k) = inner_view_type (view_alloc (label, WithoutInitializing), numInner);
  }

  // Outer and inner views are now ready for use on device

  Kokkos::RangePolicy<> range (0, numOuter);
  Kokkos::parallel_for ("my kernel label", range,
      KOKKOS_LAMBDA (const int i) {
        for (int j = 0; j < numInner; ++j) {
          device_outer(i)(j) = 10.0 * double (i) + double (j);
        }
      }
    });
  Kokkos::fence();

  // Destroy the View of Views - this will call destructors sequentially on the host!
  outer = outer_view_type ();

Another approach is to create the inner Views as nonowning, from a single pool of memory. This makes it unnecessary to invoke their destructors.

.. warning::

  `SequentialHostInit` was added in version 4.4.01. Prior to that the process was more involved.

1. The outer View must have a memory space that is both host and device accessible, such as `SharedSpace`.
2. Create the outer View without initializing it.
3. Create inner Views using placement new, in a sequential host loop.  (Prefer creating the inner Views uninitialized.  Creating the inner Views initialized launches one device kernel per inner View.  This is likely much slower than just initializing them all yourself from a single kernel over the outer View.)
4. At this point, you may access the outer and inner Views on device.
5. Before deallocating inner Views, fence to ensure all device kernels that access them have finished.
6. Destroy the inner Views explicitly.  (Otherwise, Step 7 will leak the inner Views' memory.)
7. Get rid of the outer View as you normally would.

Here is an example:

.. code-block:: c++

  using Kokkos::SharedSpace;
  using Kokkos::View;
  using Kokkos::view_alloc;
  using Kokkos::WithoutInitializing;

  using inner_view_type = View<double*>;
  using outer_view_type = View<inner_view_type*, SharedSpace>;

  const int numOuter = 5;
  const int numInner = 4;
  outer_view_type outer (view_alloc (std::string ("Outer"), WithoutInitializing), numOuter);

  // Create inner Views on host, outside of a parallel region, uninitialized
  for (int k = 0; k < numOuter; ++k) {
    const std::string label = std::string ("Inner ") + std::to_string (k);
    new (&outer(k)) inner_view_type (view_alloc (label, WithoutInitializing), numInner);
  }

  // Outer and inner views are now ready for use on device

  Kokkos::RangePolicy<> range (0, numOuter);
  Kokkos::parallel_for ("my kernel label", range, 
      KOKKOS_LAMBDA (const int i) {  
        for (int j = 0; j < numInner; ++j) {
          device_outer(i)(j) = 10.0 * double (i) + double (j);
        }
      }
    });

  // Fence before deallocation on host, to make sure 
  // that the device kernel is done first.
  Kokkos::fence ();

  // Destroy inner Views, again on host, outside of a parallel region.
  for (int k = 0; k < 5; ++k) {
    outer(k).~inner_view_type ();
  }

  // You're better off disposing of outer immediately.
  outer = outer_view_type ();

Const Views
~~~~~~~~~~~

A view can have const data semantics (i.e. its entries are read-only) by specifying a `const` data type. It is a compile-time error to assign to an entry of a "const View". Assignment semantics are equivalent to a pointer to const data. A const View means the *entries* are const; you may still assign to a const View. `View<const double*>` corresponds exactly to `const double*`, and `const View<double*>` to `double* const`. Therefore, it does not make sense to allocate a const View since you could not obtain a non-const view of the same data and you can not assign to it. You can however assign a non-const view to a const view. Here is an example:

.. code-block:: c++

  const size_t N0 = ...;
  Kokkos::View<double*> a_nonconst ("a_nonconst", N0);

  // Assign a nonconst View to a const View
  Kokkos::View<const double*> a_const = a_nonconst;
  // Pass the const View to some read-only function.
  const double result = readOnlyFunction (a_const);

Const Views often enables the compiler to optimize more aggressively by allowing it to reason about possible write conflicts and data aliasing. For example, in a vector update `a(i+1)+=b(i)` with skewed indexing, it is safe to vectorize if `b` is a View of const data.

Accessing entries (indexing)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

You may access an entry of a View using parentheses enclosing a comma-delimited list of integer indices. This looks just like a Fortran multidimensional array access. For example:

.. code-block:: c++

  const size_t N = ...;
  Kokkos::View<double*[3][4]> a ("some label", N);
  // KOKKOS_LAMBDA macro includes capture-by-value specifier [=].
  Kokkos::parallel_for (N, KOKKOS_LAMBDA (const ptrdiff_t i) {
    const size_t j = ...;
    const size_t k = ...;
    const double a_ijk = a(i,j,k);
    /* rest of the loop body */
  });

Note how in the above example, we only access the View's entries in a parallel loop body. In general, you may only access a View's entries in an execution space which is allowed to access that View's memory space. For example, if the default execution space is `Cuda`, a View for which no specific Memory Space was given may not be accessed in host code [#footnotecudauvm]_.

Furthermore, access costs (e.g., latency and bandwidth) may vary, depending on the View's "native" memory and execution spaces and the execution space from which you access it. CUDA UVM may work, but it may also be slow, depending on your access pattern and performance requirements. Thus, best practice is to access the View only in a Kokkos parallel for, reduce, or scan, using the same execution space as the View. This also ensures that access to the View's entries respect first-touch allocation. The first (leftmost) dimension of the View is the *parallel dimension* over which it is most efficient to do parallel array access if the default memory layout is used (e.g. if no specific memory layout is
specified).

.. [#footnotecudauvm] An exemption is if you specified for CUDA compilation that the default memory space is CudaUVMSpace, which can be accessed from the host.


Reference counting
~~~~~~~~~~~~~~~~~~

Kokkos automatically manages deallocation of Views through a reference-counting mechanism.  Otherwise, Views behave like raw pointers. Copying or assigning a View does a shallow copy, and changes the reference count. (The View copied has its reference count incremented, and the assigned-to View has its reference count decremented.) A View's destructor (called when the View falls out of scope or during a stack unwind due to an exception) decrements the reference count. Once the reference count reaches zero, Kokkos may deallocate the View.

For example, the following code allocates two Views, then assigns one to the other. That assignment may deallocate the first View, since it reduces its reference count to zero. It then increases the reference count of the second View, since now both Views point to it.

.. code-block:: c++

  Kokkos::View<int*> a ("a", 10);
  Kokkos::View<int*> b ("b", 10);
  a = b; // assignment does shallow copy

For efficiency, View allocation and reference counting turn off inside of Kokkos' parallel for, reduce, and scan operations. This affects what you can do with Views inside of Kokkos' parallel operations.

Lifetime
~~~~~~~~

The lifetime of an allocation begins when a View is constructed by an allocating constructor such as

.. code-block:: c++

  Kokkos::View<int*> b("b", 10);

The lifetime of an allocation ends when there are no more Views which reference that allocation (see reference counting above).

Kokkos requires that the lifetime of all allocations ends before the call to :ref:`Kokkos::finalize<kokkos_finalize>`.

For example, the following is incorrect usage of Kokkos:

.. code-block:: c++

  int main() {
    Kokkos::initialize();
    Kokkos::View<double*> p("constructed view", 100);
    Kokkos::finalize();
    // p is destroyed here, after Kokkos::finalize
  }

Resizing
~~~~~~~~

Kokkos Views can be resized using the `resize` non-member function. It takes an existing view as its input by reference and the new dimension information corresponding to the constructor arguments. A new view with the new dimensions will be created and a kernel will be run in the view's execution space to copy the data element by element from the old view to the new one. Note that the old allocation is only deleted if the view to be resized was the *only* view referencing the underlying allocation.

.. code-block:: c++

  // Allocate a view with 100x50x4 elements
  Kokkos::View<int**[4]> a( "a", 100,50);
      
  // Resize a to 200x50x4 elements; the original allocation is freed
  Kokkos::resize(a, 200,50);
      
  // Create a second view b viewing the same data as a
  Kokkos::View<int**[4]> b = a;
  // Resize a again to 300x60x4 elements; b is still 200x50x4
  Kokkos::resize(a,300,60);

Layout
------

Strides and dimensions
~~~~~~~~~~~~~~~~~~~~~~

*Layout* refers to the mapping from a logical multidimensional index *(i, j, k, . . .)* to a physical memory offset. Different programming languages may have different layout conventions. For example, Fortran uses *column-major* or "left" layout, where consecutive entries in the same column of a 2-D array are contiguous in memory. Kokkos calls this `LayoutLeft`. C, C++, and Java use *row-major* or "right" layout, where consecutive entries in the same row of a 2-D array are contiguous in memory. Kokkos calls this `LayoutRight`.

The generalization of both left and right layouts is "strided." For a strided layout, each dimension has a *stride*. The stride for that dimension determines how far apart in memory two array entries are, whose indices in that dimension differ only by one, and whose other indices are all the same. For example, with a 3-D strided view with strides *(s_1, s_2, s_3)*, entries *(i, j, k)* and *(i, j+1, k)* are *s_2* entries (not bytes) apart in memory. Kokkos calls this `LayoutStride`.

Strides may differ from dimensions. For example, Kokkos reserves the right to pad each dimension for cache or vector alignment. You may access the dimensions of a View using the (ISO/C++ form) `extent` method, which takes the index of the dimension.

Strides are accessed using the `stride` method. It takes a raw integer array, and only fills in as many entries as the rank of the View. For example:

.. code-block:: c++

  const size_t N0 = ...;
  const size_t N1 = ...;
  const size_t N2 = ...;
  Kokkos::View<int***> a ("a", N0, N1, N2);
      
  int dim1 = a.extent (1); // returns dimension 1
  size_t strides[3]
  a.stride (strides); // fill 'strides' with strides

.. code-block:: c++

  const size_t n0 = a.extent (0);
  const size_t n2 = a.extent (2);

Note the return type of `extent(N)` is the `size_type` of the views memory space. This causes some issues if warning-free compilation should be achieved since it will typically be necessary to cast the return value. In particular, in cases where the `size_type` is more conservative than required, it can be beneficial to cast the value to `int` since signed 32-bit integers typically give the best performance when used as index types. In index heavy codes, this performance difference can be significant compared to using `size_t` since the vector length on many architectures is twice as long for 32 bit values as for 64 bit values and signed integers have less stringent overflow testing requirements than unsigned integers.

Users of the BLAS and LAPACK libraries may be familiar with the ideas of layout and stride. These libraries only accept matrices in column-major format. The stride between consecutive entries in the same column is 1, and the stride between consecutive entries in the same row is `LDA` ("leading dimension of the matrix A"). The number of rows may be less than `LDA`, but may not be greater.

Other layouts
~~~~~~~~~~~~~

Other layouts are possible.  For example, Kokkos has a "tiled" layout, where a tile's entries are stored contiguously (in either row- or column-major order) and tiles have compile-time dimensions. One may also use Kokkos to implement Morton ordering or variants thereof. In order to write a custom layout one has to define a new layout class and specialise the `ViewMapping` class for that layout. The `ViewMapping` class implements the offset operator as well as stride calculation for regular layouts. A good way to start such a customization is by copying the implementation of `LayoutLeft` and its associated `ViewMapping` specialization, renaming the layout and then change the offset operator.

Default layout depends on execution space
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Kokkos selects a View's default layout for optimal parallel access over the leftmost dimension based on its execution space. For example, `View<int**, Cuda>` has `LayoutLeft`, so that consecutive threads in the same warp access consecutive entries in memory. This *coalesced access* gives the code better memory bandwidth.

In contrast, `View<int**, OpenMP>` has `LayoutRight`, so that a single thread accesses contiguous entries of the array. This avoids wasting cache lines and helps prevent false sharing of a cache line between threads. In :ref:`Managing_Data_Placement` more details will be discussed.

Explicitly specifying layout
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

We prefer that users let Kokkos determine a View's layout, based on its execution space. However, sometimes you really need to specify the layout. For example, the BLAS and LAPACK libraries only accept column-major arrays.  If you want to give a View to the BLAS or LAPACK library, that View must be `LayoutLeft`. You may specify the layout as a template parameter of View. For example:

.. code-block:: c++

  const size_t N0 = ...;
  const size_t N1 = ...;
  Kokkos::View<double**, Kokkos::LayoutLeft> A ("A", N0, N1);
      
  // Get 'LDA' for BLAS / LAPACK
  int strides[2]; // any integer type works in stride()
  A.stride (strides);
  const int LDA = strides[1];

You may ask a View for its layout via its `array_layout` typedef. This can be helpful for C++ template metaprogramming. For example:

.. code-block:: c++

  template<class ViewType>
  void callBlas (const ViewType& A) {
    typedef typename ViewType::array_layout array_layout;
    if (std::is_same<array_layout, LayoutLeft>::value) {
      callSomeBlasFunction (A.data(), ...);
    } else {
      throw std::invalid_argument ("A is not LayoutLeft");
    }
  }

.. _Managing_Data_Placement:

Managing Data Placement
-----------------------

Memory spaces
~~~~~~~~~~~~~

Views are allocated by default in the default execution space's default memory space. You may access the View's execution space via its `execution_space` typedef, and its memory space via its `memory_space` typedef. You may also specify the memory space explicitly as a template parameter. For example, the following allocates a View in CUDA device memory:

.. code-block:: c++

  Kokkos::View<int*, Kokkos::CudaSpace> a ("a", 100000);

and the following allocates a View in "host" memory, using the default host execution space for first-touch initialization:

.. code-block:: c++

  Kokkos::View<int*, Kokkos::HostSpace> a ("a", 100000);

Since there is no bijective association between execution spaces and memory spaces, Kokkos provides a way to explicitly provide both to a View as a `Device`.

.. code-block:: c++

  Kokkos::View<int*, Kokkos::Device<Kokkos::Cuda,Kokkos::CudaUVMSpace> > a ("a", 100000);
  Kokkos::View<int*, Kokkos::Device<Kokkos::OpenMP,Kokkos::CudaUVMSpace> > b ("b", 100000);

In this case `a` and `b` will live in the same memory space, but `a` will be initialized on the GPU while `b` will be
initialized on the host. The `Device` type can be accessed as a view's `device_type` typedef. A `Device` has only three typedef members: `device_type`, `execution_space` and `memory_space`. The `execution_space` and `memory_space` typedefs are the same for a view as the `device_type` typedef.

It is important to understand that accessibility of a View does not depend on its execution space directly. It is only determined by its memory space. Therefore both `a` and `b` have the same access properties. They differ only in how they are initialized and in where parallel kernels associated with operations such as resizing or deep copies are run.

The following is the accessibility matrix for execution and memory spaces:

.. csv-table::

  ,Serial, OpenMP, Threads, Cuda
  HostSpace,           :octicon:`check` , :octicon:`check` , :octicon:`check` , :octicon:`x`     ,
  CudaSpace,           :octicon:`x`     , :octicon:`x`     , :octicon:`x`     , :octicon:`check` ,
  CudaUVMSpace,        :octicon:`check` , :octicon:`check` , :octicon:`check` , :octicon:`check` ,
  CudaHostPinnedSpace, :octicon:`check` , :octicon:`check` , :octicon:`check` , :octicon:`check` ,

This relationship can be queried via the `SpaceAccessibility` class:

.. code-block:: c++

  template< typename AccessSpace , typename MemorySpace >
  struct SpaceAccessibility {
    enum { accessible };  // AccessSpace can access MemorySpace
    enum { assignable };  // Can assign View<...,AccessSpace,...> = View<...,MemorySpace,...>
    enum { deep_copy };  // Can deep copy to AccessSpace::memory_space from MemorySpace
  };

A typical use case would be:

.. code-block:: c++

  if(SpaceAccessibility<ExecSpace, ViewType::memory_space>::accessible) {
     parallel_for(RangePolicy<ExecSpace>, functor);
  }

Initialization
~~~~~~~~~~~~~~

A View's entries are initialized to zero by default. Initialization happens in parallel for first-touch allocation over the first (leftmost) dimension of the View using the execution space of the View.

You may allocate a View without initializing. For example:

.. code-block:: c++

  Kokkos::View<int*> x (Kokkos::view_alloc(Kokkos::WithoutInitializing, label), 100000);

This is mainly useful in cases when the initial values of the view are not important because
they will be overwritten without ever being read.
It is still important that the first write to each location be done within a parallel kernel
in a way that reflects how first-touch affinity to threads is desired.
Typically it is sufficient to use the parallel iteration index as the index of the location in the
view to write to.

.. warning::

  :cpp:`WithoutInitialization` implies that the destructor of each element of the :cpp:`View` **will not be called**.
  For instance, if the :cpp:`View`'s value type is not trivially destructible,
  you **should not use** :cpp:`WithoutInitialization` unless you are taking care of calling the destructor manually before the :cpp:`View` deallocates its memory.

  The mental model is that whenever placement new is used to call the constructor, the destructor also isn't called before the memory is deallocated but it needs to be called manually.

Deep copy and HostMirror
~~~~~~~~~~~~~~~~~~~~~~~~

Copying data from one view to another, in particular between views in different memory spaces, is called deep copy.
Kokkos never performs a hidden deep copy. To do so a user has to call the `deep_copy` function. For example:

.. code-block:: c++

  Kokkos::View<int*> a ("a", 10);
  Kokkos::View<int*> b ("b", 10);
  Kokkos::deep_copy (a, b); // copy contents of b into a

Deep copies can only be performed between views with an identical memory layout and padding. For example the following two operations are not valid:

.. code-block:: c++

  Kokkos::View<int*[3], Kokkos::CudaSpace> a ("a", 10);
  Kokkos::View<int*[3], Kokkos::HostSpace> b ("b", 10);
  Kokkos::deep_copy (a, b); // This will give a compiler error

  Kokkos::View<int*[3], Kokkos::LayoutLeft, Kokkos::CudaSpace> c ("c", 10);
  Kokkos::View<int*[3], Kokkos::LayoutLeft, Kokkos::HostSpace> d ("d", 10);
  Kokkos::deep_copy (c, d); // This might give a runtime error

The first one will not work because the default layouts of `CudaSpace` and `HostSpace` are different. The compiler will catch that since no overload of the `deep_copy` function exists to copy view from one layout to another. The second case will fail at runtime if padding settings are different for the two memory spaces. This would result in different allocation sizes and thus prevent a direct memcopy.

The reasoning for allowing only direct bitwise copies is that a deep copy between different memory spaces would otherwise require a temporary copy of the data to which a bitwise copy is performed followed by a parallel kernel to transfer the data element by element.

Kokkos provides the following way to work around those limitations. Firstly, views have a `HostMirror` typedef which is a view type with compatible layout inside the `HostSpace`. Additionally, there is a `create_mirror` and `create_mirror_view` function which allocate views of the `HostMirror` type of view. The difference between the two is that `create_mirror` will always allocate a new view, while `create_mirror_view` will only create a new view if the original one is not in `HostSpace`.

.. code-block:: c++

  Kokkos::View<int*[3], MemorySpace> a ("a", 10);
  // Allocate a view in HostSpace with the layout and padding of a
  typename Kokkos::View<int*[3], MemorySpace>::HostMirror b =
      create_mirror(a);
  // This is always a memcopy
  Kokkos::deep_copy (b, a);
      
  typename Kokkos::View<int*[3]>::HostMirror c =
  Kokkos::create_mirror_view(a);
  // This is a no-op if MemorySpace is HostSpace
  Kokkos::deep_copy (c, a)

How do I get the raw pointer?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

We discourage access to a View's "raw" pointer. This circumvents reference counting, that is, the memory may be deallocated once the View's reference count goes to zero so holding on to a raw pointer may result in invalid memory access. Furthermore, it may not even be possible to access the View's memory from a given execution space. For example, a View in the `Cuda` space points to CUDA device memory. Also using raw pointers would normally defeat the usability of polymorphic layouts and automatic padding. Nevertheless, for instances where you really need access to the pointer, we provide the `data()` method. For example:

.. code-block:: c++

  // Legacy function that takes a raw pointer.
  extern void legacyFunction (double* x_raw, const size_t len);
    
  // Your function that takes a View.
  void myFunction (const Kokkos::View<double*>& x) {
    // DON'T DO THIS UNLESS YOU MUST
    double* x_raw = x.data();
    const size_t N = x.extent(0);
    legacyFunction (x_raw, N);
  }

A user is in most cases also allowed to obtain a pointer to a specific element via the usual `&` operator. For example

.. code-block:: c++

  // Legacy function that takes a raw pointer.
  void someLibraryFunction (double* x_raw);
      
  KOKKOS_INLINE_FUNCTION
  void foo(const Kokkos::View<double*>& x) {
    someLibraryFunction(&x(3));
  }

This is only valid if a Views reference type is an `lvalue`. That property can be queried statically at compile time from the view through its `reference_type_is_lvalue` member.

Memory access traits
--------------------

Another way to get optimized data accesses is to specify memory traits. These traits are used to declare intended use of the particular view of an allocation. For example, a particular kernel might use a view only for streaming writes. By declaring that intention, Kokkos can insert the appropriate store intrinsics on each architecture if available. Access traits are specified through an optional template parameter which comes last in the list of parameters. Multiple traits can be combined with binary OR operators:

.. code-block:: c++

  Kokkos::View<double*, Kokkos::MemoryTraits<SomeTrait> > a;
  Kokkos::View<const double*, Kokkos::MemoryTraits<SomeTrait | SomeOtherTrait> > b;
  Kokkos::View<int*, Kokkos::LayoutLeft, Kokkos::MemoryTraits<SomeTrait | SomeOtherTrait> > c;
  Kokkos::View<int*, MemorySpace, Kokkos::MemoryTraits<SomeTrait | SomeOtherTrait> > d;
  Kokkos::View<int*, Kokkos::LayoutLeft, MemorySpace, Kokkos::MemoryTraits<SomeTrait> > e;

Unmanaged Views
~~~~~~~~~~~~~~~

.. _MemoryTraits: ../API/core/view/memoryTraits.html

.. |MemoryTraits| replace:: memory traits

It's always better to let Kokkos control memory allocation, but sometimes you don't have a choice. You might have to work with an application or an interface that returns a raw pointer, for example. Kokkos lets you wrap raw pointers in an *unmanaged View*. "Unmanaged" means that Kokkos does *neither* reference counting *nor* automatic deallocation for those Views. The following example shows how to create an unmanaged View of host memory. You may do this for CUDA device memory too, or indeed for memory allocated in any memory space, by specifying the View's execution or memory space accordingly. Note that the pointer to the allocation has to be provided to the constructor.

We would like to highlight that in Kokkos, Views are managed by default. In other words, if a View is not created as an unmanaged View, then it is managed, irrespective of other memory traits. Thus, an explicit memory trait for managed Views (with an alias called ``Kokkos::MemoryManaged``), has been deprecated in Kokkos 4.7. Since, it has no practical value. See the API reference on |MemoryTraits|_. 

.. code-block:: c++

  // Sometimes other code gives you a raw pointer, ...
  const size_t N0 = ...;
  double* x_raw = malloc (N0 * sizeof (double));
  {
    // ... but you want to access it with Kokkos.
    //
    // malloc() returns host memory, so we use the host memory space HostSpace.  
    // Unmanaged Views have no label because labels work with the reference counting system.
    Kokkos::View<double*, Kokkos::HostSpace, Kokkos::MemoryTraits<Kokkos::Unmanaged> >
      x_view (x_raw, N0);
  
    functionThatTakesKokkosView (x_view);
    
    // It's safest for unmanaged Views to fall out of scope before freeing their memory.
  }
  free (x_raw);

Random Access
~~~~~~~~~~~~~

The `RandomAccess` trait declares the intent to access a View irregularly (in particular non consecutively). If the default execution space is `Cuda`, access to a `RandomAccess` View may use `CUDA` texture fetches. In more detail, if used for a ``const`` View in the `CudaSpace` or `CudaUVMSpace`, Kokkos will use texture fetches for accesses when executing in the `Cuda` execution space. For example:

.. code-block:: c++

  const size_t N0 = ...;
  Kokkos::View<int*> a_nonconst ("a", N0); // allocate nonconst View
  // Assign to const, RandomAccess View
  Kokkos::View<const int*, Kokkos::MemoryTraits<Kokkos::RandomAccess>> a_ra = a_nonconst;

Note that texture fetches are not cache-coherent with respect to writes, so you must use read-only access. The texture cache is optimized for noncontiguous access since it has a shorter cache line than the regular cache.

While `RandomAccess` is valid for other execution spaces, currently no specific optimizations are performed. But in the future a view allocated with the `RandomAccess` attribute might for example, use a larger page size, and thus reduce page faults in the memory system.

.. _Atomic: ../API/core/atomics.html

.. |Atomic| replace:: Atomic

|Atomic|_ Access
~~~~~~~~~~~~~~~~

The `Atomic` access trait lets you create a View of data such that every read or write to any entry uses an atomic update. Kokkos supports atomics for all data types independent of size. Restrictions are that you are

#. not allowed to alias data for which atomic operations are performed, and 
#. the results of non-atomic accesses (including read) to data which is at the same time atomically accessed is not defined.

Performance characteristics of atomic operations depend on the data type. Some types (in particular integer types) are natively supported and might even provide asynchronous atomic operations. Others (such as 32 bit and 64 bit atomics for non-integer types) are often implemented using compare-and-swap (CAS) loops of integers. Everything else is implemented with a locking approach where an atomic operation acquires a lock based on a hash of the pointer value of the data element.

Types for which atomic access are performed must support the necessary operators such as =, +=, -=, +, - etc. as well as have a number of `volatile` overloads of functions such as assign and copy constructors defined. 

.. code-block:: c++

  Kokkos::View<int*> a("a" , 100);
  Kokkos::View<int*, Kokkos::MemoryTraits<Kokkos::Atomic> > a_atomic = a;
      
  a_atomic(1) += 1; // This access will do an atomic addition

Restrict
~~~~~~~~

The `Restrict` trait indicates that the memory of this View doesn't alias/overlap with another data structure in the current scope. This enables compiler optimizations.

Aligned
~~~~~~~

Allocation of Kokkos Views is 64-byte aligned. The exception being the allocation of unmanaged Views, which may or may not be aligned. The `Aligned` trait can be used to indicate to the compiler that it can expect the memory allocation of the View to be aligned by 64-bytes. The compiler can perform optimizations accordingly.

Note that it is not possible to specify this trait for sub-Views. Sub-Views may or not be aligned depending on their parent View. Assigning a View with the `Aligned` trait to an unaligned sub-View will lead to a run-time error.

Standard idiom for specifying access traits
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The standard idiom for View is to pass it around using as few template parameters as possible. Then, assign to a View with the desired access traits only at the "last moment" when those access traits are needed just before entering a computational kernel. This lets you template C++ classes and functions on the View type without proliferating instantiations. Here is an example:

.. code-block:: c++

  // Compute a sparse matrix-vector product, for a sparse
  // matrix stored in compressed sparse row (CSR) format.
  void spmatvec (const Kokkos::View<double*>& y,
        const Kokkos::View<const size_t*>& ptr,
        const Kokkos::View<const int*>& ind,
        const Kokkos::View<const double*>& val,
        const Kokkos::View<const double*>& x)
  {
    // Access to x has less locality than access to y.
    Kokkos::View<const double*, Kokkos::MemoryTraits<Kokkos::RandomAccess>> x_ra = x;
    typedef Kokkos::View<const size_t*>::size_type size_type;
      
    Kokkos::parallel_for (y.extent (0), KOKKOS_LAMBDA (const size_type i) {
      double y_i = y(i);
      for (size_t k = ptr(i); k < ptr(i+1); ++k) {
        y_i += val(k) * x_ra(ind(k));
      }
      y(i) = y_i;
    });
  }

Conversion Rules and Function Specialization
--------------------------------------------

Not all view types can be assigned to each other. Requirements are:

* the data type and dimension have to match, 
* the layout must be compatible and 
* the memory space has to match.

Examples illustrating the rules are:

#. Data Type and Rank has to Match

   .. code-block:: c++

    int*       -> int*       // ok
    int*       -> const int* // ok
    const int* -> int*       // not ok, const violation
    int**      -> int*       // not ok, rank mismatch
    int*[3]    -> int**      // ok
    int**      -> int*[3]    // ok if runtime dimension check matches
    int*       -> long*      // not ok, type mismatch

#. Layouts must be compatible

   .. code-block:: c++

    LayoutRight  -> LayoutRight   // ok
    LayoutLeft   -> LayoutRight   // not ok except for 1D Views
    LayoutLeft   -> LayoutStride  // ok
    LayoutStride -> LayoutLeft    // ok if runtime dimensions allow assignment

#. Memory Spaces must match

   .. code-block:: c++

    Kokkos::View<int*> -> Kokkos::View<int*,HostSpace> // ok if default memory space is HostSpace

#. Memory Traits
