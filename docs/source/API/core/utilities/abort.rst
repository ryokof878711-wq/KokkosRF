``Kokkos::abort``
=================

.. role:: cpp(code)
    :language: cpp

ヘッダー ``<Kokkos_Core.hpp>``　に定義。

.. code-block:: cpp

    KOKKOS_FUNCTION void abort(const char *const msg);

異常なプログラム終了が発生し、エラーの説明文字列が表示されます。

注意事項
-----

.. _KokkosAbort: https://github.com/kokkos/kokkos/blob/4.2.00/core/src/Kokkos_Abort.hpp

.. |KokkosAbort| replace:: ``<Kokkos_Abort.hpp>``

* Since バージョン　4.2　より、  ``<Kokkos_Core.hpp>``　の代わりに、|KokkosAbort|_　が含まれる場合があります。
