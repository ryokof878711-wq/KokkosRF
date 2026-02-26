# ホストとデバイスの作業の重複

ホスト実行とデバイス実行を並行して実行できるアーキテクチャを使用する場合、Kokkosはホスト操作とデバイス操作のオーバーラッピングをサポートします。これにより、アルゴリズムによっては大幅な速度向上が期待できます。 本使用事例では、デバイス実行とホスト実行のオーバーラッピングを活用するアルゴリズムの条件と設計について説明します。

## アクター
 - 異なるカーネルのセットを用いたアルゴリズムであり、一部のカーネルはホスト上で実行するのが最適であり、一部のカーネルはアクセラレータデバイス上で実行するのがより効果的。
 - 通信またはシリアライゼーション操作を計算カーネルと交互に行うことができるアルゴリズム。
 - リソースの競合なしにホストとデバイス間で作業を分割できるアルゴリズム。
 
## サブジェクト
 - Kokkos 実行空間
 - Kokkos 実行ポリシー
 - Kokkos メモリ空間

## 仮定
 - デバイスカーネルが実行中である間は、ホストがアクセス可能なメモリとの競合はほとんど、あるいは全く発生しません。
    
## 制約
 - カーネルは、ノンブロッキングです。
    
## 前提条件
 - C++ファンクタの形式で実装されている　"実行カーネル"。
    
## 使用例パターン1 - 重複する計算カーネル

```
|--- デバイスおよびホストメモリを割り当て
|--- デバイスおよびホストメモリを初期化
|------------------------------------------------
|- ホスト演算 0　を実行
|--- 反復処理 -----------------------------
    |->----------- グローバルバリア -----------------
    |  |- ホストおよびデバイスデータを同期
    |  |- デバイス演算 N \　を実行
    |  |- ホスト演算 N+1 / 非同期　を実行
    |-<--------------------------------------------
```

## コード例
デバイスが反復処理 n において、演算を実行している間、反復処理 n+1 に必要なホストデータのセットアップを実行します。

```c++
型定義 ダブル value_type;
型定義 Kokkos::OpenMP   HostExecSpace;
型定義 Kokkos::Cuda     DeviceExecSpace;
型定義 Kokkos::RangePolicy<DeviceExecSpace>  device_range_policy;
型定義 Kokkos::RangePolicy<HostExecSpace>    host_range_policy;
型定義 Kokkos::View<double*, Kokkos::CudaSpace>   ViewVectorType;
型定義 Kokkos::View<double**, Kokkos::CudaSpace>  ViewMatrixType;

// ホスト上のデータをセットアップ
// 反復処理間の変動性の論証のため、パラメータ xVal を使用
void init_src_views(ViewVectorType::HostMirror p_x,
                  ViewMatrixType::HostMirror p_A,
                  const value_type xVal ) {
    
  Kokkos::parallel_for( "init_A", host_range_policy(0,N), [=] ( int i ) {
    for ( int j = 0; j < M; ++j ) {
      h_A( i, j ) = 1;
    }
  });

  Kokkos::parallel_for( "init_x", host_range_policy(0,M), [=] ( int i ) {
    h_x( i ) = xVal;
  });
}
  
ViewVectorType y( "y", N );
ViewVectorType x( "x", M );
ViewMatrixType A( "A", N, M );

ViewVectorType::HostMirror h_y = Kokkos::create_mirror_view( y );
ViewVectorType::HostMirror h_x = Kokkos::create_mirror_view( x );
ViewMatrixType::HostMirror h_A = Kokkos::create_mirror_view( A );
  
for ( int repeat = 0; repeat < nrepeat; repeat++ ) {
  init_src_views( h_x, h_A, repeat+1);  // 次回のデバイス起動に向けた設定データの準備
  
  Kokkos::fence(); // デバイスとホスト間でデータをコピーする前に同期するために使用するバリア
    
  // この反復処理において必要となる、デバイスへのホストデータのディープコピー
  Kokkos::deep_copy( h_y, h );
  Kokkos::deep_copy( x, h_x );
  Kokkos::deep_copy( A, h_A );  // implicit barrier

  // Application: y=Ax
  Kokkos::parallel_for( "yAx", device_range_policy( 0, N ), 
                              KOKKOS_LAMBDA ( int j ) {
    double temp2 = 0;
    for ( int i = 0; i < M; ++i ) {
      temp2 += A( j, i ) * x( i );
    }

    y( j ) = temp2;
  } );
    
  // なお、ここにはバリアが存在しないことに注意。
  //　したがって、カーネルがまだ実行中の間、ホストスレッドはループして　ini_src_views　を呼び出す。
}
```

**重要注意事項**:  理論上、ホストカーネルとデバイスカーネルの起動順序は重要ではありませんが、実際には、最初にデバイスカーネルを起動する必要があります。 ほとんどのホストバックエンドは、カーネルが動作している間、"メイン"　スレッドを空いたままにしません。 ホスト並列カーネルが起動されると、そのスレッドがカーネルへの貢献を完了するまで、メインスレッドは占有された状態となります。 デバイス実行は異なるコンテキストで行われるため、カーネル起動直後にはホストスレッドは解放されます。 並列実行パターンに関連する契約についても
注意を払う必要があります。 パターンが、完了前に同期化を必要とする場合（例えば、リダクションなど）、ホストとデバイスの操作を
重複させる機会はありません。 したがって、ホスト／デバイスのオーバーラッピングパターンを活用するには、アルゴリズム全体の修正が必要となる場合があります。

## Usage pattern使用例パターン2 - デバイスがカーネルを実行している間ホスト上でシリアル化された演算を実行

```
|--- デバイスおよびホストメモリを割り当て
|--- デバイスおよびホストメモリを初期化
|---------- グローバルバリア  ------------------------------
|- ホストおよびデバイスデータを同期
|--------------------------------------------------------
    |->|- デバイス演算 N を実行        \ 非同期
    |  |- ホストデータをNからディスクへシリアル化 /
    |  |------------ グローバルバリア -----------------------
    |  |- N+1の開始に向け、ホストとデバイスのデータを同期
    |-<----------------------------------------------------
```

ディスクにシリアル化されたデータは1回の反復分遅れていますが、デバイス操作と非同期で実行することが可能です。 N+1のデバイスデータは
反復Nの後にコピーされるため、同期の前にバリアが必要となります。

## コード例

```c++
型定義 Kokkos::RangePolicy<>    range_policy;
型定義 Kokkos::View<double*>    ViewVectorType;

ViewVectorType V_r;
ViewVectorType V_r1;
ViewVectorType::HostMirror h_V = Kokkos::create_mirror_view( y );

get_initial_state(h_V); // function to initialize V on host

Kokkos::deep_copy(V_r, h_V);
Kokkos::deep_copy(V_r1, h_V)

for (int r = 0; r < R; r++) {
  
  Kokkos::parallel_for(range_policy(0,N), KOKKOS_LAMBDA (int i) {
     V_r1(i) = get_RHS_func(V_r);  //return V_r1(i) for RHS from V_r
  });

  serialize_state(h_V); // ホスト　view_r　にまだ存在するデータをシリアライズ
 
  Kokkos::fence();  // ホストおよびデバイス間を同期
 
  Kokkos::deep_copy(h_V, V_r1);  // update for next iteration
  Kokkos::deep_copy(V_r, h_V);
     
}
```

