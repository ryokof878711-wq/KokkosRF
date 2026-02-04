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

- Blocks on completion of all outstanding asynchronous works. Side effects of outstanding work will be observable upon completion of the ``fence`` call - that means ``Kokkos::fence()`` implies a memory fence.

Examples
--------

Timing kernels
~~~~~~~~~~~~~~

.. code-block:: cpp

    Kokkos::Timer timer;
    // This operation is asynchronous, without a fence 
    // one would time only the launch overhead
    Kokkos::parallel_for("Test", N, functor);
    Kokkos::fence();
    double time = timer.seconds();

Use with asynchronous deep copy
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: cpp

    Kokkos::deep_copy(exec1, a,b);
    Kokkos::deep_copy(exec2, a,b);
    // do some stuff which doesn't touch a or b
    Kokkos::parallel_for("Test", N, functor);

    // wait for all three operations to finish
    Kokkos::fence();

    // do something with a and b
