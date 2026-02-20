# 相互運用性とレガシーコード

Kokkos の目標の一つは、レガシーアプリケーションにおける段階的な導入を支援することです。 これにより段階的な変換が可能となり、機能性（および一定の範囲内での）パフォーマンスの継続的なテストを実現します。　この特徴の一つは、基盤となるバックエンドプログラミングモデルとの完全な相互運用性です。 これにより、最大性能を達成するために、バックエンドモデルに直接記述されたターゲット特化型の最適化も可能となります。

この章を読んだ後、以下のことを理解する必要があります:

* 生の　OpenMP　および　Cuda　との相互運用性に関する制限。
* 外部データ構造の処理方法。
* レガシーデータ構造を段階的に変換する方法。
* 非Kokkosサードパーティライブラリを安全に呼び出す方法。

本章のすべてのコード例では、`Kokkos`　名前空間のすべてのクラスが、作業名前空間にインポートされているものと仮定します。

## OpenMP、C++スレッド、および　CUDA　の相互運用性

Kokkos　の実装は、C++　ライブラリによって実現されているため、基盤となるバックエンドプログラミングモデルとの完全な相互運用性を提供します。特に、同一コンパイル単位内でOpenMP、CUDA、Kokkosコードの混在を可能にします。 これは、Kokkos　の並列実行レイヤーとデータレイヤーの両方に当てはまります

このことにより、特定の制限を解除するわけではないことを認識することが重要です。　例えば、OpenMP　並列領域内でビューを割り当てることは無効です。これは、[`parallel_for()`](../API/core/parallel-dispatch/parallel_for)　カーネル内でビューを割り当てるのが無効であるのと同様です。  実際に、モデルを組み合わせる際には、やや煩雑になることがあります。Cuda　カーネル内および　OpenMP　並列領域内で、あるビューを別のビューに割り当てることは、宛先ビューが管理対象外ビューである場合にのみ、可能です。 [`parallel_for()`](../API/core/parallel-dispatch/parallel_for)　によるカーネルのディスパッチ中に、 ファンクタまたはラムダ式で参照されるすべてのビューは、自動的に管理対象外モードに切り替わります。これは、 単にOpenMP並列領域に入るだけでは、このようなことは起こりません。

### Cuda　相互運用性

Cuda 相互運用性において最も重要な点は、提供されるマクロ `KOKKOS_INLINE_FUNCTION` が `__host__ __device__ inline` として評価されることです。これは、純粋な `__device__` 関数（例えば Cuda イントリンシックやライブラリのデバイス関数）を呼び出す際には、`__CUDA_ARCH__` プラグマで保護する必要があることを意味します。

```c++
__device__ SomeFunction(double* x) {
  ...
}

構造体　ファンクタ {
  型定義　Cuda execution_space;
  View<double*,Cuda> a;
  KOKKOS_INLINE_FUNCTION
 void operator(const int& i) {
    #ifdef __CUDA_ARCH__
    int block = blockIdx.x;
    int thread = threadIdx.x;
    if (thread == 0)
      SomeFunction(&a(block));
    #endif
  }
}
```

[`RangePolicy`](../API/core/policies/RangePolicy) は、1次元スレッドブロックの1次元グリッドを開始します。これにより、インデックス `i` は `blockIdx.x * blockDim.x + threadIdx.x` として計算されます。 `TeamPolicy` では、チーム数がグリッド次元となり、一方、チームごとのスレッド数は Cuda スレッドブロックの Y 座標軸（Y-dimension）に対応します。 オプションのベクトル長は、X軸にマッピングされます。 例えば、`TeamPolicy<Cuda>(100,12,16)` はブロック次元 (16,12,1) のサイズ 100 の 1D グリッドを開始し、`TeamPolicy<Cuda>(100,96)` はブロック次元 (1,96,1) のグリッドサイズ 100 を生成します。 ベクトル長に対する制限（Cuda　実行空間では2の冪乗かつ32未満）により、ベクトルループは単一のワープに属するスレッドによって実行されることが保証されます。

### OpenMP

OpenMP相互運用性における制限事項の一つは、Kokkos　の初期化後に　`omp_set_num_threads()`　を用いてスレッド数を増加させることは無効であるという点です。 この制限は、Kokkosがスレッドごとの内部データ構造の割り当てを行う際の帳簿管理のために必要です。 ただし、OpenMP　実行空間向けにコンパイルされた　Kokkos　並列カーネル内でスレッド　ID　を要求することは有効です。 並列カーネル内または、それによって呼び出される関数内で、OpenMP　アトミックなどの　OpenMP　構文を使用することも有効です。 ただし、それらが必ずしも同じ基盤メカニズムにマッピングされるとは限らないので、OpenMP　と　Kokkos　アトミックを混在させた場合の結果は未定義です。


## レガシィデータ構造

レガシーデータ構造との相互運用性を促進する主なメカニズムは、2つあります: 

1. Kokkos　は、レガシーデータ構造を作成するために抽出されるデータと生のポインタを割り当て、そして 
1. 管理対象外ビューにより、外部で割り当てられたデータを参照することができます。

いずれの場合も、Kokkos　ビューのレイアウトをレガシーデータ構造で使用されている実際のレイアウトに固定することが必須です。 ユーザーは、適切なアクセス権限を保証する責任があることに注意してください。 例えば、`CudaSpace`内のビューから取得したポインタは、Cudaカーネルからのみアクセス可能であり、また、`new`　への呼び出しを通じて取得されたメモリから構築されたビューは、通常、`HostSpace`　にアクセス可能な実行空間からのみアクセス可能です。 

## Kokkos　を介した生の割り当て

レガシーアプリに複数のメモリ空間のサポートを追加する簡単な方法は、[`kokkos_malloc`](../API/core/c_style_memory_management/malloc)、 [`kokkos_free`](../API/core/c_style_memory_management/free) および [`kokkos_realloc`](../API/core/c_style_memory_management/realloc) を使用することです。 関数は、メモリ空間に対してテンプレート化されており、これによりデータ構造のターゲットを絞った配置が可能となります:

```c++
// デフォルトのメモリ空間に、100個の　double　型要素からなる配列を割り当て
double* a = (double*) kokkos_malloc<>(100*sizeof(double));

// Cuda UVM　メモリ空間に、150個の　int*　配列を割り当て。
// この割り当てはホストからアクセス可能
int** 2d_array = (int**) kokkos_malloc<CudaUVMSpace>
                         (150*sizeof(int*));

// ポインタ配列をCuda空間内のデータへのポインタを充当
// UVM空間ではないため、2d_array[i][j]　は、Cuda　カーネル内でのみアクセス可能
for(int i=0;i<150;i++)
  2d_array[i] = (int*) kokkos_malloc<CudaSpace>(200*sizeof(int));
```

この機能の一般的な使用シナリオは、GPU向けにコンパイルする際に、CudaUVMSpace　内の全メモリを割り当てることです。 これにより、すべての割り当てが、レガシーコードセクションからだけでなく、Kokkos　で書き込まれた並列カーネルからもアクセス可能になります。

### External memory management

メモリが、外部で管理される場合、例えば、Kokkos　がデータ割り当てのポインタを入力として与えられるライブラリで使用されるので、 データを　Kokkos　のビューにラップすることが必要または便利な場合があります。 ライブラリがコピー作成用のデータを受け取った場合、内部データ構造をビューとして割り当て、並列カーネル内でデータを要素ごとにコピーすることは容易です。 実際の宛先メモリ空間にコピーする前に、まずホストビューにコピーする必要がある場合があることに、注意してください:

```c++
template<class ExecutionSpace>
void MyKokkosFunction(double* a, const double** b, int n, int m) {
  // ホスト実行空間とビュータイプを定義
  型定義 HostSpace::execution_space host_space;
  型定義 View<double*,ExecutionSpace> t_1d_device_view;
  型定義 View<double**,ExecutionSpace> t_2d_device_view;

  // ビューを　ExecutionSpace のメモリ空間に割り当て
  t_1d_device_view d_a("a",n);
  // そのビューのホストコピーを作成
  typename t_1d_device_view::HostMirror h_a = create_mirror_view(a);
  // 外部割り当てからホストビューへデータをコピー
  parallel_for(RangePolicy<host_space>(0,n),
    KOKKOS_LAMBDA (const int& i) {
    h_a(i) = a[i];
  });
  // ホストビューからデバイスビューへデータをコピー
  deep_copy(d_a,h_a);

  // ExecutionSpace　のメモリ空間内に2Dビューを割り当て
  t_2d_device_view d_b("b",n,m);
  // そのビューのホストコピーを作成
  typename t_2d_device_view::HostMirror h_b = create_mirror_view(b);

  // チームポリシーの member_type を取得
  typedef TeamPolicy<host_space>::member_type t_team;
  // TeamPolicyを使用して2Dコピーカーネルを実行
  parallel_for(TeamPolicy<host_space>(n,m),
    KOKKOS_LAMBDA (const t_team& t) {
    const int i = t.team_rank();
    parallel_for(TeamThreadRange(t,m), [&] (const int& j) {
      h_b(i,j) = b[i][j];
    });
  });
  // ホストからデバイスへデータをコピー
  deep_copy(d_b,h_b);
}
```

あるいは、外部割り当てを直接参照するビューを作成することもできます。 そのデータが多次元ビューである場合、レイアウトを明示的に指定することが重要です。さらに、すべてのデータは同じ割り当ての一部でなければなりません。

```c++
void MyKokkosFunction(int* a, const double* b, int n, int m) {
  // ホスト実行空間とビュータイプを定義
  型定義 View<int*, DefaultHostExecutionSpace, MemoryTraits<Unmanaged>> t_1d_view;
  型定義 View<double**[3],LayoutRight, DefaultHostExecutionSpace,
               MemoryTraits<Unmanaged>> t_3d_view;
  // Unmanaged views cannot have labels管理対象外のビューのラベル保持は不可能

  // Create a 1D view of the external allocation外部割り当ての1Dビューを作成
  t_1d_view d_a(a,n);

  // 2番目の外部割り当ての3Dビューを作成
  // これにおいては、データが行主記憶配置（すなわち、3番目のインデックスがストライド1）であったことが前提
  t_3d_view d_b(b,n,m);
}
```

### 基本データ所有構造体としてのビュー

別の選択肢として、Kokkos　に　Views　を用いた基本的な割り当て処理を任せ、その上に従来のデータ構造を構築する方法があります。 再度、ビューのレイアウトをレガシーデータのレイアウトに固定することが重要です。

```c++
// 行主記憶方式で、2Dビューを割り当てる
View<double**,LayoutRight,HostSpace> a("A",n,m);

// ポインタの配列を割り当てる
double** a_old = new double*[n];

// 配列に　a　の行へのポインタを格納
for(int i=0; i<n; i++)
  a_old[i] = &a(i,0);
```

### std::vector

C++　コードで最も一般的なデータオブジェクトの一つは、`std::vector`　です。 そのセマンティクスは、残念ながら、Kokko　の要件と互換性がなく、したがって十分にサポートされていません。 ファンクターとラムダ式が、並列実行に対して　const　オブジェクトとして渡されることが、主な問題です。A major problem is that functors and lambdas are passed as const objects to the parallel execution. 本デザインは以下のために選択されました (i) 競合状態の一般的な原因を防止、および (ii) ファンクタを共有する方法や配置場所について、基盤となる実装の対応を柔軟化。 特に、これにより、実装側では各スレッドにファンクタの個別のコピーを割り当てるか、読み取り専用キャッシュ構造体に配置するかの選択肢が残されます。

この場合、`std::vector`のセマンティクスにより、コンストラクタはエントリを変更できなくなります。なぜなら、const `std::vector`は読み取り専用だからであるからです。 さらに、ファンクターの複数のコピーを作成すると、コピーセマンティクスを持つため、ベクトルデータが実際に複製されることになります。

`std::vector`　のその他の問題点は、サイズ変更およびプッシュ機能に対する制限のないサポートです。スレッド環境では、それらの機能をサポートすると大幅なパフォーマンスの低下を招くことになります。　特に、 `std::vector`へのアクセスには、あるスレッドがその内容を参照している間に、別のスレッドがビューを解放するのを防ぐために、ロックが必要となります。 そして、最後に重要な点は、`std::vector`　がGPU　ではサポートされておらず、移植性を妨げる要因となることです。

Kokkosは、　`Kokkos::vector`　により　`std::vector`　のドロップイン置換を提供します。 並列カーネル以外では、そのセマンティクスは、主に　`std::vector`　と同じです; 例えば、割り当ては深部コピーを実行し、サイズ変更とプッシュ機能を提供します。重要な相違点の一つは、const　ベクトルの要素に値を代入することの有効性です。

並列セクション内で、`Kokkos::vector`はビューセマンティクスへ切り替えます。 具体的には、代入は浅いコピーであることを意味します。 特定の関数は、並列カーネル内で呼び出された場合にも実行時エラーをスローします; これには、サイズ変更およびプッシュが含まれます。

```c++
// 1000個の　double　要素からなるベクトルを作成
Kokkos::vector<double> v(1000);
//  v　のコピーとして、別のベクトルを作成;
// これにより、もう1000個の double を割り当て
Kokkos::vector<double> x = v;

parallel_for(1000, KOKKOS_LAMBDA (const int& i) {
   //  x　のビューを作成; m および x は、同じデータを参照。
   Kokkos::vector<double> m = x;
   x[i] = 2*i+1;
   v[i] = m[i] - 1;
});

// 現在、x　は、最初の1000の奇数を含有
// v　は、最初の1000の偶数を含有。
```

## 非 Kokkos ライブラリの呼び出し

並列カーネル外では、非Kokkosライブラリを呼び出すことに制限はありません。　しかしながら、 Kokkos　ビューの多態的なレイアウト構造のため、サードパーティ製ライブラリとの互換性を確認するために、レイアウトのテストが必要となる場合が多くなります。 例えば、通常のBLASインターフェースは、行列が列主形式（　Kokkos　では　LayoutLeft　）で配置されていることを想定しています。さらに、ライブラリがビューのメモリ空間にアクセスできることをテストする必要があります。

```c++
template<class Scalar, class Device>
スカラー　dot(View<const Scalar* , Device> a,
           View<const Scalar*, Device> b) {
// メモリを確認し、真の場合は　cublas　を呼び出す
#ifdef KOKKOS_HAVE_CUDA
  if(std::is_same<typename Device::memory_space,
                           CudaSpace>::value ||
     std::is_same<typename Device::memory_space,
                           CudaUVMSpace>::value) {
    返し call_cublas_dot(a.ptr_on_device(), b.ptr_on_device(),
                           a.extent(0) );
  }
#endif

// それ以外の場合、ホスト上でCBlasを呼び出す
  if(std::is_same<typename Device::memory_space,HostSpace>::value) {
    return call_cblas_dot(a.ptr_on_device(), b.ptr_on_device(),
                          a.extent(0) );
  }
}
```
