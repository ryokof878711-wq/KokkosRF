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

If a functor has an `execution_space` public typedef, a parallel dispatch will only run the functor in that execution space. That's a good way to mark a functor as specific to an execution space. If the functor lacks this typedef, [`parallel_for()`](../API/core/parallel-dispatch/parallel_for) will run it in the default execution space unless you tell it otherwise (that's an advanced topic; see "execution policies"). Lambdas do not have typedefs, so they run on the default execution space unless you tell Kokkos otherwise.

## Parallel for

The most common parallel dispatch operation is a [`parallel_for()`](../API/core/parallel-dispatch/parallel_for) call. It corresponds to the OpenMP construct `#pragma omp parallel for`. [`parallel_for()`](../API/core/parallel-dispatch/parallel_for) splits the index range over the available hardware resources and executes the loop body in parallel. Each iteration is executed independently. Kokkos promises nothing about the loop order or the amount of work which actually runs concurrently. This means in particular that not all loop iterations are active at the same time. Consequently, it is not legal to use wait constructs (e.g., wait for a prior iteration to finish). Kokkos also doesn't guarantee that it will use all available parallelism. For example, it can decide to execute in serial if the loop count is very small, and it would typically be faster to run in serial instead of introducing parallelization overhead. The `RangePolicy` allows you to specify minimal chunk sizes in order to control potential concurrency for low trip count loops.

The lambda or the `operator()` method of the functor takes one argument. That argument is the parallel loop "index." The type of the index depends on the execution policy used for the [`parallel_for()`](../API/core/parallel-dispatch/parallel_for). It is an integer type for the implicit or explicit [`RangePolicy`](../API/core/policies/RangePolicy). The former is used if the first argument to [`parallel_for()`](../API/core/parallel-dispatch/parallel_for) is an integer.

## Parallel reduce

Kokkos' [`parallel_reduce()`](../API/core/parallel-dispatch/parallel_reduce) operation implements a reduction. It is like [`parallel_for()`](../API/core/parallel-dispatch/parallel_for), except that each iteration produces a value and these iteration values are accumulated into a single value with a user-specified associative binary operation. It corresponds to the OpenMP construct `#pragma omp parallel reduction` but with fewer restrictions on the reduction operation.

In addition to the execution policy and the functor, [`parallel_reduce()`](../API/core/parallel-dispatch/parallel_reduce) takes an additional argument which is either the place where the final reduction result is stored (a simple scalar, or a [`Kokkos::View`](../API/core/view/view)) or a reducer argument which encapsulates both the place where to store the final result as well as the type of reduction operation desired (see [Custom Reductions](Custom-Reductions)). 

The lambda or the `operator()` method of the functor takes two arguments. The first argument is the parallel loop "index," the type of which depends on the execution policy used for the [`parallel_reduce()`](../API/core/parallel-dispatch/parallel_reduce). For example: when calling [`parallel_reduce()`](../API/core/parallel-dispatch/parallel_reduce) with a [`RangePolicy`](../API/core/policies/RangePolicy), the first argument to the operator is an integer type, but if you call it with a [`TeamPolicy`](../API/core/policies/TeamPolicy) the first argument is a *team handle*. The second argument is a non-const reference to a thread-local variable of the same type as the reduction result.

When not providing a `reducer` the reduction is performed with a sum reduction using the + or += operator of the scalar type. Custom reduction can also be implemented by providing a functor with a `join` and an `init` function. 

### Example using lambda

Here is an example reduction using a lambda, where the reduction result is a `double`.

```c++
const size_t N = ...;
View<double*> x ("x", N);
// ... fill x with some numbers ...
double sum = 0.0;
// KOKKOS_LAMBDA macro includes capture-by-value specifier [=].
parallel_reduce ("Reduction", N, KOKKOS_LAMBDA (const int i, double& update) {
  update += x(i); 
}, sum);
```

### Example using functor with `join` and `init`.

The following example shows a reduction using the _max-plus semiring_, where `max(a,b)` corresponds to addition and ordinary addition corresponds to multiplication:

```c++
class MaxPlus {
public:
  // Kokkos reduction functors need the value_type typedef.
  // This is the type of the result of the reduction.
  using value_type = double;

  // Just like with parallel_for functors, you may specify
  // an execution_space typedef. If not provided, Kokkos
  // will use the default execution space by default.

  // Since we're using a functor instead of a lambda,
  // the functor's constructor must do the work of capturing
  // the Views needed for the reduction.
  MaxPlus (const View<double*>& x) : x_ (x) {}

  // This is helpful for determining the right index type,
  // especially if you expect to need a 64-bit index.
  using size_type = View<double*>::size_type;

  KOKKOS_INLINE_FUNCTION void
  operator() (const size_type i, value_type& update) const
  { // max-plus semiring equivalent of "plus"
    if (update < x_(i)) {
      update = x_(i);
    }
  }

  // "Join" intermediate results from different threads.
  // This should normally implement the same reduction
  // operation as operator() above.
  KOKKOS_INLINE_FUNCTION void
  join (value_type& dst,
        const value_type& src) const
  { // max-plus semiring equivalent of "plus"
    if (dst < src) {
      dst = src;
    }
  }

  // Tell each thread how to initialize its reduction result.
  KOKKOS_INLINE_FUNCTION void
  init (value_type& dst) const
  { // The identity under max is -Inf.
     dst = reduction_identity<value_type>::max();
  }

private:
  View<double*> x_;
};
```

This example shows how to use the above functor:

```c++
const size_t N = ...;
View<double*> x ("x", N);
// ... fill x with some numbers ...

double result;
parallel_reduce ("Reduction", N, MaxPlus (x), result);
```

### Reductions with an array of results

Kokkos lets you compute reductions with an array of reduction results, as long as that array has a (run-time) constant number of entries. This currently only works with functors. Here is an example functor that computes column sums of a 2-D View.

```c++
struct ColumnSums {
  // In this case, the reduction result is an array of float.
  using value_type = float[];

  using size_type = View<float**>::size_type;

  // Tell Kokkos the result array's number of entries.
  // This must be a public value in the functor.
  size_type value_count;

  View<float**> X_;

  // As with the above examples, you may supply an
  // execution_space typedef. If not supplied, Kokkos
  // will use the default execution space for this functor.

  // Be sure to set value_count in the constructor.
  ColumnSums (const View<float**>& X) :
    value_count (X.extent(1)), // # columns in X
    X_ (X)
  {}

  // value_type here is already a "reference" type,
  // so we don't pass it in by reference here.
  KOKKOS_INLINE_FUNCTION void
  operator() (const size_type i, value_type sum) const {
    // You may find it helpful to put pragmas above this loop
    // to convince the compiler to vectorize it. This is 
    // probably only helpful if the View type has LayoutRight.
    for (size_type j = 0; j < value_count; ++j) {
      sum[j] += X_(i, j);
    }
  }

  // value_type here is already a "reference" type,
  // so we don't pass it in by reference here.
  KOKKOS_INLINE_FUNCTION void
  join (value_type dst,
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

We show how to use this functor here. The results are
stored in the 1D View `sums`.
```c++
const size_t numRows = 10000;
const size_t numCols = 10;

View<float**> X ("X", numRows, numCols);
// ... fill X before the following ...
ColumnSums cs (X);
Kokkos::View<float*> sums ("sums", numCols);
parallel_reduce (X.extent(0), cs, sums);
```

The result view could also use `Kokkos::HostSpace`, in which case
accessing the results on the host requires a fence:

```c++
Kokkos::View<float*, Kokkos::HostSpace> sums ("sums", numCols);
parallel_reduce (X.extent(0), cs, sums);
Kokkos::fence();
std::cout << sums(0) << '\n';

If the number of elements in the reduced array is a compile-time constant,
it is also possible to place the results directly into a C array:
```
float sums[10];
parallel_reduce (X.extent(0), cs, sums);
```

## Parallel scan

Kokkos' [`parallel_scan()`](../API/core/parallel-dispatch/parallel_scan) operation implements a _prefix scan_. A prefix scan is like a reduction over a 1-D array, but it also stores all intermediate results ("partial sums"). It can use any associative binary operator. The default is `operator+` and we call a scan with that operator a "sum scan" if we need to distinguish it from scans with other operators. The scan operation comes in two variants. An _exclusive scan_ excludes (hence the name) the i<sup>th</sup> entry of the input array while computing the i<sup>th</sup> prefix scan and an _inclusive scan_ includes that entry. Given an example array `(1, 2, 3, 4, 5)`, an exclusive sum scan overwrites the array with `(0, 1, 3, 6, 10)`, and an inclusive sum scan overwrites the array with `(1, 3, 6, 10, 15)`.

Many operations that "look" sequential can be parallelized with a scan. To learn more, refer to Guy Blelloch's book<sup>2</sup> (a version of his PhD dissertation).

Kokkos lets users specify a scan by either a functor or a lambda. Both look like their [`parallel_reduce()`](../API/core/parallel-dispatch/parallel_reduce) equivalents, except that the `operator()` method or lambda takes three arguments: the loop index, the "update" value by non const reference, and a `bool`. Here is a lambda example where the intermediate results have type `float`.

```c++
View<float*> x = ...; // assume filled with input values
const size_t N = x.extent(0);
parallel_scan (N, KOKKOS_LAMBDA (const int i,
          float& update, const bool final) {
    // Load old value in case we update it before accumulating
    const float val_i = x(i); 
    if (final) {
      x(i) = update; // only update array on final pass
    }
    // For exclusive scan, change the update value after
    // updating array, like we do here. For inclusive scan,
    // change the update value before updating array.
    update += val_i;
  });
```

Kokkos may use a multiple-pass algorithm to implement scan. This means that it may call your `operator()` or lambda multiple times per loop index value. The `final` Boolean argument tells you whether Kokkos is on the final pass. You must only update the array on the final pass.

For an exclusive scan, change the `update` value after updating the array, as in the above example. For an inclusive scan, change `update` _before_ updating the array. Just as with reductions, your functor may need to specify a non-default `join` or `init` method if the defaults do not do what you want.

***
<sup>2</sup>  Blelloch, Guy, _Vector Models for Data-Parallel Computing_, The MIT Press, 1990.
***

## Function Name Tags

When writing class-based applications it is often useful to make the classes themselves functors. Using that approach allows the kernels to access all other class members, both data and functions. An issue coming up in that case is the necessity for multiple parallel kernels in the same class. Kokkos supports that through function name tags. An application can use optional (unused) first arguments to differentiate multiple operators in the same class. Execution policies can take the type of that argument as an optional template parameter. The same applies to init, join and final functions.

```c++
class Foo {
  struct BarTag {};
  struct RabTag {};

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

This approach can also be used to template the operators by templating the tag classes which is useful to enable compile time evaluation of appropriate conditionals.
