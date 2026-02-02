``is_finalized``
================

.. role::cpp(code)
    :language: cpp

ヘッダー　``<Kokkos_Core.hpp>``　に定義。

インターフェイス
---------

.. cpp:function:: bool is_finalized() noexcept

    Kokkos　の最終処理完了ステータスを照会し、Kokkos　の最終処理が完了している場合は、``true``　を、の最終処理が完了していない場合は、``false``　を返します。  Kokkos の初期化または最終処理完了前後に、この関数を呼び出すことが可能です。
   
   :return:if :cpp:func:　`finalize` が呼び出されている場合には、 ``true`` ; そうでない場合には、`false` 。

例
--------

.. code-block:: cpp

    #include <Kokkos_Core.hpp>
    #include <cassert>

    int main(int argc, char* argv[]) {
        assert(!Kokkos::is_finalized());
        Kokkos::initialize(argc, argv);
	assert(!Kokkos::is_finalized());
        Kokkos::finalize();
        assert(Kokkos::is_finalized());
    }    

.. seealso::

   `Kokkos::InitializationSettings <InitializationSettings.html#kokkosInitializationSettings>`_
      Kokkos　をプログラムで初期化する設定を定義します。
   `Kokkos::ScopeGuard <ScopeGuard.html#kokkosScopeGuard>`_
      RAII　を使用して、Kokkos　を初期化および完了するクラス。
