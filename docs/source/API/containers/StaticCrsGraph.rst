
``StaticCrsGraph`` [DEPRECATED]
===============================

ヘッダーファイル: ``<Kokkos_StaticCrsGraph.hpp>`` (deprecated since Kokkos 4.5)

 ``StaticCrsGraph``　を、KokkosKernels　に移動しました。


StaticCrsGraph　は、圧縮された行ストレージ配列であり、行マップ、列インデックス、および非ゼロエントリが3つの異なる　Kokkos::Views　に格納されています。ホストまたはデバイス上で、CRSデータの操作とアクセスを簡素化するために、適切な型と関数が提供されています。


使用例
-----

.. code-block:: cpp

    using StaticCrsGraphType = Kokkos::StaticCrsGraph<unsigned, Space, void, void, unsigned>;
    StaticCrsGraphType graph();

    const int begin = graph.row_map[0];
    const int end = graph.row_map[1];

    double sum = 0;
    for (int i = begin; i < end; i++) {
        Kokkos::View<unsigned, Space> v(graph.entries(i));
        for (int j = 0; j < v.extent(0); j++) {
            sum += v(j);
        }
    }
