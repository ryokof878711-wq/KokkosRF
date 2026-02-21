# プログラミングモデル

プログラミングモデル「Kokkos」は、以下の6つのコア抽象化によって特徴づけられます: 実行空間,
実行パターン、実行ポリシー、メモリ空間、メモリレイアウト、およびメモリ特性。 これらの抽象化概念により、汎用的なアルゴリズムやデータ構造を構築することが可能となり、それらを様々な種類のアーキテクチャに適用することが可能となります。 実質的に、これらはアルゴリズムのコンパイル時変換を可能にし、ハードウェアの並列性の程度やメモリ階層構造に応じて適応させることを可能にします。

![abstractions](figures/kokkos-abstractions-doc.png)

<h4>図 3.1  プログラミングモデルのコア抽象化　</h4>

## 実行空間

実行空間とは、コードが実際に実行される場所 _Where_ を指します。 例えば、現在のハイブリッド　GPU/CPU　システムでは、実行スペースとして2種類が存在します：　GPU　コアおよび　CPU　コアです。 将来的には、メモリ内処理（PIM）モジュールや、ヘテロジニアス　CPU　上の異なるコアタイプなどが含まれる可能性があります。 原則として、これはリモートメモリ空間の導入にも利用可能です。例えば、異なるノードへ作業を送信する機能などが挙げられます。 実行空間は、アプリケーション開発者が異種混在ハードウェアアーキテクチャの異なる部分をターゲットとする手段を提供します。 これは、以前に説明した機械モデルに直接対応しております。

## 実行パターン

実行パターンは、その中でアプリケーションが表現されなければならない、 _基本的パラレルアルゴリズム_ です。　例は、以下の通りです。 

* [`parallel_for()`](../API/core/parallel-dispatch/parallel_for): 関数を指定された回数だけ、順序を定めずに実行、
* [`parallel_reduce()`](../API/core/parallel-dispatch/parallel_reduce): `parallel_for()`の実行および還元演算の組合せ、
* [`parallel_scan()`](../API/core/parallel-dispatch/parallel_scan): 各演算の出力値について、parallel_for()　演算と接頭辞または接尾辞スキャンを組み合わせ、および
* `task`: 他の関数への依存関係を持つ、単関数を実行。

アプリケーションをこれらのパターンで表現することで、基盤となる実装および使用されるコンパイラが有効な変換について、推論することが可能となります。例えば、すべての `parallel_***` パターンは、実行順序を設定せず、還元処理自体の結果のみが確定的であることを、保証します。これにより、異なるハードウェア上で、例えば、反復処理のスレッドおよびベクトルレーンの割り当てといった、異なるマッピングパターンが可能となります。 

## 実行ポリシー

実行ポリシーは、実行パターンと組み合わせて、関数が _どのように_　実行されるかを決定します。一部のポリシーは、他のポリシー内にネストすることが可能です。 

### 範囲ポリシー

実行ポリシーの中で、もっとも単純な形態は、 _範囲ポリシー_　です。それらは、範囲内の各要素に対して一度ずつ演算を実行するために使用されます。 There are no prescriptions of order of execution or concurrency, which means that it is not legal to synchronize different iterations.

### チームポリシー

チームポリシーは、階層的並列性を実装するために使用されます。 そのために、Kokkos　は、スレッドを　 _teams_　にグループ化します。 _thread team_ とは、1つ以上の並列な実行　"スレッド"　の集合体です。 Kokkos では、任意の数のチームの作成 - the _league size_　が可能です。 ハードウェアの制約により、チーム内のスレッド数 - the _team size_　は制限されます。 チーム内のすべてのスレッドは、確実に同時に実行されます

チーム内のスレッドは同期が可能です。これらは "barrier" プリミティブを備えており、一時的な保存に使用できる　"スクラッチパッド"　メモリを共有します。 Kokkos　ではすべての同期メカニズムが有効とは限らないことに注意してください。特に、チーム内のスレッドに対して、"スピンロック"　を実装すると、デッドロックが発生する可能性があります。 Kokkos　では、スレッドの順序保証は提供されておりません。つまり、チーム内の別のスレッドが取得したロックで待機状態にある単一のスレッドが、コンピューティングエンジンのリソースを、100%使用してしまう可能性があります。 明示的な Kokkos を、 _チームバリア_　と呼ぶことが、チーム内のスレッドを同期させるための唯一の安全な方法です。

Scratch pad memory exists only during parallel operations; allocations in it do not persist across kernels. Teams themselves may run in any order, and may not necessarily run all in parallel. For example, if the user asks for _T_ teams, the hardware may choose to run them one after another in sequence, or in groups of up to _G_ teams at a time in parallel.

Users may _nest_ parallel operations. Teams may perform one parallel operation (for, reduce, or scan), and threads within each team may perform another, possibly different parallel operation. Different teams may do entirely different things. For example, all the threads in one team may execute a [`parallel_for()`](../API/core/parallel-dispatch/parallel_for) and all the threads in a different team may execute a [`parallel_scan()`](../API/core/parallel-dispatch/parallel_scan). Different threads within a team may also do different things. However, performance may vary if threads in a team "diverge" in their behavior (e.g., take different sides of a branch). [Chapter 8 - Hierarchical Parallelism](HierarchicalParallelism) shows how the C++ implementation of Kokkos exposes thread teams.

NVIDIA's CUDA programming model inspired Kokkos' thread team model. The scratch pad memory corresponds with CUDA's per-team "shared memory." The "league/team" vocabulary comes from OpenMP 4.0 and has many aspects in common with our thread team model. We have found that programming to this model results in good performance, even on computer architectures that only implement parts of the full model. For example, most multicore processors in common use for high-performance computing lack "scratch pad" hardware. However, if users request a scratch pad size that fits comfortably in the largest cache shared by the threads in a team, programming as if a scratch pad exists forces users to address locality in their algorithms. This also reflects the common experience that rewriting a code for more restrictive hardware, then porting the code _back_ to conventional hardware, tends to improve performance relative to an unoptimized code.

## Memory Spaces

Memory Spaces are the places _Where_ data resides. They specify physical location of data as well as certain access characteristics. Different physical locations correspond to things such as high bandwidth memory, on die scratch memory or non-volatile bulk storage. Different logical memory spaces allow for concepts such as UVM memory in the CUDA programming model, which is accessible from Host and GPU. Memory Spaces could also be used in the future to express remote memory locations. Furthermore, they encapsulate functionality such as consistency control and persistence scopes.

## Memory Layout

Layouts express the _mapping_ from logical (or algorithmically) indices to address offset for a data allocation. By adopting appropriate layouts for memory structures, an application can optimise data access patterns in a given algorithm. If an implementation provides polymorphic layouts (i.e. a data structure can be instantiated at compile or runtime with different layouts), an architecture dependent optimisation can be performed.

## Memory Traits

Memory Traits specify _how a data structure is accessed_ in an algorithm. Traits express usage scenarios such as atomic access, random access and streaming loads or stores. By putting such attributes on data structures, an implementation of the programming model can insert optimal load and store operations. If a compiler implements the programming model, it could reason about the access modes and use that to inform code transformations.
