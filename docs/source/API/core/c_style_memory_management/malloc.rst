``kokkos_malloc``
=================

.. role:: cpp(code)
    :language: cpp

ヘッダー ``<Kokkos_Core.hpp>``　に定義

.. _MemorySpace: ../memory_spaces.html

.. |MemorySpace| replace:: ``MemorySpace``

.. _Kokkos_kokkos_free: free.html

.. |Kokkos_kokkos_free| replace:: ``Kokkos::kokkos_free()``

.. _Kokkos_realloc: realloc.html

.. |Kokkos_realloc| replace:: ``Kokkos::kokkos_realloc()``

指定されたメモリ空間 |MemorySpace|_ 上に、ラベルなどのメタデータ用に追加のスペースを加えた、初期化されていないストレージを ``size`` バイト分割り当てます。

割り当てが成功した場合、任意のスカラー型に適したアラインメントを持つ割り当てメモリブロック内で、最下位（最初の）バイトへのポインタを返す。

割り当てに失敗した場合、``Kokkos::Experimental::RawMemoryAllocationFailure`` 型の例外がスローされます。

.. warning::

    Kokkos　の管理対象外であるメモリに対して、メモリの動作を操作する関数（例：memAdvise）を呼び出すと、未定義の動作を引き起こします。

ディスクリプション
-----------

.. cpp:function:: template <class MemorySpace = Kokkos::DefaultExecutionSpace::memory_space> void* kokkos_malloc(const string& label, size_t size);

.. cpp:function:: template <class MemorySpace = Kokkos::DefaultExecutionSpace::memory_space> void* kokkos_malloc(size_t size);

    :tparam MemorySpace: ストレージ先をコントロールします。　省略された場合、デフォルトの実行領域のメモリ領域が使用されます (つまり、``Kokkos::DefaultExecutionSpace::memory_space``)。

    :param ラベル: KokkosP プロファイリングツールを介してプロファイリングおよびデバッグツールで使用される、ユーザーが提供した文字列。

    :param サイズ: 割り当てるバイト数。

    :returns: 成功した場合、新たに割り当てられたメモリの先頭へのポインタを返します。 メモリリークを避けるため、返されたポインタは |Kokkos_kokkos_free|_ または |Kokkos_realloc|_ で解放する必要があります。

    :throws: 失敗した場合、``Kokkos::Experimental::RawMemoryAllocationFailure`` をスローします。
