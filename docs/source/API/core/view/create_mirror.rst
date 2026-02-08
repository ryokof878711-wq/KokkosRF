``create_mirror[_view]``
========================

.. role:: cpp(code)
    :language: cpp

ヘッダーファイル: ``<Kokkos_Core.hpp>``

.. _deepCopy: deep_copy.html

.. |deepCopy| 置換:: :cpp:func:`deep_copy`

一般的な望ましいユースケースは、GPUメモリにメモリ割り当てを行い、CPUメモリにも同一のメモリ割り当てを行うことであり、一方から他方へのコピーが容易になるようにすることです。このユースケースやその他のユースケースを満たすため、Kokkosは、View　の　"mirrors" を扱うための機能を備えています。 ビュー型 ``A`` の　"mirror"　とは、おおまかに定義すると、ビュー型 ``B`` であり、型 ``B`` の　View が CPU からアクセス可能であり、型 ``A`` と ``B`` のビュー間の |deepCopy|_ が直接的であるようにすることである。 mirror　を処理する最も一般的な関数は、 ``create_mirror``、 ``create_mirror_view`` および ``create_mirror_view_and_copy``　です。

使用例
-----

``create_mirror`` および ``create_mirror_view`` の主な違いは、以下の通りです: ``create_mirror`` `always` は、特定スペース　(ホストスペースの下にあります)　にある新たなメモリを割り当てますが、一方では、ミラーリングすべき　View (``a_viewed``)　が特定空間からまだアクセス可能でない場合に、メモリの割り当てのみを行い、そうでない場合には、単に　``a_view``　を返します。
ミラーが異なる実行空間でのアクセス提供のみに使用される場合、``create_mirror_view``　を使用し、データが独立している必要がある場合（例：データの以前のバージョンと更新後のバージョンを保持する場合）、``create_mirror``　を使用してください。

.. code-block:: cpp

    // host_mirror と host_mirror_view の両方が、a_view からの/a_view への deep_copy に対して正しいプロパティを持っています。
    // host_mirror は、 a_view　から独立して割り当てられたメモリを持つことを保証されます。
　　auto host_mirror = create_mirror(a_view);
    // a_view がホストからアクセス可能な場合、host_mirror_view は a_view と同じメモリを指す可能性があります。
    auto host_mirror_view = create_mirror_view(a_view);

    // ミラービューが、アクセス可能でなければならないスペースを指定できます。
    auto mirror = create_mirror(memory_space_instance, a_view);
    auto mirror_view = create_mirror_view(memory_space_instance, a_view);


ディスクリプション
-----------

.. _View: view.html

.. |View| 置換:: :cpp:class:`View`

.. _ExecutionSpaceConcept: ../execution_spaces.html#executionspaceconcept

.. |ExecutionSpaceConcept| replace:: :cpp:func:`ExecutionSpaceConcept`

.. _MemorySpaceConcept: ../memory_spaces.html#memoryspaceconcept

.. |MemorySpaceConcept| 置換:: :cpp:func:`MemorySpaceConcept`


.. cpp:function:: テンプレート <class ViewType> typename ViewType::HostMirror create_mirror(ViewType const& src);

   レイアウトとパディングは ``src`` と同じで、新しいホストを作成し、アクセス可能な |View|_ を生成します。

   - ``src``: a ``Kokkos::View``.

.. cpp:function:: テンプレート <class ViewType> typename ViewType::HostMirror create_mirror(decltype(Kokkos::WithoutInitializing), ViewType const& src);

   レイアウトとパディングは``src``と同じものを使って、新しいホストを作成し、|View|_ でアクセス可能にします。　新たなビューは、 初期化されていないデータを持ちます。

   - ``src``: a ``Kokkos::View``.

.. cpp:function:: テンプレート <class Space, class ViewType> ImplMirrorType create_mirror(Space const& space, ViewType const& src);

    レイアウトとパディングは``src``と同じものを使って、新たな |View|_ を作成しますが、 ``Space::device_type``　のデバイス型を使います。

   - ``src``: a ``Kokkos::View``.

   - ``Space``:  |ExecutionSpaceConcept|_ または |MemorySpaceConcept|_　の必要要件を満たすクラス。

   - ``ImplMirrorType``: ``Kokkos::View``　の実装定義の仕様。

.. cpp:function:: テンプレート <class Space, class ViewType> ImplMirrorType create_mirror(decltype(Kokkos::WithoutInitializing), Space const& space, ViewType const& src);

   レイアウトとパディングは``src``と同じものを使って、新たな |View|_ を作成しますが、 ``Space::device_type``　のデバイス型を使います。 新たなビューは、 初期化されていないデータを持ちます。

   - ``src``: a ``Kokkos::View``.

   - ``Space``: a class meeting the requirements of |ExecutionSpaceConcept|_ または |MemorySpaceConcept|_　の必要要件を満たすクラス。

   - ``ImplMirrorType``: ``Kokkos::View``　の実装定義の仕様。

.. cpp:function:: テンプレート <class ViewType, class ALLOC_PROP> auto create_mirror(ALLOC_PROP const& arg_prop, ViewType const& src);

   Creates a new |View|_ with the same layout and padding as ``src``
   using the |View|_ constructor properties ``arg_prop``, e.g., ``Kokkos::view_alloc(Kokkos::WithoutInitializing)``.
   If ``arg_prop`` contains a memory space, a |View|_ in that space is created. Otherwise, a |View|_ in host-accessible memory is returned.

   - ``src``: a ``Kokkos::View``.

   - ``arg_prop``: |View|_ constructor properties, e.g., ``Kokkos::view_alloc(Kokkos::WithoutInitializing)``.

     .. important::

	``arg_prop`` must not include a pointer to memory, or a label, or allow padding.


.. cpp:function:: template <class ViewType> typename ViewType::HostMirror create_mirror_view(ViewType const& src);

   If ``src`` is not host accessible (i.e. if ``SpaceAccessibility<HostSpace,ViewType::memory_space>::accessible`` is ``false``)
   it creates a new host accessible |View|_ with the same layout and padding as ``src``. Otherwise returns ``src``.

   - ``src``: a ``Kokkos::View``.

.. cpp:function:: template <class ViewType> typename ViewType::HostMirror create_mirror_view(decltype(Kokkos::WithoutInitializing), ViewType const& src);

   If ``src`` is not host accessible (i.e. if ``SpaceAccessibility<HostSpace,ViewType::memory_space>::accessible`` is ``false``)
   it creates a new host accessible |View|_ with the same layout and padding as ``src``. The new view will have uninitialized data. Otherwise returns ``src``.

   - ``src``: a ``Kokkos::View``.

.. cpp:function:: template <class Space, class ViewType> ImplMirrorType create_mirror_view(Space const& space, ViewType const& src);

   If ``std::is_same<typename Space::memory_space, typename ViewType::memory_space>::value`` is ``false``, creates a new |View|_ with
   the same layout and padding as ``src`` but with a device type of ``Space::device_type``. Otherwise returns ``src``.

   - ``src``: a ``Kokkos::View``.

   - ``Space`` : a class meeting the requirements of |ExecutionSpaceConcept|_ or |MemorySpaceConcept|_

   - ``ImplMirrorType``: an implementation defined specialization of ``Kokkos::View``.

.. cpp:function:: template <class Space, class ViewType> ImplMirrorType create_mirror_view(decltype(Kokkos::WithoutInitializing), Space const& space, ViewType const& src);

   If ``std::is_same<typename Space::memory_space, typename ViewType::memory_space>::value`` is ``false``,
   creates a new |View|_ with the same layout and padding as ``src`` but with a device type of ``Space::device_type``.
   The new view will have uninitialized data. Otherwise returns ``src``.

   - ``src``: a ``Kokkos::View``.

   - ``Space``: a class meeting the requirements of |ExecutionSpaceConcept|_ or |MemorySpaceConcept|_

   - ``ImplMirrorType``: an implementation defined specialization of ``Kokkos::View``.

.. cpp:function:: template <class ViewType, class ALLOC_PROP> auto create_mirror_view(ALLOC_PROP const& arg_prop, ViewType const& src);

   If the |View|_ constructor arguments ``arg_prop`` (created by a call to `Kokkos::view_alloc`) include a memory space and the memory space
   doesn't match the memory space of ``src``, creates a new |View|_ in the specified memory_space. If the ``arg_prop`` don't include a memory
   space and the memory space of ``src`` is not host-accessible, creates a new host-accessible |View|_.
   Otherwise, ``src`` is returned. If a new |View|_ is created, the implicitly called constructor respects ``arg_prop``
   and uses the same layout and padding as ``src``.

   - ``src``: a ``Kokkos::View``.

   - ``arg_prop``: |View|_ constructor properties, e.g., ``Kokkos::view_alloc(Kokkos::WithoutInitializing)``.

     .. important::

	``arg_prop`` must not include a pointer to memory, or a label, or allow padding.

.. cpp:function:: template <class Space, class ViewType> ImplMirrorType create_mirror_view_and_copy(Space const& space, ViewType const& src);

   If ``std::is_same<typename Space::memory_space, typename ViewType::memory_space>::value`` is ``false``,
   creates a new ``Kokkos::View`` with the same layout and padding as ``src`` but with a device type of ``Space::device_type``
   and conducts a ``deep_copy`` from ``src`` to the new view if one was created. Otherwise returns ``src``.

   - ``src``: a ``Kokkos::View``.

   - ``Space``: a class meeting the requirements of |ExecutionSpaceConcept|_ or |MemorySpaceConcept|_

   - ``ImplMirrorType``: an implementation defined specialization of ``Kokkos::View``.

.. cpp:function:: template <class ViewType, class ALLOC_PROP> ImplMirrorType create_mirror_view_and_copy(ALLOC_PROP const& arg_prop, ViewType const& src);

   If the  memory space included in the |View|_ constructor arguments ``arg_prop`` (created by a call to `Kokkos::view_alloc`) does not match the memory
   space of ``src``, creates a new |View|_ in the specified memory space using ``arg_prop`` and the same layout
   and padding as ``src``. Additionally, a ``deep_copy`` from ``src`` to the new view is executed
   (using the execution space contained in ``arg_prop`` if provided). Otherwise returns ``src``.

   - ``src``: a ``Kokkos::View``.

   - ``arg_prop``: |View|_ constructor properties, e.g., ``Kokkos::view_alloc(Kokkos::HostSpace{}, Kokkos::WithoutInitializing)``.

     .. important::

	``arg_prop`` must not include a pointer to memory, or a label, or allow padding and ``arg_prop`` must include a memory space.
