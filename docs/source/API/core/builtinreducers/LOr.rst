``LOr``
=======

.. role:: cpp(code)
    :language: cpp

論理的 ``OR`` 演算を行う　`ReducerConcept <ReducerConcept.html>`_　の具体的な実装

ヘッダーファイル: ``<Kokkos_Core.hpp>``

使用例
-----

.. code-block:: cpp

   T result;
   parallel_reduce(N,Functor,LOr<T,S>(result));

概要
--------

.. code-block:: cpp

   template<class Scalar, class Space>
   class LOr{
     public:
       型定義 LOr リデューサー;
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
       LOr(value_type& value_);

       KOKKOS_INLINE_FUNCTION
       LOr(const result_view_type& value_);
   };

インターフェイス
---------

.. cpp:class:: template<class Scalar, class Space> LOr

   .. rubric:: パブリック型

   .. cpp:type:: リデューサー

      自己型。

   .. cpp:type:: value_type

      還元スカラー型。

   .. cpp:type:: result_view_type

      還元結果を参照する ``Kokkos::View``

   .. rubric:: コンストラクタ

   .. cpp:function:: KOKKOS_INLINE_FUNCTION LOr(value_type& value_);

      結果の位置としてローカル変数を参照するリデューサを構築します。

   .. cpp:function:: KOKKOS_INLINE_FUNCTION LOr(const result_view_type& value_);

      特定のビューを結果の位置として参照するリデューサーを構築します。

   .. rubric:: パブリックメンバー関数

   .. cpp:function:: KOKKOS_INLINE_FUNCTION void join(value_type& dest, const value_type& src) const;

      ``src``　の ``and``　および　``dest`` を　``dest``: ``dest = src || dest;``　にビット単位で格納します。

   .. cpp:function:: KOKKOS_INLINE_FUNCTION void init(value_type& val) const;

      ``Kokkos::reduction_identity<Scalar>::land()`` メソッドを使用して、``val``　を初期化します。 デフォルト実装は、``val=0``　を設定します。


   .. cpp:function:: KOKKOS_INLINE_FUNCTION value_type& reference() const;

      クラスコンストラクタで提供された結果への参照を返します。

   .. cpp:function:: KOKKOS_INLINE_FUNCTION result_view_type view() const;

      クラスコンストラクタで提供された結果の場所のビューを返します。


追加情報
^^^^^^^^^^^^^^^^^^^^^^

* ``LOr<T,S>::value_type`` は、 非定数 ``T``　です。

* ``LOr<T,S>::result_view_type`` は、 ``Kokkos::View<T,S,Kokkos::MemoryTraits<Kokkos::Unmanaged>>``　です。 S(メモリ空間)は結果が存在する空間と同じでなければならないことに、ご注意ください。

* 必要条件: ``Scalar`` は、 定義した ``operator =`` and ``operator ||`` を持ちます。``Kokkos::reduction_identity<Scalar>::lor()`` は、有効な式です。

* LOr をカスタム型で使用するには、 ``Kokkos::reduction_identity<CustomType>`` のテンプレート仕様を定義する必要があります。 詳細については、 `Built-In Reducers with Custom Scalar Types <../../../ProgrammingGuide/Custom-Reductions-Built-In-Reducers-with-Custom-Scalar-Types.html>`_ をご参照ください。
