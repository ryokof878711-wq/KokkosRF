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

(注意事項: Kokkos　はタスク並列パターンの具体的な戻り値型を保証せず、適切な``Kokkos::BasicFuture``型に変換可能であることのみを保証します。  何らかの理由で型に名前を付ける必要がある場合（例えばコンテナに格納する場合など）までは、``auto`` を使用してください。 Otherwise, Kokkos may be able to provide better performance if the future type is never required to be converted to a specific ``Kokkos::BasicFuture`` type.)

``TaskScheduler`` types in Kokkos have shared reference semantics; a copy of a given scheduler represents the same underlying entity and strategy as the scheduler it was copied from. Inside of a task functor, users should retrieve the scheduler instance from the team member handle passed in as the first argument rather than storing the scheduler themselves.  Use ``auto`` for this as well until you need to store it for some reason.

When a future is ready, the result of the task that a future represents as a predecessor can be retrieved using the ``get()`` method.  However, this can **only** be called from a context where the future is guaranteed to be ready—that is, in a task that was spawned with the future as a predecessor, or a task that transitively depends on that future via another task, or after a ``Kokkos::wait`` on the scheduler that spawned the task associated with the future (see below).  **Calling the** ``get()`` **method of a future in any other context results in undefined behavior** (and the worst kind of bug, at that: it may not even result in a segfault or anything until hours of execution!).  Note that this is different from ``std::future``, where the ``get()`` method blocks until it's ready.

Future types in Kokkos have shared reference semantics; a copy of a given future represents the same underlying dependency as the future it was copied from.  A default-constructed ``Kokkos::BasicFuture`` represents an always-ready dependency with no value (that is, retrieving the value is undefined behavior—practically speaking, probably a segfault).  A default-constructed future will return ``true`` for the ``is_null()`` method.  In addition to convertibility to a ``Kokkos::BasicFuture`` of the appropriate value type and scheduler type, all Kokkos futures are convertible to a ``Kokkos::BasicFuture`` of ``void`` and the appropriate scheduler type.

Waiting in Kokkos Tasking
-------------------------

Kokkos generally provides no way to block a thread of execution to wait on an individual future, and it provides no guarantee of correct execution if the user attempts to do so via external means (for instance, polling on the ``is_ready()`` method in a ``while`` loop is forbidden).  *Outside of* a Kokkos task functor (that is, anywhere that ``host_spawn`` would be allowed), Kokkos provides the ability to wait on *all* of the futures created on a given scheduler (including those created, transitively, by tasks spawned not yet completed, or potentially not even started).  This is done using the ``Kokkos::wait`` function on the scheduler:

.. code-block:: cpp

    template <class Scheduler>
    void my_function(Scheduler sched) {
        // use auto until you need to name the type for some reason
        auto fut = Kokkos::host_spawn(
            Kokkos::TaskSingle(sched),
            MyTaskFunctor()
        );
        Kokkos::wait(sched);
        auto value = fut.get();
        /* ... */
    }

Users should think of ``Kokkos::wait`` as an *extremely* expensive operation (a "sledgehammer") and use it as sparingly as possible.

"Waiting" in a task functor
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
