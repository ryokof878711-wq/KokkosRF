``LayoutRight``
===============

.. role:: cpp(code)
   :language: cpp

ヘッダーファイル: ``<Kokkos_Core.hpp>``

使用例
-----

.. code-block:: cpp

   Kokkos::View<float*, Kokkos::LayoutRight> my_view;

ディスクリプション
-----------

.. cpp:struct:: LayoutRight

   多次元ビューに提供された場合、  **最後の指数が隣接したものである**　ために、
   メモリを配置します。 これは、割り当てに関する　C　の慣習に合致します。

   .. rubric:: ネストされた型定義

   .. cpp:type:: array_layout

       このモデルがレイアウト概念を表現していることを示すタグ。

   .. rubric:: メンバー変数

   .. cpp:member:: static constexpr bool is_extent_constructible = true

       A boolean to allow detection that this class is extent constructible.

   .. cpp:member:: size_t dimension[8]

       An array containing the size of each dimension of the Layout.

   .. rubric:: Constructors

   .. cpp:function:: KOKKOS_INLINE_FUNCTION explicit constexpr LayoutRight(size_t N0 = 0, size_t N1 = 0, \
				       size_t N2 = 0, size_t N3 = 0, size_t N4 = 0, \
				       size_t N5 = 0, size_t N6 = 0, size_t N7 = 0)

      Constructor that takes in up to 8 sizes, to set the sizes of the corresponding dimensions of the Layout.

   .. cpp:function:: LayoutRight(LayoutRight const&) = default

       Default copy constructor, element-wise copies the other Layout.

   .. cpp:function:: LayoutRight(LayoutRight&&) = default

       Default move constructor, element-wise moves the other Layout.

   .. rubric:: Assignment operators

   .. cpp:function:: LayoutRight& operator=(LayoutRight const&) = default

       Default copy assignment, element-wise copies the other Layout.

   .. cpp:function:: LayoutRight& operator=(LayoutRight&&) = default

       Default move assignment, element-wise moves the other Layout.
