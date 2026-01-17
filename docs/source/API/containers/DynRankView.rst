
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

       割り当てプロパティとレイアウトオブジェクトを持つ割り当てコンストラクタ。

       * ``layout``: レイアウトクラスのインスタンス。

   .. cpp:function:: DynRankView(const pointer_type& ptr, const IntType& ... indices)

       Requires: ``array_layout::is_regular == true``

       管理対象外データのラップコンストラクタ

       * ``ptr``: ユーザーが提供したメモリ割り当てへのポインタ。``DynRankView::required_allocation_size(n0,...,nR)`` のストレージを提供する必要があります。
       * ``indices``: ビューのランタイムディメンション。

   .. cpp:function:: DynRankView(const pointer_type& ptr, const array_layout& layout)

       管理対象外データのラップコンストラクタ。

       * ``ptr``: ユーザーが提供したメモリ割り当てへのポインタ。  ``DynRankView::required_allocation_size(layout)`` (\ *NEEDS TO BE IMPLEMENTED*\ )　のストレージを提供する必要があります。
       * ``layout``: ビューのランタイムディメンション。

   .. cpp:function:: DynRankView(const ScratchSpace& space, const IntType& ... indices)

       Requires: ``sizeof(IntType...)==rank_dynamic()`` and ``array_layout::is_regular == true``

       スクラッチメモリハンドルからメモリを取得するコンストラクタ。

       * ``space``: スクラッチメモリハンドル。通常、``TeamPolicy``　カーネルの　``team_handles``　から返されます。
       * ``indices``: ビューのランタイムディメンション。

   .. cpp:function:: DynRankView(const ScratchSpace& space, const array_layout& layout)

       スクラッチメモリハンドルからメモリを取得するコンストラクタ。

       * ``space``: スクラッチメモリハンドル。通常、``TeamPolicy``　カーネルの　``team_handles``　から返されます。
       * ``layout``: レイアウトクラスのインスタンス。

   .. cpp:function:: DynRankView(const DynRankView<DT, Prop...>& rhs, Args ... args)

       サブビューコンストラクタ。引数については、　``subview``　関数を参照してください。

   .. rubric:: データアクセル機能

   .. cpp:function:: reference_type operator() (const IntType& ... indices) const

      参照型である場合もそうでない場合もある ``reference_type`` の値を返します。
      インデックス引数の数は、ビューの ``rank`` と一致する必要があります。リターン型の特性については、``reference_type`` の注記を参照してください。

   .. cpp:function:: reference_type access (const IntType& i0=0, const IntType& i1=0, \
			   const IntType& i2=0, const IntType& i3=0, const IntType& i4=0, \
			   const IntType& i5=0, const IntType& i6=0) const

      参照型である場合もそうでない場合もある ``reference_type`` の値を返します。
      インデックス引数の数は、ビューの　``rank``　以上でなければなりません。
      ``rank``を超えるインデックス引数は``0``でなければならず、 ``KOKKOS_DEBUG`` が定義されている場合に有効になります。
      戻り値の型の特性については、``reference_type`` の注記を参照してください。


   .. rubric:: データレイアウト、ディメンション、ストライド

   .. cpp:function:: constexpr array_layout layout() const

      レイアウトオブジェクトを返します。 同じディメンションを持つ他のビューを構築するために使用できます。

   .. cpp:function:: template<class iType> constexpr size_t extent(const iType& dim) const

      指定されたディメンションの範囲を返します。 ``iType`` は整数型でなければならず、``dim`` は ``rank`` より小さくなければならない。

   .. cpp:function:: template<class iType> constexpr int extent_int(const iType& dim) const

    　``int``　として、指定されたディメンションの範囲を返します。 ``iType`` は整数型でなければならず、``dim`` は ``rank`` より小さくなければならない。``extent`` と比較して、この関数は　``int``　演算が　``size_t``　よりも効率的なアーキテクチャで有用である。
      また、そうでなければすべてのインデックス操作を　``int``　で行っているアプリケーションにおいて、型キャストの必要性を排除する可能性があります。

   .. cpp:function:: template<class iType> constexpr size_t stride(const iType& dim) const

       指定されたディメンションの範囲を返します。 ``iType`` は整数型でなければならず、``dim`` は ``rank`` より小さくなければならない。 例　: ``a.stride(3) == (&a(i0,i1,i2,i3+1,i4)-&a(i0,i1,i2,i3,i4))``

   .. cpp:function:: constexpr size_t stride_0() const

       指定されたディメンション 0　の範囲を返します。

   .. cpp:function:: constexpr size_t stride_1() const

       ディメンション 1　のストライドを返します。

   .. cpp:function:: constexpr size_t stride_2() const

       ディメンション 2　のストライドを返します。

   .. cpp:function:: constexpr size_t stride_3() const

       ディメンション 3　のストライドを返します。

   .. cpp:function:: constexpr size_t stride_4() const

       ディメンション 4　のストライドを返します。

   .. cpp:function:: constexpr size_t stride_5() const

       ディメンション 5　のストライドを返します。

   .. cpp:function:: constexpr size_t stride_6() const

       ディメンション 6　のストライドを返します。

   .. cpp:function:: constexpr size_t stride_7() const

       ディメンション 7　のストライドを返します。

   .. cpp:function:: constexpr size_t span() const

       最下位アドレスと最上位アドレスを持つエレメントの間にあるエレメント内のメモリスパンを返します。
       これはパディングによりエクステントの積よりも大きくなる可能性があり、 またはおよび、たとえば ``LayoutStride`` が許容するような、非連続的なデータレイアウトです。

   .. cpp:function:: constexpr pointer_type data() const

       基盤となるデータ割り当てへのポインタを返します。

   .. cpp:function:: bool span_is_contiguous() const

       スパンが連続しているかどうか (つまり、スパン内の各メモリ位置が、ビューによってカバーされるインデックス空間に属するか否か)。

   .. cpp:function:: static constexpr size_t required_allocation_size(size_t N0 = 0, size_t N1 = 0, \
			   size_t N2 = 0, size_t N3 = 0, size_t N4 = 0, \
			   size_t N5 = 0, size_t N6 = 0);

       指定された次元の非管理ビューに必要なバイト数を返します。 この関数は、 ``array_layout::is_regular == true``　である場合にのみ有効です。

   .. cpp:function:: static constexpr size_t required_allocation_size(const array_layout& layout);

       :return: 提供されたレイアウトのの非管理ビューに必要なバイト数を返します。

   .. rubric:: 他のパブリックメソッド

   .. cpp:function:: int use_count() const

       :return: 基盤となる割り当ての現在の参照カウント。

   .. cpp:function:: const char* label() const;

       :return:  ``DynRankView``　のラベル。

   .. cpp:function:: constexpr unsigned rank() const

       :return:  ``DynRankView``　のダイナミックランク。

   .. cpp:function:: constexpr bool is_allocated() const

       :return: ビューが有効なメモリ領域を指している場合に真となります。
		この関数は管理ビューと非管理ビューの両方で機能します。
		非管理ビューでは、参照されるアドレスが有効である保証はなく、単にヌルポインタでないことのみが保証されます。

割当ルール
----------------

割当ルールは割り当て演算子とコピー構造体の両方を網羅します。 すべての論理的かた合法な割り当てを可能にする一方で、可能であればコンパイル時に、そうでなければ実行時に不正な割り当てを傍受することを目指しています。 以下では、それぞれ ``DstType`` と ``SrcType`` を宛先ビューとソースビューの型として使用します。 ``dst_view`` および ``src_view`` は、宛先ビューとソースビューの実行時インスタンスを示しています。すなわち、:

.. code-block:: cpp

    ScrType src_view(...);
    DstType dst_view(src_view);
    dst_view = src_view;

以下の条件はコンパイル時に満たされ、評価されます :

* ``DstType::rank == SrcType::rank``
* ``DstType::non_const_value_type`` は、 ``SrcType::non_const_value_type``　と同じです。
*  ``std::is_const<DstType::value_type>::value == true``　よりも ``std::is_const<SrcType::value_type>::value == true`` である場合。
* ``MemorySpaceAccess<DstType::memory_space,SrcType::memory_space>::assignable == true``

さらに、``DstType::array_layout`` が ``SrcType::array_layout``と同じでない場合、充足すべきルールもあります。これらのルールは、両方のレイアウトが、 ``LayoutLeft`` , ``LayoutRight`` or ``LayoutStride``、``LayoutRight`` または ``LayoutStride``　のいずれかの場合のみを対象としています。

* ``DstType::array_layout`` も ``SrcType::array_layout`` も ``LayoutStride``　ではない場合　:

    -  ``DstType::array_layout``　よりも``DstType::rank > 1`` である場合、 ``SrcType::array_layout``　と同じである必要があります。

*  ``DstType::array_layout`` または ``SrcType::array_layout`` が ``LayoutStride``　である場合　：
    - 各ディメンション ``k`` については、その ``dst_view.extent(k) == src_view.extent(k)``　を保持する必要があります。

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
