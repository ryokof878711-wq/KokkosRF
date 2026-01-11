
.. include:: ../../mydefs.rst

.. role:: cpp(code)
    :language: cpp

``DynRankView``
===============

ヘッダーファイル: ``<Kokkos_DynRankView.hpp>``

使用
-----

``DynRankView`` は、コンパイル時にレイアウトとメモリ空間が確保される、参照カウント型の多次元配列です。
``DynRankView`` は、ランクが ``std::shared_ptr`` テンプレートパラメータで提供されない点で 
[[View|Kokkos::View]] とは異なります; それは、コンストラクタに渡された引数の数に基づいて動的に決定されます。
動的に決定されます。ランクの上限は 7 次元です。


ディスクリプション
-----------

.. cpp:class:: template <class DataType, class LayoutType, class MemorySpace, class MemoryTraits> DynRankView;

   :tparam DataType: ``DynRankView``の基本的なスカラー型を定義します。

		     .. attention:: 本パラメータは、必須です。

		     基本構造は、 ``ScalarType``です。
		     例:

		     * ``double``: a ``DynRankView``で、 次元数はコンストラクタの引数として渡され、その数がランクを決定します。

   :tparam LayoutType: インデックスの基盤となる1次元メモリストレージへのマッピングを決定します。

		       .. important:: 本パラメータは、オプションです。

		       カスタムレイアウトは実装可能ですが、Kokkosにはいくつかの組み込みレイアウトが付属しています:

		       * ``LayoutRight``: ストライドは、右端から左端に向かって増加します。
			 最後の次元は、ストライドが1です。これはC言語の多次元配列（``[][][]``）がメモリ上に配置される方法に対応します。

		       * ``LayoutLeft``: ストライドは、右端から左端に向かって増加します。
			 最初の次元は、ストライドが1です。これはFortranが配列に使用するレイアウトです。

		       * ``LayoutStride``: ストライドは各次元で任意に設定できます。

   :tparam MemorySpace: 保管場所を管理します。

			.. important:: 本パラメータは、オプションです。

			省略された場合、デフォルトの実行領域のデフォルトメモリ領域が使用されます　（つまり、``Kokkos::DefaultExecutionSpace::memory_space``)

   :tparam MemoryTraits: メモリアクセスに対するより細かい制御

			 .. important:: 本パラメータは、オプションです。

			 * ``Unmanaged``: DynRankViewは参照カウントされません。割り当てはコンストラクタに提供されなければなりません。
			 * ``Atomic``: ビューへのすべてのアクセスはアトミック操作を使用する。
			 * ``RandomAccess``: ビューがランダムアクセス方式で使用されていることを示唆します。
			   ビューもまた　``const``であれば、これにより、GPU上で特別なロード操作（すなわちテクスチャフェッチ）がトリガーされます。
			 * ``Restrict``: 現在のスコープ内で、他のデータ構造によるビューのアリアシングは存在しません

   .. important::

      ``DataType``以外のテンプレートパラメータはオプションですが、順序は強制されます。
      つまり、例えば、``LayoutType``は省略可能ですが、 ``MemorySpace`` および ``MemoryTraits``の両方が
      特定されれば、 ``MemoryTraits`` の前に ``MemorySpace``が来なければなりません。


   .. rubric:: Public Static Variables

   * ``rank``: ビューのランク (つまり、次元性).
   * ``rank_dynamic``: 実行時に決定される次元の数です。
   * ``reference_type_is_lvalue_reference``: 参照型がC++の左辺値参照であるかどうか。


   .. rubric:: パブリックデータタイプ 型定義

   .. cpp:type:: data_type

      DynRankView の ``DataType``　

   .. cpp:type:: const_data_type

      ``DataType`` の定数バージョンであり、 それがすでに定数である場合には、 ``data_type``　と同じです。

   .. cpp:type:: non_const_data_type

      ``DataType``　の非定数バージョンであり、それがすでに非定数である場合には、 ``data_type``　と同じです。

   .. cpp:type:: scalar_array_type

      もし　``DataType``　が、Sacado FAD型のような適切に特化した配列データ型を表す場合、 ``scalar_array_type`` は、基礎となる基本的なスカラー型です。

   .. cpp:type:: const_scalar_array_type

      ``scalar_array_type``　の定数バージョンであり、 それがすでに定数である場合には、``scalar_array_type`` と同じです。

   .. cpp:type:: non_const_scalar_array_type

      ``scalar_array_type``　の非定数バージョンであり、 それがすでに非定数である場合には、 ``scalar_array_type`` と同じです。

   .. rubric:: パブリックスカラー型定義

   .. cpp:type:: value_type

      配列指定子を削除した　``data_type`` で、つまりビューが参照しているデータの
スカラー型です (例えば、 ``data_type`` が ``const int*******``　である場合、 ``value_type`` は ``const int``　です)。

   .. cpp:type:: const_value_type

      ``value_type``　の定数バージョン。

   .. cpp:type:: non_const_value_type

      ``value_type``　の非定数バージョン。

   .. rubric:: パブリックスペース型定義

   .. cpp:type:: execution_space

      ビューに関連付けられた実行領域は、ビューの初期化を実行するために使用され、そして特定の deep_copy 演算です。

   .. cpp:type:: memory_space

      データ保存場所の種類。

   .. cpp:type:: device_type

      ``Device<execution_space,memory_space>``　に定義された複合型。

   .. cpp:type:: memory_traits

      ビューの記憶特性。

   .. cpp:type:: host_mirror_space

      ``HostMirror``　に使用されるホストがアクセス可能なメモリ空間。

   .. rubric:: Public View Typedefsパブリックビュー型定義

   .. cpp:type:: non_const_type

      すべてのテンプレートパラメータが明示的に定義されたこのビュータイプ。

   .. cpp:type:: const_type

      ``const`` data type　を使用してすべてのテンプレートパラメータが明示的に定義されたこのビュータイプ。

   .. cpp:type:: HostMirror

      ホストアクセス可能メモリ空間に格納された、同一の　``DataType``　および　``LayoutType``　を持つ互換ビュータイプ。

   .. rubric:: パブリックデータハンドルズ型定義

   .. cpp:type:: reference_type

      ビューアクセス演算子の戻り値の型。

   .. cpp:type:: pointer_type

      スカラー型へのポインタ。

   .. rubric:: 他のパブリック型定義

   .. cpp:type:: array_layout

      ``DynRankView``　のレイアウト。

   .. cpp:type:: size_type

      このビューのメモリ空間に関連付けられたインデックス型。

   .. cpp:type:: dimension

      型のような整数配列で、ビューの範囲を表現できます。

   .. cpp:type:: specialize

     ``DynRankView``.Kokkosの　``DynRankView``　の基盤となるマッピング構造の部分的な特殊化に使用される特殊化タグです。

   .. rubric:: コンストラクタ

   .. cpp:function:: DynRankView()

       デフォルトコンストラクタ。 割り当ては行われず、参照カウントも発生しません。すべてのエクステントはゼロであり、そのデータポインタは　``nullptr``　であり、そのランクは0に設定されます。

   .. cpp:function:: DynRankView(const DynRankView<DT, Prop...>& rhs)

       互換性のある　DynRankView　を持つコピーコンストラクタ。 DynRankViewの割り当てルールに従います。

   .. cpp:function:: DynRankView(DynRankView&& rhs)

       コンストラクタを移動します。

   .. cpp:function:: DynRankView(const View<RT,RP...> & rhs)

       Viewを入力とするコピーコンストラクタ。

   .. cpp:function:: DynRankView(const std::string& name, const IntType& ... indices)

       ``array_layout::is_regular == true``　を必要とします。

       標準的な割り当てコンストラクタ。

       * ``name``: ユーザーが提供したラベル。プロファイリングおよびデバッグ目的で使用されます。名前は一意である必要はありません。
       * ``indices``: ビューのランタイムディメンション。

   .. cpp:function:: DynRankView(const std::string& name, const array_layout& layout)

        標準的な割り当てコンストラクタ。

       * ``name``: ユーザーが提供したラベル。プロファイリングおよびデバッグ目的で使用されます。名前は一意である必要はありません。
       * ``layout``: レイアウトクラスのインスタンス。

   .. cpp:function:: DynRankView(const AllocProperties& prop, const IntType& ... indices)

       Requires: ``array_layout::is_regular == true``

       割り当てプロパティを持つ割り当てコンストラクタ。 割り当てプロパティオブジェクトは、``view_alloc``　関数によって返されます。

       * ``indices``: ビューのランタイムディメンション。

   .. cpp:function:: DynRankView(const AllocProperties& prop, const array_layout& layout)

       Allocating constructor with allocation properties and a layout object.

       * ``layout``: an instance of a layout class.

   .. cpp:function:: DynRankView(const pointer_type& ptr, const IntType& ... indices)

       Requires: ``array_layout::is_regular == true``

       Unmanaged data wrapping constructor.

       * ``ptr``: pointer to a user provided memory allocation. Must provide storage of size ``DynRankView::required_allocation_size(n0,...,nR)``.
       * ``indices``: runtime dimensions of the view.

   .. cpp:function:: DynRankView(const pointer_type& ptr, const array_layout& layout)

       Unmanaged data wrapper constructor.

       * ``ptr``: pointer to a user provided memory allocation. Must provide storage of size ``DynRankView::required_allocation_size(layout)`` (\ *NEEDS TO BE IMPLEMENTED*\ )
       * ``layout``: an instance of a layout class.

   .. cpp:function:: DynRankView(const ScratchSpace& space, const IntType& ... indices)

       Requires: ``sizeof(IntType...)==rank_dynamic()`` and ``array_layout::is_regular == true``

       Constructor which acquires memory from a Scratch Memory handle.

       * ``space``: scratch memory handle. Typically returned from ``team_handles`` in ``TeamPolicy`` kernels.
       * ``indices``: runtime dimensions of the view.

   .. cpp:function:: DynRankView(const ScratchSpace& space, const array_layout& layout)

       Constructor which acquires memory from a Scratch Memory handle.

       * ``space``: scratch memory handle. Typically returned from ``team_handles`` in ``TeamPolicy`` kernels.
       * ``layout``: an instance of a layout class.

   .. cpp:function:: DynRankView(const DynRankView<DT, Prop...>& rhs, Args ... args)

       Subview constructor. See ``subview`` function for arguments.

   .. rubric:: Data Access Functions

   .. cpp:function:: reference_type operator() (const IntType& ... indices) const

      Returns a value of ``reference_type`` which may or not be reference itself.
      The number of index arguments must match the ``rank`` of the view. See notes on ``reference_type`` for properties of the return type.

   .. cpp:function:: reference_type access (const IntType& i0=0, const IntType& i1=0, \
			   const IntType& i2=0, const IntType& i3=0, const IntType& i4=0, \
			   const IntType& i5=0, const IntType& i6=0) const

      Returns a value of ``reference_type`` which may or not be reference itself.
      The number of index arguments must be equal or larger than the ``rank`` of the view.
      Index arguments beyond ``rank`` must be ``0`` , which will be enforced if ``KOKKOS_DEBUG`` is defined.
      See notes on ``reference_type`` for properties of the return type.


   .. rubric:: Data Layout, Dimensions, Strides

   .. cpp:function:: constexpr array_layout layout() const

      Returns the layout object. Can be used to to construct other views with the same dimensions.

   .. cpp:function:: template<class iType> constexpr size_t extent(const iType& dim) const

      Returns the extent of the specified dimension. ``iType`` must be an integral type, and ``dim`` must be smaller than ``rank``.

   .. cpp:function:: template<class iType> constexpr int extent_int(const iType& dim) const

      Returns the extent of the specified dimension as an ``int``. ``iType`` must be an integral type,
      and ``dim`` must be smaller than ``rank``. Compared to ``extent`` this function can be useful
      on architectures where ``int`` operations are more efficient than ``size_t``.
      It also may eliminate the need for type casts in applications which otherwise perform all index operations with ``int``.

   .. cpp:function:: template<class iType> constexpr size_t stride(const iType& dim) const

       Returns the stride of the specified dimension. ``iType`` must be an integral type, and ``dim`` must be smaller than ``rank``. Example: ``a.stride(3) == (&a(i0,i1,i2,i3+1,i4)-&a(i0,i1,i2,i3,i4))``

   .. cpp:function:: constexpr size_t stride_0() const

       Return the stride of dimension 0.

   .. cpp:function:: constexpr size_t stride_1() const

       Return the stride of dimension 1.

   .. cpp:function:: constexpr size_t stride_2() const

       Return the stride of dimension 2.

   .. cpp:function:: constexpr size_t stride_3() const

       Return the stride of dimension 3.

   .. cpp:function:: constexpr size_t stride_4() const

       Return the stride of dimension 4.

   .. cpp:function:: constexpr size_t stride_5() const

       Return the stride of dimension 5.

   .. cpp:function:: constexpr size_t stride_6() const

       Return the stride of dimension 6.

   .. cpp:function:: constexpr size_t stride_7() const

       Return the stride of dimension 7.

   .. cpp:function:: constexpr size_t span() const

       Return the memory span in elements between the element with the lowest and the highest address.
       This can be larger than the product of extents due to padding, and or non-contiguous data layout as for example ``LayoutStride`` allows.

   .. cpp:function:: constexpr pointer_type data() const

       Return the pointer to the underlying data allocation.

   .. cpp:function:: bool span_is_contiguous() const

       Whether the span is contiguous (i.e. whether every memory location between in span belongs to the index space covered by the view).

   .. cpp:function:: static constexpr size_t required_allocation_size(size_t N0 = 0, size_t N1 = 0, \
			   size_t N2 = 0, size_t N3 = 0, size_t N4 = 0, \
			   size_t N5 = 0, size_t N6 = 0);

       Returns the number of bytes necessary for an unmanaged view of the provided dimensions. This function is only valid if ``array_layout::is_regular == true``.

   .. cpp:function:: static constexpr size_t required_allocation_size(const array_layout& layout);

       :return: the number of bytes necessary for an unmanaged view of the provided layout.

   .. rubric:: Other Public Methods

   .. cpp:function:: int use_count() const

       :return: the current reference count of the underlying allocation.

   .. cpp:function:: const char* label() const;

       :return: the label of the ``DynRankView``.

   .. cpp:function:: constexpr unsigned rank() const

       :return: the dynamic rank of the ``DynRankView``.

   .. cpp:function:: constexpr bool is_allocated() const

       :return: true if the view points to a valid memory location.
		This function works for both managed and unmanaged views.
		With the unmanaged view, there is no guarantee that referenced address is valid, only that it is a non-null pointer.

Assignment Rules
----------------

Assignment rules cover the assignment operator as well as copy constructors. We aim at making all logically legal assignments possible, while intercepting illegal assignments if possible at compile time, otherwise at runtime. In the following, we use ``DstType`` and ``SrcType`` as the type of the destination view and source view respectively. ``dst_view`` and ``src_view`` refer to the runtime instances of the destination and source views, i.e.:

.. code-block:: cpp

    ScrType src_view(...);
    DstType dst_view(src_view);
    dst_view = src_view;

The following conditions must be met at and are evaluated at compile time:

* ``DstType::rank == SrcType::rank``
* ``DstType::non_const_value_type`` is the same as ``SrcType::non_const_value_type``
* If ``std::is_const<SrcType::value_type>::value == true`` than ``std::is_const<DstType::value_type>::value == true``.
* ``MemorySpaceAccess<DstType::memory_space,SrcType::memory_space>::assignable == true``

Furthermore there are rules which must be met if ``DstType::array_layout`` is not the same as ``SrcType::array_layout``. These rules only cover cases where both layouts are one of ``LayoutLeft`` , ``LayoutRight`` or ``LayoutStride``

* If neither ``DstType::array_layout`` nor ``SrcType::array_layout`` is ``LayoutStride``:
    - If ``DstType::rank > 1`` than ``DstType::array_layout`` must be the same as ``SrcType::array_layout``.

* If either ``DstType::array_layout`` or ``SrcType::array_layout`` is ``LayoutStride``
    - For each dimension ``k`` it must hold that ``dst_view.extent(k) == src_view.extent(k)``

例
--------

.. code-block:: cpp

    #include<Kokkos_Core.hpp>
    #include<cstdio>

    int main(int argc, char* argv[]) {
        Kokkos::initialize(argc,argv);

        int N0 = atoi(argv[1]);
        int N1 = atoi(argv[2]);

        Kokkos::DynRankView<double> a("A",N0);
        Kokkos::DynRankView<double> b("B",N1);

        Kokkos::parallel_for("InitA", N0, KOKKOS_LAMBDA (const int& i) {
            a(i) = i;
        });

        Kokkos::parallel_for("InitB", N1, KOKKOS_LAMBDA (const int& i) {
            b(i) = i;
        });

        Kokkos::DynRankView<double,Kokkos::LayoutLeft> c("C",N0,N1);
        {
            Kokkos::DynRankView<const double> const_a(a);
            Kokkos::DynRankView<const double> const_b(b);
            Kokkos::parallel_for("SetC", Kokkos::MDRangePolicy<Kokkos::Rank<2,Kokkos::Iterate::Left>>({0,0},{N0,N1}),
                KOKKOS_LAMBDA (const int& i0, const int& i1) {
                c(i0,i1) = a(i0) * b(i1);
            });
        }

        Kokkos::finalize();
    }
