``LAnd``
========

.. role:: cpp(code)
    :language: cpp

論理的 ``AND`` 演算を行う　`ReducerConcept <ReducerConcept.html>`_　の具体的実装

ヘッダーファイル: ``<Kokkos_Core.hpp>``

使用例
-----

.. code-block:: cpp

   T result;
   parallel_reduce(N,Functor,LAnd<T,S>(result));

概要
--------

.. code-block:: cpp

   template<class Scalar, class Space>
   class LAnd{
     public:
       型定義 LAnd リデューサー;
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
       LAnd(value_type& value_);

       KOKKOS_INLINE_FUNCTION
       LAnd(const result_view_type& value_);
   };

インターフェイス
---------

.. cpp:class:: template<class Scalar, class Space> LAnd

   .. rubric:: Public Types

   .. cpp:type:: リデューサー

      自己型

   .. cpp:type:: value_type

      還元スカラー型。

   .. cpp:type:: result_view_type

      還元結果を参照する ``Kokkos::View``。

   .. rubric:: コンストラクタ

   .. cpp:function:: KOKKOS_INLINE_FUNCTION LAnd(value_type& value_);

      結果の位置としてローカル変数を参照するリデューサを構築します。

   .. cpp:function:: KOKKOS_INLINE_FUNCTION LAnd(const result_view_type& value_);

      特定のビューを結果の位置として参照するリデューサーを構築します。

   .. rubric:: パブリックメンバー関数

   .. cpp:function:: KOKKOS_INLINE_FUNCTION void join(value_type& dest, const value_type& src) const;

      ``src``　の ``and``　および　``dest`` を ``dest``:  ``dest = src && dest;``　にビット単位で格納します。

   .. cpp:function:: KOKKOS_INLINE_FUNCTION void init(value_type& val) const;

      ``Kokkos::reduction_identity<Scalar>::land()`` メソッドを使用して、``val``　を初期化します。 デフォルト実装は、``val=1``　を設定します。

   .. cpp:function:: KOKKOS_INLINE_FUNCTION value_type& reference() const;

      Returns a reference to the result provided in class constructor.

   .. cpp:function:: KOKKOS_INLINE_FUNCTION result_view_type view() const;

      Returns a view of the result place provided in class constructor.


Additional Information
^^^^^^^^^^^^^^^^^^^^^^

* ``LAnd<T,S>::value_type`` is non-const ``T``

* ``LAnd<T,S>::result_view_type`` is ``Kokkos::View<T,S,Kokkos::MemoryTraits<Kokkos::Unmanaged>>``. Note that the S (memory space) must be the same as the space where the result resides.

* Requires: ``Scalar`` has ``operator =`` and ``operator &&`` defined. ``Kokkos::reduction_identity<Scalar>::land()`` is a valid expression.

* In order to use LAnd with a custom type, a template specialization of ``Kokkos::reduction_identity<CustomType>`` must be defined.  See `Built-In Reducers with Custom Scalar Types <../../../ProgrammingGuide/Custom-Reductions-Built-In-Reducers-with-Custom-Scalar-Types.html>`_ for details
