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

 Kokkos のタスク処理において、すべてのタスクファンクタは、一度開始されたらブロックすることなく完了まで実行可能である必要があります (　Kokkos　スケジューラは、ファンクタがKokkosタスクシステムへ呼び戻す任意の時点、例えば、あらゆる``task_spawn``で、他のタスクを実行*可能*ですが、放置されたユーザーファンクタは完了まで実行されると仮定しても差し支えありません)。 これは、あるタスクの結果を待機している別のタスクをブロックする方法がないことを意味します。 この種の設計選択を行う他のタスクシステムでは、予測される作業の新たな部分ごとにユーザーが新しいタスクを生成する必要がありますが、これはまた　Kokkos　でもオプションの一つです。しかしながら、 but Kokkos はまた、別のオプションも提供します。 従来の非ブロッキングタスクシステムのアプローチに伴う割り当てコストを削減するため、Kokkosではユーザーが現在のタスクを将来の何らかの後継タスクとして、"再利用"　できるようにします。 Kokkos は、 ``Kokkos::respawn()`` 関数を提供します。  例えば:

.. code-block:: cpp

    テンプレート <class Scheduler>
    構造体 MyTaskFunctor {
        value_type = void;　を使用
        using future_type = Kokkos::BasicFuture<double, Scheduler>;　を使用
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
            // これはリスポーン後なので、　f　が確実に準備完了している状態です
            printf("Got result %f\n", f.get());
            }
        }
    };

タスクファンクタは、``operator()``の　タスクファンクタは、``operator()``の　*実行ごとに*（つまり、生成または再生成されるたびに）最大1回のみ再生成できます。 同じ　``operator()``　の実行内で ``respawn`` を複数回呼び出すことは冗長であり、未定義挙動を引き起こします。``respawn``　への呼び出しは常に遅延評価されます。つまり、Kokkos　による後続の呼び出しは、現在実行中の呼び出しが返った後（および先行する呼び出しがある場合は、それらが準備完了した後）に初めて発生します。

``Kokkos::respawn`` の最初の引数は、常に ``Kokkos::respawn`` が呼び出されている現在実行中のタスクファンクタ（またはその基底クラスのいずれか）へのポインタでなければなりません。 第2引数は、現在実行中のタスクファンクタと同じスケジューラのフューチャー、またはスケジューラ自体のインスタンスのいずれかです。 第三引数（オプション）はタスク優先度であり、以下で説明します。 

集計前段階
----------------------

Kokkos　のタスク処理は、すべての　TaskScheduler　型に対して、2種類の ``when_all()`` メソッドを提供します。 どちらも複数の先行オブジェクトを1つに集約する役割を果たし、どちらも　``void`` 型およびそのスケジューラ型の　``Kokkos::BasicFuture``に変換可能な値を返します。　最初のものは、スケジューラ型の　``Kokkos::BasicFuture``　の配列とその配列内のエントリ数を引数として受け取ります。 二つ目は、``count``　と一項関数またはファンクタを受け取り、その関数は範囲　``[0, count)``　内の各整数で呼び出されることを想定しています。 いずれの場合も、戻り値は入力　Future がすべて準備完了状態になった時点で準備完了状態になる　Future です。

タスク優先度
---------------

Kokkos　では、タスク並列実行ポリシーに対して、優先度ヒントをオプションの第三引数として、または　``Kokkos::respawn``　のオプションの第三引数として指定できます。 これはプログラミングモデルには目に見える影響を与えず、パフォーマンスのみに影響します。 スケジューラは、これらの優先度を無視する場合があります。 認められたタスク優先度は、``Kokkos::TaskPriority::High``、 ``Kokkos::TaskPriority::Regular``　および　``Kokkos::TaskPriority::Low``　であり、 それは、引数が指定されていない場合、デフォルトで2番目の値が使用されるということです。

..
     Kokkos タスクプログラミングモデルにおける不変条件
    ==================================================

..
    TODO
