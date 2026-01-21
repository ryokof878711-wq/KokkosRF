``ReducerConcept``
==================

.. role:: cpp(code)
    :language: cpp

Reducerの概念とは、並列　Reduce　実行パターンにおいて　"Reduction"　がどのように実行されるかを定義する抽象化です。Reducer　の概念とは、並列　Reduce　実行パターンの間に　Reduce　が　"how"　（どのように）　実行されるかを定義する抽象化である。抽象化　"what"　はテンプレート引数として与えられ、`parallel_reduce <../parallel-dispatch/parallel_reduce.html>`_ 操作において還元対象となる　"what" に対応します。本ページでは、仮説的な　'Reducer'クラス定義を用いたリデューサーに予想される定義と関数について説明します。組み込みリデューサーについての簡単なディスクリプションも含まれています。

ヘッダーファイル: ``<Kokkos_Core.hpp>``

使用例
-----

.. code-block:: cpp

    T result;
    parallel_reduce(N,Functor,ReducerConcept<T>(result));


概要
--------

.. code-block:: cpp

    クラスリデューサー {
        public:
            //Required for Concept
            typedef Reducer reducer;
            typedef ... value_type;
            typedef Kokkos::View<value_type, ... > result_view_type;

            KOKKOS_INLINE_FUNCTION
            void join(value_type& dest, const value_type& src) const;

            KOKKOS_INLINE_FUNCTION
            value_type& reference() const;

            KOKKOS_INLINE_FUNCTION
            result_view_type view() const;

            //Optional
            KOKKOS_INLINE_FUNCTION
            void init(value_type& val) const;

            KOKKOS_INLINE_FUNCTION
            void final(value_type& val) const;

            //Part of Build-In reducers for Kokkos
            KOKKOS_INLINE_FUNCTION
            Reducer(value_type& value_);

            KOKKOS_INLINE_FUNCTION
            Reducer(const result_view_type& value_);
    };

パブリッククラスメンバー
--------------------

型定義
~~~~~~~~

* ``reducer``: 自己型。
* ``value_type``: 還元スカラー型。
* ``result_view_type``: 削減結果を配置すべき場所を参照する ``Kokkos::View``。 スカラーまたは複素データ型（クラスまたは構造体）の管理対象外ビューとなる可能性があります。管理対象外ビューは、参照されるスカラー（または複素データ型）が存在するのと同じメモリ領域を指定する必要があります。

コンストラクタ
~~~~~~~~~~~~

コンストラクタは概念の一部ではありません。 カスタムリデューサーは、複雑なユーザー定義コンストラクターを持つことができます。 Kokkosのすべてのビルトインリデューサーは、以下の2つのコンストラクタを持ちます:

.. cpp:function:: KOKKOS_INLINE_FUNCTION Reducer(value_type& value_);

    * 結果の保存先としてローカル変数を参照するリデューサーを構築します。

.. cpp:function:: KOKKOS_INLINE_FUNCTION Reducer(const result_view_type& value_);

    * 結果の保存先として特定ビューを参照するリデューサーを構築します。

関数
~~~~~~~~~

.. cpp:function:: KOKKOS_INLINE_FUNCTION void join(value_type& dest, const value_type& src) const;

    * ``src``　を into ``dest``　に組み合わせます。 例えば、``Add`` は、 ``dest+=src;``　を実行します。

.. cpp:function:: KOKKOS_INLINE_FUNCTION void init(value_type& val) const;

    * 適切な初期値で　``val``　を初期化するオプションコールバック。　例えば、'Add'　は、``val = 0;``を代入するが、Prod　は、``val = 1;``　を代入します。
      デフォルトではデフォルトコンストラクタの呼び出しを行います。

.. cpp:function:: KOKKOS_INLINE_FUNCTION void final(value_type& val) const;

    * 結果 ``val`` を変更するオプションのコールバック。 デフォルトでは何もしない。

.. cpp:function:: KOKKOS_INLINE_FUNCTION value_type& reference() const;

    * Returns a reference to the result place.結果の保存先への参照を返します。

.. cpp:function:: KOKKOS_INLINE_FUNCTION result_view_type view() const;

    * 結果の保存先のビューを返します。

必要条件
~~~~~~~~~~~~

リデューサーは、使用される値型に関して可交換半群を定義すると予測されますが、すなわち、それは二元演算です。

.. code-block:: cpp

    value_type op(const value_type& val1, const value_type& val2) {
      value_type result = val1;
      reducer.join(result, val2);
      return result;
    }

is commutative and associative with identity element that can be set by calling ``reducer.init(el)``.


組み込みリデューサー
~~~~~~~~~~~~~~~~~

Kokkos provides a number of built-in reducers that automatically work with the intrinsic C++ types as well as ``Kokkos::complex``. In order to use a Built-in reducer with a custom type, a template specialization of ``Kokkos::reduction_identity<CustomType>`` must be defined. A simple example is shown below and more information can be found under `Custom Reductions <../../../ProgrammingGuide/Custom-Reductions.html>`_.

* `Kokkos::BAnd <BAnd.html>`_
* `Kokkos::BOr <BOr.html>`_
* `Kokkos::LAnd <LAnd.html>`_
* `Kokkos::LOr <LOr.html>`_
* `Kokkos::Max <Max.html>`_
* `Kokkos::MaxLoc <MaxLoc.html>`_
* `Kokkos::Min <Min.html>`_
* `Kokkos::MinLoc <MinLoc.html>`_
* `Kokkos::MinMax <MinMax.html>`_
* `Kokkos::MinMaxLoc <MinMaxLoc.html>`_
* `Kokkos::Prod <Prod.html>`_
* `Kokkos::Sum <Sum.html>`_

Examples
--------

.. code-block:: cpp

    #include<Kokkos_Core.hpp>

    int main(int argc, char* argv[]) {

        long N = argc>1 ? atoi(argv[1]):100;
        long result;
        Kokkos::parallel_reduce("ReduceSum: ", N, KOKKOS_LAMBDA (const int i, long& lval) {
            lval += i;
        }, Sum<long>(result));

        printf("Result: %l Expected: %l\n",result,N*(N-1)/2);
    }
