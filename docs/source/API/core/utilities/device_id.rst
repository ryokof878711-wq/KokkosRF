``Kokkos::device_id``
=====================

.. role:: cpp(code)
    :language: cpp

ヘッダー ``<Kokkos_Core.hpp>``　に定義。

.. code-block:: cpp

    [[nodiscard]] int device_id() noexcept;  // (4.1より)

ホストバックエンドのみが有効な場合、 ``DefaultExecutionSpace`` または　``-1`` により使用されるデバイスの
id　を返します。

----

**以下も参照**

.. _num_devices : num_devices.html

.. |num_devices| replace:: ``num_devices``

.. _num_threads : num_threads.html

.. |num_threads| replace:: ``num_threads``

.. _initialize: ../initialize_finalize/initialize.html

.. |initialize| replace:: ``initialize``

.. _InitializationSettings: ../initialize_finalize/InitializationSettings.html

.. |InitializationSettings| replace:: ``InitializationSettings``

|num_devices|_:  Kokkos　に利用可能なデバイスの数を返します。

|num_threads|_:  Kokkos　が使用するスレッド数を返します。

|initialize|_:  Kokkos 実行環境を初期化します。

|InitializationSettings|_: Kokkos　を初期化するための設定
