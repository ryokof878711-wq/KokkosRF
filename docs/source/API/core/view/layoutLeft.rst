``LayoutLeft``
==============

.. role:: cpp(code)
   :language: cpp

ヘッダーファイル: ``<Kokkos_Core.hpp>``

使用例
-----

.. code-block:: cpp

   Kokkos::View<float*, Kokkos::LayoutLeft> my_view;

ディスクリプション
-----------

.. cpp:struct:: LayoutLeft

   多次元ビューに提供された場合、最初のインデックスが連続したインデックスとなるように、
   メモリを配置します。 これは、割り当てに関する　Fortran　の慣習に合致します。

   .. rubric:: ネストされた型定義

   .. cpp:type:: array_layout

       このモデルがレイアウト概念を表現していることを示すタグ。

   .. rubric:: メンバー変数

   .. cpp:member:: static constexpr bool is_extent_constructible = true

      このクラスが拡張可能なコンストラクタを持つかどうかを検出するためのブール値。

   .. cpp:member:: size_t dimension[8]

       レイアウトの各次元のサイズを含む配列。

   .. rubric:: コンストラクタ

   .. cpp:function:: KOKKOS_INLINE_FUNCTION 明示的 constexpr LayoutLeft(size_t N0 = 0, size_t N1 = 0, \
				       size_t N2 = 0, size_t N3 = 0, size_t N4 = 0, \
				       size_t N5 = 0, size_t N6 = 0, size_t N7 = 0)

      レイアウトの対応する次元のサイズを設定するための、最大8つのサイズを受け取るコンストラクタ。

   .. cpp:function:: LayoutLeft(LayoutLeft const&) = default

       デフォルトのコピーコンストラクタは、要素単位で他のレイアウトをコピーします。

   .. cpp:function:: LayoutLeft(LayoutLeft&&) = default

       デフォルトの移動コンストラクタは、要素単位で他のレイアウトをコピーします。

   .. rubric:: 代入演算子

   .. cpp:function:: LayoutLeft& operator=(LayoutLeft const&) = default

       デフォルトのコピー代入は、要素単位で他のレイアウトをコピーします。

   .. cpp:function:: LayoutLeft& operator=(LayoutLeft&&) = default

       デフォルトの移動演算子は、要素単位で他のレイアウトをコピーします。
