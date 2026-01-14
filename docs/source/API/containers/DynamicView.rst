
.. role:: cpp(code)
    :language: cpp

``DynamicView``
===============

ヘッダーファイル　: ``<Kokkos_DynamicView.hpp>``


ディスクリプション
-----------

.. cpp:class:: template<typename DataType , typename ... P> DynamicView : public Kokkos::ViewTraits<DataType , P ...>

    ホスト上で動的にサイズ変更可能であり、レイアウトを持たない、参照カウント可能なランク1配列。

    .. rubric:: パブリックメンバー変数

    .. cpp:member:: 静的定数コンストラクタ付きブール値　reference_type_is_lvalue_reference

        レファレンス型が、 C++ lvalueレファレンスかどうか。

    .. rubric:: パブリックネスティッド型定義

    .. cpp:type:: Kokkos::ViewTraits< DataType , P ... > traits

        ``Kokkos::ViewTraits`` 親クラス型

    .. cpp:type:: array_type

        ``traits::data_type`` および ``traits::device_type``　上にテンプレートされた ``DynamicView`` 型。

    .. cpp:type:: const_type

        ``traits::const_data_type`` および ``traits::device_type``　上にテンプレートされた ``DynamicView`` 型。

    .. cpp:type:: non_const_type

        ``traits::non_const_data_type`` および ``traits::device_type``　上にテンプレートされた ``DynamicView`` 型。

    .. cpp:type:: HostMirror

        ホストアクセス可能メモリ空間に格納された、同一の``DataType``　および　``LayoutType``　を持つ互換性のあるビュータイプ

    .. rubric:: パブリックデータハンドル型

    .. cpp:type:: reference_type

        ビューアクセス演算子の戻り値の型。

    .. cpp:type:: pointer_type

       スカラー型へのポインタ。

    .. rubric:: コンストラクタ

    .. cpp:function:: DynamicView()

        デフォルトコンストラクタ。割り当ては行われず、参照カウントも発生しません。 すべてのエクステントはゼロであり、そのデータポインタはNULLです。

    .. cpp:function:: DynamicView(const DynamicView<RT, RP...>& rhs)

        互換性のあるViewからのコピーコンストラクタ。View　代入ルールに従ってください。

    .. cpp:function:: DynamicView(DynamicView&& rhs)

        ムーブコンストラクタ。

    .. cpp:function:: DynamicView(const std::string & arg_label, \
			    const unsigned min_chunk_size,  \
			    const unsigned max_extent)

        標準の割り当てコンストラクタ。

        :param arg_label: ユーザーに提供されたレベルで、プロファイリングおよびデバッグ目的に使用されます。 名前が一意である必要はありません。
        :param min_chunk_size: メモリ割り当てに必要なユーザー指定の最小チャンクサイズで、 より効率的なメモリアクセスのため、2乗に最も近い数値に丸められます。
        :param max_extent: ユーザーに提供された最大サイズで、 チャンクポインタ配列を割り当てる必要があります。

        必要な量のメモリを確保するために、 ``max_extent``で制限し、``resize_serial``　メソッドを構築後に呼び出す必要があります。

    .. rubric:: パブリックデータアクセス関数

    .. cpp:function:: KOKKOS_INLINE_FUNCTION reference_type operator() (const I0 & i0 , const Args & ... args) const

        :return: それ自身が参照可能であるか否かに関わらず、`reference_type`型の値。 インデックス引数の数は 1 である必要があります (非推奨ではないコードについて)。

    .. rubric:: データリサイズ、ディメンション、ストライド


    .. cpp:function:: template< typename IntType > inline void resize_serial(IntType const & n)

       要求された要素数 `n` を格納するのに十分な `chunk_size` のメモリチャンクで、ダイナミックビューをリサイズします。
       この方法は並列領域の外側からのみ呼び出し可能です。
       ``n`` は、DynamicView コンストラクタに渡された ``max_extent`` 値よりも小さい値に制限されます。
       コンストラクタが　``chunk_size`` および ``max_extent``　について要求サイズを設定するので、このメソッドはDynamicViewの構築後に呼び出さなければなりませんが、 実際の使
用メモリ量についての入力は選択しません。

    .. cpp:function:: KOKKOS_INLINE_FUNCTION size_t allocation_extent() const noexcept;

        :return: チャンクの数とチャンクサイズを掛けた積の合計サイズ。 これは完全なメモリチャンクの総数に対する合計サイズを含むため、``size``　よりも大きくなる可能性があります。

    .. cpp:function:: KOKKOS_INLINE_FUNCTION size_t chunk_size() const noexcept;

        :return: メモリのチャンクが格納できるエントリ数は、常に2の累乗です。

    .. cpp:function:: KOKKOS_INLINE_FUNCTION size_t size() const noexcept;

        :return: ``resize_serial``　に渡された数値に基づく割当で利用可能なエントリの数。　この数値は``allocation_extent``により制限されます。

    .. cpp:function:: template< typename iType > KOKKOS_INLINE_FUNCTION size_t extent(const iType& dim) const;

        :return: 特定ディメンションの範囲。 ``iType`` は整数型でなければならず、　``dim`` は、 ``rank``　よりも小さくなければなりません。 rank > 1については、1を返してください。

    .. cpp:function:: template< typename iType > KOKKOS_INLINE_FUNCTION int extent_int(const iType& dim) const;

        :return:  ``int``　としての特定ディメンションの範囲。 ``iType`` は整数型でなければならず、　``dim`` は、 ``rank``　よりも小さくなければなりません。  ``extent``　と比較して、この関数は　``int``　演算が　``size_t``　よりも効率的なアーキテクチャにおいて有用です。また、そうでなければすべてのインデックス操作を　``int``　で行っているアプリケーションにおいて、型キャストの必要性を排除する可能性があります。rank > 1については、1　を返してください。

    .. cpp:function:: template< typename iType > KOKKOS_INLINE_FUNCTION void stride(const iType& dim) const;

        :return: 特定ディメンションのストライドは、常に ``DynamicView``　については、0 を返します。

    .. cpp:function:: KOKKOS_INLINE_FUNCTION constexpr size_t stride_0() const;

        :return: ディメンション 0　のストライドは、常に  ``DynamicView`` については、0　を返します。

    .. cpp:function:: KOKKOS_INLINE_FUNCTION constexpr size_t stride_1() const;

        :return: ディメンション 1　のストライドは、常に  ``DynamicView`` については、0　を返します。

    .. cpp:function:: KOKKOS_INLINE_FUNCTION constexpr size_t stride_2() const;

        :return: ディメンション 2　のストライドは、常に  ``DynamicView`` については、0　を返します。

    .. cpp:function:: KOKKOS_INLINE_FUNCTION constexpr size_t stride_3() const;

        :return: ディメンション 3　のストライドは、常に  ``DynamicView`` については、0　を返します。

    .. cpp:function:: KOKKOS_INLINE_FUNCTION constexpr size_t stride_4() const;

        :return: ディメンション 4　のストライドは、常に  ``DynamicView`` については、0　を返します。

    .. cpp:function:: KOKKOS_INLINE_FUNCTION constexpr size_t stride_5() const;

        :return: ディメンション 5　のストライドは、常に  ``DynamicView`` については、0　を返します。

    .. cpp:function:: KOKKOS_INLINE_FUNCTION constexpr size_t stride_6() const;

        :return: ディメンション 6　のストライドは、常に  ``DynamicView`` については、0　を返します。

    .. cpp:function:: KOKKOS_INLINE_FUNCTION constexpr size_t stride_7() const;

        :return: ディメンション 7　のストライドは、常に  ``DynamicView`` については、0　を返します。

    .. cpp:function:: KOKKOS_INLINE_FUNCTION constexpr size_t span() const;

        :return: ``DynamicView`` については、0　を返します。

    .. cpp:function:: KOKKOS_INLINE_FUNCTION constexpr pointer_type data() const;

        :return: 基盤となるデータ割当てへのポインタ。

    .. cpp:function:: KOKKOS_INLINE_FUNCTION constexpr bool span_is_contiguous() const;

        :return: スパンは連続しており、``DynamicView`` については、常に偽となります。

    .. rubric:: その他

    .. cpp:function:: KOKKOS_INLINE_FUNCTION int use_count() const;

        :return: 基盤となる割当ての現在の参照カウント。

    .. cpp:function:: inline const std::string label();

        :return: ``DynamicView``　のラベル。

    .. cpp:function:: bool is_allocated() const

        :return: ビューが割り当てられたメモリチャンクの有効なセットを指している場合に真です。 resize_serial が 0 より大きいサイズで呼び出されるまで、これが偽を返すことにご注意ください。


例
-------

.. code-block:: cpp

   const int chunk_size = 16*1024;
   Kokkos::Experimental::DynamicView<double*> view("v", chunk_size, 10*chunk_size);
   view.resize_serial(3*chunk_size);
   Kokkos::parallel_for("InitializeData", 3*chunk_size, KOKKOS_LAMBDA ( const int i) {
     view(i) = i;
   });
