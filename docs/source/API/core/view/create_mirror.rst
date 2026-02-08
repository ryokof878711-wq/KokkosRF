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

    |View|_ constructor properties ``arg_prop``, 例えば、 ``Kokkos::view_alloc(Kokkos::WithoutInitializing)``　を使って、
   レイアウトとパディングは``src``と同じものを使って、新たな |View|_ を作成します。
   　``arg_prop`` がメモリスペースを含んでいれば、 そのスペース内の |View|_ が作成されます。そうでない場合、 ホストアクセス可能なメモリ内の |View|_  を返します。

   - ``src``: a ``Kokkos::View``.

   - ``arg_prop``: |View|_ コンストラクタ特性、例えば、  ``Kokkos::view_alloc(Kokkos::WithoutInitializing)``.

     .. 重要事項::

	``arg_prop`` は、メモリまたはラベルへのポインタを含んではならず、 またはパディングを認めてはなりません。


.. cpp:function:: テンプレート <class ViewType> typename ViewType::HostMirror create_mirror_view(ViewType const& src);

  ``src`` が、ホストアクセス可能でない場合には (つまり　``SpaceAccessibility<HostSpace,ViewType::memory_space>::accessible`` が ``false``　である場合)、
　``src``　と同じレイアウトおよびパディングを使って、 |View|_　とアクセス可能な新たなホストを作成します。そうでない場合には、 ``src``　を返します。

   - ``src``: a ``Kokkos::View``.

.. cpp:function:: テンプレート <class ViewType> typename ViewType::HostMirror create_mirror_view(decltype(Kokkos::WithoutInitializing), ViewType const& src);

   ``src`` が、ホストアクセス可能でない場合には (つまり　``SpaceAccessibility<HostSpace,ViewType::memory_space>::accessible`` が ``false``　である場合)、
   ``src``　と同じレイアウトおよびパディングを使って、 |View|_　とアクセス可能な新たなホストを作成します。 新たなビューは、 初期化されていないデータを持ちます。 そうでない場合には、 ``src``　を返します。

   - ``src``: a ``Kokkos::View``.

.. cpp:function:: テンプレート <class Space, class ViewType> ImplMirrorType create_mirror_view(Space const& space, ViewType const& src);

   ``std::is_same<typename Space::memory_space　である場合には、 typename ViewType::memory_space>::value`` は、 ``false``　です。  ``src`` と同じレイアウトおよびパディングを使いますが、　``Space::device_type``　のデバイスタイプを使って新たな |View|_　を作成します。　そうでない場合には、 ``src``　を返します。

   - ``src``: a ``Kokkos::View``.

   - ``Space`` :  |ExecutionSpaceConcept|_ または |MemorySpaceConcept|_　の必要要件を満たすクラス。

   - ``ImplMirrorType``: ``Kokkos::View``　の実装定義の仕様。

.. cpp:function:: テンプレート <class Space, class ViewType> ImplMirrorType create_mirror_view(decltype(Kokkos::WithoutInitializing), Space const& space, ViewType const& src);

   ``std::is_same<typename Space::memory_space, typename ViewType::memory_space>::value``  が ``false``　であれば、
    ``src`` と同じレイアウトおよびパディングを使いますが、``Space::device_type``　のデバイスタイプを使って新たな |View|_　を作成します。
   新たなビューは、 初期化されていないデータを持ちます。 そうでない場合には、 ``src``　を返します。

   - ``src``: a ``Kokkos::View``.

   - ``Space``: a class meeting the requirements of |ExecutionSpaceConcept|_ or |MemorySpaceConcept|_

   - ``ImplMirrorType``:　 |ExecutionSpaceConcept|_ または |MemorySpaceConcept|_　の必要要件を満たすクラス。

.. cpp:function:: テンプレート <class ViewType, class ALLOC_PROP> auto create_mirror_view(ALLOC_PROP const& arg_prop, ViewType const& src);

    |View|_ constructor 引数 ``arg_prop`` ( `Kokkos::view_alloc`　への呼び出しにより、作成) がメモリ空間を含み、 そのメモリ空間が
    ``src``　のメモリ空間と一致しない場合には、 特定　memory_space　内の新たな |View|_  を作成します。　``arg_prop`` が、メモリ空間を含まず、 ``src`` のメモリ空間が、ホストアクセス可能でない場合には、新たなホストアクセス可能な |View|_　を作成します。
   そうでない場合には、 ``src`` を、返します。 新たな |View|_ が作成される場合には、 暗示的に呼び出されるコンストラクタは、``arg_prop``　を尊重し、
    ``src``　と同じレイアウトおよびパディングを使います。　

   - ``src``: a ``Kokkos::View``.

   - ``arg_prop``: |View|_ コンストラクタ特性、例えば、 ``Kokkos::view_alloc(Kokkos::WithoutInitializing)``.

     .. 重要事項::

	``arg_prop`` は、メモリまたはラベルへのポインタを含んではならず、 またはパディングを認めてはなりません。

.. cpp:function:: テンプレート <class Space, class ViewType> ImplMirrorType create_mirror_view_and_copy(Space const& space, ViewType const& src);

   ``std::is_same<typename Space::memory_space, typename ViewType::memory_space>::value`` が ``false``　であれば、
   ``src``　と同じレイアウトおよびパディングを使いますが、 ``Space::device_type``　のデバイス型を使って、作成し、作成されれば、  ``deep_copy`` を``src`` から新たなビューに実行します。そうでない場合には、 ``src``　を返します。

   - ``src``: a ``Kokkos::View``.

   - ``Space``:  |ExecutionSpaceConcept|_ または |MemorySpaceConcept|_　　の必要要件を満たすクラス。

   - ``ImplMirrorType``:  ``Kokkos::View``　　の実装定義の仕様。

.. cpp:function:: テンプレート <class ViewType, class ALLOC_PROP> ImplMirrorType create_mirror_view_and_copy(ALLOC_PROP const& arg_prop, ViewType const& src);

   If the  memory space included in the |View|_ コンストラクタ引数 ``arg_prop`` に含まれるメモリ空間　(　`Kokkos::view_alloc`　への呼び出しにより作成) が、 ``src``　のメモリ空間と一致しない場合には、 creates a new |View|_ in the specified memory space using ``arg_prop`` およびand the same layout
   and padding as ``src``　と同じレイアウトおよびパディングを使って、特定メモリ空間内の新たな  |View|_ 　を作成します。　さらに、Additionally, a from ``src`` から新たなビューへの ``deep_copy``　が実行されます (提供されれば、 ``arg_prop`` 内に含まれた実行空間を使用)。
   そうでない場合には、 ``src``　を返します。

   - ``src``: a ``Kokkos::View``.

   - ``arg_prop``: |View|_ コンストラクタ特性、例えば、 ``Kokkos::view_alloc(Kokkos::HostSpace{}, Kokkos::WithoutInitializing)``.

     .. 重要事項::

	``arg_prop`` は、メモリまたはラベルへのポインタを含んではならず、 またはパディングを認めてはなりません。そして、``arg_prop`` は、メモリ空間を含まなければなりません。
