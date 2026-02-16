.. _api-memory-spaces:

メモリ空間
=============

.. role:: cpp(code)
    :language: cpp

.. _MemorySpaceType: #kokkos-memoryspaceconcept

.. |MemorySpaceType| replace:: :cpp:func:`MemorySpace` type

.. _TheDocumentationOnTheMemorySpaceConcept: #kokkos-memoryspaceconcept

.. |TheDocumentationOnTheMemorySpaceConcept| replace:: the documentation on the :cpp:func:`MemorySpace` concept

.. _Experimental: utilities/experimental.html#experimentalnamespace

.. |Experimental| replace:: Experimental

.. _ExecutionSpaceType: ./execution_spaces.html#kokkos-executionspaceconcept

.. |ExecutionSpaceType| 置換:: :cpp:func:`ExecutionSpace` 型

.. _ExecutionSpaceTypes: ./execution_spaces.html#kokkos-executionspaceconcept

.. |ExecutionSpaceTypes| 置換:: :cpp:func:`ExecutionSpace` 型

``Kokkos::CudaSpace``
---------------------

``Kokkos::CudaSpace`` は、Cuda対応GPU上のデバイスメモリを表すMemorySpaceTypeです。  ごく稀な場合を除き、直接使用すべきではなく、代わりに汎用的な実行空間として使用される必要があります。詳細については、 |TheDocumentationOnTheMemorySpaceConcept|_　を参照してください。

``Kokkos::CudaHostPinnedSpace``
-------------------------------

``Kokkos::CudaHostPinnedSpace`` は、Cuda対応GPUからアクセス可能なホスト側の固定メモリを表す。このメモリは通常、ホストとデバイスの両方の実行空間からアクセス可能です。  ごく稀な場合を除き、直接使用すべきではなく、代わりに汎用的な実行空間として使用される必要があります。 詳細については、 |TheDocumentationOnTheMemorySpaceConcept|_　を参照してください。

``Kokkos::CudaUVMSpace``
------------------------

``Kokkos::CudaUVMSpace`` は、Cuda　対応　GPU　システム上の統合仮想メモリを表す　MemorySpaceType　です。 統合仮想メモリは、ほとんどのホスト実行空間からもアクセス可能です。 ごく稀な場合を除き、直接使用すべきではなく、代わりに汎用的な実行空間として使用される必要があります。 詳細については、 |TheDocumentationOnTheMemorySpaceConcept|_　を参照してください。 

``Kokkos::HIPSpace``
--------------------

``Kokkos::HIPSpace`` :sup:`promoted from` |Experimental|_ :sup:`バージョン4.0以降` は、HIP GPUプログラミング環境においてGPU上のデバイスメモリを表すMemorySpaceTypeです。 ごく稀な場合を除き、直接使用すべきではなく、代わりに汎用的な実行空間として使用される必要があります。 詳細については、 |TheDocumentationOnTheMemorySpaceConcept|_　を参照してください。 

``Kokkos::HIPHostPinnedSpace``
------------------------------

``Kokkos::HIPHostPinnedSpace`` :sup:`promoted from` |Experimental|_ :sup:`バージョン4.0以降` は、HIP GPU　プログラミング環境において、GPU　からアクセス可能なホスト側の固定メモリを表す　MemorySpaceType　です。このメモリはホストとデバイスの両方の実行空間からアクセス可能です。 ごく稀な場合を除き、直接使用すべきではなく、代わりに汎用的な実行空間として使用される必要があります。 詳細については、 |TheDocumentationOnTheMemorySpaceConcept|_　を参照してください。

``Kokkos::HIPManagedSpace``
---------------------------

``Kokkos::HIPManagedSpace`` :sup:`promoted from` |Experimental|_ :sup:`バージョン4.0以降`  は、HIP GPU　プログラミング環境における　GPU　上のページ移行メモリを表す　MemorySpaceType　です。ページ移行メモリは、ほとんどのホスト実行空間からアクセス可能です。 すべてのオペレーティングシステムと　HIP　対応ハードウェアの組み合わせで利用可能ですが、``xnack``　機能をサポートし有効化するには、オペレーティングシステムとハードウェアの両方が対応している必要があります。ごく稀な場合を除き、直接使用すべきではなく、代わりに汎用的な実行空間として使用される必要があります。 詳細については、 |TheDocumentationOnTheMemorySpaceConcept|_　を参照してください。

``Kokkos::SYCLDeviceUSMSpace``
--------------------------------------------

``Kokkos::SYCLDeviceUSMSpace`` :sup:`promoted from` |Experimental|_ :sup:`バージョン4.5以降` は、SYCL GPU　プログラミング環境において、GPU　上のデバイスメモリを表します。本メモリは、SYCL　実行空間からのみアクセス可能です

``Kokkos::SYCLHostUSMSpace``
------------------------------------------

``Kokkos::SYCLHostUSMSpace`` :sup:`promoted from` |Experimental|_ :sup:`バージョン4.5以降` は、SYCL GPUプログラミング環境において、GPUからアクセス可能なホスト側の固定メモリを表す　|MemorySpaceType|_ です。このメモリは、ホスト実行空間とSYCL　実行空間の両方からアクセス可能です。 

``Kokkos::SYCLSharedUSMSpace``
--------------------------------------------

``Kokkos::SYCLSharedUSMSpace`` :sup:`promoted from` |Experimental|_ :sup:`バージョン4.5以降` は、SYCL GPUプログラミング環境において、GPU上のページ移行メモリを表す |MemorySpaceType|_ です。 このメモリは、ホスト実行空間とSYCL　実行空間の両方からアクセス可能です。 

``Kokkos::HostSpace``
---------------------

``Kokkos::HostSpace`` is CPUからアクセス可能な従来のランダムアクセスメモリを表す |MemorySpaceType|_ です。 ごく稀な場合を除き、直接使用すべきではなく、代わりに汎用的な実行空間として使用される必要があります。 詳細については、 |TheDocumentationOnTheMemorySpaceConcept|_　を参照してください。

``Kokkos::SharedSpace``
-----------------------

``Kokkos::SharedSpace`` :sup:`バージョン4.0以降` |MemorySpaceType| の別名であり、有効化された任意の |ExecutionSpaceType| からアクセス可能なメモリを表します。これを実現するため、メモリは「実行空間」で表される処理ユニットのローカルメモリとの間で移動させることができます。この動作は、アクセス時にOSとドライバによって自動的に行われます。アクセスする処理ユニットのローカルメモリに現在存在しない場合、メモリはチャンク単位で移動されます（サイズはバックエンドに依存します）。 これらのチャンクは独立して移動可能であり（例：GPU　上でアクセスされる部分のみをGPUへ移動）、処理ユニット上に存在する間はローカルメモリと同様に扱われます。 詳細については、 |TheDocumentationOnTheMemorySpaceConcept|_　を参照してください。

利用可能性は、プリプロセッサ定義 ``KOKKOS_HAS_SHARED_SPACE`` または ``constexpr bool Kokkos::has_shared_space`` で確認できます。
以下のバックエンドにおいて、``Kokkos::SharedSpace`` は対応する |MemorySpaceType|_ を指しています:

* Cuda -> ``CudaUVMSpace``
* HIP -> ``HIPManagedSpace``
* SYCL -> ``SYCLSharedUSMSpace``
* Only backends running on host -> ``HostSpace``

``Kokkos::SharedHostPinnedSpace``
---------------------------------

``Kokkos::SharedHostPinnedSpace`` :sup:`バージョン4.0以降` は、|MemorySpaceType|_ の別名であり、有効なすべての |ExecutionSpaceTypes|_ からアクセス可能です。メモリは、ホストに固定されたまま保持され、デバイス上ではゼロコピーアクセスにより小さなチャンク（バックエンドに応じて、キャッシュライン、メモリページ等）単位で利用可能となります。1つの　``実行空間``　への書き込みは、同期イベント時に他の　``実行空間``　で可視化されます。同期をトリガーするイベントは、バックエンドの仕様によって異なります。 しかしながら、フェンスはすべてのバックエンドにおいて同期化イベントです。

利用可能性は、プリプロセッサ定義 `` KOKKOS_HAS_SHARED_HOST_PINNED_SPACE`` または ``constexpr bool Kokkos::has_shared_host_pinned_space`` で確認できます。
以下のバックエンドにおいて、``Kokkos::SharedHostPinnedSpace`` は対応する |MemorySpaceType|_ を指しています:

* Cuda -> ``CudaHostPinnedSpace``
* HIP -> ``HipHostPinnedSpace``
* SYCL -> ``SYCLHostUSMSpace``
* Only backends running on host -> ``HostSpace``

``Kokkos::MemorySpaceConcept``
------------------------------

 ``MemorySpace`` の概念は、Kokkos　において、割り当ておよびアクセスが発生する　"場所"　と "方法"　を表すための、基本的な抽象化です。Kokkos　を使用するコードの大半は、特定のインスタンスよりもむしろ、``MemorySpace``　という　*汎用的な概念*　について、記述される必要があります。このページでは、Kokkos　の実行空間の一般的な機能を　実際にどのように*使用*　するかを説明しています；より正式で理論的な処理については、本文書 <KokkosConcepts.html>`_ 　を参照してください。

    *免責事項*: C++における　"概念"　という用語に目新しい点はありません; C++でテンプレートを使ったことがある人は
知っていようといまいと、コンセプトを使っています。「概念」という言葉自体に惑わされないでください。
この言葉は現在、C++20の新たな言語機能と結びつけられることが多くなっています。 ここで　"概念"　とは単に
"特定の場所でテンプレートパラメータとなる型に対してできること"　を意味します。

シノプシス
~~~~~~~~

.. code-block:: cpp

    // これは実際のクラスではなく、概念を簡略に記述したものです。
    class MemorySpaceConcept {
    パブリック:
        typedef MemorySpaceConcept memory_space;
        typedef ... execution_space;
        typedef Device<execution_space, memory_space> device_type;

        MemorySpaceConcept();
        MemorySpaceConcept(const MemorySpaceConcept& src);
        const char* name() const;
        void * allocate(ptrdiff_t size) const;
        void deallocate(void* ptr, ptrdiff_t size) const;
    };

    template<class MS>
    struct is_memory_space {
    enum { value = false };
    };

    template<>
    構造体 is_memory_space<MemorySpaceConcept> {
    enum { value = true };
    };  

型定義
~~~~~~~~

.. _ExecutionSpace: execution_spaces.html#executionspaceconcept

.. |ExecutionSpace| replace:: :cpp:func:`ExecutionSpace`

.. _DeepCopyDocumentation: view/deep_copy.html

.. |DeepCopyDocumentation| replace:: :cpp:func:`deep_copy` documentation

.. _KokkosSpaceAccessibility: SpaceAccessibility.html

.. |KokkosSpaceAccessibility| replace:: :cpp:func:`Kokkos::SpaceAccessibility`

* ``memory_space``: 自己型;
* ``execution_space``: デフォルトの |ExecutionSpace|_ は、``MemorySpace`` のインスタンスが提供するメモリ内でオブジェクトを構築するとき、または（場合によっては）そのようなメモリからの深部コピーまたは深部コピーを行う際に使用されます（詳細は |DeepCopyDocumentation|_ を参照）。 Kokkos は、``Kokkos::SpaceAccessibility<execution_space, memory_space>::accessible`` が ``true`` であることを保証します（|KokkosSpaceAccessibility|_ を参照）。
* ``device_type``: ``DeviceType<execution_space,memory_space>``.

コンストラクタ
~~~~~~~~~~~~

* ``MemorySpaceConcept()``: デフォルトコンストラクタ。
* ``MemorySpaceConcept(const MemorySpaceConcept& src)``: コピーコンストラクタ。

関数
~~~~~~~~~

* ``const char* name() const;``: メモリ空間のインスタンスのラベルを返します。
* ``void * allocate(ptrdiff_t size) const;``: ``MemorySpaceConcept`` が表すメモリリソースを使用して、少なくとも「size」バイトのバッファを割り当てます。
* ``void deallocate(void* ptr, ptrdiff_t size) const;``: 以前、正確に `allocate(size)` で割り当てられた、`ptr`（型 `void*`）から始まるバッファを解放します。

非メンバーファシリティ
~~~~~~~~~~~~~~~~~~~~~

* ``template<class MS> struct is_memory_space;``: クラスがメモリ空間であるかどうかを確認するための型特性。
* ``template<class S1, class S2> struct SpaceAccessibility;``: 2つのスペースが互換性があるか（割り当て可能、deep_copy可能、アクセス可能）を確認するための型特性。
