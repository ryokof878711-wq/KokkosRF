``fence``
=========

.. role::cpp(code)
    :language: cpp

ヘッダーファイル: ``<Kokkos_Core.hpp>``

使用例:

.. code-block:: cpp

    Kokkos::fence();

未完了の非同期　Kokkos　演算がすべて完了した時点でブロックします。
それは、三引数 `deep_copy <../view/deep_copy.html>`_　だけでなく、並列ディスパッチ　(例えば、 `parallel_for() <parallel_for.html#kokkosparallel_for>`_, `parallel_reduce() <parallel_reduce.html#kokkosparallel_reduce>`_ 
and `parallel_scan() <parallel_scan.html#kokkosparallel_scan>`_) を
含みます。

注意事項: 実行空間インスタンス固有の　``fence`` も存在します。

インターフェイス
---------

.. code-block:: cpp

    void Kokkos::fence();

.. code-block:: cpp

    void Kokkos::fence(const std::string& label);

パラメータ
~~~~~~~~~~

- ``label``: A label to identify a specific fence in fence profiling operations. ``label`` does not have to be unique.

必要要件
~~~~~~~~~~~~

- ``Kokkos::fence()`` は、既存の並列領域内では呼び出すことはできません (つまり、  ``operator()`` of a functor or lambda内)。

セマンティクス
---------

- すべての未処理の非同期作業が完了した時点でブロックします。 未処理作業の副作用は、 ``fence`` 呼び出しの完了時に観測可能となります。つまり ``Kokkos::fence()`` は、メモリフェンスを意味します。

例
--------

タイミングカーネル
~~~~~~~~~~~~~~

.. code-block:: cpp

    Kokkos::Timer timer;
    // This operation is asynchronous, without a fence この演算は非同期であり、フェンスなしで行われます。
    // ローンチオーバーヘッドのみを計測します。
    Kokkos::parallel_for("Test", N, functor);
    Kokkos::fence();
    double time = timer.seconds();

非同期の深部コピーと併用
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: cpp

    Kokkos::deep_copy(exec1, a,b);
    Kokkos::deep_copy(exec2, a,b);
    // aやbに干渉しない何かを行います。
    Kokkos::parallel_for("Test", N, functor);

    // Kokkos::fence();　を完了するために
    3つの演算すべてを待ちます。

    // a および bを使って何かを行います。
