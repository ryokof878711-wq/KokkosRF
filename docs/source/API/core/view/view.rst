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

   実際には、:cpp:func:`rank()` および :cpp:func:`rank_dynamic()` は、静的メンバ関数として実装されていませんが、しかし、``rank``および``rank_dynamic``の基底型には、引数なしのメンバ関数があります。 (つまり、 引数なしで、呼び出し可能)。

.. versionchanged:: 4.1

   :cpp:func:`rank` および　:cpp:func:`rank_dynamic` は、 :cpp:`size_t`　に変換可能な静的メンバ変数です。
   それらの基本型は未指定ですが、ホスト側とデバイス側の双方から呼び出せるゼロ引数メンバー関数を持つ :cpp:`std::integral_constant` と同等です。
   ユーザーには、整数型への暗示的変換に依存する代わりに、:cpp:`rank()` および :cpp:`rank_dynamic()`（静的メンバ関数呼び出しに類似）の使用を推奨します。

   Kokkos 4.1 まで定義されていた :cpp:func:`rank` および :cpp:func:`rank_dynamic` の実際の型は、実装（つまり Kokkos ではなくコンパイラ）に委ねられていましたが、実際には、 :cpp:`int` であることが多く、この変更により符号付きおよび符号なしの整数型の比較に関する警告が発生する可能性があります。
   また、:cpp:func:`rank` 型を使用していたコードが動作しなくなる可能性があります
   さらに、MSVCには、特定の　constexpr　コンテキストにおいて、:cpp:`size_t`　への暗示的な変換には、問題があるように見えます。 その場合には、Calling :cpp:func:`rank()` または :cpp:func:`rank_dynamic()` を呼び出すと、上手く機能します。

.. cpp:function:: constexpr array_layout layout() const

   :return: 同じ寸法で他のビューを構築するために使用可能なレイアウトオブジェクト。

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

   :tparam iType: 整数型
   :param dim: ストライドを獲得するための次元。
   :return: 次元 :cpp:any:`dim`　のストライド。

   例: :cpp:expr:`a.stride(3) == (&a(i0,i1,i2,i3+1,i4)-&a(i0,i1,i2,i3,i4))`

   .. rubric::前提条件:

   - :cpp:any:`dim` は、 :cpp:func:`rank`よりも小さくなければなりません。

.. cpp:function:: constexpr size_t stride_0() const

   :return: 次元 0　のストライド。

.. cpp:function:: constexpr size_t stride_1() const

   :return: 次元 1　のストライド。

.. cpp:function:: constexpr size_t stride_2() const

   :return: 次元 2　のストライド。

.. cpp:function:: constexpr size_t stride_3() const

   :return: 次元 3　のストライド。

.. cpp:function:: constexpr size_t stride_4() const

   :return: 次元 4　のストライド。

.. cpp:function:: constexpr size_t stride_5() const

   :return: 次元 5　のストライド。

.. cpp:function:: constexpr size_t stride_6() const

   :return: 次元 6　のストライド。

.. cpp:function:: constexpr size_t stride_7() const

   :return: 次元 7　のストライド。

.. cpp:function:: template<class iType> void stride(iType* strides) const

   :tparam iType: 整数型
   :param strides: 長さの出力配列 :cpp:expr:`rank() + 1`

   Sets :cpp:expr:`strides[r]` to :cpp:expr:`stride(r)` for all :math:`r` with :math:`0 \le r \lt \texttt{rank()}`.
   Sets :cpp:expr:`strides[rank()]` to :cpp:func:`span()`.

   .. rubric:: 前提条件:

   - :cpp:any:`strides` は、長さ :cpp:expr:`rank() + 1`　の配列である必要があります。

.. cpp:function:: constexpr size_t span() const

   :return: 最低アドレスおよび最高アドレスを持つ要素間の記憶範囲のサイズ。

   最低アドレスおよび最高アドレスを持つ要素間の要素における記憶範囲を取得します。
   これは、パディングによる範囲の積よりも大きくなる可能性があり、
   または、例えば、 :cpp:struct:`LayoutStride` が認める通りに、非連続データレイアウトとなる可能性があります。

.. cpp:function:: constexpr size_t size() const

   :return: 範囲の積、つまり、 :cpp:class:`View`　における論理的要素の数。

.. cpp:function:: constexpr pointer_type data() const

   :return: 基盤データは位置へのポインタ

   .. 警告::
   
      Kokkos　によって管理されているメモリに対して、メモリの動作を操作する関数（例：``memAdvise``）を呼び出すと、未定義の動作を引き起こします。

.. cpp:function:: bool span_is_contiguous() const

   :return: 範囲が連続しているかどうか (つまり、 範囲内のすべてのメモリ位置が、:cpp:class:`View`　に含まれるインデックス空間に属しているかどうか )。

.. cpp:function:: static constexpr size_t required_allocation_size(size_t N0=0, size_t N1=0, \
         size_t N2=0, size_t N3=0, \
         size_t N4=0, size_t N5=0, \
         size_t N6=0, size_t N7=0);
   
   :param N0, N1, N2, N3, N4, N5, N6, N7: 照会対象となる次元
   :return: 指定された次元を持つ管理対象外の :cpp:class:`View` に必要なバイト数

   .. rubric:: 必要要件:
   
   - :cpp:expr:`array_layout::is_regular == true`.

.. cpp:function:: static constexpr size_t required_allocation_size(const array_layout& layout);

   :param layout: 照会対象となるレイアウト
   :return: 提供されたレイアウトの管理対象外である :cpp:class:`View` に必要なバイト数。

その他のユーティリティメソッド
^^^^^^^^^^^^^^^^^^^^^

.. cpp:function:: int use_count() const;

   :return: 基盤となる割り当ての現在の参照カウント。

.. cpp:function:: const std::string label() const;

   :return: ビューのレベル。

.. cpp:function:: void assign_data(pointer_type arg_data);

   :param arg_data: 基盤の :cpp:class:`View` データポインタを設定するポインタ。

   以前に割り当てられたデータの参照カウントを減算し、基底ポインタを arg_data に設定します。
   この演算の有効な結果は、ビューが管理対象外となったことに注意してください; このように、　arg_data　に関連付けられたメモリの解放は、ビューの解放とはまったく関連していません。

.. cpp:function:: constexpr bool is_allocated() const;

   :return: ビューが、有効なメモリ位置付けを指す場合には、真。

   この関数は、管理ビューと管理対象外ビューの両方で機能します。
   管理対象外ビューでは、参照されるアドレスが有効であることが保証されるのではなく、単にヌルポインタでないことのみが保証されます。

mdspanへの変換
^^^^^^^^^^^^^^^^^^^^

.. cpp:function:: テンプレート <class OtherElementType, class OtherExtents, class OtherLayoutPolicy, class OtherAccessor> constexpr　演算子 mdspan<OtherElementType, OtherExtents, OtherLayoutPolicy, OtherAccessor>()

   :tparam OtherElementType: 対象 mdspan 要素型。
   :tparam OtherExtents: 対象 mdspan 範囲。
   :tparam OtherLayoutPolicy: 対象 mdspan　レイアウト。
   :tparam OtherAccessor: 対象 mdspan　アクセサ。

   :constraints: :cpp:class:`View`\ 's :ref:`natural mdspan <api-view-natural-mdspans>` は、must be assignable to :cpp:`mdspan<OtherElementType, OtherExtents, OtherLayoutPolicy, OtherAccessor>`　に割り当て可能でなければなりません。

   :returns: :cpp:class:`View`　の *natural mdspan*　から変換された範囲およびレイアウトを持つ mdspan。

.. cpp:function:: テンプレート <class OtherAccessorType = default_accessor<typename traits::value_type>> constexpr auto to_mdspan(const OtherAccessorType& other_accessor = OtherAccessorType{})

   :tparam OtherAccessor: 対象 mdspan アクセサ

   :constraints: :cpp:`typename OtherAccessorType::data_handle_type` は、 :cpp:`value_type*`　に割り当て可能でなければなりません。

   :returns: :cpp:class:`View`\ 's :ref:`natural mdspan <api-view-natural-mdspans>`　ですが、 :cpp:any:`other_accessor`　～構築されたアクセサポリシーを伴います。

.. cpp:namespace-pop::


非メンバー関数
--------------------

.. cpp:function:: template <class... ViewTDst, class... ViewTSrc> bool is_assignable(const View<ViewTDst...>& dst, const View<ViewTSrc...>& src)

   :return:  src が dst　に代入可能である場合には、真。

   .. seealso:: :ref:`api-view-assignment`

.. cpp:function:: テンプレート <class LT, class... LP, class RT, class... RP> bool operator==(const View<LT, LP...>& lhs, const View<RT, RP...>& rhs)

   :return: :cpp:type:`~View::value_type`, :cpp:type:`~View::array_layout`, :cpp:type:`~View::memory_space`, :cpp:func:`~View::rank()`, :cpp:func:`~View::data()` および :cpp:expr:`extent(r)`, for :math:`0 \le r \lt \texttt{rank()}`が一致すれば、 :cpp:`true` 。

.. cpp:function:: テンプレート <class LT, class... LP, class RT, class... RP> bool operator!=(const View<LT, LP...>& lhs, const View<RT, RP...>& rhs)

   :return: :cpp:expr:`!(lhs == rhs)`

.. _api-view-assignment:

代入ルール
----------------

代入規則は、代入演算子とコピーコンストラクタの両方に適用されます。
論理的に合法な代入をすべて可能にすることを目指しつつ、一方では、違法な代入はコンパイル時に可能な限り介入し、それが不可能な場合には実行時に介入します。
以下では、宛先ビューと送信元ビューのタイプをそれぞれ　``DstType`` および　``SrcType``　として使用します。
``dst_view`` および ``src_view`` は、宛先ビューと送信元ビューのランタイムインスタンスを指します。 つまり:

.. code-block:: cpp

    SrcType src_view(...);
    DstType dst_view(src_view);
    dst_view = src_view;

以下の条件は、コンパイル時に満たされ、かつ評価されなければなりません:

* :cpp:`DstType::rank() == SrcType::rank()`
* :cpp:`DstType::non_const_value_type` が :cpp:`SrcType::non_const_value_type`　と同じ。
* :cpp:`std::is_const_v<SrcType::value_type> == true`　であれば、 :cpp:`std::is_const_v<DstType::value_type>` もまた、 :cpp:`true`　である必要があります。
* :cpp:`MemorySpaceAccess<DstType::memory_space,SrcType::memory_space>::assignable == true`
* :cpp:`DstType::rank_dynamic() != DstType::rank()` および :cpp:`SrcType::rank_dynamic() != SrcType::rank()`　であれば、 各次元 :cpp:`k` について、両方のコンパイル時に、:cpp:`dst_view.extent(k) == src_view.extent(k)`　が真である必要があります。

さらに、実行時には、以下の条件を満たす必要があります:

* :cpp:`DstType::rank_dynamic() != DstType::rank()`　であれば、  各コンパイル時次元 :cpp:`k`　について、 :cpp:`dst_view.extent(k) == src_view.extent(k)`　が真である必要があります。

さらに、:cpp:`DstType::array_layout` が :cpp:`SrcType::array_layout` と同一でない場合、満たさなければならない規則があります。
これらのルールは、両方のレイアウトが :cpp:class:`LayoutLeft`、:cpp:class:`LayoutRight`、または :cpp:class:`LayoutStride` のいずれかである場合にのみ適用されます。

* :cpp:`DstType::array_layout`　または :cpp:`SrcType::array_layout`　が :cpp:class:`LayoutStride`　である場合:

  - If :cpp:`DstType::rank > 1`　であれば、 :cpp:`DstType::array_layout` は、:cpp:`SrcType::array_layout`　と同じである必要があります。

* :cpp:`DstType::array_layout` または :cpp:`SrcType::array_layout` が :cpp:class:`LayoutStride`　である場合

  - 各次元 :cpp:`k` について、:cpp:`dst_view.extent(k) == src_view.extent(k)`でなければなりません。

.. code-block:: cpp
   :caption: 代入例

    View<int*>       a1 = View<int*>("A1",N);     // OK
    View<int**>      a2 = View<int*[10]>("A2",N); // OK
    View<int*[10]>   a3 = View<int**>("A3",N,M);  // OK M == 10　の場合に、OK。そうでなければ実行不能。 
    View<const int*> a4 = a1;                     // OK
    View<int*>       a5 = a4;                     // エラー: const から 非const への代入。
    View<int**>      a6 = a1;                     // エラー: ランクが不一致。
    View<int*[8]>    a7 = a3;                     // エラー: コンパイル時次元が不一致。
    View<int[4][10]> a8 = a3;                     // OK if N == 4 の場合に、OK。そうでなければ実行不能。
    View<int*, LayoutLeft>    a9  = a1;           // a1 が、LayoutLeft または LayoutRight　のいずれかであれば、OK。
    View<int**, LayoutStride> a10 = a8;           // OK
    View<int**>               a11 = a10;          // OK
    View<int*, HostSpace> a12 = View<int*, CudaSpace>("A12",N); // エラー: non-assignable memory spaces代入不可能メモリ空間。
    View<int*, HostSpace> a13 = View<int*, CudaHostPinnedSpace>("A13",N); // OK

.. _api-view-natural-mdspans:

Natural mdspans
---------------

.. versionadded:: 4.4.0

C++23 は、非所有の多次元配列ビュー`　である、mdspan <https://en.cppreference.com/w/cpp/container/mdspan>`_　を導入します。
:cpp:class:`View` は :cpp:`std::mdspan` と互換性があり、有効な mdspan との間で、暗示的に変換可能です。
これらの変換ルールは、ビューの　*natural mdspan*　によって規定されます。
For an mdspan :cpp:`m` of type :cpp:`M` that is the *natural mdspan* of a :cpp:class:`View` :cpp:`v` of type :cpp:`V`, 型 :cpp:`M` の mdspan :cpp:`m` が、型 :cpp:`V` の :cpp:class:`View` :cpp:`v` の *natural mdspan* である場合、以下のプロパティが成立します:

#. :cpp:`M::value_type` は、 :cpp:`V::value_type`
#. :cpp:`M::index_type` は、 :cpp:`std::size_t`.
#. :cpp:`M::extents_type` は、 :cpp:`std::extents<M::index_type, Extents...>` 、ただし

   * :cpp:`sizeof(Extents...)` は、 :cpp:`V::rank()`
   * およびand each element at インデックス :cpp:`r` of :cpp:`Extents...` における各要素は、　:cpp:`V::static_extents(r) != 0`　であれば、:cpp:`V::static_extents(r)`　であり、そうでない場合には、 :cpp:`std::dynamic_extent`　です。

#. :cpp:`M::layout_type` は、

   * :cpp:`V::array_layout` が :cpp:`LayoutLeft`　であれば、:cpp:`std::layout_left_padded<std::dynamic_extent>`
   * :cpp:`V::array_layout` が :cpp:`LayoutRight`　であれば、:cpp:`std::layout_right_padded<std::dynamic_extent>`
   * :cpp:`V::array_layout` が :cpp:any:`LayoutStride`　であれば、:cpp:`std::layout_stride` 。

#. :cpp:`M::accessor_type` が :cpp:`std::default_accessor<V::value_type>`　です。

さらに、*natural mdspan* は、:cpp:`m.data() == v.data()` および、各範囲  :cpp:`r`　について、 :cpp:`m.extents().extent(r) == v.extent(r)`　となるように、構築されています。

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
