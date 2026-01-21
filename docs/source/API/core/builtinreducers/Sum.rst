``Sum``
=======

.. role:: cpp(code)
    :language: cpp

``add`` operation演算を行う `ReducerConcept <ReducerConcept.html>`_　の具体的な実装。

ヘッダーファイル: ``<Kokkos_Core.hpp>``

使用例
-----

.. code-block:: cpp

   T result;
   parallel_reduce(N,Functor,Sum<T,S>(result));

概要
--------

.. code-block:: cpp

   template<class Scalar, class Space>
   class Sum{
     public:
       typedef Sum reducer;
       typedef typename std::remove_cv<Scalar>::type value_type;
       typedef Kokkos::View<value_type, Space> result_view_type;

       KOKKOS_INLINE_FUNCTION
       void join(value_type& dest, const value_type& src) const;

       KOKKOS_INLINE_FUNCTION
       void init( value_type& val) const;

       KOKKOS_INLINE_FUNCTION
       value_type& reference() const;

       KOKKOS_INLINE_FUNCTION
       result_view_type view() const;

       KOKKOS_INLINE_FUNCTION
       Sum(value_type& value_);

       KOKKOS_INLINE_FUNCTION
       Sum(const result_view_type& value_);
   };

インターフェイス
---------

.. cpp:class:: template<class Scalar, class Space> Sum

   .. rubric:: パブリック型

   .. cpp:type:: reducer

      自己型。

   .. cpp:type:: value_type

      The reduction scalar type.還元スカラー型。

   .. cpp:type:: result_view_type

      還元結果を参照する　``Kokkos::View``。

   .. rubric:: コンストラクタ

   .. cpp:function:: KOKKOS_INLINE_FUNCTION Sum(value_type& value_);

      結果の保存先としてローカル変数を参照するリデューサを構築します。

   .. cpp:function:: KOKKOS_INLINE_FUNCTION Sum(const result_view_type& value_);

      特定のビューを結果の保存先として参照するリデューサーを構築します。

   .. rubric:: パブリックメンバー関数

   .. cpp:function:: KOKKOS_INLINE_FUNCTION void join(value_type& dest, const value_type& src) const;

      ``src``を ``dest``:  ``dest+=src;``　に追加します。

   .. cpp:function:: KOKKOS_INLINE_FUNCTION void init(value_type& val) const;

      Initialize ``val`` using the ``Kokkos::reduction_identity<Scalar>::sum()`` メソッドを使用して、``val``　を初期化します。 デフォルト実装は、``val=0``　を設定します。

   .. cpp:function:: KOKKOS_INLINE_FUNCTION value_type& reference() const;

      クラスコンストラクタで提供された結果への参照を返します。

   .. cpp:function:: KOKKOS_INLINE_FUNCTION result_view_type view() const;

      クラスコンストラクタで提供された結果の保存先のビューを返します。

追加情報
^^^^^^^^^^^^^^^^^^^^^^

* ``Sum<T,S>::value_type`` は、 非定数 ``T``　です。

* ``Sum<T,S>::result_view_type`` は、 ``Kokkos::View<T,S,Kokkos::MemoryTraits<Kokkos::Unmanaged>>``　です。 S(メモリ空間)は結果が存在する空間と同じでなければならないことに、ご注意ください。

* Requires: ``Scalar`` has ``operator =`` and ``operator +=`` defined. ``Kokkos::reduction_identity<Scalar>::sum()`` is a valid expression.

* In order to use Sum with a custom type, a template specialization of ``Kokkos::reduction_identity<CustomType>`` must be defined.  See `Built-In Reducers with Custom Scalar Types <../../../ProgrammingGuide/Custom-Reductions-Built-In-Reducers-with-Custom-Scalar-Types.html>`_ for details
