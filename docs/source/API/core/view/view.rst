``View``
========

ヘッダーファイル: ``<Kokkos_Core.hpp>``

.. _CppReferenceSharedPtr: https://en.cppreference.com/w/cpp/memory/shared_ptr

.. |CppReferenceSharedPtr| 置換:: ``std::shared_ptr``

.. _ProgrammingGuide: ../../../ProgrammingGuide/View.html#memory-access-traits

.. |ProgrammingGuide| 置換:: プログラミングガイド

クラスインターフェイス
---------------

.. cpp:class:: テンプレート <class DataType, class... Properties> View

   Kokkos Viewは、コンパイル時のレイアウトとメモリ空間を持つ、参照カウント可能な多次元配列です。
   そのセマンティクスは、|CppReferenceSharedPtr|_　のものと同様です。
   
   :tparam DataType: :cpp:class:`View` の基本スカラー型とその次元性を定義します。

      基本構造は、　``ScalarType STARS BRACKETS`` であり、ここで　``STARS``　の数は実行時長さの次元数を示し、
　　　``BRACKETS`` の数はコンパイル時次元数を定義します。
      Due to C++ type restrictions runtime dimensions must come first.C++の型制限により、実行時の次元は最初に指定されなければなりません。

      例:

      - :cpp:`double**`: 2つのランタイム次元に持つ :cpp:`double` の2Dビュー
      - :cpp:`const int***[5][3]`: 3つの実行時次元と2つのコンパイル時次元を持つ　cpp:`int`（）の5D ビュー：
         データは、:cpp:`const`.
      - :cpp:`Foo[6][2]`: コンパイル時2次元を持つ :cpp:`Foo` クラスの　2D　ビュー

   :tparam Properties...: レイアウト、メモリ空間、メモリ特性等、:cpp:class:`View` の様々なプロパティを定義します。
   
      :cpp:class:`View`　クラスのテンプレートパラメータにおいて、　`DataType`　以降のパラメータは可変長かつオプションですが、必ず指定順序で指定する必要があります。 例えば、:cpp:any:`LayoutType` は省略可能であることを意味しますが、:cpp:any:`MemorySpace` と :cpp:`MemoryTraits` の両方が指定される場合、:cpp:any:`MemorySpace` は :cpp:any:`MemoryTraits` の前に記述されなければならない。 

      .. code-block:: cpp
         :caption: ビューテンプレートパラメータの順序付け。

         テンプレート <class DataType [, class LayoutType] [, class MemorySpace] [, class MemoryTraits]>
         クラスビュー;

   :tparam LayoutType: インデックスの基盤となる1次元メモリストレージへのマッピングを決定します。
   
      Kokkos　には、いくつかの組み込みレイアウトが付属しています:

      - :cpp:struct:`LayoutRight`: ストライドは、右端から左端の次元に向かって増加します。
         最後の次元は、ストライドが1です。
         これはC言語の多次元配列（例：　`foo[][][]`　）がメモリ上に配置される方法に対応しています。
      - :cpp:struct:`LayoutLeft`: ストライドは、左端から右端の次元に向かって増加します。
         最初の次元は、ストライドが1です。これは、Fortran　が配列に使用するレイアウトです。
      - :cpp:struct:`LayoutStride`: 各次元ごとのストライドは、任意に設定できます。
   
   :tparam MemorySpace: ビューの保存場所を、管理します。

      省略された場合、デフォルトの実行スペースのデフォルトのメモリ領域が使用されます (つまり、:cpp:expr:`DefaultExecutionSpace::memory_space`)。

   :tparam MemoryTraits: 構造体テンプレート :cpp:struct:`MemoryTraits` の列挙型パラメータを介して、アクセプロパティを設定します。可能なテンプレート引数は、以下のフラグの　bitwise OR　です: 

      - ``Unmanaged``
      - ``RandomAccess``
      - ``Atomic``
      - ``Restrict``
      - ``Aligned``

      詳細については、ProgrammingGuide|_ のメモリアクセス特性に関するサブセクションも、参照してださい。

..
   ここで、"namespace"　を押します; これは名前空間エンティティを作成するものではなく、ここからポップまでのすべて　ビュークラスの一部であることを、Sphinx　に伝えます。
   すべてのエンティティは、依然としてスコープを介して参照されます (つまり、 View::data_type)。

.. cpp:namespace-push:: テンプレート <class DataType, class... Properties> ビュー

パブリック定数
^^^^^^^^^^^^^^^^

.. cpp:member:: static constexpr bool reference_type_is_lvalue_reference

   参照型が、C++左辺値参照であるかどうかの確認

データ型
^^^^^^^^^^

.. cpp:type:: data_type

   :cpp:any: :cpp:class:`View`　の `DataType`　であり、　:cpp:type:`data_type` には配列指定子（例: :cpp:`int**[3]`）が含まれることに、注意してください。

.. cpp:type:: const_data_type

   `DataType`の　:cpp:`const` バージョンであり,  それがすでに  :cpp:`const`　であれば、:cpp:type:`data_type`　と同じです。

.. cpp:type:: non_const_data_type

   :cpp:any:`DataType`　の非　:cpp:`const` バージョンであり、 それがすでに  non-:cpp:`const`　であれば、:cpp:type:`data_type`　と同じです。

.. cpp:type:: scalar_array_type

   If :cpp:any:`DataType` が Sacado FAD 型のような適切に特化された配列データ型を表す場合、 :cpp:type:`scalar_array_type` は、基礎となる基本スカラー型です。

.. cpp:type:: const_scalar_array_type

   :cpp:type:`scalar_array_type`　の  :cpp:`const`　バージョンであり、それがすでに  :cpp:`const`　であれば、:cpp:type:`scalar_array_type` と同じです。

.. cpp:type:: non_const_scalar_array_type

    :cpp:type:`scalar_array_type`　の非 　:cpp:`const` バージョンであり、それがすでに  non-:cpp:`const`　 :cpp:type:`scalar_array_type` と同じです。


スカラー型
^^^^^^^^^^^^

.. cpp:type:: value_type

   配列指定子を削除した :cpp:type:`data_type`、つまり、 ビューが参照しているデータのスカラー型　(例えば、:cpp:type:`data_type` が :cpp:`const int**[3]`　であれば、 :cpp:type:`value_type` は、 :cpp:`const int`　です)。

.. cpp:type:: const_value_type

   :cpp:type:`value_type`　の :cpp:`const`　バージョン。

.. cpp:type:: non_const_value_type

   :cpp:type:`value_type`　の　non-:cpp:`const` バージョン。


空間
^^^^^^

.. cpp:type:: execution_space

   ビューに関連付けられた :ref:`execution space <api-execution-spaces>` は、ビュー初期化実行、および特定の and certain deep_copy 演算子に使用されます。

.. cpp:type:: memory_space

    :cpp:class:`View` データが格納されている、:ref:`memory space <api-memory-spaces>` 。

.. cpp:type:: device_type

   :cpp:expr:`Device<execution_space, memory_space>`　で定義される複合型。

.. cpp:type:: memory_traits

   ビューのメモリトレイト。

.. cpp:type:: host_mirror_space

   :cpp:type:`HostMirror`　で使用されるホストアクセス可能メモリ領域。

ビュー型
^^^^^^^^^^

.. cpp:type:: non_const_type

   :cpp:any:`DataType` テンプレートパラメータとして渡された、　:cpp:class:type with :cpp:type:`non_const_data_type` を持つ、この　:cpp:class:`View` 。

.. cpp:type:: const_type

   :cpp:any:`DataType` テンプレートパラメータとして渡された、:cpp:type:`const_data_type`　を持つ、この　:cpp:class:`View`。

.. cpp:type:: HostMirror

   同じ :cpp:type:`data_type` を持つ、互換性のあるビュー型、およびホストアクセスっ可能なメモリ空間内に格納された :cpp:type:`array_layout` 。


データハンドル
^^^^^^^^^^^^

.. cpp:type:: reference_type

   ビューアクセス演算子の戻り値の型。

   .. seealso::
      :cpp:func:`operator()`

      :cpp:func:`access()`


.. cpp:type:: pointer_type

   :cpp:type:`value_type`　へのポインタ。


他の型
^^^^^^^^^^^

.. cpp:type:: array_layout

    :cpp:class:`View`　の　:cpp:any:`LayoutType`。

.. cpp:type:: size_type

   この :cpp:class:`View`　の　メモリ空間に関するインデックス型。

.. cpp:type:: 次元

   :cpp:class:`View`　の範囲を表すことができる整数配列のような型。

.. cpp:type:: 特殊化

   :cpp:class:`View` の基盤となるマッピング構造の部分的な特殊化に使用される特殊化タグ。


コンストラクタ
^^^^^^^^^^^^

.. cpp:function:: View()

   デフォルトコンストラクタ。 割り当ては行われず、参照カウントも発生しません。すべての領域はゼロであり、データ指針は :cpp:`nullptr` です。

.. cpp:function:: template<class DT, class... Prop> View( const View<DT, Prop...>& rhs)

   互換性のあるビューを持つコピーコンストラクタ。以下の　`View`　クラスの代入ルールに従います。

   .. seealso:: :ref:`api-view-assignment`

.. cpp:function:: View(View&& rhs)

   移動コンストラクタ

.. cpp:function:: template<class IntType> View( const std::string& name, const IntType& ... extents)

   標準割り当てコンストラクタ。初期化は、:cpp:type:`memory_space`　に対応する実行空間のデフォルトインスタンス上で実行され、それをフェンスします。

   :tparam IntType: 整数型

   :param name: ユーザーにより提供されたラベルで、 プロファイリングおよびデバッグの目的で使用されます。 名前は、独自のものである必要はありません。

   :param extents: :cpp:class:`View`　の範囲。

   .. rubric:: 必要要件:

   - :cpp:expr:`sizeof(IntType...) == rank_dynamic()` or :cpp:expr:`sizeof(IntType...) == rank()`.
      後者の場合、コンパイル時の次元に対応する範囲は、:cpp:class:`View` 型のコンパイル時の範囲と一致する必要があります。
   - :cpp:expr:`array_layout::is_regular == true`.

.. cpp:function:: View( const std::string& name, const array_layout& layout)

    標準割り当てコンストラクタ。初期化は、:cpp:type:`memory_space`　に対応する実行空間のデフォルトインスタンス上で実行され、それをフェンスします。

   :param name: ユーザーにより提供されたラベルで、 プロファイリングおよびデバッグの目的で使用されます。
      名前は、独自のものである必要はありません。

   :param layout: レイアウトクラスのインスタンス。
      有効な範囲の数は、:cpp:func:`rank_dynamic` または :cpp:func:`rank` と一致する必要があります。
      後者の場合、コンパイル時の次元に対応する範囲は、:cpp:class:`View` 型のコンパイル時の範囲と一致する必要があります。

.. cpp:function:: template<class IntType> View( const ALLOC_PROP &prop, const IntType& ... extents)

   割り当てプロパティを持つ割り当てコンストラクタ（　:cpp:func:`view_alloc` の呼び出しによって作成される）。実行空間が、 
   :cpp:any:`prop`　において特定される場合には、 初期化ではそれは使われず、フェンスは設定されません。
   そうでない場合には、:cpp:class:`View` は、:cpp:type:`memory_space` に対応するデフォルトの実行空間インスタンスを使用して初期化され、フェンスが設定されます。

   :tparam IntType: 整数型

   :param prop: :cpp:func:`view_alloc`　によって返される割り当てプロパティオブジェクト。

   :param extents: ビューの範囲

   .. rubric:: 必要要件:

   - :cpp:expr:`sizeof(IntType...) == rank_dynamic()` or :cpp:expr:`sizeof(IntType...) == rank()`.
      後者の場合、コンパイル時の次元に対応する範囲は、:cpp:class:`View` 型のコンパイル時の範囲と一致する必要があります。
   - :cpp:expr:`array_layout::is_regular == true`.

.. cpp:function:: View( const ALLOC_PROP &prop, const array_layout& layout)

   割り当てプロパティ　( :cpp:func:`view_alloc`　への呼び出しにより作成) および　レイアウトオブジェクトを使って、コンストラクタを割り当てます。実行空間が、
  :cpp:any:`prop`　において、特定される場合には、  初期化ではそれは使われず、フェンスは設定されません。
   そうでない場合には、:cpp:class:`View` は、:cpp:type:`memory_space` に対応するデフォルトの実行空間インスタンスを使用して初期化され、フェンスが設定されます。

   :param prop: `view_alloc`　によって返される割り当てプロパティオブジェクト。

   :param layout: レイアウトクラスのインスタンス。
      有効な範囲の数は、:cpp:func:`rank_dynamic` または :cpp:func:`rank` と一致する必要があります。
      後者の場合、コンパイル時の次元に対応する範囲は、:cpp:class:`View` 型のコンパイル時の範囲と一致する必要があります。

.. cpp:function:: template<class IntType> View( pointer_type ptr, const IntType& ... extents)

   管理対象外データラッピングコンストラクタ。

   :tparam IntType: 整数型。

   :param ptr: ユーザーにより提供されたメモリ割り当てへのポインタ。
      サイズ :cpp:expr:`required_allocation_size(extents...)`　のストレージを提供する必要があります。

   :param extents: :cpp:class:`View`　の範囲。

   .. rubric:: 必要要件:

   - :cpp:expr:`sizeof(IntType...) == rank_dynamic()` or :cpp:expr:`sizeof(IntType...) == rank()`.
       後者の場合、コンパイル時の次元に対応する範囲は、:cpp:class:`View` 型のコンパイル時の範囲と一致する必要があります。
   - :cpp:expr:`array_layout::is_regular == true`.

.. cpp:function:: View( pointer_type ptr, const array_layout& layout)

   管理対象外データラッピングコンストラクタ。

   :param ptr: ユーザーにより提供されたメモリ割り当てへのポインタ
      :cpp:expr:`View::required_allocation_size(layout)`　のストレージを提供する必要があります。

   :param layout: レイアウトクラスのインスタンス。
      有効な範囲の数は、ダイナミックまたはトータルランクと一致する必要があります。 
      後者の場合、コンパイル時の次元に対応する範囲は、:cpp:class:`View` 型のコンパイル時の範囲と一致する必要があります。

.. cpp:function:: template<class IntType> View( const ScratchSpace& space, const IntType& ... extents)

   スクラッチメモリハンドルからメモリを取得するコンストラクタ。

   :tparam IntType: 整数型

   :param space: 
     一般的には、``TeamPolicy`` カーネル内の、:cpp:func:`team_shmem`, :cpp:func:`team_scratch`, または、 :cpp:func:`thread_scratch` から返されます。

   :param extents: Extents of the :cpp:class:`View`.

   .. rubric:: Requirements:スクラッチメモリハンドル

   - :cpp:expr:`sizeof(IntType...) == rank_dynamic()` or :cpp:expr:`sizeof(IntType...) == rank()`.
       後者の場合、コンパイル時の次元に対応する範囲は、:cpp:class:`View` 型のコンパイル時の範囲と一致する必要があります。
   - :cpp:expr:`array_layout::is_regular == true`.

.. cpp:function:: View( const ScratchSpace& space, const array_layout& layout)

   スクラッチメモリハンドルからメモリを取得するコンストラクタ。

   :param space: scratch memory handle.
       一般的には、``TeamPolicy`` カーネル内の、 :cpp:func:`team_shmem`, :cpp:func:`team_scratch`、または :cpp:func:`thread_scratch` から返されます。

   :param layout: an instance of a layout class.
       有効な範囲の数は、ダイナミックまたはトータルランクと一致する必要があります。 後者の場合、コンパイル時の次元に対応する範囲は、:cpp:class:`View` 型のコンパイル時の範囲と一致する必要があります。

.. cpp:function:: template<class DT, class... Prop> View( const View<DT, Prop...>& rhs, Args ... args)

   :param rhs: サブビューを取得する　:cpp:class:`View`。
   :param args...:　:cpp:func:`subview` で指定されたサブビューのスライス。

   サブビューコンストラクタ。

   .. seealso:: :cpp:func:`subview`

.. cpp:function:: explicit(traits::is_managed) View( const NATURAL_MDSPAN_TYPE& mds )

   :param mds: 変換元のmdspan。

   .. 警告::

      :cpp:`explicit(bool)` は、C++20　以降でのみ利用可能です。 C++17を使って、Kokkos　を構築する場合、このコンストラクタは、完全に暗黙的に定義されます。
      C++20への、その後のアップグレードでは、:cpp:`traits::is_managed` が、 :cpp:`false` である場合、コンパイルエラーが発生する可能性があることに、注意してください。

   :cpp:`NATURAL_MDSPAN_TYPE` は、ビューの　:ref:`natural mdspan <api-view-natural-mdspans>` です。 :cpp:type:`array_layout` が、 :cpp:struct:`LayoutLeft`、 :cpp:struct:`LayoutRight`、または :cpp:class:`LayoutStride`のうちの1つである場合にのみ、 *natural mdspan* が利用可能です。 *natural mdspan* が利用可能である場合にのみ、このコンストラクタは、利用可能です。

   :cpp:any:`mds` から変換して :cpp:class:`View` を構築します。 :cpp:class:`View` は、管理対象外となり、 :cpp:`View(mds.data(), array_layout_from_mapping(mds.mapping()))`　によるかの様に構築されます。

   .. seealso:: :ref:`Natural mdspans <api-view-natural-mdspans>`

   .. versionadded:: 4.4.0

.. cpp:function:: template <class ElementType, class ExtentsType, class LayoutType, class AccessorType> explicit(SEE_BELOW) View(const mdspan<ElementType, ExtentsType, LayoutType, AccessorType>& mds)

   :tparam ElementType:  mdspan 要素型
   :tparam ExtentsType:  mdspan 範囲
   :tparam LayoutType:  mdspan レイアウト
   :tparam AccessorType: mdspan 範囲

   :param mds: 変換元のmdspan。

   .. 警告::

      :cpp:`explicit(bool)` は、C++20　以降でのみ利用可能です。 C++17を使って、Kokkos　を構築する場合、このコンストラクタは、完全に暗黙的に定義されます。
      C++20への、その後のアップグレードでは、:cpp:`traits::is_managed` が、 :cpp:`false` である場合、コンパイルエラーが発生する可能性があることに、注意してください。
   :cpp:any:`mds`　～変換することにより、:cpp:class:`View` を構築します。
   The :cpp:class:`View`'s :ref:`natural mdspan <api-view-natural-mdspans>` は、 :cpp:any:`mds`　から構築可能でなければなりません。 :cpp:class:`View` は、 :cpp:`View(NATURAL_MDSPAN_TYPE(mds))`　によるかの様に、構築されます。

   In C++20:
      このコンストラクタは、This constructor is implicit if :cpp:any:`mds` が、 :cpp:class:`View`　の　*natural mdspan* に暗示的に変換可能です。

   .. versionadded:: 4.4.0


データアクセス関数
^^^^^^^^^^^^^^^^^^^^^

.. cpp:function:: template<class IntType> reference_type operator() (const IntType& ... indices) const

   :tparam IntType: an integral type

   :param indices: 要素のインデックスを取得して参照を取得します。
   :return: 指定されたインデックスの要素への参照。

   :cpp:type:`reference_type` の値を返しますが、この型自体は参照可能である場合もあれば、そうでない場合もあります。
   インデックス引数の数は、ビューの　:cpp:func:`rank`　に一致する必要があります。

   .. rubric:: 必要要件:
   
   - :cpp:expr:`sizeof(IntType...) == rank_dynamic()`

.. cpp:function:: template<class IntType> reference_type access(const IntType& i0=0, const IntType& i1=0, \
         const IntType& i2=0, const IntType& i3=0, const IntType& i4=0, \
         const IntType& i5=0, const IntType& i6=0, const IntType& i7=0) const

   :tparam IntType: 整数型
   
   :param i0, i1, i2, i3, i4, i5, i6, i7: 参照を取得する要素のインデックス。
   :return: 指定されたインデックスの要素への参照。

   :cpp:type:`reference_type` の値を返しますが、この型自体は参照可能である場合もあれば、そうでない場合もあります。
   インデックス引数の数は、ビューの :cpp:func:`rank` 以上である必要があります。
   Index arguments beyond :cpp:func:`rank` must be :cpp:`0`, which will be enforced if :cpp:any:`KOKKOS_DEBUG` is defined.:cpp:func:`rank`を超えるインデックス引数は、　:cpp:`0`　でなければなりませんが、これは、　:cpp:any:`KOKKOS_DEBUG`　が定義されている場合に、必ず行われます。


データレイアウト、次元、ストライド
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. cpp:function:: static constexpr size_t rank()

   :return: ビューのランク

   .. versionadded:: 4.1

.. cpp:function:: static constexpr size_t rank_dynamic()

   :return: 実行時に決定される次元の数。

   .. versionadded:: 4.1

.. 注意事項::

   In practice, :cpp:func:`rank()` and :cpp:func:`rank_dynamic()` are not actually implemented as static member functions but ``rank`` and ``rank_dynamic`` underlying types have a nullary member function (i.e. callable with no argument).

.. versionchanged:: 4.1

   :cpp:func:`rank` and :cpp:func:`rank_dynamic` are static member constants that are convertible to :cpp:`size_t`.
   Their underlying types are unspecified, but equivalent to :cpp:`std::integral_constant` with a nullary member function callable from host and device side.
   Users are encouraged to use :cpp:`rank()` and :cpp:`rank_dynamic()` (akin to a static member function call) instead of relying on implicit conversion to an integral type.

   The actual type of :cpp:func:`rank` and :cpp:func:`rank_dynamic` as they were defined until Kokkos 4.1 was left up to the implementation (that is, up to the compiler not to Kokkos) but in practice it was often :cpp:`int` which means this change may yield warnings about comparing signed and unsigned integral types.
   It may also break code that was using the type of :cpp:func:`rank`.
   Furthermore, it appears that MSVC has issues with the implicit conversion to :cpp:`size_t` in certain constexpr contexts. Calling :cpp:func:`rank()` or :cpp:func:`rank_dynamic()` will work in those cases.

.. cpp:function:: constexpr array_layout layout() const

   :return: the layout object that can be used to to construct other views with the same dimensions.

.. cpp:function:: template<class iType> constexpr size_t extent( const iType& dim) const

   :tparam iType: an integral type
   :param dim: the dimension to get the extent of
   :return: the extent of dimension :cpp:any:`dim`

   .. rubric:: Preconditions:

   - :cpp:any:`dim` must be smaller than :cpp:func:`rank`.

.. cpp:function:: template<class iType> constexpr int extent_int( const iType& dim) const

   :tparam iType: an integral type
   :param dim: the dimension to get the extent of
   :return: the extent of dimension :cpp:any:`dim` as an :cpp:`int`

   Compared to :cpp:func:`extent` this function can be
   useful on architectures where :cpp:`int` operations are more efficient than :cpp:`size_t`.
   It also may eliminate the need for type casts in applications which
   otherwise perform all index operations with :cpp:`int`.

   .. rubric:: Preconditions:

   - :cpp:any:`dim` must be smaller than :cpp:func:`rank`.

.. cpp:function:: template<class iType> constexpr size_t stride(const iType& dim) const

   :tparam iType: an integral type
   :param dim: the dimension to get the stride of
   :return: the stride of dimension :cpp:any:`dim`

   Example: :cpp:expr:`a.stride(3) == (&a(i0,i1,i2,i3+1,i4)-&a(i0,i1,i2,i3,i4))`

   .. rubric:: Preconditions:

   - :cpp:any:`dim` must be smaller than :cpp:func:`rank`.

.. cpp:function:: constexpr size_t stride_0() const

   :return: the stride of dimension 0.

.. cpp:function:: constexpr size_t stride_1() const

   :return: the stride of dimension 1.

.. cpp:function:: constexpr size_t stride_2() const

   :return: the stride of dimension 2.

.. cpp:function:: constexpr size_t stride_3() const

   :return: the stride of dimension 3.

.. cpp:function:: constexpr size_t stride_4() const

   :return: the stride of dimension 4.

.. cpp:function:: constexpr size_t stride_5() const

   :return: the stride of dimension 5.

.. cpp:function:: constexpr size_t stride_6() const

   :return: the stride of dimension 6.

.. cpp:function:: constexpr size_t stride_7() const

   :return: the stride of dimension 7.

.. cpp:function:: template<class iType> void stride(iType* strides) const

   :tparam iType: an integral type
   :param strides: the output array of length :cpp:expr:`rank() + 1`

   Sets :cpp:expr:`strides[r]` to :cpp:expr:`stride(r)` for all :math:`r` with :math:`0 \le r \lt \texttt{rank()}`.
   Sets :cpp:expr:`strides[rank()]` to :cpp:func:`span()`.

   .. rubric:: Preconditions:

   - :cpp:any:`strides` must be an array of length :cpp:expr:`rank() + 1`

.. cpp:function:: constexpr size_t span() const

   :return: the size of the span of memory between the element with the lowest and highest address

   Obtains the memory span in elements between the element with the
   lowest and the highest address. This can be larger than the product
   of extents due to padding, and or non-contiguous data layout as for example :cpp:struct:`LayoutStride` allows.

.. cpp:function:: constexpr size_t size() const

   :return: the product of extents, i.e. the logical number of elements in the :cpp:class:`View`.

.. cpp:function:: constexpr pointer_type data() const

   :return: the pointer to the underlying data allocation.

   .. warning::
   
      Calling any function that manipulates the behavior of the memory (e.g. ``memAdvise``) on memory managed by Kokkos results in undefined behavior.

.. cpp:function:: bool span_is_contiguous() const

   :return: whether the span is contiguous (i.e. whether every memory location between in span belongs to the index space covered by the :cpp:class:`View`).

.. cpp:function:: static constexpr size_t required_allocation_size(size_t N0=0, size_t N1=0, \
         size_t N2=0, size_t N3=0, \
         size_t N4=0, size_t N5=0, \
         size_t N6=0, size_t N7=0);
   
   :param N0, N1, N2, N3, N4, N5, N6, N7: the dimensions to query
   :return: the number of bytes necessary for an unmanaged :cpp:class:`View` of the provided dimensions.

   .. rubric:: Requirements:
   
   - :cpp:expr:`array_layout::is_regular == true`.

.. cpp:function:: static constexpr size_t required_allocation_size(const array_layout& layout);

   :param layout: the layout to query
   :return: the number of bytes necessary for an unmanaged :cpp:class:`View` of the provided layout.

Other Utility Methods
^^^^^^^^^^^^^^^^^^^^^

.. cpp:function:: int use_count() const;

   :return: the current reference count of the underlying allocation.

.. cpp:function:: const std::string label() const;

   :return: the label of the View.

.. cpp:function:: void assign_data(pointer_type arg_data);

   :param arg_data: the pointer to set the underlying :cpp:class:`View` data pointer to

   Decrement reference count of previously assigned data and set the underlying pointer to arg_data.
   Note that the effective result of this operation is that the view is now an unmanaged view; thus, the deallocation of memory associated with arg_data is not linked in anyway to the deallocation of the view.

.. cpp:function:: constexpr bool is_allocated() const;

   :return: true if the view points to a valid memory location.

   This function works for both managed and unmanaged views.
   With the unmanaged view, there is no guarantee that referenced address is valid, only that it is a non-null pointer.

Conversion to mdspan
^^^^^^^^^^^^^^^^^^^^

.. cpp:function:: template <class OtherElementType, class OtherExtents, class OtherLayoutPolicy, class OtherAccessor> constexpr operator mdspan<OtherElementType, OtherExtents, OtherLayoutPolicy, OtherAccessor>()

   :tparam OtherElementType: the target mdspan element type
   :tparam OtherExtents: the target mdspan extents
   :tparam OtherLayoutPolicy: the target mdspan layout
   :tparam OtherAccessor: the target mdspan accessor

   :constraints: :cpp:class:`View`\ 's :ref:`natural mdspan <api-view-natural-mdspans>` must be assignable to :cpp:`mdspan<OtherElementType, OtherExtents, OtherLayoutPolicy, OtherAccessor>`

   :returns: an mdspan with extents and a layout converted from the :cpp:class:`View`'s *natural mdspan*.

.. cpp:function:: template <class OtherAccessorType = default_accessor<typename traits::value_type>> constexpr auto to_mdspan(const OtherAccessorType& other_accessor = OtherAccessorType{})

   :tparam OtherAccessor: the target mdspan accessor

   :constraints: :cpp:`typename OtherAccessorType::data_handle_type` must be assignable to :cpp:`value_type*`

   :returns: :cpp:class:`View`\ 's :ref:`natural mdspan <api-view-natural-mdspans>`, but with an accessor policy constructed from :cpp:any:`other_accessor`

.. cpp:namespace-pop::


Non-Member Functions
--------------------

.. cpp:function:: template <class... ViewTDst, class... ViewTSrc> bool is_assignable(const View<ViewTDst...>& dst, const View<ViewTSrc...>& src)

   :return: true if src can be assigned to dst.

   .. seealso:: :ref:`api-view-assignment`

.. cpp:function:: template <class LT, class... LP, class RT, class... RP> bool operator==(const View<LT, LP...>& lhs, const View<RT, RP...>& rhs)

   :return: :cpp:`true` if :cpp:type:`~View::value_type`, :cpp:type:`~View::array_layout`, :cpp:type:`~View::memory_space`, :cpp:func:`~View::rank()`, :cpp:func:`~View::data()` and :cpp:expr:`extent(r)`, for :math:`0 \le r \lt \texttt{rank()}`, match.

.. cpp:function:: template <class LT, class... LP, class RT, class... RP> bool operator!=(const View<LT, LP...>& lhs, const View<RT, RP...>& rhs)

   :return: :cpp:expr:`!(lhs == rhs)`

.. _api-view-assignment:

Assignment Rules
----------------

Assignment rules cover the assignment operator as well as copy constructors.
We aim at making all logically legal assignments possible, while intercepting illegal assignments if possible at compile time, otherwise at runtime.
In the following we use ``DstType`` and ``SrcType`` as the type of the destination view and source view respectively. 
``dst_view`` and ``src_view`` refer to the runtime instances of the destination and source views, i.e.:

.. code-block:: cpp

    SrcType src_view(...);
    DstType dst_view(src_view);
    dst_view = src_view;

The following conditions must be met at and are evaluated at compile time:

* :cpp:`DstType::rank() == SrcType::rank()`
* :cpp:`DstType::non_const_value_type` is the same as :cpp:`SrcType::non_const_value_type`
* If :cpp:`std::is_const_v<SrcType::value_type> == true` then :cpp:`std::is_const_v<DstType::value_type>` must also be :cpp:`true`.
* :cpp:`MemorySpaceAccess<DstType::memory_space,SrcType::memory_space>::assignable == true`
* If :cpp:`DstType::rank_dynamic() != DstType::rank()` and :cpp:`SrcType::rank_dynamic() != SrcType::rank()` then for each dimension :cpp:`k` that is compile time for both it must be true that :cpp:`dst_view.extent(k) == src_view.extent(k)`

Additionally the following conditions must be met at runtime:

* If :cpp:`DstType::rank_dynamic() != DstType::rank()` then for each compile time dimension :cpp:`k` it must be true that :cpp:`dst_view.extent(k) == src_view.extent(k)`.

Furthermore there are rules which must be met if :cpp:`DstType::array_layout` is not the same as :cpp:`SrcType::array_layout`.
These rules only cover cases where both layouts are one of :cpp:class:`LayoutLeft`, :cpp:class:`LayoutRight` or :cpp:class:`LayoutStride`

* If neither :cpp:`DstType::array_layout` nor :cpp:`SrcType::array_layout` is :cpp:class:`LayoutStride`:

  - If :cpp:`DstType::rank > 1` then :cpp:`DstType::array_layout` must be the same as :cpp:`SrcType::array_layout`.

* If either :cpp:`DstType::array_layout` or :cpp:`SrcType::array_layout` is :cpp:class:`LayoutStride`

  - For each dimension :cpp:`k` it must hold that :cpp:`dst_view.extent(k) == src_view.extent(k)`

.. code-block:: cpp
   :caption: Assignment Examples

    View<int*>       a1 = View<int*>("A1",N);     // OK
    View<int**>      a2 = View<int*[10]>("A2",N); // OK
    View<int*[10]>   a3 = View<int**>("A3",N,M);  // OK if M == 10 otherwise runtime failure
    View<const int*> a4 = a1;                     // OK
    View<int*>       a5 = a4;                     // Error: const to non-const assignment
    View<int**>      a6 = a1;                     // Error: Ranks do not match
    View<int*[8]>    a7 = a3;                     // Error: compile time dimensions do not match
    View<int[4][10]> a8 = a3;                     // OK if N == 4 otherwise runtime failure
    View<int*, LayoutLeft>    a9  = a1;           // OK since a1 is either LayoutLeft or LayoutRight
    View<int**, LayoutStride> a10 = a8;           // OK
    View<int**>               a11 = a10;          // OK
    View<int*, HostSpace> a12 = View<int*, CudaSpace>("A12",N); // Error: non-assignable memory spaces
    View<int*, HostSpace> a13 = View<int*, CudaHostPinnedSpace>("A13",N); // OK

.. _api-view-natural-mdspans:

Natural mdspans
---------------

.. versionadded:: 4.4.0

C++23 introduces `mdspan <https://en.cppreference.com/w/cpp/container/mdspan>`_, a non-owning multidimensional array view.
:cpp:class:`View` is compatible with :cpp:`std::mdspan` and can be implicitly converted from and to valid mdspans.
These conversion rules are dictated by the *natural mdspan* of a view.
For an mdspan :cpp:`m` of type :cpp:`M` that is the *natural mdspan* of a :cpp:class:`View` :cpp:`v` of type :cpp:`V`, the following properties hold:

#. :cpp:`M::value_type` is :cpp:`V::value_type`
#. :cpp:`M::index_type` is :cpp:`std::size_t`.
#. :cpp:`M::extents_type` is :cpp:`std::extents<M::index_type, Extents...>` where

   * :cpp:`sizeof(Extents...)` is :cpp:`V::rank()`
   * and each element at index :cpp:`r` of :cpp:`Extents...` is :cpp:`V::static_extents(r)` if :cpp:`V::static_extents(r) != 0`, otherwise :cpp:`std::dynamic_extent`

#. :cpp:`M::layout_type` is

   * :cpp:`std::layout_left_padded<std::dynamic_extent>` if :cpp:`V::array_layout` is :cpp:`LayoutLeft`
   * :cpp:`std::layout_right_padded<std::dynamic_extent>` if :cpp:`V::array_layout` is :cpp:`LayoutRight`
   * :cpp:`std::layout_stride` if :cpp:`V::array_layout` is :cpp:any:`LayoutStride`

#. :cpp:`M::accessor_type` is :cpp:`std::default_accessor<V::value_type>`

Additionally, the *natural mdspan* is constructed so that :cpp:`m.data() == v.data()` and for each extent :cpp:`r`, :cpp:`m.extents().extent(r) == v.extent(r)`.

例
--------

.. code-block:: cpp

    #include<Kokkos_Core.hpp>
    #include<cstdio>

    int main(int argc, char* argv[]) {
        Kokkos::initialize(argc,argv);

        int N0 = atoi(argv[1]);
        int N1 = atoi(argv[2]);

        Kokkos::View<double*> a("A",N0);
        Kokkos::View<double*> b("B",N1);

        Kokkos::parallel_for("InitA", N0, KOKKOS_LAMBDA (const int& i) {
            a(i) = i;
        });

        Kokkos::parallel_for("InitB", N1, KOKKOS_LAMBDA (const int& i) {
            b(i) = i;
        });

        Kokkos::View<double**,Kokkos::LayoutLeft> c("C",N0,N1);
        {
            Kokkos::View<const double*> const_a(a);
            Kokkos::View<const double*> const_b(b);
            Kokkos::parallel_for("SetC", Kokkos::MDRangePolicy<Kokkos::Rank<2,Kokkos::Iterate::Left>>({0,0},{N0,N1}),
                KOKKOS_LAMBDA (const int& i0, const int& i1) {
                c(i0,i1) = a(i0) * b(i1);
            });
        }

        Kokkos::finalize();
    }
