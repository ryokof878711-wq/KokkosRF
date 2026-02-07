``Kokkos::num_devices``
=======================

.. role:: cpp(code)
    :language: cpp

ヘッダー ``<Kokkos_Core.hpp>``　に定義。

.. code-block:: cpp

    [[nodiscard]] int num_devices() noexcept;  // (since 4.3)

Returns the number of available devices on the system or ``-1`` if only host backends are enabled.ホストバックエンドのみが有効な場合は、システム上で利用可能なデバイスの数、または　``-1``を返します。

注意事項
-----

``Kokkos::num_devices()`` は、Kokkosが実行に利用できるデバイスの数を
決定するために使用される場合があります。
これは、``Kokkos::initialize()`` の前または ``Kokkos::finalize()`` の後に
呼び出される可能性のある数少ないランタイム関数の1つです。

例
-------

.. code-block:: cpp

   #include <Kokkos_Core.hpp>
   #include <iostream>

   int main(int argc, char* argv[]) {
     if (Kokkos::num_devices() == 0) {
       std::cerr << "no device available for execution\n";
       return 1;
     }
     Kokkos::initialize(argc, argv);
     // 何かを行う
     Kokkos::finalize();
     return 0;
   }


----

**以下も参照**

.. _device_id : device_id.html

.. |device_id| replace:: ``device_id``

.. _num_threads : num_threads.html

.. |num_threads| replace:: ``num_threads``

.. _initialize: ../initialize_finalize/initialize.html

.. |initialize| replace:: ``initialize``

.. _InitializationSettings: ../initialize_finalize/InitializationSettings.html

.. |InitializationSettings| replace:: ``InitializationSettings``

|device_id|_:  Kokkos　により使用されるデバイスの　id を返します。

|num_threads|_:  Kokkos　により使用されるスレッド数を返します。

|initialize|_:  Kokkos 実行環境を初期化します。execution environment

|InitializationSettings|_:  Kokkos　初期化の設定
