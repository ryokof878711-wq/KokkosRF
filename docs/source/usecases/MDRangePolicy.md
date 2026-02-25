# MDRangePolicy使用事例

## 多次元配列に関する演算

本例は、多次元配列またはテンソルデータに対する演算を実行するコンテクストにおける　[`MDRangePolicy`](../API/core/policies/MDRangePolicy)　の使用方法を示しています。
このような使用事例は、例えば有限要素法などの偏微分方程式の数値解法に取り組む際、空間の離散化により、`C`　個のセル（要素）が生成される場合、および　`P`　個の評価点を引数として受け取り、そのランクと次元　`D`　が基底関数のフィールドランク　`F`　に依存する出力を返す基底関数の定義に発生します。


## 問題の形式化

**入力**:
  `inputData(C,P,D,D)` - ランク4ビュー
  `inputField(C,F,P,D)` - ランク4ビュー


**戻し**:
  `outputField(C,F,P,D)` - ランク4ビュー


**計算**: 
  `C,F,P`の各トリプルについて、2つの入力ビューから出力フィールドを計算します:
  
``` c++
 (C,F,P)における各 (c,f,p) について
  積 inputData(c,p,:,:) * inputField(c,f,p,:)　を計算
  outputField(c,f,p,:)　に結果を格納
```

## シリアル実装

``` c++

for (int c = 0; c < C; ++c)
for (int f = 0; f < F; ++f)
for (int p = 0; p < P; ++p)
{

  自動結果 = Kokkos::subview(outputField, c, f, p, Kokkos::ALL);
  自動左   = Kokkos::subview(inputData, c, p, Kokkos::ALL, Kokkos::ALL);
  自動右  = Kokkos::subview(inputField, c, f, p, Kokkos::ALL);
  
  for (int i=0;i<D;++i) {
  
    double tmp(0);
    
    for (int j=0;j<D;++j)
      tmp += left(i, j)*right(j);
    
    result(i) = tmp;
  }
}

```

## Kokkos　を使った並列化

### 初期実装 - `RangePolicy`

上記のシリアルコードを並列化するための、最も直接的な方法は、セルを順次反復する外側の　`for`　ループを、[`RangePolicy`](../API/core/policies/RangePolicy)　を使用した並列　for　ループに変換することです。

``` c++

Kokkos::parallel_for("for_all_cells", 
  Kokkos::RangePolicy<>(0,C),
   KOKKOS_LAMBDA (const int c) {
     for (int f = 0; f < F; ++f)
     for (int p = 0; p < P; ++p)
     {

      自動結果 = Kokkos::subview(outputField, c, f, p, Kokkos::ALL);
      自動左   = Kokkos::subview(inputData, c, p, Kokkos::ALL, Kokkos::ALL);
      自動右  = Kokkos::subview(inputField, c, f, p, Kokkos::ALL);
  
      for (int i=0;i<D;++i) {
  
        double tmp(0);
    
        for (int j=0;j<D;++j)
          tmp += left(i, j)*right(j);
    
        result(i) = tmp;
      }
     }
  });

```


セル数が十分に多く、並列化が有効となる場合、すなわち並列ディスパッチのオーバーヘッドと計算時間の合計が、シリアル実行の総時間よりも短い場合には、上記のシンプルな実装によりパフォーマンスが向上します。

特に、体 `F` と点 `P` に対する for ループ内において、さらに活用できる並列処理の余地がございます。 これを実現する一つの方法は、3つの反復範囲の積である`C*F*P`を取り、その積に対して[`parallel_for`](../API/core/parallel-dispatch/parallel_for)　を実行することです。 ただし、これには抽出ルーチンが必要となります。具体的には、平坦化された反復範囲 `C*F*P` のインデックスと、この例におけるデータ構造が要求する多次元インデックスとの間の対応付けを行う必要があります。 さらに、パフォーマンスの移植性を実現するためには、1次元積分反復範囲と多次元3Dインデックス間のマッピングにアーキテクチャ認識が必要となりますが、これは、Kokkosでデータアクセスパターンを確立するために使用される　[`LayoutLeft`](../API/core/view/layoutLeft)　および[`LayoutRight`](../API/core/view/layoutRight)　の概念に類似したものです。

 [`MDRangePolicy`](../API/core/policies/MDRangePolicy) は、反復範囲の積を手動で計算したり、1次元と3次元の多次元インデックス間のマッピングを行ったりする必要なく、3つの反復範囲すべてに対して並列化を行うという目標を達成するための自然な方法を提供します。[`MDRangePolicy`](../API/core/policies/MDRangePolicy) は、最初の実装である [`RangePolicy`](../API/core/policies/RangePolicy) の使用例で示された内容の通り、きっちりと入れ子になった for ループでの使用に適しており、単一次元での並列化を超える計算における追加の並列性を実現する手法を提供します。

### 実装　- `MDRangePolicy`

``` c++
Kokkos::parallel_for("mdr_for_all_cells", 
  Kokkos::MDRangePolicy< Kokkos::Rank<3> > ({0,0,0}, {C,F,P}),
   KOKKOS_LAMBDA (const int c, const int f, const int p) {
    自動結果 = Kokkos::subview(outputField, c, f, p, Kokkos::ALL);
    自動左   = Kokkos::subview(inputData, c, p, Kokkos::ALL, Kokkos::ALL);
    自動右 = Kokkos::subview(inputField, c, f, p, Kokkos::ALL);
  
    for (int i=0;i<D;++i) {
  
      ダブル tmp(0);
    
      for (int j=0;j<D;++j)
        tmp += left(i, j)*right(j);
    
      result(i) = tmp;
    }
  });
```

## MDRangePolicy使用例

[`MDRangePolicy`](../API/core/policies/MDRangePolicy) は、　[`RangePolicy`](../API/core/policies/RangePolicy) と同じテンプレートパラメータを受け付けますが、また追加の型である `Kokkos::Rank<R>` パラメータを必要とします。そこでは、 `R` がランクであり、 入れ子になった　for-loops　の数であり、コンパイル時に提供される必要があります。 

ポリシーには、以下の2つの引数が必要です:
  1) "begin"　インデックスの、初期化子リスト、または　Kokkos::Array`
  2) "end" インデックスの、初期化子リスト、または　Kokkos::Array`

内部的には、[`MDRangePolicy`](../API/core/policies/MDRangePolicy) は多次元反復空間に対してタイリングを使用します。カスタマイズのため、ポリシーにはオプションの第三引数を渡すことが可能ですーこれはタイル寸法サイズの初期化子リストとなります。 単純なデフォルトサイズは問題に依存する場合があり、自動的に決定することが難しいため、この引数は、パフォーマンスチューニングの際には重要となる可能性があります。

ラムダ関数の署名（またはファンクタのアクセス演算子）は、各ランクごとに引数を必要とします。

[`MDRangePolicy`](../API/core/policies/MDRangePolicy) は、Kokkos の [`parallel_for`](../API/core/parallel-dispatch/parallel_for) および [`parallel_reduce`](../API/core/parallel-dispatch/parallel_reduce) パターン双方で使用可能です。


## 資料

[`MDRangePolicy`](../API/core/policies/MDRangePolicy) の　API　資料は、Kokkos　の　Wiki　でご覧いただけます：
[Wikiリンク](https://github.com/kokkos/kokkos/wiki/Kokkos%3A%3AMDRangePolicy)。
 
この例が基づいている使用事例は、Trilinos の Intrepid2 パッケージに由来するものです。 具体例については、Trilinos　のコードを以下のファイルでご確認ください：`Trilinos/packages/intrepid2/src/Shared/Intrepid2_ArrayToolsDef*.hpp`。

以下のリンクは、Intrepid　パッケージの概要をいくつか紹介しています: 
  [documentation link](https://trilinos.org/packages/intrepid/)。

