``BAnd``
========

.. role:: cpp(code)
    :language: cpp

ビット単位の　``AND``　演算を実行する `ReducerConcept <ReducerConcept.html>`_ 具体的な実装

ヘッダーファイル: ``<Kokkos_Core.hpp>``

使用例
-----

.. code-block:: cpp

    T result;
    parallel_reduce(N,Functor,BAnd<T,S>(result));

概要
--------

.. code-block:: cpp

    template<class Scalar, class Space>
    class BAnd{
        public:
            typedef BAnd reducer;
            typedef typename std::remove_cv<Scalar>::type value_type;
            typedef Kokkos::View<value_type, Space> result_view_type;

            KOKKOS_INLINE_FUNCTION
            void join(value_type& dest, const value_type& src) const;

            KOKKOS_INLINE_FUNCTION
            void init(value_type& val) const;

            KOKKOS_INLINE_FUNCTION
            value_type& reference() const;

            KOKKOS_INLINE_FUNCTION
            result_view_type view() const;

            KOKKOS_INLINE_FUNCTION
            BAnd(value_type& value_);

            KOKKOS_INLINE_FUNCTION
            BAnd(const result_view_type& value_);
    };

インターフェイス
---------

.. cpp:class:: template<class Scalar, class Space> BAnd

   .. rubric:: パブリック型。

   .. cpp:type:: リデューサー

      自分型

   .. cpp:type:: value_type

      還元スカラー型。

   .. cpp:type:: result_view_type

      還元結果を参照する ``Kokkos::View`` 

   .. rubric:: コンストラクタ

   .. cpp:function:: KOKKOS_INLINE_FUNCTION BAnd(value_type& value_);

      結果の位置としてローカル変数を参照するリデューサを構築します。

   .. cpp:function:: KOKKOS_INLINE_FUNCTION BAnd(const result_view_type& value_);

      特定のビューを結果の位置として参照するリデューサーを構築します。

   .. rubric:: パブリックメンバー関数

   .. cpp:function:: KOKKOS_INLINE_FUNCTION void join(value_type& dest, const value_type& src) const;

     ``src``　の ``and``　および　``dest`` を ``dest``:  ``dest = src & dest;``　にビット単位で格納します。

   .. cpp:function:: KOKKOS_INLINE_FUNCTION void init(value_type& val) const;

      ``Kokkos::reduction_identity<Scalar>::land()`` メソッドを使用して、``val``　を初期化します。 デフォルト実装は、``val=~(0)``　を設定します。

   .. cpp:function:: KOKKOS_INLINE_FUNCTION value_type& reference() const;

      クラスコンストラクタで提供された結果への参照を返します。

   .. cpp:function:: KOKKOS_INLINE_FUNCTION result_view_type view() const;

      クラスコンストラクタで提供された結果の場所のビューを返します。


追加情報
~~~~~~~~~~~~~~~~~~~~~~

* ``BAnd<T,S>::value_type`` は、非定数 ``T``　です。

* ``BAnd<T,S>::result_view_type`` は　``Kokkos::View<T,S,Kokkos::MemoryTraits<Kokkos::Unmanaged>>``　です。　S(メモリ空間)は結果が存在する空間と同じでなければならないことに、ご注意ください。

* 必要条件: ``Scalar`` は、 定義した　``operator =`` and ``operator &`` を持ちます。``Kokkos::reduction_identity<Scalar>::band()`` は有効な式です。

* BAnd をカスタム型でしようするには、with a custom type, a template specialization of ``Kokkos::reduction_identity<CustomType>`` を定義する必要があります。詳細については、 `Built-In Reducers with Custom Scalar Types <../../../ProgrammingGuide/Custom-Reductions-Built-In-Reducers-with-Custom-Scalar-Types.html>`_ をご参照ください。
