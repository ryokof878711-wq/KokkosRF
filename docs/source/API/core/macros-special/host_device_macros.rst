
``関数アノテーションマクロ``
==============================

.. role::cpp(code)
    :language: cpp

ヘッダー ``<Kokkos_Macros.hpp>``　に定義。

使用例:

.. code-block:: cpp

    KOKKOS_FUNCTION void foo();
    KOKKOS_INLINE_FUNCTION void foo();
    KOKKOS_FORCEINLINE_FUNCTION void foo();
    KOKKOS_RELOCATABLE_FUNCTION void foo();
    auto l = KOKKOS_LAMBDA(int i) { ... };
    auto l = KOKKOS_CLASS_LAMBDA(int i) { ... };

これらのマクロは、デバイスコードとホストコードの分割コンパイルの管理を行います。
それらは、CUDAおよびHIPにおける　``__host__ __device__``　マークアップと同じ目的を果たします。
一般的に、これらのマクロのいずれかでマークされた関数のみが、並列　Kokkos　コード内で使用可能です。
つまり、並列アルゴリズムで実行されるすべてのコードは、これらのマクロのいずれかで
マークされなければなりません。

``KOKKOS_FUNCTION``
-------------------

このマクロは、CUDAおよびHIPにおける　``__host__ __device__``　マークアップに相当します。
クラスおよびテンプレートのフリー関数におけるインライン定義のメンバ関数に、
主に使用します

.. code-block:: cpp

    class Foo {
      public:
        // インライン定義されたコンストラクタ
        KOKKOS_FUNCTION Foo() { ... };

        // インライン定義されたメンバー関数
        template<class T>
        KOKKOS_FUNCTION void bar() const { ... }
    };

    template<class T>
    KOKKOS_FUNCTION void foo(T v) { ... }


``KOKKOS_INLINE_FUNCTION``
--------------------------

このマクロは、CUDAおよびHIPにおける　``__host__ __device__ inline``　マークアップに相当します。
主にテンプレート化されていないフリー関数に使用します:

.. code-block:: cpp

    KOKKOS_INLINE_FUNCTION void foo() {}

このマクロをクラスのインライン定義メンバー関数またはテンプレート化されたフリー関数に使用することはバグではないことに
ご注意ください。 それはデフォルトでインラインになるため、単に冗長なだけです。

``KOKKOS_FORCEINLINE_FUNCTION``
-------------------------------

このマクロは、CUDAおよびHIPにおける　``__host__ __device__``　マークアップに相当しますが、また 
コンパイラ依存のヒント（利用可能な場合）を使用して、インライン化を強制します。
これは頻繁に使用される一部の機能において役立ちますが。 しかし、コンパイル時間に悪影響を及ぼす可能性もあり、
コードの肥大化により実行時のパフォーマンスも低下する恐れがあります。 一部の例においては、``KOKKOS_FORCEINLINE_FUNCTION``　を過剰に使用すると、
コンパイラ固有の最大インライン制限によりコンパイルエラーが発生することさえあります。
このマクロは、広範なパフォーマンスチェックの実施と併せてのみ使用してください。

.. code-block:: cpp

    class Foo {
      public:
        KOKKOS_FORCEINLINE_FUNCTION
        Foo() { ... };

        template<class T>
        KOKKOS_FORCEINLINE_FUNCTION
        void bar() const { ... }
    };

    template<class T>
    KOKKOS_FORCEINLINE_FUNCTION
    void foo(T v) { ... }

``KOKKOS_RELOCATABLE_FUNCTION``
-------------------------------

このマクロは、CUDAおよびHIPにおける　``__host__ __device__``　マークアップ、およびSYCLにおける ``SYCL_EXTERNAL`` に相当します。
同一コンパイル単位でコンパイルされるが、別のコンパイル単位で定義されたKokkos　並列構造から
呼び出される関数に対して使用します。

.. code-block:: cpp

    // functor.cpp
    #include <Kokkos_Macros.hpp>

    KOKKOS_RELOCATABLE_FUNCTION void count_even(const long i, long& lcount) {
      lcount += (i % 2) == 0;
    }

.. code-block:: cpp

    // main.cpp
    #include <Kokkos_Core.hpp>

    KOKKOS_RELOCATABLE_FUNCTION void count_even(const long i, long& lcount);

    int main(int argc, char* argv[]) {
      Kokkos::ScopeGuard scope_guard(argc, argv);
      long count = 0;
      Kokkos::parallel_reduce(
        n, KOKKOS_LAMBDA(const long i, long& lcount) { count_even(i, lcount); },
        count);
    }

このマクロは、Kokkosがホスト実行スペースのみで構成されている場合、またはCUDA、HIP、SYCLバックエンドで
再配置可能デバイスコードサポートが明示的に有効化されている場合にのみ使用可能です。

``KOKKOS_LAMBDA``
-----------------

このマクロは、ラムダ関数用のデフォルトのキャプチャ句とホストデバイスマークアップを提供します。 それは、CUDAおよびHIPにおける
``[=] __host__ __device__`` に相当します。
C++ラムダ式を作成するよりも、Kokkosの並列ディスパッチ機構
``parallel_for``、 ``parallel_reduce`` および　``parallel_scan``　等に渡すために使用されます。

.. code-block:: cpp

    void foo(...) {
      ...
      parallel_for("Name", N, KOKKOS_LAMBDA(int i) {
        ...
      });
      ...
      parallel_reduce("Name", N, KOKKOS_LAMBDA(int i, double& v) {
        ...
      }, result);
      ...
    }

.. warning:: ``KOKKOS_FUNCTION`` などのマークが付いた関数内や、``KOKKOS_LAMBDA`` でマークされたラムダ式内で ``KOKKOS_LAMBDA`` を使用しないでください。 特に、ネストされた並列呼び出し用のラムダ式を定義する際には、　``KOKKOS_LAMBDA``　を使用しないでください。CUDAはこれをサポートしていません。代わりに通常のC++構文を使用してください：``[=] (int i) {...}``。 

.. warning:: クラスメンバ関数内でラムダ式を作成する場合、代わりに　``KOKKOS_CLASS_LAMBDA``　を使用する必要がある場合があります。

``KOKKOS_CLASS_LAMBDA``
-----------------------

このマクロは、クラスメンバ関数内で作成されるラムダ式に対して、デフォルトのキャプチャ句とホストデバイスマークアップを提供します。 
これはCUDAおよびHIPにおける　``[=, *this] __host__ __device__`` に相当し、親クラスを参照ではなく値でキャプチャします。

.. code-block:: cpp

    class Foo {
      public:
        Foo() { ... };
        int data;

        KOKKOS_FUNCTION print_data() const {
          printf("Data: %i\n",data);
        }
        void bar() const {
          parallel_for("Name", N, KOKKOS_CLASS_LAMBDA(int i) {
            ...
            print_data();
            printf("%i %i\n",i,data);
          });
        }
    };

注意事項: ラムダ式内でクラス全体のコピーを取得することを避けたい場合、 アクセスされたデータメンバーのローカルコピーを作成する必要があり、ラムダ関数内では非静的メンバ関数を使用できません:

.. code-block:: cpp

    class Foo {
      public:
        Foo() { ... };
        int data;

        KOKKOS_FUNCTION print_data() const {
          printf("Data: %i\n",data);
        }
        void bar() const {
          int data_copy = data;
          parallel_for("Name", N, KOKKOS_LAMBDA(int i) {
            ...
            // can't call member functions
            // print_data();
            // use the copy of data
            printf("%i %i\n",i,data_copy);
          });
        }
    };


``KOKKOS_DEDUCTION_GUIDE``
--------------------------

このマクロは、ユーザー定義の推論ガイドに注釈を付けるために使用されます。


.. code-block:: cpp

    template<class T, size_t N>
    class Foo {
      T data[N];
      public:
        template<class ... Args>
        KOKKOS_FUNCTION
        Foo(Args ... args):data{static_cast<T>(args)...} {}

        KOKKOS_FUNCTION void print(int i) const {
          printf("%i\n",static_cast<int>(data[i]));
        }
    };

    template<class T, class ... Args>
    KOKKOS_DEDUCTION_GUIDE
    Foo(T, Args...) -> Foo<T, 1+sizeof...(Args)>;

    void bar() {
      Kokkos::parallel_for(1, KOKKOS_LAMBDA(int) {
        Foo f(1, 2., 3.2f);
        f.print(0);
        f.print(1);
        f.print(2);
      });
    }
