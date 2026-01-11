
``DualView``
============

ヘッダーファイル: ``<Kokkos_DualView.hpp>``

|

デバイスメモリを参照する ``Kokkos::View`` とホストから
アクセス可能な ``Kokkos::View`` 間のミラーリングを管理するコンテナです。このクラスは、同時に2つの異なるメモリ空間に存在する
データを管理する機能を提供します。
両方の割り当てに対して変更フラグ同様に、同一レイアウトのViewを2つのメモリ空間上でサポートします。
ユーザーは、いずれかのメモリ空間でデータを変更した場合、``modify()`` 関数を呼び出すことで、変更フラグを手動で更新する責任を負いますが、それは変更されたデータを用いてデバイス上でテンプレート化されます。
ユーザーは ``sync()`` メソッドを呼び出すことでデータを同期することもできますが、それは同期を必要とするデバイス上でテンプレート化されます（すなわち、一方向コピー操作の対象）。

DualViewクラスは、基盤となる`Kokkos::View <../core/view/view.html>`_ objectsの適切なメソッドを呼び出す
realloc、resize、capacityなどの便利なメソッドも提供します。

4つのテンプレート引数は、``Kokkos::View``の引数と同じです。

* DataType、 コンテナに格納されるエントリの型。

* Layout、メモリ上の配列の配置。

* Device, Kokkos Device型。そのメモリ領域がホストからアクセス不可の場合、
デュアルビューは2つの独立したビューを含む：1つはデバイスメモリ内、
もう1つはホストメモリ内。それ以外の場合、DualViewは1つのビューのみを保存します。

* MemoryTraits (オプショナル) ユーザーの意図するメモリアクセス動作。 
例については、`Kokkos::View <../core/view/view.html>`_ のドキュメントを参照してください。ほとんどのユーザーにとって、デフォルト設定で十分です。

使用
-----

.. code-block:: cpp

    view_type = Kokkos::DualView<Scalar**　使用、
                                       Kokkos::LayoutLeft,
                                       Device>
    view_type a("A", n, m);

    Kokkos::deep_copy(a.view_device(), 1); １への// set device-side entries
    a.template modify<typename view_type::execution_space>(); // mark device-side as modified
    a.template sync<typename view_type::host_mirror_space>(); // sync modified data to host

    Kokkos::deep_copy(a.view_host(), 2); ２への// set host-side entries
    a.template modify<typename ViewType::host_mirror_space>(); // mark host-side as modified
    a.template sync<typename ViewType::execution_space>(); // sync modified data to device

ディスクリプション
-----------



.. cpp:class:: template <class DataType, class Arg1Type = void, class Arg2Type = void, class Arg3Type = void> DualView

    |

    .. rubric:: *Public* typedefs

    .. cpp:type:: ViewTraits<DataType, Arg1Type, Arg2Type, Arg3Type> traits

       デバイスタイプと様々な ``Kokkos::View`` の特殊化に対する型定義。

    .. cpp:type:: traits::host_mirror_space host_mirror_space

       Kokkosホストデバイス型

    .. cpp:type:: View<typename traits::data_type, Arg1Type, Arg2Type, Arg3Type> t_dev

       デバイス上の``Kokkos::View``の型。

    .. cpp:type:: typename t_dev::HostMirror t_host

       ``t_dev``の``Kokkos::View``ホストミラーの型。

    .. cpp:type:: View<typename traits::const_data_type, Arg1Type, Arg2Type, Arg3Type> t_dev_const

       デバイス上のconst Viewの型。

    .. cpp:type:: typename t_dev_const::HostMirror t_host_const

       ``t_dev_const``のconst Viewホストミラーの型。

    .. cpp:type:: View<typename traits::const_data_type, typename traits::array_layout, typename traits::device_type, Kokkos::MemoryTraits<Kokkos::RandomAccess> > t_dev_const_randomread

       デバイス上の const、 random-access Viewの型。

    .. cpp:type:: typename t_dev_const_randomread::HostMirror t_host_const_randomread

       ``t_dev_const_randomread``のconst, random-access Viewホストミラーの型。

    .. cpp:type:: View<typename traits::data_type, typename traits::array_layout, typename traits::device_type, MemoryUnmanaged> t_dev_um

       デバイス上の被管理Viewの型。

    .. cpp:type:: View<typename t_host::data_type, typename t_host::array_layout, typename t_host::device_type, MemoryUnmanaged> t_host_um

       \\c t_dev_umの非管理Viewホストミラーの型。

    .. cpp:type:: View<typename traits::const_data_type, typename traits::array_layout, typename traits::device_type, MemoryUnmanaged> t_dev_const_um

       デバイス上のconst非管理Viewの型。

    .. cpp:type:: View<typename t_host::const_data_type, typename t_host::array_layout, typename t_host::device_type, MemoryUnmanaged> t_host_const_um

       \\c t_dev_const_umのconst非管理ホストミラーの型。

　　.. cpp:type:: View<typename t_host::const_data_type, typename t_host::array_layout, typename t_host::device_type, Kokkos::MemoryTraits<Kokkos::Unmanaged | Kokkos::RandomAccess> > t_dev_const_randomread_um

       デバイス上のconst, random-access Viewの型。

    .. cpp:type:: typename t_dev_const_randomread::HostMirror t_host_const_randomread_um

       ``t_dev_const_randomread``のconst, random-access Viewミラーの型。

    .. cpp:type:: View<unsigned int[2], LayoutLeft, typename t_host::execution_space> t_modified_flags;

    .. cpp:type:: View<unsigned int, LayoutLeft, typename t_host::execution_space> t_modified_flag;

    .. rubric:: Data Members

    .. cpp:member:: t_dev d_view

       *device*上のビューインスタンスは、 Kokkos 4.6 以降、public アクセスは非推奨となります。


    .. cpp:member:: t_host h_view

       *host*上のビューインスタンスは、 Kokkos 4.6 以降、public アクセスは非推奨となります。

    .. cpp:member:: t_modified_flags modified_flags

    .. cpp:member:: t_modified_flag modified_host;

    .. cpp:member:: t_modified_flag modified_device;

    |

    .. rubric:: *Public* constructors

    .. cpp:function:: DualView();

       空のコンストラクタ。デバイスとホストの両方のビューオブジェクトは、それぞれのデフォルトコンストラクタを使用して構築されます。
       "修正済み"フラグは両方とも"未修正"として初期化される。

    .. cpp:function:: DualView(const std::string& label, const size_t n0 = KOKKOS_IMPL_CTOR_DEFAULT_ARG, const size_t n1 = KOKKOS_IMPL_CTOR_DEFAULT_ARG, const size_t n2 = KOKKOS_IMPL_CTOR_DEFAULT_ARG, const size_t n3 = KOKKOS_IMPL_CTOR_DEFAULT_ARG, const size_t n4 = KOKKOS_IMPL_CTOR_DEFAULT_ARG, const size_t n5 = KOKKOS_IMPL_CTOR_DEFAULT_ARG, const size_t n6 = KOKKOS_IMPL_CTOR_DEFAULT_ARG, const size_t n7 = KOKKOS_IMPL_CTOR_DEFAULT_ARG);

       ホストとデバイスの両方でビューオブジェクトを割り当てるコンストラクタ。
       最初の引数は文字列ラベルであり、これは完全にあなたの便宜のために用意されています。 (異なるDualViewオブジェクトは、必要に応じて同じラベルを持つことができます。)
       以下に示す引数は、Viewオブジェクトの次元です。例えば、Viewが3次元であれば、
       最初の3つの整数引数はゼロ以外になり、また、続く整数引数は省略できます。

    .. cpp:function:: DualView(ALLOC_PROP const& arg_prop, const size_t n0 = KOKKOS_IMPL_CTOR_DEFAULT_ARG, const size_t n1 = KOKKOS_IMPL_CTOR_DEFAULT_ARG, const size_t n2 = KOKKOS_IMPL_CTOR_DEFAULT_ARG, const size_t n3 = KOKKOS_IMPL_CTOR_DEFAULT_ARG, const size_t n4 = KOKKOS_IMPL_CTOR_DEFAULT_ARG, const size_t n5 = KOKKOS_IMPL_CTOR_DEFAULT_ARG, const size_t n6 = KOKKOS_IMPL_CTOR_DEFAULT_ARG, const size_t n7 = KOKKOS_IMPL_CTOR_DEFAULT_ARG);

       ホストとデバイスの両方でViewオブジェクトを割り当てるコンストラクタ。最初の引数として``Kokkos::view_alloc``で作成されたオブジェクトを渡すことを可能にします。
       例えば、ラベルを提供し、初期化を回避し、または実行空間インスタンスを指定します。
       以下の引数は、Viewオブジェクトの次元です。
       例えば、Viewが3次元であれば、
       最初の3つの整数引数はゼロ以外になり、
　　　　また、続く整数引数は省略できます。

    .. cpp:function:: DualView(const DualView<SS, LS, DS, MS>& src);

       コピーコンストラクタ (シャローコピー)

    .. cpp:function:: DualView(const DualView<SD, S1, S2, S3>& src, const Arg0& arg0, Args... args);

       サブビューコンストラクタ

    .. cpp:function:: DualView(const t_dev& d_view_, const t_host& h_view_);

       既存のデバイスおよびホストビューオブジェクトからデュアルビューを作成します。
       このコンストラクタは、デバイスとホストのViewオブジェクトが同期されていることを前提としています。 発信者である貴方は、このコンストラクタを呼び出す前にこれが事実であることを
       確認する責務を負います。このコンストラクタが戻った後、DualViewの``sync()``および``modify()``メソッドを使用して、
       Viewオブジェクトの同期を確保できます。デュアルビューが1つのビューのみを格納する場合、すなわちデュアルビューのメモリ領域がホストからアクセス可能な場合、
       両引数は同一の割り当てを参照しなければなりません。

       - ``d_view_`` デバイスビュー

       - ``h_view_`` ホストビュー (``t_host = t_dev::HostMirror``型を持つ必要があります)

    |

    .. rubric:: 同期化、変更済みとしてマーク、およびビューの取得のための*Public*メソッド。

    .. cpp:function:: テンプレート <class Device> KOKKOS_INLINE_FUNCTION const auto& view();

    .. cpp:function:: テンプレート <class Device> static int get_device_side();

       * 特定のデバイス ``Device`` 上のビューを返します。 ``Device`` は、``Kokkos::Device`` 型、メモリ空間、またはデバイスビューもしくはホストアクセス可能ビューに対応する実行空間のいずれかである。
       * 例えば、Cuda上で次のようにDualViewを作成するとします：

         .. code-block:: cpp

           using dual_view_type = Kokkos::DualView<float, Kokkos::Cuda>;
           dual_view_type DV ("my dual view", 100);

         CUDAデバイスのビューを取得したい場合は、次の操作を行ってください：

         .. code-block:: cpp

           dual_view_type::t_dev cudaView = DV.view<dual_view_type::t_dev::memory_space>();

         そのビューのホストミラーを取得したい場合は、次のようにします：

         .. code-block:: cpp

           dual_view_type::t_host hostView = DV.view<dual_view_type::t_host::memory_space>();

    .. cpp:function:: const t_host& view_host() const;

       *  ホストがアクセス可能なビューを返します。 `Kokkos_ENABLE_DEPRECATED_CODE_4=ON`を持つ値によりViewを返します。

    .. cpp:function:: const t_dev& view_device() const;

       * Return the View on the device.デバイス上のViewを返します。 `Kokkos_ENABLE_DEPRECATED_CODE_4=ON`を持つ値によりViewを返します。

    .. cpp:function:: template <class Device> void sync(const typename Impl::enable_if<(std::is_same<typename traits::data_type, typename traits::non_const_data_type>::value) || (std::is_same<Device, int>::value), int>::type& = 0);

    .. cpp:function:: template <class Device> void sync(const typename Impl::enable_if<(!std::is_same<typename traits::data_type, typename traits::non_const_data_type>::value) || (std::is_same<Device, int>::value), int>::type& = 0);

       * デバイスまたはホスト上のデータは、他方の領域のデータが変更済みとしてマークされた場合にのみ更新します。
       * ``デバイス``がこのDualViewのデバイスタイプと同じ場合、ホストからデバイスへデータを複製します。それ以外の場合には、デバイスからホストへデータをコピーします。いずれの場合も、コピー元のソースが変更された場合にのみコピーしてください。
       * これは一方向の同期のみです。コピー先の対象が変更されている場合、この操作はその変更を破棄します。また、デバイスとホストの変更フラグの両方をリセットします。
       * このメソッドでは、どちらのビューでデータを変更したかを独自に判断できません。貴方は、変更されたデータを、適切なテンプレートパラメータを指定して``modify()``メソッドを呼び出すことで、手動で変更済みとしてマークする必要があります。

    .. cpp:function:: template <class Device> bool need_sync() const;

    .. cpp:function:: template <class Device> void modify();

    .. cpp:function:: inline void clear_sync_state();

       特定のdevice \\c Device上でデータを変更済みとしてマークします。 ``Device``が　本DualViewのデバイスタイプと同一の場合、
そのデバイスのデータを変更済みとしてマークしてください。
        そうでない場合は、ホストのデータを変更済みとしてマークしてください。

    |

    .. rubric:: *Public*メソッド：View オブジェクトの再割り当てまたはサイズ変更。

    .. cpp:function:: constexpr boolは、_allocated() constです。;

       基底ビューの割り当て状態を返します。 ホストビューとデバイスビューの両方が、
       有効なメモリ位置を指している場合に真を返します。この関数は、管理ビューと非管理ビューの両方で動作します。
       非管理ビューでは、参照されるアドレスが有効である保証はなく、単にヌルポインタでないことのみが保証されます。

    .. cpp:function:: void realloc(const size_t n0 = KOKKOS_IMPL_CTOR_DEFAULT_ARG, const size_t n1 = KOKKOS_IMPL_CTOR_DEFAULT_ARG, const size_t n2 = KOKKOS_IMPL_CTOR_DEFAULT_ARG, const size_t n3 = KOKKOS_IMPL_CTOR_DEFAULT_ARG, const size_t n4 = KOKKOS_IMPL_CTOR_DEFAULT_ARG, const size_t n5 = KOKKOS_IMPL_CTOR_DEFAULT_ARG, const size_t n6 = KOKKOS_IMPL_CTOR_DEFAULT_ARG, const size_t n7 = KOKKOS_IMPL_CTOR_DEFAULT_ARG);

       両方のビューオブジェクトを再割り当てします。これにより、オブジェクトの既存の内容はすべて破棄され、
       その変更されたフラグはリセットされます。古いViewのどちらの内容も、新しいViewオブジェクトにはコピー*されません*。

    .. cpp:function:: void resize(const size_t n0 = KOKKOS_IMPL_CTOR_DEFAULT_ARG, const size_t n1 = KOKKOS_IMPL_CTOR_DEFAULT_ARG, const size_t n2 = KOKKOS_IMPL_CTOR_DEFAULT_ARG, const size_t n3 = KOKKOS_IMPL_CTOR_DEFAULT_ARG, const size_t n4 = KOKKOS_IMPL_CTOR_DEFAULT_ARG, const size_t n5 = KOKKOS_IMPL_CTOR_DEFAULT_ARG, const size_t n6 = KOKKOS_IMPL_CTOR_DEFAULT_ARG, const size_t n7 = KOKKOS_IMPL_CTOR_DEFAULT_ARG);

       両方のビューのサイズを変更し、必要に応じて古い内容を新しいビューにコピーします。このメソッドでは、
       最後に変更済みとマークされたデバイスの古い内容を新しいViewオブジェクトにのみコピーします。
　　　　従って、ユーザーはサイズ変更されたオブジェクトを使用する前に``sync()``を呼び出す必要があります。

    |

    .. rubric:: *Public* Methods for querying capacity, stride, or dimension(s).キャパシティ、ストライド、または次元を問い合わせのための*Pubilic* メソッド。

    .. cpp:function:: KOKKOS_INLINE_FUNCTION constexpr size_t span() const;

       


(same as ``Kokkos::View::span``と同様)。

    .. cpp:function:: KOKKOS_INLINE_FUNCTION bool span_is_contiguous();

       スパンが連続している場合に true を返します。

    .. cpp:function:: template <typename iType> void stride(iType* stride_) const;

       各次元ごとにストライドを取得します。 ``stride_`` [rank] をspan()に設定します。

    .. cpp:function:: template <typename iType> KOKKOS_INLINE_FUNCTION constexpr typename std::enable_if<std::is_integral<iType>::value, size_t>::type extent(const iType& r) const;

       要求されたランクの範囲を返します。

    .. cpp:function:: template <typename iType> KOKKOS_INLINE_FUNCTION constexpr typename std::enable_if<std::is_integral<iType>::value, int>::type extent_int(const iType& r) const;

       要求されたランクに対する整数の範囲を返します。
