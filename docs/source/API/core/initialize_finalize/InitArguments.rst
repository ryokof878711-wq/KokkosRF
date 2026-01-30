InitArguments
=============

.. role:: cpp(code)
   :language: cpp

.. _KokkosInitialize: initialize.html
.. |KokkosInitialize| replace:: ``Kokkos::initialize``

.. _KokkosInitializationSetting: InitializationSettings.html
.. |KokkosInitializationSetting| replace:: ``Kokkos::InitializationSettings``

``<Kokkos_Core.hpp>`` ヘッダーに定義。

.. warning:: 3.7より非推奨、 4.3において削除、 **代わりに** ``Kokkos::InitializationSettings``　を　**使用** 

インターフェイス
---------

.. cpp:struct:: InitArguments

   .. cpp:member:: int num_threads

   .. cpp:member:: int num_numa

   .. cpp:member:: int device_id

   .. cpp:member:: int ndevices

   .. cpp:member:: int skip_device

   .. cpp:member:: bool disable_warnings

   .. cpp:function:: InitArguments()

``InitArguments`` は、|KokkosInitialize|_ に渡される引数をプログラムで定義するために使用可能な構造です。 Iバージョン3.7で廃止され、|KokkosInitializationSetting|_　が推奨されるようになりました。

その置換の主な理由の一つは、ユーザー指定のデータメンバーがデフォルト値を持つものと区別できない点にあります。

例
~~~~~~~

.. code-block:: cpp

   #include <Kokkos_Core.hpp>

   int main() {
     Kokkos::InitArguments arguments;
     arguments.num_threads = 2;
     arguments.device_id = 1;
     arguments.disable_warnings = true;
     Kokkos::initialize(arguments);
     // ...
     Kokkos::finalize();
   }


以下も参照
~~~~~~~~

* |KokkosInitializationSetting|_
* |KokkosInitialize|_
