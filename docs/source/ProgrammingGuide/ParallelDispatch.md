# 並列ディスパッチ

おそらく、Kokkosがどのようにコードの並列化を実現するのかを学びたいと考えられたので、本ガイドを読み始めたのでしょう。 本章では、Kokkos　が実行可能な各種の並列演算について解説いたします。 Kokkos　が特定の実行空間による実行のためにそれらを　"ディスパッチ"　するため、これらの操作を総称して、 _並列ディスパッチ_　と呼びます。 Kokkos　は、3つの異なる並列演算を提供します:

* [`parallel_for()`](../API/core/parallel-dispatch/parallel_for) 独立した反復処理を持つ "forループ" を実装します。
* [`parallel_reduce()`](../API/core/parallel-dispatch/parallel_reduce) 還元を実装。
* [`parallel_scan()`](../API/core/parallel-dispatch/parallel_scan) プレフィックススキャンを実施。

Kokkos　は、並列ループの本体を定義する際に、ユーザーに2つの選択肢を提供します: ファンクタおよびラムダ。 また、ユーザーは　_実行ポリシー_　を指定することで、並列操作の実行方法を制御することも可能です。 後述の章では、ネストされた並列処理を可能にするより高度な実行ポリシーについて解説いたします。

構文に関する重要な注意事項:

* Kokkos　が並列で呼び出すファンクタのメソッドには、`KOKKOS_INLINE_FUNCTION`　マクロを使用して、明示的にマークしてください
* `KOKKOS_LAMBDA` マクロを使用し、ラムダ式を Kokkos に並列実行のために渡す際に、ラムダのキャプチャ句を置き換えてください

## 並列ループボディを指定

### ファンクタ

 _ファンクタ_ は、並列ループの本体を定義する方法の一つです。 それは、パブリックな `operator()` インスタンスメソッドを持つクラスまたは　struct<sup>1</sup> です。 そのメソッドの引数は、実行したい並列操作（for、reduce、または　scan）、およびループの実行ポリc（例えば、範囲またはチーム）の両方に依存します。ファンク他の一例については、 各並列操作の種類についての、本章の該当セクションを参照してください。 最も一般的なケースである　[`parallel_for()`](../API/core/parallel-dispatch/parallel_for)　では、for　ループのインデックスである、整数型の引数を取ります。これはとなります。他の引数である可能性もあります; [Chapter 8 - Hierarchical Parallelism](HierarchicalParallelism)　を参照してください。

`operator()`　メソッドは、const　でなければならず、`KOKKOS_FUNCTION`　または　`KOKKOS_INLINE_FUNCTION`　マクロでマークされなければなりません。一部のバックエンド (CUDA および　HIP等) については、このマクロは、ご自身のメソッドがアクセラレータデバイスとホストの両方で実行可能であることを示すために必要です。 マークアップを必要とするバックエンドでビルドしない場合、`KOKKOS_INLINE_FUNCTION` は `inline` に展開され、そｓて and `KOKKOS_FUNCTION` は不要ですが、害はありません。 そのようなメソッドのシグネチャの例を以下に示します：

```c++
KOKKOS_INLINE_FUNCTION void operator() (...) const;
```

並列操作全体（for、reduce、または scan）は、同じファンクタのインスタンスを共有します。 ただし、`operator()`メソッド内で宣言された変数は、その並列ループの反復内でのみ有効です。Kokkos　は、コードを実行する実行領域に対して、ファンクタインスタンスをポインタや参照ではなく、"コピー，によって渡すことがあります。 特に、ファンクターはホストとは異なる実行空間にコピーする必要がある場合があります。 Kokkos ビューもコピーで渡してください。これは浅いコピーとして機能します。 ファンクタも、const　オブジェクトとして渡されますので、ファンクタのメンバを変更することはできません。 (ただし、ファンクターが、例えばビューやファンクターのメンバーである生の配列といった内容を変更することは有効です。)

***
<sup>1</sup>　C++ 内の  "構造体" は、ほんの一例であり、そのメンバーのすべてが、デフォルトで公開されています。
***

### ラムダ

2011年版のC++標準　("C++11")　では、新しい言語構文であるラムダ式（_ラムダ_）が提供されています。Kokkos　では、並列ループ本体をファンクタ（上記参照）またはラムダ式として指定することが可能です。 ラムダ関数は、自動的に生成されるファンクタのように機能します。 クラスと同様に、ラムダ式にも状態が存在する可能性があります。 唯一の違いは、ラムダ関数の場合、状態が環境から渡される点です。（　"closure" という名称は、関数が環境から状態を "closes over"　ことを意味します。）ファンクタによる場合と同様に、ラムダ式は状態を "value" で（コピーによって）取り込む必要があり、参照やポインタによって取り込むことはできません。 

デフォルトでは、ラムダ式は何もキャプチャしません（デフォルトのキャプチャ指定子 `[]` が示す通りです）。 [`parallel_for()`](../API/core/parallel-dispatch/parallel_for) は、一般的にその副作用によって動作するため、これはおそらく役に立たないでしょう。 Kokkos では閉鎖のコピーを作成する権利を留保しており、その動作は非同期となる可能性があるため、意味論的に正しい動作を実現するには、``capture by value'' を行う必要があります。 最外層の並列化については、KOKKOS_LAMBDA マクロを介して行うことをお勧めいたします（詳細は[第8章]　(HierarchicalParallelism)　を参照してください）。
一部のバックエンドでは、これは通常のcapture-by-value　句 `[=]` に置き換わります。 それは周囲のスコープから変数を値渡しで取得します。 参照渡しで取得しないでください！　その他のバックエンドについては (例えば、 CUDA および　HIP)、 このマクロには、`KOKKOS_INLINE_FUNCTION`　マクロ同様に、ラムダ式を正しく動作させるようにする、特別な定義が存在する可能性があります。 

参照渡し `[&]` を用いることは、二つの理由から Kokkos のセマンティクスに違反します。 第一の理由は、Kokkos　が、ラムダを、ディスパッチングスレッドのスタックにアクセスできない実行空間に渡す可能性があることです。第二の理由は、参照渡しによる取得により、プログラマーはラムダのconstセマンティクスを破ることができるということです。 正確性と移植性の観点から、ラムダ式およびファンクタは並列コードセクション内で　const　オブジェクトとして扱われます。 参照渡しによる取得は、その　const　プロパティを回避することを可能にし、スレッドセーフでないコードを書く可能性をさらに多く生み出します。

ラムダ式を用いたネストされた並列処理（詳細は、[第8章](HierarchicalParallelism)　を参照してください。）において、参照による取得は、パフォーマンス上の理由から、有用な場合がありますが、そのコードは、コピーによるキャプチャでも動作する場合にのみ、Kokkosコードとして有効です。

### ファンクタまたはラムダを使用するべきでしょうか？

Kokkosでは、ファンクタとラムダのどちらを使用するかを選択できます。 ラムダ式は短いループ本体に便利です。 より複雑なループ本体については、テストを容易にするために、それを分離してファンクタとして名前を付ける方が良いかもしれません。 ラムダ関数は、定義上　"無名関数"、であり、つまり名前を持たないものです。これにより、それらのテストを行うことが難しくなります。 さらに、 CUDA　でラムダ式をご利用になる場合には、十分に新しいバージョンのCUDAが必要です。 書き込み時点において、CUDA 7.5以降のバージョンでは、特別なフラグを用いたホスト-デバイスラムダ式をサポートしております。 CUDA 8.0では、ホストコンパイラとの相互運用性が向上しました。このサポートを有効にするためには、`KOKKOS_CUDA_OPTIONS=enable_lambda` オプションをご使用ください

最後に、"実行タグ"　機能は、複数の異なる並列ループ本体を、単一のファンクタにまとめることができますが、ファンクタでのみ動作します。  (詳細についは、[第8章](HierarchicalParallelism) 参照してください。)

### 実行空間の指定

ファンクタに、 `execution_space` パブリック型定義がある場合には、 並列ディスパッチでは、その実行空間内でのみファンクタが動作します。 その方法は、ファンクタを特定の実行領域に固有のものとしてマークする良い方法です。 このファンクタに、この　`型定義`　が指定されていない場合、[`parallel_for()`](../API/core/parallel-dispatch/parallel_for)　は、特に指定がない限り、デフォルトの実行空間でこれを実行します（これは高度なトピックです；"実行ポリシー"　を参照してください）。 ラムダ式には、型定義がありません。そのため、Kokkosに別の指定をしない限り、デフォルトの実行空間で動作します。

## 並列for

最も一般的な並列ディスパッチ演算は、 [`parallel_for()`](../API/core/parallel-dispatch/parallel_for) 呼び出しです。 それは、OpenMP　の構文　`#pragma omp parallel for`に対応します。 [`parallel_for()`](../API/core/parallel-dispatch/parallel_for) は、インデックス範囲を利用可能なハードウェアリソースに分割し、ループ本体を並列に実行します。各反復処理は独立して実行されます。 Kokkos は、ループの順序や実際に同時に実行される作業量について、一切保証しません。 これは、特にすべてのループ反復が同時に有効であるわけではないことを意味します。　したがって、待機構造（例：前の反復処理が終了するのを待つ）を使用することは、法的に認められていません。 Kokkosは、利用可能な並列処理をすべて使用することを保証するものではありません。　例えば、 ループ回数が非常に少ない場合、シリアル実行を選択することが可能であり、また、並列化のオーバーヘッドを導入するよりも、シリアルで実行する方が通常は高速となります。 実行回数の少ないループにおける潜在的な並列処理を制御するために、The `RangePolicy` `RangePolicy` の使用が、最小チャンクサイズの指定を可能にします。

ラムダ式またはファンクタの`operator()`　メソッドは、1つの引数を取ります。 その引数は、並列ループの　"インデックス"　です。インデックスの型は、[`parallel_for()`](../API/core/parallel-dispatch/parallel_for) で使用される実行ポリシーによって異なります。 それは、暗示的または明示的なimplicit or explicit [`RangePolicy`](../API/core/policies/RangePolicy).　の整数型です。  [`parallel_for()`](../API/core/parallel-dispatch/parallel_for) への最初の引数は、整数である場合に、前者が使用されます。

## 並列リデュース

Kokkos　の [`parallel_reduce()`](../API/core/parallel-dispatch/parallel_reduce) 演算は、還元を実装します。  各反復処理では値が生成され、これらの反復値はユーザーが指定した連想二項演算によって単一の値に集約される場合を除いて、 [`parallel_for()`](../API/core/parallel-dispatch/parallel_for)　の様なものです。 これは、OpenMP　の構文　`#pragma omp 並列演算`に対応しますが、還元演算に対する制限が少なくなっています。

実行ポリシーとファンクタに加えて、 [`parallel_reduce()`](../API/core/parallel-dispatch/parallel_reduce) は、最終的な集約結果を格納する場所（単純なスカラー、または`Kokkos::View` (../API/core/view/view)) であるか、または最終結果の保存場所と希望する還元演算の型の両方をカプセル化したリデューサー引数である追加の引数を取ります。 ( [Custom Reductions](Custom-Reductions)　を参照してください)。

ファンクタのラムダ式または　`operator()`　メソッドは、2つの引数を取ります。 第一の引数は、並列ループの　"インデックス"　であり、その型は　[`parallel_reduce()`](../API/core/parallel-dispatch/parallel_reduce)　に使用される実行ポリシーによって異なります。　例えば: [`RangePolicy`](../API/core/policies/RangePolicy)　で [`parallel_reduce()`](../API/core/parallel-dispatch/parallel_reduce) を呼び出す場合、 the first argument to the operator演算への第一の引数は、整数型ですが、もし  [`TeamPolicy`](../API/core/policies/TeamPolicy) でそれを呼び出した場合には、第一の引数は、 *チームハンドル*　です。 第二引数は、還元結果と同じ型のスレッドローカル変数への、非　const　参照です。

`リデューサ`を指定しない場合、スカラー型の　`+`　または　`+=`　演算子を用いた、合計による還元処理が行われます。 カスタム還元はまた、 `join` および  `init` 関数をファンクタに提供することにより、実装可能です。 

### ラムダ使用例

以下にラムダを使用した還元例を示します。ここでは、還元結果は、 `ダブル`　です。

```c++
const size_t N = ...;
View<double*> x ("x", N);
// ...  x　をいくつかの数で満たします ...
double sum = 0.0;
// KOKKOS_LAMBDA マクロは、 capture-by-value 指定子 [=]　を含みます。
parallel_reduce ("還元", N, KOKKOS_LAMBDA (const int i, double& update) {
  update += x(i); 
}, sum);
```

### `join` および `init`を伴うファンクタ使用例

以下の例は、_max-plus半環_ を用いた還元を示しており、ここでは、`max(a,b)`は加法に対応し、通常の加法は乗法に対応しますThe following example shows a reduction using the, where `max(a,b)` corresponds to addition and ordinary addition corresponds to multiplication:

```c++
クラス MaxPlus {
public:
  // Kokkos の還元ファンクタには、value_type の型定義が必要です
  // これは還元結果の型です。
  using value_type = double　を使用;

  // parallel_forファンクタと同様に
  // 　execution_space の型定義を指定することができます。指定がない場合、
  // Kokkos はデフォルトでデフォルトの実行スペースを使用します。

  // ラムダ式の代わりに、ファンクタを使用しているため
  // ファンクタのコンストラクタは、
  // 還元に必要なビューをキャプチャする作業を必ず行わなければなりません。
  MaxPlus (const View<double*>& x) : x_ (x) {}

  // これは適切なインデックスの種類を決定するのに役立ちます
  // 特に、ビットインデックスを必要とすることが予測される場合
  using size_type = View<double*>::size_type　を使用;

  KOKKOS_INLINE_FUNCTION void
  operator() (const size_type i, value_type& update) const
  { // max-plus semiring equivalent of "plus"
    if (update < x_(i)) {
      update = x_(i);
    }
  }

  // 異なるスレッドからの中間結果を"結合"。
  // 通常、これは上記の　operator()　と同じ演算を
  // 実装する必要があります。
  KOKKOS_INLINE_FUNCTION void
  結合 (value_type& dst,
        const value_type& src) const
  { // "プラス"　に相当する最大プラス半環
    if (dst < src) {
      dst = src;
    }
  }

  // 各スレッドに対し、その還元結果を初期化する方法を指示。
  KOKKOS_INLINE_FUNCTION void
  init (value_type& dst) const
  { //  最大値の下での値は、-Inf。
     dst = reduction_identity<value_type>::max();
  }

プライベート:
  View<double*> x_;
};
```

この例は、上記のファンクターの使用方法を示します:

```c++
const size_t N = ...;
View<double*> x ("x", N);
// ... fill x with some numbers ...

double 結果;
parallel_reduce ("Reduction", N, MaxPlus (x), result);
```

### 結果の配列を用いた削減

Kokkosでは、(実行時の)定数個の要素を持つ配列であれば、その配列を用いて還元結果の計算を行うことが可能です。 以下に、2次元ビューの列の合計を計算するファンクタの例を示します。

```c++
構造体 ColumnSums {
  // この場合、還元結果は浮動小数点数の配列となります。
  using value_type = float[] を使用;

  using size_type = View<float**>::size_type　を使用;

  // Kokkos に結果配列の要素数を伝達します。
  // これはファンクタ内のパブリック値である必要があります。
  size_type value_count;

  View<float**> X_;

  // 上記の例により、 execution_space　型定義を
  // 行うことができます。 定義されない場合には、 
  // Kokkos　は、このファンクタのデフォルト実行空間を使用します。

  // コンストラクタ内で、必ず　value_count　を設定してください。
  ColumnSums (const View<float**>& X) :
    value_count (X.extent(1)), // # columns in X
    X_ (X)
  {}

  // ここでの　value_type は、既に "参照" 型ですので、
  // ここでは参照によっては、それを渡しません。
  KOKKOS_INLINE_FUNCTION void
  operator() (const size_type i, value_type sum) const {
    // このループの上部にプリプロセッサディレクティブを記述すると、
    // コンパイラにベクトル化を促すのに役立つかもしれません。  
    // おそらく、この機能はビューのタイプに　LayoutRight　が設定されている場合にのみ有効です。
    for (size_type j = 0; j < value_count; ++j) {
      sum[j] += X_(i, j);
    }
  }

  // ここでの　value_type は、既に "参照" 型ですので、
  // ここでは参照によっては、それを渡しません。
  KOKKOS_INLINE_FUNCTION void
  結合 (value_type dst,
        const value_type src) const {
    for (size_type j = 0; j < value_count; ++j) {
      dst[j] += src[j];
    }
  }

  KOKKOS_INLINE_FUNCTION void init (value_type sum) const {
    for (size_type j = 0; j < value_count; ++j) {
      sum[j] = 0.0;
    }
  }
};
```

ここでは、このファンクターの使い方を説明します。 結果は、
1次元ビュー `sums` に保存されます。
```c++
const size_t numRows = 10000;
const size_t numCols = 10;

View<float**> X ("X", numRows, numCols);
// ... 以下の前に　X を満たします ...
ColumnSums cs (X);
Kokkos::View<float*> sums ("sums", numCols);
parallel_reduce (X.extent(0), cs, sums);
```

結果ビューでは、　`Kokkos::HostSpace`　を使用することも可能です。その場合、
ホスト上で結果にアクセスするにはフェンスが必要となります:

```c++
Kokkos::View<float*, Kokkos::HostSpace> sums ("sums", numCols);
parallel_reduce (X.extent(0), cs, sums);
Kokkos::fence();
std::cout << sums(0) << '\n';

還元された配列の要素数がコンパイル時の定数である場合、
結果をC言語の配列に直接格納することも可能です：
```
フロート　sums[10];
parallel_reduce (X.extent(0), cs, sums);
```

## 並列スキャン

Kokkos　の　`parallel_scan()`　演算は、_プレフィックススキャン_　を実装します。 プレフィックススキャンは、1次元配列に対する還元演算に似ていますが、中間結果（　"部分和"　）をすべて保存する点も特徴であります。それは、任意の結合性を持つ二項演算子を使用できます。デフォルトは、 `operator+` であり、他の演算子を用いたスキャンと区別する必要がある場合、その演算子を用いたスキャンを、"和スキャン"　と呼びます。スキャン演算には、２種類あります。 _エクスクルーシブスキャン_ は、入力配列のi<sup>th</sup>　入力を計算対象から除外し（名称の通り）、一方では、_インクルーシブスキャン_では、その要素を含みます。 例の配列 `(1, 2, 3, 4, 5)`　を前提とすれば、 an エクスクルーシブサムスキャンは、scan overwrites the array with `(0, 1, 3, 6, 10)`　を使った配列を上書きし、インクルーシブサムスキャンは、`(1, 3, 6, 10, 15)`　を使った配列を上書きします。　

 "一見" 連続的であるような演算の多くは、走査処理によって並列化することが可能です。 詳細については、Guy Blelloch氏の著書　<sup>2</sup>（博士論文を基にした書籍）を参照してください。

Kokkos　では、関数型またはラムダ式によってスキャンを指定することが可能です。 `operator()`　メソッドまたはラムダ式が3つの引数を取る点が異なりますことを除いて:具体的には、ループインデックス、非const参照による「更新」値、そして`bool`型、どちらも[`parallel_reduce()`](../API/core/parallel-dispatch/parallel_reduce)　の対応する関数のように見えます。
以下に、中間結果は、`浮動小数点`　型となるラムダの例を示します。

```c++
View<float*> x = ...; // 入力値が入力されていると仮定
const size_t N = x.extent(0);
parallel_scan (N, KOKKOS_LAMBDA (const int i,
          float& update, const bool final) {
    // 蓄積する前に更新した場合に備え、古い値を読み込みます
    const float val_i = x(i); 
    if (final) {
      x(i) = update; // only update array on final pass
    }
    // エクスクルーシブスキャンについては、
    // ここで行った通りに、配列更新後、更新値を変えます。インクルーシブスキャンについては、
    // 配列更新前に、更新値を変えます。
    更新 += val_i;
  });
```

Kokkos は、スキャンを実装するために複数パスアルゴリズムを使用する場合があります。 これは、ループのインデックス値ごとに、`operator()` またはラムダ式が複数回呼び出される可能性があることを意味します。 `最終`ブール引数は、Kokkos　が最終パス中であるかどうかを示します。 配列の更新は、最終パスでのみ行う必要があります。

エクスクルーシブスキャンについては、上記例にある通り、配列　`更新`　後、　`更新`　値を変えます。 インクルーシブスキャンについては、 配列　`更新` _前_ に、更新値を変えます。 同様に、還元により、デフォルト値が、望むものでない場合には、ファンクタが、非デフォルト `結合`　または　`init` メソッドを指定する必要がある場合があります。

***
<sup>2</sup>  Blelloch, Guy, _データ並列計算のベクトルモデル_, The MIT 出版, 1990.
***

## 関数ネームタグ

クラスベースのアプリケーションを作成する際には、クラス自体をファンクターとして扱うことがしばしば有用です。 の方法を用いることで、カーネルは他のすべてのクラスメンバー（データと関数の両方）にアクセスすることが可能となります。 その場合の問題点として、同一クラス内に複数の並列カーネルが必要となる点が挙げられます。 Kokkos は、関数ネームタグを通じてそれをサポートしております。 アプリケーションは、同じクラス内の複数の演算子を区別するために、オプション（未使用）の第一引数を使用することができます。 実行ポリシーは、その引数の型をオプションのテンプレートパラメータとして指定できます。 init関数、join関数、final関数についても同様です。

```c++
class Foo {
  構造体 BarTag {};
  構造体 RabTag {};

  void compute() {
     Kokkos::parallel_for(RangePolicy<BarTag>(0, 100), *this);
     Kokkos::parallel_for(RangePolicy<RabTag>(0, 1000), *this);
  }

 KOKKOS_FUNCTION
  void operator() (BarTag, const int i) const {
    ...
    foobar();
    ...
  }

  KOKKOS_FUNCTION
  void operator() (RabTag, const int i) const {
    ...
    foobar();
    ...
  }

  KOKKOS_FUNCTION
  void foobar() {
    ...
  }
};
```

この手法は、タグクラスをテンプレート化することで演算子をテンプレート化するのにも利用可能であり、これにより、適切な条件式をコンパイル時に評価できるようになる点が有用です。
