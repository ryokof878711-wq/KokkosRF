# ScatterViewの要素をノードに平均化

 [`Kokkos::ScatterView`](../API/containers/ScatterView)　について典型的な使用事例を示すために、有限要素法プログラムにおいて、利用可能な情報が要素から節点への対応関係のみである場合を考えることが可能であり、要素からノードへ、ある量を平均化したいと思います。
この平均値は、二つの和の比率、つまり、隣接する要素の量の和を
隣接する要素の和で割ったものです。


## 隣接する要素の数を計算

ノードに隣接する要素の数を計算するだけでも、[`Kokkos::ScatterView`](../API/containers/ScatterView) に関する必要なワークフローの大半が明らかになります。
アルゴリズムは以下の通りです: メッシュ要素を並列に反復処理し、並行して、各メッシュ要素は自身のノードを特定し、そのノード固有の配列エントリに1を加算します。
それらのエントリは最終的に、ノードごとに1つのエントリを持つ　[`Kokkos::View`](../API/core/view/view)　に格納されますが、
アルゴリズムの間に、 データ競合を防止するため、これらは、[`Kokkos::ScatterView`](../API/containers/ScatterView)　を介してアクセスされます。

```c++
Kokkos::View<int*> count_adjacent_elements(Kokkos::View<int**> elements_to_nodes, int number_of_nodes) {
  Kokkos::View<int*> elements_per_node("elements_per_node", number_of_nodes);
  auto scatter_elements_per_node = Kokkos::Experimental::create_scatter_view(elements_per_node);
  Kokkos::parallel_for(elements_to_nodes.extent(0), KOKKOS_LAMBDA(int element) {
    auto access_elements_per_node = scatter_elements_per_node.access();
    for (int node_of_element = 0; node_of_element < elements_to_nodes.extent(1); ++node_of_element) {
      int node = elements_to_nodes(element, node_of_element);
      access_elements_per_node(node) += 1;
    }
  });
  Kokkos::Experimental::contribute(elements_per_node, scatter_elements_per_node);
  return elements_per_node;
}
```

## ノードにおける値の合計を計算

ノードに隣接する要素の値の合計を計算することは、ノード周辺の要素の数を計算することとほぼ同じです:

```c++
Kokkos::View<double*> sum_to_nodes(Kokkos::View<int**> elements_to_nodes, int number_of_nodes,
    Kokkos::View<double*> element_values) {
  Kokkos::View<double*> node_values("node_values", number_of_nodes);
  auto scatter_node_values = Kokkos::Experimental::create_scatter_view(node_values);
  Kokkos::parallel_for(elements_to_nodes.extent(0), KOKKOS_LAMBDA(int element) {
    auto access_node_values = scatter_node_values.access();
    for (int node_of_element = 0; node_of_element < elements_to_nodes.extent(1); ++node_of_element) {
      int node = elements_to_nodes(element, node_of_element);
      access_node_values(node) += element_values(element);
    }
  });
  Kokkos::Experimental::contribute(node_values, scatter_node_values);
  return node_values;
}
```

## 完全な平均値の計算

各ノードで2つの合計値が得られたため、ノードを最終的に1回ループ処理し、
これら2つの合計値の比率を求め、平均値を定義すれば十分です。
この関数は、各ノードに隣接する要素の数が事前に計算済みであると仮定した上で構成されます。.

```c++
Kokkos::View<double*> average_to_nodes(Kokkos::View<int**> elements_to_nodes, int number_of_nodes,
    Kokkos::View<double*> element_values,
    Kokkos::View<int*> elements_per_node) {
  auto node_values = sum_to_nodes(elements_to_nodes, number_of_nodes, element_values);
  Kokkos::parallel_for(number_of_nodes, KOKKOS_LAMBDA(int node) {
    node_values[node] /= elements_per_node[node];
  });
  node_values　を返す;
}
```
