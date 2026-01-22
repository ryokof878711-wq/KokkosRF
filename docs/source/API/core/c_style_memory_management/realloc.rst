``kokkos_realloc``
==================

.. role:: cpp(code)
    :language: cpp

ヘッダー ``<Kokkos_Core.hpp>``　に定義。

.. _Kokkos_kokkos_malloc: malloc.html

.. |Kokkos_kokkos_malloc| replace:: ``Kokkos::kokkos_malloc()``

.. _Kokkos_kokkos_realloc: realloc.html

.. |Kokkos_kokkos_realloc| replace:: ``Kokkos::kokkos_realloc()``

.. _MemorySpace: ../memory_spaces.html

.. |MemorySpace| replace:: ``MemorySpace``

.. _Kokkos_kokkos_free: free.html

.. |Kokkos_kokkos_free| replace:: ``Kokkos::kokkos_free()``

指定されたメモリ領域を再割り当てします。それは、同じメモリースペース |MemorySpace|_　上で、以前に |Kokkos_kokkos_malloc|_ または |Kokkos_kokkos_realloc|_　により割り当てられる必要があり、
まだ |Kokkos_kokkos_free|_ で解放されておらず、そうでない場合には、結果は未定義となる。

.. warning::

   Kokkos　の管理対象外であるメモリに対して、メモリの動作を操作する関数（例：memAdvise）を呼び出すと、未定義の動作を引き起こします。

ディスクリプション
-----------

.. cpp:function:: template <class MemorySpace = Kokkos::DefaultExecutionSpace::memory_space> void* kokkos_realloc(void* ptr, size_t new_size);

  :tparam MemorySpace: Controls the storage location. If omitted the memory space of the default execution space is used (i.e. ``Kokkos::DefaultExecutionSpace::memory_space``).

  :param ptr: The pointer to the memory area to be reallocated.

  :param new_size: The new size in bytes.

  :returns: On success, returns a pointer to the beginning of the newly allocated memory. To avoid a memory leak, the returned pointer must be deallocated with |Kokkos_kokkos_free|_, the original pointer ``ptr`` is invalidated and any access to it is undefined behavior (even if reallocation was in-place). On failure, returns a null pointer. The original pointer ptr remains valid and may need to be deallocated with |Kokkos_kokkos_free|_.

  :throws: On failure, throws ``Kokkos::Experimental::RawMemoryAllocationFailure``.
