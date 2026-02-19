# 階層的並列処理

本章では、Kokkos　を使用して複数のレベルの共有メモリ並列性を活用する方法について説明します。　これらのレベルには、スレッドチーム、チーム内のスレッド、およびベクトルレーンが含まれます。 これらの並列処理レベルをネストし、[`parallel_for()`](../API/core/parallel-dispatch/parallel_for)　および　[`parallel_scan()`](../API/core/parallel-dispatch/parallel_scan)　を実行できます。各レベルで　[`parallel_reduce()`](../API/core/parallel-dispatch/parallel_reduce)　または　[`parallel_scan()`](../API/core/parallel-dispatch/parallel_scan)　を実行できます。 構文は実行ポリシーのみが異なり、これは`parallel_*`　演算の最初の引数です。Kokkos　は、また "scratch pad" メモリを公開しており、スレッド固有およびチーム固有の割り当てを提供します。

## モチベーション

現代の高性能コンピュータにおけるノードアーキテクチャは、ますます高度化する _階層的並列処理_　によって特徴づけられます。
階層内のレベルは、そのレベルにある演算ユニット間で共有されるハードウェアリソースによって決定されます。
階層の上位レベルは、その階層の下位レベルにあるブランチ内の全リソースにもアクセス権を持ちます。
この概念は、異質性の概念とは直交する。例えば、典型的なCPUベースのクラスターにおけるノードは、複数のマルチコア　CPU　で構成されます。 各コアは1つ以上のハイパースレッドをサポートし、各ハイパースレッドはベクトル命令を実行できます。 これは、並列性の階層構造に4つのレベルがあることを意味します:

1. CPU　ソケットは同じメモリおよびネットワークリソースへのアクセスを共有し、
1. ソケット内のコアは通常、共有の最終レベルキャッシュ（LLC）を備えており、
1. 同一コア上のハイパースレッドは共有L1（およびL2）キャッシュにアクセス可能であり、同じ実行ユニットに命令を送信し、そして
1. ベクトル演算ユニットは、複数のデータ項目に対して共有命令を実行します。

GPU　ベースのシステムも、4レベルの階層構造を持ちます:

1. 同一ノード内の複数　GPU　が、同じホストメモリとネットワークリソースへのアクセスを共有し、
1. GPU　コアクラスタ（例：NVIDIA GPU　上の　SM　）は共有キャッシュを持ち、単一　GPU　上で同じ高帯域メモリにアクセスし、
1. 同一のコアクラスタ上で実行されるスレッドは、同じ　L1　キャッシュとスクラッチメモリにアクセス可能であり、そして
1. それらは、いわゆるワープまたは波面内にグループ化され、その内部ではスレッドは常に同期しており、例えば直接的なレジスタ交換を通じて、より緊密に連携できます。

Kokkos　は、複数の抽象化された並列処理レベルを提供しますが、それらを適切なハードウェア機能にマッピングします。 このマッピングは必ずしも静的または事前定義されたものではなく、カーネルごとに異なる場合があります。さらに、一部のマッピング決定は実行時に行われます。これにより、ワークセットのサイズに応じて作業を、異なるハードウェアリソースにマッピングする適応型カーネルが、有効となります。Kokkos　はデフォルト値と提案を提供する一方、最適なマッピングはアルゴリズムに依存する可能性があります。階層的並列性には、実行ポリシーを通じて、アクセス可能である。

特に、以下のケースでは階層的並列処理を使用する必要があります:

1. 非密接ネストループ：階層的並列処理により、より多くの並列性を引き出せます。
1. データ収集と再利用：外側のループの特定の反復でデータを収集し、それを内側のループで繰り返し使用する場合、スクラッチメモリを用いた階層的並列処理がそのユースケースに適している可能性があります。
1. 階層的並列処理を利用すると、開発者はキャッシュブロッキングに適したアルゴリズム選択を迫られ、これにより、単純なフラット並列処理よりも優れたパフォーマンスを発揮するアルゴリズムが得られる場合があります：

一方、ネストループが密接に存在する場合は、階層的並列処理を使用すべきではないでしょう。そのようなユースケースには、多次元範囲ポリシーの方が適しています

(HP_thread_teams)=
## スレッドチーム

Kokkos　の最も基本的な階層的並列処理の概念はスレッドチームです。 _thread team_　とは、同期化が可能で、 "scratch pad" メモリを共有するスレッドの集合体です　( [Section 7.3](Team_scratch_pad_memory)　参照)。

Kokkos　のスレッドチームは、1次元のインデックス範囲をハードウェアリソースにマッピングする代わりに、2次元のインデックス範囲をマッピングします。 最初のインデックスは、_league rank_　、つまり _team rank_　を示すインデックスです。 CUDA　では、これは1次元のブロックの1次元グリッドを起動することに相当します。 リーグの規模は任意です――つまり、整数サイズの型によってのみ制限されますが―一方で、チームの規模はハードウェアの制約に収まる必要があります。 CUDAと同様に、同時に実際にアクティブなチームは限られており、新しいチームが実行される前に、それらは完了まで実行されなければなりません。したがって、他のスレッドチームによって開始されたイベントを待機するといった、スレッドチーム間の同期メカニズムを使用することは有効ではありません。

### ポリシーインスタンスの作成

Kokkosは、　[`Kokkos::TeamPolicy`](../API/core/policies/TeamPolicy)　実行ポリシーを使用してスレッドチームの使用を明示します。  スレッドチームを使用するには、　[`Kokkos::TeamPolicy`](../API/core/policies/TeamPolicy)　のインスタンスを作成する必要があります。並列ディスパッチ呼び出しに対して、インラインで作成できます。 コンストラクタは2つの引数（リーグサイズとチームサイズ）を必要とします：チームサイズの代わりに、　`Kokkos::AUTO`　を使用することで、　既定のアーキテクチャに適したチームサイズを推測させることができます。そうすることが、[`TeamPolicy`](../API/core/policies/TeamPolicy)を利用するための、ほとんどの開発者によって推奨される方法です。 [`Kokkos::RangePolicy`](../API/core/policies/RangePolicy)の場合と同様に、特定の実行タグ、特定の実行空間、`Kokkos::IndexType`、および`Kokkos::Schedule`をオプションのテンプレート引数として指定できます。

```c++
// デフォルトの実行スペースを使用し、起動
// それぞれ team_size スレッドを持つ league_size teams を伴うリーグ
Kokkos::TeamPolicy<>
        policy( league_size, team_size );

// Using  a specific execution space to特定実行を使用
// チームサイズを選択する　n_workset　並列処理を、Kokkos を使用して実行
Kokkos::TeamPolicy<ExecutionSpace>
        policy( league_size, Kokkos::AUTO() );

// 特定の実行スペースと実行タグを使用
Kokkos::TeamPolicy<SomeTag, ExecutionSpace>
        policy( league_size, team_size );
```

### 基本カーネル

チームポリシーの　`member_type`　は、並列カーネル内でチームを使用するために必要な機能を提供します。リーグ順位や規模、およびチーム順位および規模等のスレッド識別子へのアクセスを可能にします。 また、チームバリア、削減、スキャンなどのチーム同期アクションも提供します。

```c++
Kokkos::atomic_add　を使用;
Kokkos::PerTeam　を使用;
Kokkos::Sum　を使用;
Kokkos::TeamPolicy　を使用;
Kokkos::parallel_for　を使用;

typedef TeamPolicy<ExecutionSpace>::member_type member_type;
// ポリシーのインスタンスを作成
TeamPolicy<ExecutionSpace> policy (league_size, Kokkos::AUTO() );
// カーネルを起動
parallel_for (policy, KOKKOS_LAMBDA (member_type team_member) {
    // Calculate a global thread idグローバルスレッドIDを計算
    int k = team_member.league_rank () * team_member.team_size () +
            team_member.team_rank ();

    // このチームのグローバルスレッドIDの合計を計算
    int team_sum = k;
    team_member.team_reduce(Sum<int, typename ExecutionSpace::memory_space>(team_sum));

    // 計算された合計をグローバル値にアトミック演算で加算
    Kokkos::single (PerTeam (team_member), [=] () {
      atomic_add(&global_value(), team_sum);
    });
  });
```

 [`TeamPolicy`](../API/core/policies/TeamPolicy) という名称は、これを使用するカーネルがチームに対して並列領域を構成することを明示的に示しています。

チームメンバー間の作業調整を可能にするため、つまり一部のスレッドが値を計算し、グローバルメモリに保存した後、誰もがそれを消費する仕組みを実現するために、チームはバリアを提供します。これらの障壁は、同じチーム内の全チームメンバーにとって共通のものですが、他のチームとは関係がありません。こ子にその一例があります:

```c++
Kokkos::TeamPolicy　を使用;
Kokkos::parallel_for　を使用;

typedef TeamPolicy<ExecutionSpace>::member_type member_type;
// ポリシーのインスタンスを作成
TeamPolicy<ExecutionSpace> policy (league_size, Kokkos::AUTO() );
// カーネルを起動
parallel_for (policy, KOKKOS_LAMBDA (member_type team_member) {
    // 各チームのThread 0　は、間接的にデータを収集。
    if( team_member.team_rank() == 0 ) {
      a(team_member.league_rank()) = b(indices(team_member.league_rank()));
    }
    //各チームメンバーが更新を待つためのバリアを作成
    team_member.team_barrier();

    // すべてのチームメンバーが、a を利用可能
    c(team_member.league_rank(),team_member.team_rank()) = a(team_member.league_rank();
  });
```

(Team_scratch_pad_memory)=
## チームスクラッチパッドメモリ

各Kokkos　チームには "scratch pad"　が存在します。　これは、そのチーム内のスレッドのみがアクセス可能なメモリ空間のインスタンスです。 スクラッチパッドは、アルゴリズムがワークセットを共有スペースにロードし、その後チームの全メンバーと共同で作業することを可能にします。 スクラッチパッド内のデータの存続期間は、チームの存続期間です。特に、スクラッチメモリは、同一の物理コア群上で動作する全ての論理チームによって再利用されます。チームの存続期間中、グローバルメモリで許可される全ての操作はスクラッチメモリでも許可されるます。 チームの存続期間中、グローバルメモリで可能になるすべての演算は、スクラッチメモリでも可能になります。 これには、アドレスの取得およびスクラッチ領域にある要素に対するアトミック演算の実行が含まれます。 チームレベルのスクラッチパッドは、Cudaにおけるブロックごとの共有メモリ、あるいはCellプロセッサ上の「ローカルストア」メモリに対応します。

Kokkos は、実行スペースに関連付けられた特別なメモリ空間を通じてスクラッチパッドを公開します: `execution_space::scratch_memory_space`。  [`TeamPolicy`](../API/core/policies/TeamPolicy) メンバ型を通じてスクラッチメモリの一部を割り当てることができます。ユーザー指定の最大合計サイズまで、最初から複数回の割り当てを要求できます。 最大値は、ファンクタ内の　`team_shmem_size`　関数によって提供されますが、この関数はチームサイズに依存する可能性のある値を返します。あるいは、TeamPolicy　の　`set_scratch_size`　設定を通じて指定することも可能です。 両方の値を同時に指定することは無効です。TeamPolicy　への引数は、ファンクタを使用する際の共有メモリサイズを設定するために使用できます。 共有メモリ割り当てに関する制約の一つは、チームの存続期間中に解放できない点です。 これが、メモリプールの複雑さの回避および、割り当て取得にかかる時間をの短縮につながります（現在、オフセット計算には、数十回の整数演算が必要です）。

以下は、ファンクターインターフェースの使用例です:

```c++
template<class ExecutionSpace>
struct functor構造体ファンクタ {
  typedef ExecutionSpace execution_space;
  typedef execution_space::member_type member_type;

  KOKKOS_INLINE_FUNCTION
  void operator() (member_type team_member) const {
    size_t double_size = 5*team_member.team_size()*sizeof(double);

    // スクラッチパッド上で共有チームの割り当てを取得
    double* team_shared_a = (double*)
      team_member.team_shmem().get_shmem(double_size);

    // スクラッチパッド上で別の割り当てを取得
    int* team_shared_b = (int*)
      team_member.team_shmem().get_shmem(160*sizeof(int));

    // ... スクラッチ割り当てを使用 ...
  }

  // 共有メモリ容量を提供。
  // この関数は、引数として team_size を選択し、
  // それにより、team_size　に応じた割り当てが可能になります
  size_t team_shmem_size (int team_size) const {
    return sizeof(double)*5*team_size +
           sizeof(int)*160;
  }
};
```
[`TeamPolicy`](../API/core/policies/TeamPolicy) の `set_scratch_size` 関数は、2つまたは3つの引数を取ります。 最初の引数は、特定のサイズが要求されるスクラッチ階層内のレベルを指定します。 様々なレベルには、様々名制約があります。 一般的に、第1レベルは数十キロバイト程度に制限され、おおむねL1キャッシュサイズに対応します。 第2レベルは、数ギガバイト規模の全チームに対する集計値を取得するために使用でき、これは高帯域メモリの空き領域に対応します。 第3レベルは、主にノード内の容量メモリに依存します。 2番目と3番目の引数は、スクラッチメモリのスレッド単位またはチーム単位のサイズです。

ここにいくつかの例があります:

```c++
TeamPolicy<> policy_1 = TeamPolicy<>(league_size, team_size).
                          set_scratch_size(1, PerTeam(1024), PerThread(32));
TeamPolicy<> policy_2 = TeamPolicy<>(league_size, team_size).
                          set_scratch_size(1, PerThread(32));
TeamPolicy<> policy_3 = TeamPolicy<>(league_size, team_size).
                          set_scratch_size(0, PerTeam(1024));
```

各チームが利用できるスクラッチスペースの総量は、チームごとの値にスレッドごとの値を乗じた値にチームサイズを乗じた値となります。 このインターフェイスでは、ユーザーがそれらの設定を直接指定することが可能です:

```c++
parallel_for(TeamPolicy<>(league_size, team_size).set_scratch_size(1, PerTeam(1024)),
  KOKKOS_LAMBDA (const TeamPolicy<>::member_type& team) {
    ...
});
```

メモリ内で単純に生の割り当てを取得する代わりに、ユーザーはスクラッチメモリ内に直接ビューを割り当てることもできます。 これは、View　コンストラクタの第1引数として共有メモリハンドルを渡すことで実現されます。 ビューには、共有メモリのサイズ要件を返す静的メンバ関数も備わっています。 その関数は実行時次元を引数として想定し、これは　ビューコンストラクタに対応します。 Note that the view must be unmanaged (i.e. it must have the `Unmanaged` memory trait).ビューは、管理対象外（つまり、`Unmanaged`　メモリ特性を持つ必要がある）でなければならないことに注意してください。

```c++
型定義 Kokkos::DefaultExecutionSpace::scratch_memory_space
  ScratchSpace;
// Define a view type in ScratchSpace
型定義 Kokkos::View<int*[4],ScratchSpace,
          Kokkos::MemoryTraits<Kokkos::Unmanaged>> shared_int_2d;

// 共有メモリ割り当てのサイズを取得
size_t shared_size = shared_int_2d::shmem_size(team_size);
Kokkos::parallel_for(Kokkos::TeamPolicy<>(league_size,team_size).
                       set_scratch_size(0,Kokkos::PerTeam(shared_size)),
                     KOKKOS_LAMBDA ( member_type team_member) {
  // チーム共有メモリ内でビューを割り当て。
  // コンストラクタが、共有メモリハンドルおよび
  // 実行時次元を選択
  shared_int_2d A(team_member.team_scratch(0), team_member.team_size());
  ...
});
```

## ネスト並列処理

リーグやチームの順位インデックスを明示的に使用するコードを書く代わりに、ネスト並列処理を用いて階層的アルゴリズムを実装することが可能です。 Kokkos はユーザーが最大3層のネストされた並列処理を可能にします。 チームレベルとスレッドレベルが、最初の2つのレベルです。第3レベルは、  _vector_ 並列処理です。

各レベル　<sup>1</sup>　において、3つの並列パターン（for、reduce、scan）のいずれかを使用できます。
それらをネストしたり、リーグやチームの順位を意識したコードと連携させて使用できます。 異なるレイアは特別な実行ポリシーを介してアクセス可能です: `TeamThreadLoop` および `ThreadVectorLoop`.

***
<sup>1</sup> スレッドレベルでは、すべての実行空間に対して並列スキャン演算が実装されているわけではなく、最上位レベルでのTeamPolicy　もサポートしていません。
***

### チームループ

最初のネストレベルの並列ループは、チームの各スレッド間でインデックス範囲を分割します。 これが、This motivates the policy name ポリシー名　[`TeamThreadRange`](../API/core/policies/TeamThreadRange)　を動かし、 それは、ループがチームによって1回実行され、インデックス範囲がスレッド間で分割されることを示しています。 ループ回数はチーム内のスレッド数に制限されず、インデックス範囲がスレッドにマッピングされる方法はアーキテクチャに依存します。 [`TeamThreadRange`](../API/core/policies/TeamThreadRange) ポリシーを使用して複数の並列ループをネストすることは許可されていません。 しかしながら、[`TeamThreadRange`](../API/core/policies/TeamThreadRange) ポリシーを使用した複数の並列ループが、同じカーネル内で、順番に連続して実行されることは有効です。 ネスト並列層のクロージャ外で、POD　データへの書き込みアクセスを行うことは認められないことに注意してください。 これは、スレッドプライベート変数、チーム共有変数、およびグローバル共有変数に関連するデバッグが困難な問題を防止するための意識的な選択です。これを強制する簡単な方法は、ラムダ式で　 "capture by value"　句を使用することですが、
ただし、通常、パフォーマンスが向上するため、リリースビルドでは、"capture by reference"　が推奨されます。
ラムダ式が　[`TeamThreadRange`](../API/core/policies/TeamThreadRange)　ループ内で`const`と見なされるため、コンパイラは、コンパイル時に　`const`　違反として不正なアクセスを検出します。

The simplest use case is to have another [`parallel_for()`](../API/core/parallel-dispatch/parallel_for) nested inside a kernel.最も単純な使用例は、カーネル内に別の　[`parallel_for()`](../API/core/parallel-dispatch/parallel_for)　をネストさせることです。

```c++
Kokkos::parallel_for　を使用;
Kokkos::TeamPolicy　を使用;
Kokkos::TeamThreadRange　を使用;

parallel_for (TeamPolicy<> (league_size, team_size),
                    KOKKOS_LAMBDA (member_type team_member)
{
  Scalar tmp;
  parallel_for (TeamThreadRange (team_member, loop_count),
    [=] (int& i) {
      // ...
      // tmp += i; // This would be an illegal access
    });
});
```

 [`parallel_reduce()`](../API/core/parallel-dispatch/parallel_reduce)  構文は、最適化されたチームレベルの削減を実行するために使用できます:

```c++
Kokkos::parallel_reduce　を使用;
Kokkos::TeamPolicy　を使用;
Kokkos::TeamThreadRange　を使用;
parallel_for (TeamPolicy<> (league_size, team_size),
                 KOKKOS_LAMBDA (member_type team_member) {
    // デフォルトの削減では、スカラーの += 演算子を使用して
    // スレッドの貢献度を結合します。
    Scalar sum;
    parallel_reduce (TeamThreadRange (team_member, loop_count),
      [=] (int& i, Scalar& lsum) {
        // ...
        lsum += ...;
      }, sum);

    // ここでスレッドを同期させるためのチームバリアを導入
    team_member.team_barrier();

    // カスタムの縮約をファンクタとして提供できます。。
    // Kokkos　が提供するものの1つ（例：Prod<Scalar>）を含みます。
    スカラー積;
    Scalar init_value = 1;
    parallel_reduce (TeamThreadRange (team_member, loop_count),
      [=] (int& i, Scalar& lsum) {
        // ...
        lsum *= ...;
      }, Kokkos::Experimental::Prod<Scalar>(product);
  });
```
カスタムリダクションでは、Kokkos　が認識するファンクター結合パターンのいずれかを使用する必要があることに、注意してください; これらには、`Sum, Prod, Min, Max, LAnd, LOr, BAnd, BOr, ValLocScalar, MinLoc, MaxLoc, MinMaxScalar, MinMax, MinMaxLocScalar` および `MinMaxLoc`　が含まれます。

第3のパターンは、プレフィックススキャン実行に使用可能な　[`parallel_scan()`](../API/core/parallel-dispatch/parallel_scan) です。

#### チームバリア

あるループ演算を別のループ演算と順序付けする必要が生じる場合（例えば、後続の計算のための準備段階として配列を埋める場合など）、スレッドを時間的に制御できることが重要です; これは、バリアの使用によって実現可能です。 ネストループでは、外側のループ（[`TeamPolicy<> ()`](../API/core/policies/TeamPolicy)）に組み込み（暗示的）チームバリアが存在します; 内側のループ ( [`TeamThreadRange ()`](../API/core/policies/TeamThreadRange) ) do not.には、存在しません。 この後者の条件は、多くの場合に、'non-blocking' 条件と呼ばれます。 必要に応じて、チームスレッドを同期化するための明示的なバリアを導入できます；その一例は前の例で示されています。 

### ベクターループ

カーネル内のネストされた並列ループの最内層は、_vector_　ループで構成されます。ベクトルレベルの並列処理は、実行ポリシー[`ThreadVectorRange`](../API/core/policies/ThreadVectorRange)　を使用するチームレベルループと同一の動作をします。 チームレベルとは対照的に、[`ThreadVectorRange`](../API/core/policies/ThreadVectorRange)　を使用した並列パターン以外で、ベクターレベルを合法的に利用する方法はありません。 ただし、このような並列構造は、[`TeamThreadRange`](../API/core/policies/TeamThreadRange) 並列演算の内外を問わず、使用できます。

```c++
Kokkos::parallel_reduce　を使用;
Kokkos::TeamPolicy　を使用;
Kokkos::TeamThreadRange　を使用;
Kokkos::ThreadVectorRange　を使用;
parallel_for (TeamPolicy<> (league_size, team_size),
                 KOKKOS_LAMBDA (member_type team_member) {

    int k = team_member.team_rank();
    // デフォルト還元では、スカラーの += 演算子を使用して
    // レッドの貢献度を結合します。
    Scalar sum;
    parallel_reduce (ThreadVectorRange (team_member, loop_count),
      [=] (int& i, Scalar& lsum) {
        // ...
        lsum += ...;
      }, sum);

    parallel_for (TeamThreadRange (team_member, workset_size),
      [&] (int& j) {
      // カスタムの還元をファンクタとして提供できます。
      // Kokkos　が提供するもののいずれかを含めることも可能です。例：Prod<Scalar>。
      スカラー積;
      Scalar init_value = 1;
     parallel_reduce (ThreadVectorRange (team_member, loop_count),
        [=] (int& i, Scalar& lsum) {
          // ...
          lsum *= ...;
        }, Kokkos::Experimental::Prod<Scalar>(product);
      });
  });
```

名前が示す通り、ベクトルレベルはベクトル化可能でなければなりません。 並列パターンは、コンパイラによるベクトル化を促進するために利用可能なメカニズムを活用します。 例えば、インテルコンパイラを使用する場合、ベクトルレベルループは内部的に　`#pragma ivdep`　で装飾され、コンパイラに想定されるベクトル依存関係を無視するよう指示します。

### 実行を単一の実行者に制限

前述の通り、カーネルはチーム内のスレッド（およびベクトルレーン）に対して並列領域です。 これは、それぞれのネストレベルの外側にあるグローバルメモリアクセスが、繰り返し実行から保護される必要がある可能性があることを意味します。 よくある例として、チームが計算を実行するが、チームごとに1つの結果のみをグローバルメモリに書き戻す必要がある場合が考えられます。.

このケースでは、Kokkos　は`Kokkos::single(Policy,Lambda)`　関数を提供します。それは現在、以下の2つのポリシーを、受け入れています:

* 　`Kokkos::PerTeam` はラムダの本体実行をチームごとに1回に制限します
* 　`Kokkos::PerThread` はラムダ式の実行をスレッドごとに1回（つまり、スレッド内のベクトルレーン1つに限定）に制限します。

`single`　関数は、第二引数としてラムダ式を受け取ります。このラムダ式は引数を全く取らないか、参照渡しで1つの引数を取ります。そのラムダ式は、引数を全く取らないか、参照による1つの引数を取ります。引数を全く取らない場合、その本体は効果を発揮するために副作用を実行しなければなりません。 引数を1つ取る場合、その引数の最終的な値はレベル上のすべての実行者にブロードキャストされます: つまり、スレッドのベクトルレーンごと、またはチームの各スレッド（およびベクトルレーン）。 ラムダ式が変数を値渡しでキャプチャする場合（`[=]`、`[&]`ではない）、常に正しくある必要があります。 したがって、ラムダ式が参照によってキャプチャする場合、参照によってキャプチャした変数を変更しては _なりません_ 。

```c++
Kokkos::parallel_for　を使用;
Kokkos::parallel_reduce　を使用;
Kokkos::TeamThreadRange　を使用;
Kokkos::ThreadVectorRange　を使用;
Kokkos::PerThread　を使用;

TeamPolicy<...> policy (...);
型定義 TeamPolicy<...>::member_type team_member;

parallel_for (policy, KOKKOS_LAMBDA (const team_member& thread) {
 // ...

  parallel_for (TeamThreadRange (thread, 100),
    KOKKOS_LAMBDA (const int& i) {
      double sum = 0;
      // スレッドを使って、ベクター還元を実行
      parallel_reduce (ThreadVectorRange (thread, 100),
        [=] (int i, double& lsum) {
          // ...
          lsum += ...;
      }, sum);
      // Add the result value into a team shared array.
      // Make sure it is only added once per thread.
      Kokkos::single (PerThread (), [=] () {
          shared_array(i) += sum;
      });
  });

  double sum;
  parallel_reduce (TeamThreadRange (thread, 99),
    KOKKOS_LAMBDA (int i, double& lsum) {
      // Add the result value into a team shared array.
      // Make sure it is only added once per thread.
      Kokkos::single (PerThread (thread), [=] () {
          lsum += someFunction (shared_array(i),
                                shared_array(i+1));
      });
  }, sum);

  // Add the per team contribution to global memory.
  Kokkos::single (PerTeam (thread), [=] () {
    global_array(thread.league_rank()) = sum;
  });
});
```

Here is an example of using the broadcast capabilities to determine the start offset for a team in a buffer:

```c++
using Kokkos::parallel_for;
using Kokkos::parallel_reduce;
using Kokkos::TeamThreadRange;
using Kokkos::ThreadVectorRange;
using Kokkos::PerThread;

TeamPolicy<...> policy (...);
typedef TeamPolicy<...>::member_type team_member;

Kokkos::View<int> offset("Offset");
offset() = 0;

parallel_for (policy, KOKKOS_LAMBDA (const team_member& thread) {
  // ...

  parallel_reduce (TeamThreadRange (thread, 100),
    KOKKOS_LAMBDA (const int& i, int& lsum) {
      if(...) lsum++;
  });
  Kokkos::single (PerTeam (thread), [=] (int& my_offset) {
   my_offset = Kokkos::atomic_fetch_add(&offset(),lsum);
  });
  ...
});
```

To further illustrate the "parallel region" semantics of the team execution consider the following code:

```c++
using Kokkos::parallel_reduce;
using Kokkos::TeamThreadRange;
using Kokkos::TeamPolicy;

parallel_reduce(TeamPolicy<>(N,team_size),
  KOKKOS_LAMBDA (const member_type& teamMember, int& lsum) {
    int s = 0;
    for(int i = 0; i<10; i++) s++;
    lsum += s;
},sum);
```

In this example `sum` will contain the value `N * team_size * 10`. Every thread in each team will compute `s=10` and then contribute it to the sum.

Let's go one step further and add a nested [`parallel_reduce()`](../API/core/parallel-dispatch/parallel_reduce). By choosing the loop bound to be `team_size` every thread still only runs once through the inner loop.

```c++
using Kokkos::parallel_reduce;
using Kokkos::TeamThreadRange;
using Kokkos::TeamPolicy;

parallel_reduce(TeamPolicy<>(N,team_size),
  KOKKOS_LAMBDA (const member_type& teamMember, int& lsum) {

  int s = 0;
  parallel_reduce(TeamThreadRange(teamMember, team_size),
    [=] (const int k, int & inner_lsum) {
    int inner_s = 0;
    for(int i = 0; i<10; i++) inner_s++;
    inner_lsum += inner_s;
  },s);
  lsum += s;
},sum);
```

The answer in this case is nevertheless `N * team_size * team_size * 10`. Each thread computes `inner_s = 10`. But all threads in the team combine their results to compute a `s` value of `team_size * 10`. Since every thread in each team contributes that value to the global sum, we arrive at the final value of `N * team_size * team_size * 10`. If the intended goal was for each team to only contribute `s` once to the global sum, the contribution should have been protected with a `single` clause.
