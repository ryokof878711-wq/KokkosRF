
``DualView``
============

ヘッダーファイル: ``<Kokkos_DualView.hpp>``

|

デバイスメモリを参照するKokkos::Viewとホストから
アクセス可能な``Kokkos::View``間のミラーリングを管理するコンテナです。このクラスは、同時に2つの異なるメモリ空間に存在する
データを管理する機能を提供します。
両方の割り当てに対して変更フラグ同様に、同一レイアウトのViewを2つのメモリ空間上でサポートします。
ユーザーは、いずれかのメモリ空間でデータを変更した場合、modify()関数を呼び出すことで、変更フラグを手動で更新する責任を負いますが、それは変更されたデータを用いてデバイス上でテンプレート化されます。
ユーザーは``sync()``メソッドを呼び出すことでデータを同期することもできますが、それは同期を必要とするデバイス上でテンプレート化されます（すなわち、一方向コピー操作の対象）。

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

       The view instance on the *device*状の public access deprecated from Kokkos 4.6 on

    .. cpp:member:: t_host h_view

       The view instance on the *host*, public access deprecated from Kokkos 4.6 on.

    .. cpp:member:: t_modified_flags modified_flags

    .. cpp:member:: t_modified_flag modified_host;

    .. cpp:member:: t_modified_flag modified_device;

    |

    .. rubric:: *Public* constructors

    .. cpp:function:: DualView();

       Empty constructor. Both device and host View objects are constructed using their default constructors.
       The "modified" flags are both initialized to "unmodified."

    .. cpp:function:: DualView(const std::string& label, const size_t n0 = KOKKOS_IMPL_CTOR_DEFAULT_ARG, const size_t n1 = KOKKOS_IMPL_CTOR_DEFAULT_ARG, const size_t n2 = KOKKOS_IMPL_CTOR_DEFAULT_ARG, const size_t n3 = KOKKOS_IMPL_CTOR_DEFAULT_ARG, const size_t n4 = KOKKOS_IMPL_CTOR_DEFAULT_ARG, const size_t n5 = KOKKOS_IMPL_CTOR_DEFAULT_ARG, const size_t n6 = KOKKOS_IMPL_CTOR_DEFAULT_ARG, const size_t n7 = KOKKOS_IMPL_CTOR_DEFAULT_ARG);

       Constructor that allocates View objects on both host and device.
       The first argument is a string label, which is entirely for your benefit. (Different DualView objects may have the same label if you like.)
       The arguments that follow are the dimensions of the View objects. For example, if the View has three dimensions,
       the first three integer arguments will be nonzero, and you may omit the integer arguments that follow.

    .. cpp:function:: DualView(ALLOC_PROP const& arg_prop, const size_t n0 = KOKKOS_IMPL_CTOR_DEFAULT_ARG, const size_t n1 = KOKKOS_IMPL_CTOR_DEFAULT_ARG, const size_t n2 = KOKKOS_IMPL_CTOR_DEFAULT_ARG, const size_t n3 = KOKKOS_IMPL_CTOR_DEFAULT_ARG, const size_t n4 = KOKKOS_IMPL_CTOR_DEFAULT_ARG, const size_t n5 = KOKKOS_IMPL_CTOR_DEFAULT_ARG, const size_t n6 = KOKKOS_IMPL_CTOR_DEFAULT_ARG, const size_t n7 = KOKKOS_IMPL_CTOR_DEFAULT_ARG);

       Constructor that allocates View objects on both host and device allowing to pass an object created by ``Kokkos::view_alloc`` as first argument,
       e.g., to provide a label, avoid initialization, or specifying an execution space instance.
       The arguments that follow are the dimensions of the View objects.
       For example, if the View has three dimensions, the first three integer arguments will be nonzero, and you may omit the integer arguments that follow.

    .. cpp:function:: DualView(const DualView<SS, LS, DS, MS>& src);

       Copy constructor (shallow copy)

    .. cpp:function:: DualView(const DualView<SD, S1, S2, S3>& src, const Arg0& arg0, Args... args);

       Subview constructor

    .. cpp:function:: DualView(const t_dev& d_view_, const t_host& h_view_);

       Create DualView from existing device and host View objects.
       This constructor assumes that the device and host View objects are synchronized. You, the caller, are responsible for making sure this
       is the case before calling this constructor. After this constructor returns, you may use DualView's ``sync()`` and ``modify()``
       methods to ensure synchronization of the View objects. In case the DualView only stores one View, i.e., DualView's memory space is host-accessible,
       both arguments must reference the same allocation.

       - ``d_view_`` Device View

       - ``h_view_`` Host View (must have type ``t_host = t_dev::HostMirror``)

    |

    .. rubric:: *Public* Methods for synchronizing, marking as modified, and getting Views.

    .. cpp:function:: template <class Device> KOKKOS_INLINE_FUNCTION const auto& view();

    .. cpp:function:: template <class Device> static int get_device_side();

       * Return a View on a specific device ``Device``. ``Device`` can be a ``Kokkos::Device`` type, a memory space or a execution space corresponding to either the device View or the host-accessible View.
       * For example, suppose you create a DualView on Cuda, like this:

         .. code-block:: cpp

           using dual_view_type = Kokkos::DualView<float, Kokkos::Cuda>;
           dual_view_type DV ("my dual view", 100);

         If you want to get the CUDA device View, do this:

         .. code-block:: cpp

           dual_view_type::t_dev cudaView = DV.view<dual_view_type::t_dev::memory_space>();

         and if you want to get the host mirror of that View, do this:

         .. code-block:: cpp

           dual_view_type::t_host hostView = DV.view<dual_view_type::t_host::memory_space>();

    .. cpp:function:: const t_host& view_host() const;

       *  Return the host-accessible View. Returns the View by value with `Kokkos_ENABLE_DEPRECATED_CODE_4=ON`

    .. cpp:function:: const t_dev& view_device() const;

       * Return the View on the device. Returns the View by value with `Kokkos_ENABLE_DEPRECATED_CODE_4=ON`.

    .. cpp:function:: template <class Device> void sync(const typename Impl::enable_if<(std::is_same<typename traits::data_type, typename traits::non_const_data_type>::value) || (std::is_same<Device, int>::value), int>::type& = 0);

    .. cpp:function:: template <class Device> void sync(const typename Impl::enable_if<(!std::is_same<typename traits::data_type, typename traits::non_const_data_type>::value) || (std::is_same<Device, int>::value), int>::type& = 0);

       * Update data on device or host only if data in the other space has been marked as modified.
       * If ``Device`` is the same as this DualView's device type, then copy data from host to device. Otherwise, copy data from device to host. In either case, only copy if the source of the copy has been modified.
       * This is a one-way synchronization only. If the target of the copy has been modified, this operation will discard those modifications. It will also reset both device and host modified flags.
       * This method doesn't know on its own whether you modified the data in either View. You must manually mark modified data as modified, by calling the ``modify()`` method with the appropriate template parameter.

    .. cpp:function:: template <class Device> bool need_sync() const;

    .. cpp:function:: template <class Device> void modify();

    .. cpp:function:: inline void clear_sync_state();

       Mark data as modified on the given device \\c Device. If ``Device`` is the same as this
       DualView's device type, then mark the device's data as modified. Otherwise, mark the host's data as modified.

    |

    .. rubric:: *Public* Methods for reallocating or resizing the View objects

    .. cpp:function:: constexpr bool is_allocated() const;

       Return allocation state of underlying views. Returns true if both the host and device
       views points to a valid memory location. This function works for both managed and unmanaged views.
       With the unmanaged view, there is no guarantee that referenced address is valid, only that it is a non-null pointer.

    .. cpp:function:: void realloc(const size_t n0 = KOKKOS_IMPL_CTOR_DEFAULT_ARG, const size_t n1 = KOKKOS_IMPL_CTOR_DEFAULT_ARG, const size_t n2 = KOKKOS_IMPL_CTOR_DEFAULT_ARG, const size_t n3 = KOKKOS_IMPL_CTOR_DEFAULT_ARG, const size_t n4 = KOKKOS_IMPL_CTOR_DEFAULT_ARG, const size_t n5 = KOKKOS_IMPL_CTOR_DEFAULT_ARG, const size_t n6 = KOKKOS_IMPL_CTOR_DEFAULT_ARG, const size_t n7 = KOKKOS_IMPL_CTOR_DEFAULT_ARG);

       Reallocate both View objects. This discards any existing contents of the objects,
       and resets their modified flags. It does *not* copy the old contents of either View into the new View objects.

    .. cpp:function:: void resize(const size_t n0 = KOKKOS_IMPL_CTOR_DEFAULT_ARG, const size_t n1 = KOKKOS_IMPL_CTOR_DEFAULT_ARG, const size_t n2 = KOKKOS_IMPL_CTOR_DEFAULT_ARG, const size_t n3 = KOKKOS_IMPL_CTOR_DEFAULT_ARG, const size_t n4 = KOKKOS_IMPL_CTOR_DEFAULT_ARG, const size_t n5 = KOKKOS_IMPL_CTOR_DEFAULT_ARG, const size_t n6 = KOKKOS_IMPL_CTOR_DEFAULT_ARG, const size_t n7 = KOKKOS_IMPL_CTOR_DEFAULT_ARG);

       Resize both views, copying old contents into new if necessary. This method only copies the old
       contents into the new View objects for the device which was last marked as modified. Thus, users are required to call ``sync()`` before using the resized object.

    |

    .. rubric:: *Public* Methods for querying capacity, stride, or dimension(s).

    .. cpp:function:: KOKKOS_INLINE_FUNCTION constexpr size_t span() const;

       Return the allocation size (same as ``Kokkos::View::span``).

    .. cpp:function:: KOKKOS_INLINE_FUNCTION bool span_is_contiguous();

       Return true if the span is contiguous

    .. cpp:function:: template <typename iType> void stride(iType* stride_) const;

       Get stride(s) for each dimension. Sets ``stride_`` [rank] to span().

    .. cpp:function:: template <typename iType> KOKKOS_INLINE_FUNCTION constexpr typename std::enable_if<std::is_integral<iType>::value, size_t>::type extent(const iType& r) const;

       Return the extent for the requested rank

    .. cpp:function:: template <typename iType> KOKKOS_INLINE_FUNCTION constexpr typename std::enable_if<std::is_integral<iType>::value, int>::type extent_int(const iType& r) const;

       Return integral extent for the requested rank
