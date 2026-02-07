``Kokkos::num_threads``
=======================

.. role:: cpp(code)
    :language: cpp

 ヘッダー ``<Kokkos_Core.hpp>``　に定義。

.. code-block:: cpp

    [[nodiscard]] int num_threads() noexcept;  // (since 4.1)

 ``DefaultHostExecutionSpace``　により使用された同時実行スレッド数を返します。

----

**以下も参照**

.. _device_id : device_id.html

.. |device_id| replace:: ``device_id``

.. _num_devices : num_devices.html

.. |num_devices| replace:: ``num_devices``

.. _initialize: ../initialize_finalize/initialize.html

.. |initialize| replace:: ``initialize``

.. _InitializationSettings: ../initialize_finalize/InitializationSettings.html

.. |InitializationSettings| replace:: ``InitializationSettings``

|num_devices|_:  Kokkos　が利用可能なデバイス数を返します。

|device_id|_:  Kokkos　により使用されるデバイスの　id を返します。

|initialize|_:  Kokkos　実行環境を初期化します。

|InitializationSettings|_: settings for initializing Kokkos
