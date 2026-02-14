タスク並列処理
================

.. role:: cpp(code)
    :language: cpp

.. 警告::
    Kokkos 4.5より非推奨

Kokkos　は、軽量なタスクベースのプログラミングをサポートしていますが、現時点ではかなり限定的ですが、将来的には大幅に拡張する予定です。

私の問題に対して、Kokkos Tasking　は有効ですか？
----------------------------------------

現在の　Kokkos　のタスク処理アプローチは、すべてのタスクベースの問題に適しているわけではありません。 現在、Kokkosのタスク割り当てインターフェースは、トップレベルのKokkosデータ並列起動に内在するオーバーヘッドを克服するには小さすぎるカーネルを対象としています。つまり、非自明な依存構造を持つ、小規模だが大量に存在するデータ並列タスクが対象です。この一般的なスケールモデルに適合するが（非常に）単純な依存構造を持つタスクについては、 必要に応じて負荷分散を行う場合、`階層的並列処理 <../../ProgrammingGuide/HierarchicalParallelism.html>`_ を、場合によっては、 ``Kokkos::Schedule<Dynamic>`` スケジューリングポリシー（例えば `このページ <policies/RangePolicy.html>`_ を参照）と組み合わせて使用すると、容易になる場合があります。

基本使用例
-----------

基本的には、タスク並列処理とは、　Kokkos　における並列処理の単なる別の形態に過ぎません。通常の `parallel dispatch <../../ProgrammingGuide/ParallelDispatch.html>`_ に関して、同様のパターン、ポリシー、ファンクタの一般的なイディオムが適用されます:

.. image:: ../../ProgrammingGuide/figures/parallel-dispatch.png

同様に、タスクについては、以下を持っています:

.. image:: ../../ProgrammingGuide/figures/task-dispatch.png

タスクファンクタ
------------

Kokkos　におけるタスク並列処理のイディオムのファンクタ部分は、ほとんどの点で、データ並列処理の場合と同様です。 タスクファンクタには追加要件があり、タスクの出力型はユーザーがネスト型（別名　"typedef"　）として``value_type``を指定することで提供する必要があります:

.. code-block:: cpp

    struct MyTask {
        value_type = double;を使用
        テンプレート <class TeamMember>
        KOKKOS_INLINE_FUNCTION
        void operator()(TeamMember& member, double& result);
    };

`team parallelism <../../ProgrammingGuide/HierarchicalParallelism.html>`_　と同様に、最初のパラメータはチームハンドルであり、``Kokkos::TeamPolicy``によって生成されるものと同等の機能に加え、いくつかの追加機能を備えています。 ``Kokkos::parallel_reduce()``と同様に、出力は第2パラメータを通じて表現されます。 現在、Kokkos タスクへのラムダインターフェースは存在しないことに、注意してください。

タスクパターン
-------------

タスク処理における　``Kokkos::parallel_for()``　およびフレンド関数の主要アナログは、``Kokkos::task_spawn()``および``Kokkos::host_spawn()``です。 両者は適切な　``Scheduler``　に関連付けられた　``Kokkos::Future``　を返しますが、 ``host_spawn`` は、タスクファンクタの　*外部*　からのみ呼び出せますが、``task_spawn`` はタスクファンクタの　*内部*　からのみ呼び出せます。

タスクポリシー
-------------

現在、Kokkos タスクには、2つのタスクポリシーが存在します：``TaskSingle``　および　``TaskTeam``　です。前者では、先行タスクが完了した時点で関連するタスクファンクタを単一ワーカーで起動するようコッコスに指示しますが（詳細は後述）、 後者は、データ並列処理における ``Kokkos::TeamPolicy`` 起動の単一チームと同様に、作業者チームを伴って起動するよう　Kokkos　に指示します。 ``TaskTeam``　で生成されたタスクでは、ユーザーは単一のワーカーからのみ、``task_spawn``　を呼び出すことが許可されます。この目的には、　``Kokkos::single``　を使用してください。

先行者およびスケジューラ
---------------------------

Kokkos　における依存関係は、``Kokkos::BasicFuture``　クラステンプレートのインスタンスによって表現されます。各タスク（``task_spawn`` または ``host_spawn`` パターンで作成）は、先行タスクをゼロ個または一つ持つことができます（より多くの先行タスクを持つタスクグラフを作成するには、後述の ``Kokkos::when_all`` を使用してください）。 先例は、ポリシーへの引数としてパターンに指定されます:

.. code-block:: cpp

    cheduler_type = /* ... discussed below ... */;を使用。
    auto scheduler = scheduler_type(/* ... discussed below ... */);
    // Launch with no predecessor:
    auto fut = Kokkos::host_spawn(
        Kokkos::TaskSingle(scheduler),
        MyTaskFunctor()
    );
    // 最も早くて、 `fut` の準備が整った時に起動:
    auto fut2 = Kokkos::host_spawn(
        Kokkos::TaskSingle(scheduler, fut),
        MyOtherTaskFunctor()
    );
    /* ... */

Kokkosの　``TaskScheduler``　概念は、タスクベースシステムにおけるタスクスケジューリングの多様な戦略を一般化した抽象概念です。Kokkos  概念は、タスクベースシステムにおけるタスクスケジューリングの多様な戦略を一般化した抽象概念です。 Kokkos　の他の概念と同様に、ユーザーは特定の``TaskScheduler``　に直接依存するコードを書くべきではなく、すべての　``TaskScheduler``　型が保証する汎用モデルに依存すべきです。

依存関係を表現するために使用される　``Kokkos::BasicFuture``　クラステンプレートは、それが表現するタスクの戻り値の型と、そのタスクの実行に使用されたスケジューラの型に対して、テンプレート化されています:

.. code-block:: cpp

    テンプレート <class Scheduler>
    void my_function(Scheduler sched) {
        // 何らかの理由で型に名前を付ける必要が生じるまで、auto　を使用してください。
        auto fut = Kokkos::host_spawn(
            Kokkos::TaskSingle(sched),
            MyTaskFunctor()
        );
        /* ... */
        my_result_type = MyTaskFunctor::value_type;　を使用。
        // 兌換性が保証されています:
        Kokkos::BasicFuture<my_task_result, Scheduler> ff = fut;
    }

(注意事項: Kokkos　はタスク並列パターンの具体的な戻り値型を保証せず、適切な``Kokkos::BasicFuture``型に変換可能であることのみを保証します。  何らかの理由で型に名前を付ける必要がある場合（例えばコンテナに格納する場合など）までは、``auto`` を使用してください。 そうでない場合、将来の型が特定の ``Kokkos::BasicFuture`` 型に変換される必要が全くないならば、Kokkos　の方がより優れたパフォーマンスを提供できる可能性があります。)

Kokkos　における ``TaskScheduler`` 型は、共有参照セマンティクスを持ちます; 特定のスケジューラのコピーは、それがコピーされた元のスケジューラと同じ基盤となるエンティティと戦略を表します。 タスクファンクタ内部では、ユーザーはスケジューラを自身で保持するのではなく、最初の引数として渡されたチームメンバーハンドルからスケジューラインスタンスを取得すべきです。これについても、何らかの理由で保存する必要が生じるまでは``auto``　を使用してください。

　Future　が準備完了状態になると、その　　Future　が先行タスクとして表すタスクの結果を　``get()``　メソッドを使用して取得できます。ただし、これは将来が確実に準備されているコンテキストから　**のみ**、呼び出すことができます。つまり、未来を先行タスクとして生成されたタスク、または別のタスクを介してその未来に間接的に依存するタスク、もしくは未来に関連付けられたタスクを生成したスケジューラについての ``Kokkos::wait`` 実行後です（下記参照）。 **フューチャーの``get()``メソッドを他のコンテキストで　**呼び出す**　と、未定義動作が発生します**（しかも最悪の種類のバグです：実行開始から数時間経ってもセグメンテーションフォルトすら発生しない可能性があります！）。これは、``std::future``　とは異なり、``get()``　メソッドは準備が整うまでブロックすることに注意してください。 

Kokkos　における　Future　型は、共有参照セマンティクスを持ちます。ある　Future　のコピーは、それがコピーされた元の　Future　と同じ基盤の依存関係を表します。デフォルトコンストラクタで生成された ``Kokkos::BasicFuture`` は、値を持たない常に準備状態の依存関係を表します（つまり、値を取得しようとすると未定義動作となり、実際にはおそらくセグメンテーションフォルトが発生します）。デフォルトコンストラクタで生成された　Future　は、　``is_null()``　メソッドについて、`true`　を返します。　適切な値型およびスケジューラ型の　``Kokkos::BasicFuture``　への変換可能性に加えて、すべての　Kokkos　Future　は、``void``　型および適切なスケジューラ型の　``Kokkos::BasicFuture``　に変換可能です。

 Kokkos タスク待機中
-------------------------

Kokkosは通常、個々の　Future を待機するために実行スレッドをブロックする手段を提供せず、Kokkos generally provides no way to block a thread of execution to wait on an individual future, ユーザーが外部手段（例えば、``while``　ループ内で``is_ready()``　メソッドをポーリングするといった行為）でこれを試みた場合、正しい実行が保証されます。Kokkos タスクファンクタの　*外側*（つまり、``host_spawn`` が許可されるあらゆる場所）では、Kokkos は特定のスケジューラ上で作成された *すべての* Future（間接的に作成されたものも含む。つまり、まだ完了していない、あるいは開始すらされていない可能性のあるタスクによって生成されたものも含む）を待機する機能を提供します。 これはスケジューラ上で　``Kokkos::wait``　関数を使用して行われます:

.. code-block:: cpp

    テンプレート <class Scheduler>
    void my_function(Scheduler sched) {
        // 何らかの理由で型に名前を付ける必要が生じるまでは、autoを使用してください
        auto fut = Kokkos::host_spawn(
            Kokkos::TaskSingle(sched),
            MyTaskFunctor()
        );
        Kokkos::wait(sched);
        auto value = fut.get();
        /* ... */
    }

ユーザーは、``Kokkos::wait``　を　*非常に*　高コストな演算（　"sledgehammer"　）と捉え、可能な限り控えめに使用する必要があります。

タスクファンクタにおける　"待機"
~~~~~~~~~~~~~~~~~~~~~~~~~~~

In Kokkos tasking, all task functors must be able to run to completion without blocking once they are started (the Kokkos scheduler *can* run other tasks at any point that the functor calls back into the Kokkos tasking system, such as any ``task_spawn``, but it is allowed to assume user functors will run to completion if left alone).  This means that there is no way to block a task pending the result of another task.  Other tasking systems that make this kind of design decision require the user to spawn a new task for each new piece of predicated work, which is an option in Kokkos as well, but Kokkos also provides another option.  To help reduce the allocation cost associated with the traditional approach to never-blocking task systems, Kokkos allows users to "reuse" the current task as a successor to some future.  Kokkos provides the ``Kokkos::respawn()`` function.  For example:

.. code-block:: cpp

    template <class Scheduler>
    struct MyTaskFunctor {
        using value_type = void;
        using future_type = Kokkos::BasicFuture<double, Scheduler>;
        future_type f;
        template <class TeamMember>
        KOKKOS_INLINE_FUNCTION
        void operator()(TeamMember& member) {
            if(f.is_null()) {
            f = Kokkos::task_spawn(
                Kokkos::TaskSingle(member.scheduler()),
                MyOtherTaskFunctor()
            );
            Kokkos::respawn(this, f);
            }
            else {
            // This is after the respawn so we're guaranteed that f is ready
            printf("Got result %f\n", f.get());
            }
        }
    };

A task functor can only be respawned up to once *per execution of* ``operator()`` (that is, once per time it is spawned or respawned).  Multiple calls to ``respawn`` in the same execution of ``operator()`` are redundant and lead to undefined behavior.  Calls to ``respawn`` are always lazy—the subsequent call to ``operator()`` by Kokkos will only happen after the currently executing one returns (and after the predecessors, if any, are ready) at the earliest.

The first argument to ``Kokkos::respawn`` must always be a pointer to the currently executing task functor (or one of its base classes) from which ``Kokkos::respawn`` is called.  The second argument can be either a future of the same scheduler as the currently executing task functor or an instance of the scheduler itself.  The third (optional) argument is a task priority, discussed below.

Aggregate Predecessors
----------------------

Kokkos tasking provides two forms of the ``when_all()`` method on every ``TaskScheduler`` type. Both serve to aggregate multiple predecessors into one, and both return a value convertible to a ``Kokkos::BasicFuture`` of ``void`` and that scheduler type.  The first takes an array of ``Kokkos::BasicFuture`` of the scheduler type and a count of entries in that array.  The second takes a ``count`` and a unary function or functor that should expect to be called with each integer in the range ``[0, count)``.  In both cases, the return value is a future that will become ready when all of the input futures become ready.

Task Priorities
---------------

Kokkos allows users to provide a priority hint to task parallel execution policies as an optional third argument, or as an optional third argument to ``Kokkos::respawn``.  This has no observable effect on the programming model—only on the performance.  A scheduler may ignore these priorities.  The allowed task priorities are ``Kokkos::TaskPriority::High``, ``Kokkos::TaskPriority::Regular``, and ``Kokkos::TaskPriority::Low``, which the second being the default if the argument isn't given.

..
    Invariants in the Kokkos Tasking Programming Model
    ==================================================

..
    TODO
