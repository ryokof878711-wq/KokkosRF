# アトミック演算

本章を読んだ後に、以下を理解する必要があります:

* Atomic operations can be used to resolve write conflictsアトミック演算は、書き込み競合を解決するために使用。
* フリー関数の使用方法。
* アトミックメモリ特性の使用方法。
* 散布図の追加パターンに　[ScatterView](../API/containers/ScatterView)　を使用

## 書き込み競合およびそのアトミック演算による解決

数値集合のヒストグラムを作成するという単純な作業を考案。

```c++
void create_histogram(View<int*> histogram, int min, int max, View<int*> values) {
  deep_copy(histogram,0);
  for(int i=0; i<values.extent(0); i++) {
    int index = (1.0*(values(i)-min)/(max-min)) * histogram.extent(0);
    histogram(index)++;
  }
}
```

このループを単純な　[`parallel_for()`](../API/core/parallel-dispatch/parallel_for)　で並列化すると、複数のスレッドが同時に同じ　`index`　をインクリメントしようとする可能性があります。その一方で、増分は実際には、３つの演算です:
  1. `histogram(index)` をレジスタへ読み込み、
  2. レジスタを増分し、
  3. レジスタを　`&histogram(index)`　に格納。

2つのスレッドが同時に同じインデックスに対してこの操作を試みると、両方のスレッドが値を読み込み、増分し、保存するという状況が発生する可能性があります。 どちらも同じ元の値を読み込んだため、更新のうち1つだけが適用されますが、一方、2番目の増加分は失われます。 これは、*競合状態*　と呼ばれます。

この状況におけるもう一つの典型的な例が、いわゆる*scatter-add* アルゴリズムです。
例えば粒子コードでは、全ての粒子をループ処理し、各粒子に対して
その近傍の粒子それぞれに何かを寄与させます（例えば部分的な力など）。 2つのスレッドが同時に共有隣接要素を持つ2つの粒子に対して作業する場合、それらは同じ隣接要素粒子への貢献について、競合する可能性があります。

```c++
void compute_force(View<int**> neighbours, View<double*> values) {
  parallel_for("ForceLoop", neighbors.extent(0), KOKKOS_LAMBDA (const int particle_id) {
    for(int i = 0; i < neighbours.extent(1); i++) {
      int neighbour_id = neighbours(i);
      // This contribution will potentially race with other threads
      values(neighbour_id) += some_contribution(...);
    }
  });
}
```


こうした状況に対処するには、いくつかの方法があります: (i) 各色の部分集合間で競合が発生しないように着色を適用しアルゴリズムを複数回実行することができ、(ii) 各スレッドに対して出力配列を複製することができ、または (iii) アトミック演算を使用することができます。 これらすべてに欠点があります。

着色には、セットを作成しなければならないという欠点があります。 ヒストグラムの例では、セットの作成コストは演算自体よりも大きい可能性が高いです。さらに、各色別々に処理する必要があるため、メモリ転送の総量はかなり大きくなる可能性があります。各キャッシュラインの一部しか使用しないにもかかわらず、すべての割り当てを複数回ループ処理する傾向があるためです。

出力配列の複製はスレッド数が少ない場合（2～8）には有効な戦略だが、それ以上になるとしばしば破綻する傾向にあります。

アトミック演算は、論理演算全体を中断なく実行します。アトミック演算は、論理演算全体を中断なしで実行します。例えば、上記の例のロード・変更・保存サイクルは、アトミック演算が完了するまで、他のスレッドが変更されたライブラリに（別のアトミック演算を介して）アクセスできない状態で実行されます。非アトミック演算は、アトミック演算と競合する可能性があることに注意してください。アトミック操作の欠点は、特定のコンパイラ最適化を阻害することであり、 アトミック演算のスループットは、アーキテクチャやスカラー型によっては良好でない場合があります。

## アトミックフリー関数

アトミック自由関数は、更新対象値へのポインタと更新内容を引数とする関数です。すべての典型的な操作には独自の原子フリー関数があります。上記の2つの例は次のようになります:

```c++
void create_histogram(View<int*> histogram, int min, int max, View<int*> values) {
  deep_copy(histogram,0);
  parallel_for("CreateHistogram", values.extent(0), KOKKOS_LAMBDA(const int i) {
    int index = (1.0*(values(i)-min)/(max-min)) * histogram.extent(0);
    atomic_inc(&histogram(index));
  });
}
```
```c++
void compute_force(View<int**> neighbours, View<double*> values) {
  parallel_for("ForceLoop", neighbors.extent(0), KOKKOS_LAMBDA (const int particle_id) {
    for(int i = 0; i < neighbours.extent(1); i++) {
      int neighbour_id = neighbours(i);
      atomic_add(&values(neighbour_id), some_contribution(...));
    }
  });
}
```

また、古い値または新しい値を返すアトミック演算も存在します。これらは、[`atomic_fetch_[op]`](../API/core/atomics/atomic_fetch_op)　および　[`atomic_[op]_fetch`](../API/core/atomics/atomic_op_fetch)　の命名規則に従います。例えば、配列内の負の値のすべてのインデックスを見つけ、それらをリストに格納したい場合、以下のアルゴリズムが適用されます:
```c++
void find_indices(View<int*> indices, View<double*> values) {
  View<int> count("Count");
  parallel_for("FindIndices", values.extent(0), KOKKOS_LAMBDA(const int i) {
    if(values(i) < 0) {
      int index = atomic_fetch_add(&count(),1);
      indices(index) = i;
    }
  });
}
```

アトミック演算の完全なリストはこちらで見られます:

| 名前                                                                                  | ライブラリ                   | カテゴリ | ディスクリプション                  |
|:--------------------------------------------------------------------------------------|:--------------------------|:-----------|:----------------------------|
| [atomic_compare_exchange](../API/core/atomics/atomic_compare_exchange)                | [Core](../API/core-index) | [Atomic-Operations](Atomic-Operations) | 比較対象の値と一致した場合にのみ値を交換し、そうでない場合は元の値を返すアトミック演算。 |
| [atomic_exchange](../API/core/atomics/atomic_exchange)                                | [Core](../API/core-index) | [Atomic-Operations](Atomic-Operations) | 値を交換し、元の値を返すアトミック演算。 |
| [atomic_fetch_\[op\]](../API/core/atomics/atomic_fetch_op)                            | [Core](../API/core-index) | [Atomic-Operations](Atomic-Operations) | 古い値を返す様々なアトミック演算。 [op] は、`add`, `and`, `div`, `lshift`, `max`, `min`, `mod`, `mul`, `or`, `rshift`, `sub` or `xor`　である場合があります。|
| [atomic_load](../API/core/atomics/atomic_load)                                        | [Core](../API/core-index) | [Atomic-Operations](Atomic-Operations) | 値を読み込むアトミック演算。 |
| [atomic_\[op\]](../API/core/atomics/atomic_op)                                        | [Core](../API/core-index) | [Atomic-Operations](Atomic-Operations) | 何も返さないアトミック演算。 [op] は、 `and`, `add`, `dec`, `max`, `min`, `inc`, `or` or `sub`　である場合があります。 |
| [atomic_\[op\]_fetch](../API/core/atomics/atomic_op_fetch)                            | [Core](../API/core-index) | [Atomic-Operations](Atomic-Operations) | 更新後の値を返す様々なアトミック演算。 [op] は、 `add`, `and`, `div`, `lshift`, `max`, `min`, `mod`, `mul`, `or`, `rshift`, `sub` or `xor` である場合があります。|
| [atomic_store](../API/core/atomics/atomic_store)                                      | [Core](../API/core-index) | [Atomic-Operations](Atomic-Operations) | 値を格納するアトミック演算。 |

## アトミックメモリ特性

カーネル中の特定の`View`に対するすべての演算がアトミックである場合、アトミックメモリ特性も使用できます。
通常、*非アトミック*　な　`View`　から　*アトミック*　な　`View`　を作成するのは、そのカーネルの処理のためだけであり、その後は通常の操作を適用します：

```c++
void create_histogram(View<int*> histogram, int min, int max, View<int*> values) {
  deep_copy(histogram,0);
  View<int*,MemoryTraits<Atomic> > histogram_atomic = histogram;
  parallel_for("CreateHistogram", values.extent(0), KOKKOS_LAMBDA(const int i) {
    int index = (1.0*(values(i)-min)/(max-min)) * histogram.extent(0);
    histogram_atomic(index)++;
  });
}
```

## ScatterView

CPU　では、特にKokkos　を　MPI　と併用する場合に、スレッド数を低く設定することが多いです。
データレプリケーションはアトミック演算を使用する場合よりも高いパフォーマンスを発揮することが多いです。
ポータブルコードを維持するためには、[`ScatterView`](../API/containers/ScatterView)　を使用できます。 これにより、基盤となるハードウェアに応じて、アトミック演算の使用からデータ複製の使用へ、コンパイル時に透過的に切り替えることが可能になります

詳細はこちらで確認できます: [ScatterView](../API/containers/ScatterView)
