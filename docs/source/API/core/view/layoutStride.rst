``LayoutStride``
================

.. role:: cpp(code)
    :language: cpp

ヘッダーファイル: ``<Kokkos_Core.hpp>``

使用例
-----

.. code-block:: c++

    Kokkos::View<float***> full_mesh; // 網状の構造
    Kokkos::View<float**, Kokkos::LayoutStride> mesh_subcomponent;
    mesh_subcomponent = Kokkos::subview(full_mesh,Kokkos::ALL(), 0, Kokkos::ALL()); // x および z コンポーネントを選択

ディスクリプション
-----------

.. cpp:class:: LayoutStride

   多次元ビューに提供された場合、 任意のストライドでメモリを配置します。
   より大きなビューの一部として非連続なサブビューを取得する際に最も頻繁に遭遇します。

   .. rubric:: パブリッククラスメンバー

   .. cpp:member:: size_t dimension[ARRAY_LAYOUT_MAX_RANK];

      レイアウトの各次元のサイズを含む配列。

   .. cpp:member:: size_t stride[ARRAY_LAYOUT_MAX_RANK];

      レイアウトの各次元のサイズを含む配列。

   .. cpp:member:: static constexpr bool is_extent_constructible = false;

      このクラスが拡張可能なコンストラクタを持つかどうかを検出するためのブール値。

   .. rubric:: パブリック型定義

   .. cpp:type:: array_layout

      このモデルがレイアウト概念を表現していることを示すタグ。

   .. rubric:: コンストラクタ

   .. cpp:function:: KOKKOS_INLINE_FUNCTION 明示的 constexpr LayoutStride(size_t N0 = 0, size_t S0 = 0, \
			   size_t N1 = 0, \
                           size_t S1 = 0, size_t N2 = 0, size_t S2 = 0, \
                           size_t N3 = 0, size_t S3 = 0, size_t N4 = 0, \
                           size_t S4 = 0, size_t N5 = 0, size_t S5 = 0, \
                           size_t N6 = 0, size_t S6 = 0, size_t N7 = 0, size_t S7 = 0);

      レイアウトの対応する次元のサイズを設定するための、最大8つのサイズを受け取るコンストラクタ。

   .. cpp:function:: LayoutStride(LayoutStride const&) = default;

      デフォルトのコピーコンストラクタは、要素単位で他のレイアウトをコピーします。

   .. cpp:function:: LayoutStride(LayoutStride&&) = default;

      デフォルトの移動コンストラクタは、要素単位で他のレイアウトを移動します。

   .. rubric:: 代入演算子

   .. cpp:function:: LayoutStride& operator=(LayoutStride const&) = default;

      Default copy assignment, element-wise copies the other Layout

   .. cpp:function:: LayoutStride& operator=(LayoutStride&&) = default;

      Default move assignment, element-wise moves the other Layout

   .. rubric:: Functions

   .. cpp:function:: KOKKOS_INLINE_FUNCTION static LayoutStride order_dimensions(int const rank, \
		   iTypeOrder const* const order, iTypeDimen const* const dimen);

      Calculates the strides given ordered dimensions

Example
-------

Creating a 3D unmanaged strided view around a ptr. (You can also just have a view allocate itself by providing a label)

.. code-block:: cpp

    #include<Kokkos_Core.hpp>
    int main(int argc, char* argv[]) {
        Kokkos::initialize(argc,argv);
        {
            // Some storage
            int* ptr = new int[80];
            // Creating a layout object
            Kokkos::LayoutStride layout(3,1,3,5,4,20);
            // Create a unmanaged view from a pointer and a layout
            Kokkos::View<int***, Kokkos::LayoutStride, Kokkos::HostSpace> a(ptr,layout);

            // Get strides
            int strides[8];
            a.stride(strides);

            // Print extents and strides
            printf("Extents: %d %d %d\n",a.extent(0),a.extent(1),a.extent(2));
            printf("Strides: %i %i %i\n",strides[0],strides[1],strides[2]);

            // delete storage
            delete [] ptr;
        }
        Kokkos::finalize();
    }

Output:

.. code-block::

    Extents: 3 3 4
    Strides: 1 5 20
