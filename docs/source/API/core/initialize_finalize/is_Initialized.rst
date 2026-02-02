``is_initialized``
==================

.. role::cpp(code)
    :language: cpp

ヘッダー ``<Kokkos_Core.hpp>``　に定義。

インターフェイス
---------

.. cpp:function:: bool is_initialized() noexcept

   Kokkos　の初期化ステータスを照会し、Kokkos　を初期化している場合は、``true``　を、初期化していない場合は、``false``　を返します。 Kokkos の初期化または最終処理完了前後に、この関数を呼び出すことが可能です。 
:return: if :cpp:function:   `initialize`　が呼び出されている場合には、 ``true`` ; そうでない場合には、`false` 。

例
--------

.. code-block:: cpp

    #include <Kokkos_Core.hpp>
    #include <cassert>

    int main(int argc, char* argv[]) {
        assert(!Kokkos::is_initialized());
        Kokkos::initialize(argc, argv);
        assert(Kokkos::is_initialized());
        Kokkos::finalize();
        assert(!Kokkos::is_initialized());
    }    

以下も参照。
--------

* `Kokkos::InitializationSettings <InitializationSettings.html#kokkosInitializationSettings>`_
* `Kokkos::ScopeGuard <ScopeGuard.html#kokkosScopeGuard>`_
