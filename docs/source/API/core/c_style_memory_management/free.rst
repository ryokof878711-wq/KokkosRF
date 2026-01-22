``kokkos_free``
===============

.. role:: cpp(code)
    :language: cpp

ヘッダー ``<Kokkos_Core.hpp>``　に定義

.. _Kokkos_kokkos_malloc: ./malloc.html

.. |Kokkos_kokkos_malloc| replace:: ``Kokkos::kokkos_malloc()``

.. _Kokkos_kokkos_realloc: ./realloc.html

.. |Kokkos_kokkos_realloc| replace:: ``Kokkos::kokkos_realloc()``

指定されたメモリ空間 ``MemorySpace`` 上で、|Kokkos_kokkos_malloc|_ または |Kokkos_kokkos_realloc|_ によって以前に割り当てられた領域を解放します。

``ptr``　がヌルポインタの場合、この関数は何も行いません。

ディスクリプション
-----------

.. cpp:function:: template <class MemorySpace = Kokkos::DefaultExecutionSpace::memory_space> void kokkos_free(void* ptr);

    :tparam MemorySpace: 保存先を制御します。 省略された場合、デフォルトの実行領域のメモリ領域が使用されます。 (つまり、 ``Kokkos::DefaultExecutionSpace::memory_space``　です).

    :param ptr: 指定されたメモリ領域上で解放するメモリへのポインタ。

    :returns: (無し)

    :throws: Throws ``std::runtime_error`` on failure to deallocate.解放に失敗した場合、``std::runtime_error`` をスローします。
