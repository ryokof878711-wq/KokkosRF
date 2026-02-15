ビューおよびその関連
================

データ管理は、あらゆるプログラムにおいて極めて重要な要素です。 Kokkos における主要な機能は ``Kokkos::View``　である。 
以下のファシリティが利用可能です:

.. リスト表::
   :幅: 25 75
   :ヘッダー列: 1

   * - クラス
     - ディスクリプション
   * - `create_mirror[_view] <view/create_mirror.html>`__
     - 別のメモリ空間に、　``Kokkos::View``　のコピーを作成。
   * - `deep_copy() <view/deep_copy.html>`__
     - ビューとスカラー間のデータコピー。
   * - `LayoutLeft <view/layoutLeft.html>`__
     - Fortran　に準拠したメモリ配置。
   * - `LayoutRight <view/layoutRight.html>`__
     -  C に準拠したメモリレイアウト。
   * - `LayoutStride <view/layoutStride.html>`__
     - 任意のストライドに対するメモリ配置。
   * - `MemoryTraits <view/memoryTraits.html>`__
     - メモリアクセストレイト。
   * - `realloc <view/realloc.html>`__
     -  ``Kokkos::View``　の再割り当て。
   * - `resize <view/resize.html>`__
     - Resizing a ``Kokkos::View``　のリサイズ。
   * - `subview <view/subview.html>`__
     - ``Kokkos::View``からスライスを取得。
   * - `View <view/view.html>`__
     -  Kokkos 主要データ構造、多次元メモリ空間およびレイアウト認識配列。
   * - `view_alloc() <view/view_alloc.html>`__
     - 引数リストからビュー割り当てパラメータバンドルを作成。
   * - `View-like Types <view/view_like.html>`__
     - おおまかに定義すると、``Kokkos::View`` のように振る舞うクラステンプレートの集合。


.. toctree::
   :hidden:
   :maxdepth: 1

   ./view/create_mirror
   ./view/deep_copy
   ./view/layoutLeft
   ./view/layoutRight
   ./view/layoutStride
   ./view/memoryTraits
   ./view/realloc
   ./view/resize
   ./view/subview
   ./view/Subview_type
   ./view/view
   ./view/view_alloc
   ./view/view_like
