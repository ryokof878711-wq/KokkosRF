初期化及び最終処理完了
=======================

Kokkos::初期化
------------------

Kokkos　の内部オブジェクトと、有効化されたすべての　Kokkos　バックエンドを初期化します

詳細については、 `Kokkos::initialize <initialize_finalize/initialize.html>`_ を参照してください。


Kokkos::最終処理完了
----------------

Shutdown Kokkos は、実行空間を初期化し、内部管理リソースを解放します。

詳細については、 `Kokkos::finalize <initialize_finalize/finalize.html>`_ を参照してください。


Kokkos::is_initialized
----------------------
Kokkos　の初期化状態を照会することができ、Kokkos　が初期化されている場合に、`true`　を返します。

詳細については、 `Kokkos::is_initialized <initialize_finalize/is_Initialized.html>`_ を参照してください。

Kokkos::is_finalized
--------------------
Allows to query finalizaton status of Kokkos and returns `true` is Kokkos is finalized.Kokkos　の最終処理完了状態を照会することができ、Kokkos　が最終処理が完了している場合に、`true`　を返します。


詳細については、 `Kokkos::is_finalized <initialize_finalize/is_Finalized.html>`_ を参照してください。

Kokkos::ScopeGuard
------------------

``Kokkos::ScopeGuard`` は、Kokkos　によって管理されるリソースを集約するクラスです。 ScopeGuard は構築時に``Kokkos::initialize``を呼び出し、破棄時に ``Kokkos::finalize``　を呼び出します。このように、　Kokkos のコンテキストは、ScopeGuard　オブジェクトの範囲を通じて、自動的に管理されます。

詳細については、 `Kokkos::ScopeGuard <initialize_finalize/ScopeGuard.html>`_ を参照してください。

ScopeGuard は、Kokkos　オブジェクトが``Kokkos::finalize``を実行した後も存続してしまうという、一般的なミスを防ぐのに役立ちます:

.. code-block:: cpp

  int main(int argc, char** argv) {
    Kokkos::initialize(argc, argv);
    Kokkos::View<double*> my_view("my_view", 10);
    Kokkos::finalize();
    // my_view destructor called after Kokkos::finalize !
  }

 ``Kokkos::ScopeGuard`` に切り替えると修正されます:

.. code-block:: cpp

  int main(int argc, char** argv) {
    Kokkos::ScopeGuard kokkos(argc, argv);
    Kokkos::View<double*> my_view("my_view", 10);
    // my_view destructor called before Kokkos::finalize
    // ScopeGuard destructor called, calls Kokkos::finalize
  }

上記の例においては、 ``my_view`` は、　main()　関数の終了時まで、範囲から外れません。  ``ScopeGuard``　がなければ、 ``my_view`` が範囲から外れる前に、　``Kokkos::finalize`` が呼び出されます。  ``ScopeGuard``　があれば、 ``my_view`` の参照が解除された後に、``ScopeGuard`` の参照が解除されますが (その後、ubsequently calling ``Kokkos::finalize``を呼び出します) 、それがシャットダウン間の適切な順序を保証します。

.. toctree::
   :hidden:
   :maxdepth: 1

   ./initialize_finalize/initialize
   ./initialize_finalize/finalize
   ./initialize_finalize/is_Initialized
   ./initialize_finalize/is_Finalized
   ./initialize_finalize/ScopeGuard
   ./initialize_finalize/InitializationSettings
   ./initialize_finalize/InitArguments
   ./initialize_finalize/push_finalize_hook
