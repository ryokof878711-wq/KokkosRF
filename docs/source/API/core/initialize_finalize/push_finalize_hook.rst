``push_finalize_hook``
======================

.. role::cpp(code)
    :language: cpp

ヘッダー ``<Kokkos_Core.hpp>``　に定義。

使用例
-----

.. code-block:: cpp

    Kokkos::push_finalize_hook(func);

Kokkos 実行環境が完了した場合に、呼び出されるべき、呼び出し可能なオブジェクト ``func``　を
登録。

 ``Kokkos::push_finalize_hook()`` 経由で登録された関数は、
取得したリソースを解放し、すべてのバックエンドの最終処理を完了する前に、
``Kokkos::finalize()``　を入力する際、逆順で呼び出されます。

関数が例外をスローして終了した場合、``std::terminate`` が呼び出されます。

インターフェイス
---------

.. cpp:Function:: void push_finalize_hook(std::function<void()> func);

   ``Kokkos::finalize()`` 入力の際に
   呼び出されるべき関数オブジェクト ``func`` を登録。




例
-------

.. code-block:: cpp

    #include <Kokkos_Core.hpp>
    #include <iostream>

    void my_hook() {
      std::cout << "Cruel world!\n";
    }

    int main(int argc, char* argv[]) {
        Kokkos::initialize(argc, argv);
        Kokkos::push_finalize_hook(my_hook);
        Kokkos::push_finalize_hook([]{ std::cout << "Goodbye\n"; });
        std::cout << "Calling Kokkos::finalize() ...\n";
        Kokkos::finalize();
    }


出力:

.. code-block::

     Kokkos::finalize() ...　呼び出し。
    Goodbye
    Cruel world!


以下も参照
--------
* `Kokkos::finalize <finalize.html>`_: Kokkos 実行環境を完了します。
