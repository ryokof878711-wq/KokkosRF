機械モデル
=============

.. role:: cpp(code)
   :language: cpp

.. |node| image:: figures/kokkos-node-doc.png
   :alt: 図 2.1 次世代高性能計算ノードの概念モデル

.. _Chap7ParallelDispatch: ParallelDispatch.html
.. |Chap7ParallelDispatch| replace:: 第７章 - 並列ディスパッチ

.. |execution-space| image:: figures/kokkos-execution-space-doc.png
   :alt: 図 2.2 次世代計算ノードの実行空間例

.. |memory-space| image:: figures/kokkos-memory-space-doc.png
   :alt: 図2.3 次世代計算ノードのメモリ空間例

.. _ViewAllocation: View.html
.. |ViewAllocation| replace:: ビュー配置

.. _Initialization: Initialization.html
.. |Initialization| replace:: 初期化

.. _Section82: HierarchicalParallelism.html#hp-thread-teams
.. |Section82| replace:: セクション 8.2

.. _Chap8HierarchicalParallelism: HierarchicalParallelism.html
.. |Chap8HierarchicalParallelism| replace:: 第8章 - 階層的並列

.. _Section231: Machine-Model.html#thread-safety
.. |Section231| replace:: セクション 2.3.1

.. _ParallelFor: ../API/core/parallel-dispatch/parallel_for.html
.. |ParallelFor| replace:: ``parallel_for()``

.. _Fence: ../API/core/parallel-dispatch/fence.html
.. |Fence| replace:: ``fence()``

 本章を読めば、Kokkos　フレームワークの設計上の選択と構造の基盤となる、並列計算ノードの抽象モデルについて、理解できます。このマシンモデルにより、Kokkos　を使用して記述されたアプリケーションは、様々なハードウェア上で高いパフォーマンスを発揮しつつ、アーキテクチャ間の移植性を確保します。

機械モデルには二つの重要な構成要素があります:

* *メモリ空間*、 その中では、データ構造の配置が可能です。
* *実行空間*、 1つ以上の *メモリ空間*　からのデータを使用して、並列演算を実行します。

モチベーション
-----------

Kokkos には、機械モデルには二つの重要な構成要素があります。 その第一の要素は、
将来の移植性と高性能を兼ね備えたハイパフォーマンスコンピューティングアプリケーションの開発に
必要な基本概念を記述する、基盤となる　*抽象マシンモデル*　です; 第二の要素は、C++　で記述された　*プログラミングモデルの具体的な実装*　であり、これによりプログラマーは概念的なマシンモデルに対して記述することが可能となります。 Kokkos で使用されている基盤となるモデルは、将来的に、C++　以外の追加言語で実装される可能性がありますが、アルゴリズム仕様は有効なまま維持されているので、Kokkos という概念のこの二つの側面を、別個の存在として扱うことが重要です。 

Kokkos 抽象機械モデル
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Kokkos　は、将来の共有メモリコンピューティングアーキテクチャの設計において、*抽象マシンモデル*　を想定しています。 モデル (図 2.1に示す) は、1つの計算ノード内に、複数の実行ユニットが存在する場合があることを想定しています。 エクサスケール計算における抽象機械モデルに関するより一般的な議論については、参考文献　Ang\ :sup:`1`　を参照してください。ここで示す図においては、 2種類の異なる計算ユニットを表示することを選択しました  - 現代のプロセッサコアと同様に、複数のレイテンシ最適化コアを代表するもの、および オフダイアクセラレータという形態による、第二の演算リソースです。 特筆すべき点は、プロセッサとアクセラレータがそれぞれ独立したメモリを有しており、各メモリは固有の性能特性を備えていることです。これらのメモリはノード全体でアクセス可能である場合もあれば、そうでない場合もあります（つまり、メモリは全ての実行ユニットから到達可能、あるいは　*共有*　される可能性がありますが、特定のメモリ空間は、特定の実行ユニットのみがアクセス可能な場合もあります）。 図2.1に示された特定のレイアウトは、単一ノード内に複数のタイプの演算エンジンとメモリを実装する可能性を記述するために用いられる　Kokkos　抽象マシンモデルの具体例です。 将来のシステムにおいては、ノード内で使用される実行エンジンの種類が多様化する可能性がありますが、現在広く普及しているマルチコアプロセッサのように単一タイプのコアから始まり、マルチコアプロセッサが様々なタイプのアクセラレータコアと結合されるような、多様な実行ユニットに至るまで広がっていくでしょう。 潜在的なノードの範囲への移植性を確保するためには、計算エンジンと利用可能なメモリの抽象化が必要となります。

-----

:sup:`1` Ang, J.A., et. al., **エクサスケール計算のための抽象機械モデルとプロキシアーキテクチャ**,
2014年, サンディア国立研究所およびローレンスバークレイ国立研究所、 DOE コンピュータアーキテクチャ研究所プロジェクト

-----

|ノード|

図 2.1 Conceptual Model of a Future High Performance Computing Node将来の高性能計算ノードの概念モデル
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Kokkos 空間
-------------

Kokkos では、同一の性能特性を共有する計算ユニットの論理的なグループ化を、*実行空間*　という用語で表現しています。実行空間は、プログラマーがいくつかの種類の基本的な並列操作を用いて利用できる、一連の並列実行リソースを提供します。 利用可能な演算のリストについては、 |Chap7ParallelDispatch|_　を参照してください。 *メモリ空間* という用語は、論理的に独立したメモリリソースを表すために使用され、それはデータ割り当てに使用できます。

実行空間インスタンス
~~~~~~~~~~~~~~~~~~~~~~~~~

実行空間の *インスタンス* とは、プログラマが並列処理を割り当てることができる、実行空間の特定の具体化を指します。例を挙げますと、実行空間は、マルチコアプロセッサを記述するために使用される場合があります。本例においては、 実行空間には、いくつかの同種のコアが含まれており、それらは論理的なグループ分けを共有しています。 Kokkos　モデル用に書かれたプログラムでは、この実行空間のインスタンスが提供され、その上で並列カーネルを実行することが可能となります。 二番目の例として、マルチコアプロセッサにGPUを追加し、システム内で第二の実行空間タイプを利用可能とした場合、アプリケーションプログラマーは二つの実行空間インスタンスから選択できるようになります。 ここでの重要な考慮事項としては、異なる実行空間向けのコードコンパイル方法と、カーネルをインスタンスへディスパッチする処理が、Kokkosモデルによって抽象化されているということです。これにより、アプリケーションプログラマーはハードウェア固有の言語でアルゴリズムを記述する必要がなくなります。

|実行-空間|

図 2.2 次世代計算ノードの実行空間例
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Kokkos メモリ空間
~~~~~~~~~~~~~~~~~~~~

将来の計算ノードで利用可能となる複数のメモリ型は、Kokkos　によって、*メモリ空間*　を通じて抽象化されます。 各メモリ空間は、データ構造を割り当ててアクセスできる有限の記憶容量を提供します。 異なるメモリ空間の種類は、実行空間からのアクセス可能性およびパフォーマンス特性に関して、それぞれ異なる特徴を有しています。

Kokkos メモリ空間のインスタンス
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

実行空間が、*インスタンス*の可用性を通じて特定のインスタンス化を持つのと同様に、メモリ空間もまた同様の仕組みで特定のインスタンス化を持ちます。 メモリ空間のインスタンスは、アプリケーションプログラマーが、データ格納領域の割り当てを要求するための具体的な方法を提供します。 実行スペースの例に戻りますと、マルチコアプロセッサには、パッケージ内蔵メモリ、低速な　DRAM　、および追加の不揮発性メモリセットを含む、複数のメモリスペースが利用可能な場合があります。 GPU　は、パッケージ内蔵メモリを通じて、追加のメモリ領域を提供する場合があります。プログラマーは、各データ構造をどのメモリ領域に関連付けられた特定のインスタンスから要求するかによって、その配置場所を、自由に決定できます。 Kokkos は、メモリ割り当てルーチンおよび関連するデータ管理操作（メモリ解放、将来の使用のための返却、ならびにコピー操作を含む）の適切な抽象化を提供します。

|メモリ-空間|

図 2.3 次世代計算ノードのメモリ空間例
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

**Kokkosにおけるメモリへのアトミックアクセス** 複数の実行スレッドが同一のメモリアドレスを読み取り、その項目に対する計算を完了し、同じメモリアドレスに書き戻そうとする場合、順序衝突が発生する可能性があります。 これらの状況は、*競合状態*（スレッドが完了する際にメモリに格納されるデータ値が、どのスレッドが最後にメモリ操作を完了したかに依存するため）として知られており、並列プログラムにおける非決定性の原因となる場合が多くあります。 並列プログラムにおいて競合状態が発生しないようにするためには、ロック（一度に単一のスレッドのみがデータ構造にアクセスできるようにする）、クリティカルセクション（任意の時点で単一のスレッドのみがコードシーケンスを実行できるようにする）、および*アトミック*　演算の使用等、いくつかの方法が用いられます。 アトミックなメモリ演算は、メモリへの読み取り、単純な計算、書き込みが単一の単位として完了することを保証します。これにより、アプリケーションプログラマーは、例えばメモリ値を安全にインクリメントしたり、より一般的なケースとして、複数のスレッドからの値を単一のメモリ位置に安全に蓄積したりすることが可能になる場合があります。

**Kokkos　におけるメモリ一貫性** メモリ一貫性モデルは、それ自体が複雑なテーマであり、通常はハードウェアキャッシュやメモリアクセスの一貫性に関連する複雑な演算に依存しています（詳細については、へニッセイおよび パターソン\ :sup:`2`　ご参照してください）。Kokkos は、ハードウェアにキャッシュが存在することを　*要求*　せず、したがって非常に弱いメモリ一貫性モデルを前提としています。 Kokkos モデルにおいては、プログラマーはカーネルによって発行されるメモリ演算の特定の順序を想定すべきではありません。 これらの演算が適切に保護されていない場合、メモリ演算間で競合状態が発生する可能性があります。 メモリ操作が確実に完了することを保証するため、Kokkosでは、計算エンジンは新規のメモリ操作を発行する前に、未処理のメモリ演算をすべて完了させることを強制する　*フェンス*　演算を提供しております。フェンスを適切に使用することで、プログラマーはデータが確実にメモリに書き込まれるタイミングについて保証を確立することが可能となります。

-----

:sup:`2` へニッセイ J.L. およびパターソン D.A., **コンピュータアーキテクチャ、第5版 : 定量的アプローチ**, モーガン・カーフマン, 2011.

-----

プログラム実行
-----------------

It is tempting to try to define formally what it means for a processor to execute code. None of us authors have a background in logic or what computer scientists call "formal methods," so our attempt might not go very far! We will stick with informal definitions and rely on Kokkos' C++ implementation as an existence proof that the definitions make sense.

Kokkos lets users tell execution spaces to execute parallel operations. These include parallel for, reduce, and scan (see |Chap7ParallelDispatch|_) as well as |ViewAllocation|_ and |Initialization|_. We name the class of all such operations *parallel dispatch*.

From our perspective, there are three kinds of code:

#. Code executing inside of a Kokkos parallel operation
#. Code outside of a Kokkos parallel operation that asks Kokkos to do something (e.g., parallel dispatch itself)
#. Code that has nothing to do with Kokkos

The first category is the most restrictive. |Section82|_ explains restrictions on inter-team synchronization. In general, we limit the ability of Kokkos-parallel code to invoke Kokkos operations (other than for nested parallelism; see |Chap8HierarchicalParallelism|_ and especially |Section82|_). We also forbid dynamic memory allocation (other than from the team's scratch pad) in parallel operations. Whether Kokkos-parallel code may invoke operating system routines or third-party libraries depends on the execution and memory spaces being used. Regardless, restrictions on inter-team synchronization have implications for things like filesystem access.

*Kokkos threads are for computing in parallel*, not for overlapping I/O and computation, and not for making graphical user interfaces responsive. Use other kinds of threads (e.g., operating system threads) for the latter two purposes. You may be able to mix Kokkos' parallelism with other kinds of threads; see |Section231|_. Kokkos' developers are also working on a task parallelism model that will work with Kokkos' existing data-parallel constructs.

**Reproducible reductions and scans** Kokkos promises *nothing* about the order in which the iterations of a parallel loop occur. However, it *does* promise that if you execute the same parallel reduction or scan, using the same hardware resources and run-time settings, then you will get the same results each time you run the operation. "Same results" even means "with respect to floating-point rounding error."

**Asynchronous parallel dispatch** This concerns the second category of code that calls Kokkos operations. In Kokkos, parallel dispatch executes *asynchronously*. This means that it may return "early," before it has actually completed. Nevertheless, it executes *in sequence* with respect to other Kokkos operations on the same execution or memory space. This matters for things like timing. For example, a |ParallelFor|_ may return "right away," so if you want to measure how long it takes, you must first call |Fence|_ on that execution space. This forces all functors to complete before |Fence|_ returns.

Thread safety?
~~~~~~~~~~~~~~

Users may wonder about "thread safety," that is, whether multiple operating system threads may safely call into Kokkos concurrently. Kokkos' thread safety depends on both its implementation and on the execution and memory spaces that the implementation uses. The C++ implementation has made great progress towards (non-Kokkos) thread safety of View memory management. For now, however, the most portable approach is for only one (non-Kokkos) thread of execution to control Kokkos. Also, be aware that operating system threads might interfere with Kokkos' performance depending on the execution space that you use.
