``parallel_reduce``
===================

.. role::cpp(code)
    :language: cpp

ヘッダーファイル: ``<Kokkos_Core.hpp>``

Usage
-----

.. code-block:: cpp

    Kokkos::parallel_reduce(name, policy, functor, reducer...);
    Kokkos::parallel_reduce(name, policy, functor, result...);
    Kokkos::parallel_reduce(name, policy, functor);
    Kokkos::parallel_reduce(policy, functor, reducer...);
    Kokkos::parallel_reduce(policy, functor, result...);
    Kokkos::parallel_reduce(policy, functor);

``functor``で定義された並列作業を、*ExecutionPolicy*　に従ってディスパッチし、実行ポリシーで定義されたワーカーからの貢献を削減します。オプションのラベル名は、プロファイリングおよびデバッグツールで使用されます。 還元型は、``sum``　であるか、``reducer``　によって定義されるか、あるいはファンクタ上のオプションの　``join``　演算子から演繹されます。削減結果は、``result``　に格納されるか、``reducer``　ハンドルを通じて格納されます。 また、そのような関数が存在する場合、``functor.final()``　関数にも提供されます。 単一の　``parallel_reduce``　内で複数の　``reducers``　を使用できるため、単一の　``parallel_reduce``　内で　``min``　値と　``max``　値を計算することが可能です。

インターフェイス
---------

.. code-block:: cpp

    template <class ExecPolicy, class FunctorType>
    Kokkos::parallel_reduce(const std::string& name,
                            const ExecPolicy& policy,
                            const FunctorType& functor);

.. code-block:: cpp

    template <class ExecPolicy, class FunctorType>
    Kokkos::parallel_reduce(const ExecPolicy& policy,
                            const FunctorType& functor);

.. code-block:: cpp

    template <class ExecPolicy, class FunctorType, class... ReducerArgument>
    Kokkos::parallel_reduce(const std::string& name,
                            const ExecPolicy& policy,
                            const FunctorType& functor,
                            const ReducerArgument&... reducer);

.. code-block:: cpp

    template <class ExecPolicy, class FunctorType, class... ReducerArgument>
    Kokkos::parallel_reduce(const ExecPolicy& policy,
                            const FunctorType& functor,
                            const ReducerArgument&... reducer);

.. code-block:: cpp

    template <class ExecPolicy, class FunctorType, class... ReducerArgumentNonConst>
    Kokkos::parallel_reduce(const std::string& name,
                            const ExecPolicy& policy,
                            const FunctorType& functor,
                            ReducerArgumentNonConst&... reducer);

.. code-block:: cpp

    template <class ExecPolicy, class FunctorType, class... ReducerArgumentNonConst>
    Kokkos::parallel_reduce(const ExecPolicy& policy,
                            const FunctorType& functor,
                            ReducerArgumentNonConst&... reducer);

Parametersパラメータ:
~~~~~~~~~~~

* ``name``: ユーザーが提供した文字列で、Kokkos Profiling Hooksを介してプロファイリングおよびデバッグツールで使用されます。
* ExecPolicy: 反復空間およびその他の実行プロパティを定義する *ExecutionPolicy*　。 有効なポリシーは以下の通り:

  - ``IntegerType``: 1D反復範囲を定義し、0からカウント値までを範囲とします。
  - `RangePolicy <../policies/RangePolicy.html>`_: 1次元反復範囲を定義します。
  - `MDRangePolicy <../policies/MDRangePolicy.html>`_: 多次元反復空間を定義します。
  - `TeamPolicy <../policies/TeamPolicy.html>`_: 1次元反復範囲を定義し、それぞれがスレッドチームに割り当てらます。
  - `TeamVectorRange <../policies/TeamVectorRange.html>`_: スレッドチームによって実行される1次元の反復範囲を定義します。 ``TeamPolicy`` または ``TaskTeam`` を通じて実行される並列領域内でのみ有効です。
  - `TeamVectorMDRange <../policies/TeamVectorMDRange.html>`_: チーム内のスレッドにより実行されるべき多次元反復範囲を定義します。``TeamPolicy`` または ``TaskTeam`` を通じて実行される並列領域内でのみ有効です。
  - `TeamThreadRange <../policies/TeamThreadRange.html>`_: チーム内のスレッドにより実行されるべき1次元反復範囲を定義します。``TeamPolicy`` または ``TaskTeam`` を通じて実行される並列領域内でのみ有効です。
  - `TeamThreadMDRange <../policies/TeamThreadMDRange.html>`_: チーム内のスレッドにより実行されるべき多次元反復範囲を定義します。``TeamPolicy`` または ``TaskTeam`` を通じて実行される並列領域内でのみ有効です。
  - `ThreadVectorRange <../policies/ThreadVectorRange.html>`_: チーム内のスレッドを分割するベクトル並列化を通じて実行されるべき1次元反復範囲を定義します。 TeamPolicy または TaskTeam を通じて実行される並列領域内でのみ有効です。
  - `ThreadVectorMDRange <../policies/ThreadVectorMDRange.html>`_: チーム内のスレッドを分割するベクトル並列化を通じて実行されるべき多次元反復範囲を定義します。``TeamPolicy`` または ``TaskTeam`` を通じて実行される並列領域内でのみ有効です。
* FunctorType: 有効なファンクタで、少なくとも　``ExecPolicy``　と縮小型との組み合わせに対応するシグネチャを持つ　``operator()`` を備えるもの。
* ReducerArgument: ``Reducer``　の概念を満たすクラス、または ``Kokkos::View`` のいずれか。
* ReducerArgumentNonConst: スカラー型または配列型; ファンクタの要件については以下を参照してください。

必要要件:
~~~~~~~~~~~~~

*  ``ExecPolicy`` が ``MDRangePolicy``　ではない場合、 the ``functor`` は、 ``operator() (const HandleType& handle, ReducerValueType& value) const`` または ``operator() (const WorkTag, const HandleType& handle, ReducerValueType& value) const``　の形式のメンバー関数を持ちます。

  -  ``ExecPolicy::work_tag`` が ``void`` または ``ExecPolicy`` が ``IntegerType``　である場合、 ``WorkTag``　引数を使わないオーバーロードが使用されます。
  - ``HandleType`` is an ``IntegerType`` if ``ExecPolicy`` is an ``IntegerType`` else it is ``ExecPolicy::member_type``.``HandleType`` は、``ExecPolicy`` が ``IntegerType`` の場合、``IntegerType`` であり、そうでない場合は ``ExecPolicy::member_type`` です。
*  ``ExecPolicy`` が ``MDRangePolicy``である場合、 the ``functor`` は、 ``operator() (const IntegerType& i0, ... , const IntegerType& iN, ReducerValueType& value) const`` または　``operator() (const WorkTag, const IntegerType& i0, ... , const IntegerType& iN, ReducerValueType& value) const``　の形式のメンバー関数を持ちます。

  - If ``ExecPolicy::work_tag`` が ``void``　の場合,  ``WorkTag`` 引数を持たないオーバーロードが使用されます。 
  - ``N`` は ``ExecPolicy::rank`` と一致する必要があります。
* ``functor``がラムダ式である場合、``ReducerArgument``　が　``Reducer``　概念を満たす、または ``ReducerArgumentNonConst`` が、 ``operator +=`` および ``operator =`` の POD型または ``Kokkos::View``　である必要があります。  後者の場合、値型のデフォルトコンストラクタ（　``reduction_identity``` ではなく）によって同一性が与えられると仮定する場合、和の削減が適用されます。 提供されている場合、``init``/ ``join``/ ``final`` メンバ関数は、タグ付き削減であっても ``WorkTag`` 引数を取ってはいけません
* ``ExecPolicy`` が ``TeamThreadRange`` である場合、 "reducing" ``functor`` は認められず、   ``ReducerArgument``　が　``Reducer``　概念を満たす、または ``ReducerArgumentNonConst`` が、 ``operator +=`` および ``operator =`` の POD型または ``Kokkos::View``　である必要があります。後者の場合、値型のデフォルトコンストラクタ（　``reduction_identity``` ではなく）によって同一性が与えられると仮定する場合、和の削減が適用されます。
* ``ExecPolicty`` が　``TeamVectorMDRange``、　``TeamThreadMDRange`` または ``ThreadVectorMDRange`` である場合、 ``ReducerArgumentNonConst``　のみが認められ、   ``operator +=`` and ``operator =``　を持つPOD型でなければなりません。
* The reduction argument typeof the ``functor`` 演算子の削減引数 ``ReducerValueType`` は、operator must be compatible with the ``ReducerArgument`` (または　``ReducerArgumentNonConst``) と互換性がなければならず、``init``、``join``、および``final``関数の引数が存在し、リデューサーが特定されない場合には、ファクターのそれらの引数は一致する必要があります（``ReducerArgument``は``Reducer``概念を満たさないが、スカラー、配列、または``Kokkos::View``です）。In case of tagged reductions, タグ削減の場合、つまりポリシー内でタグを特定する場合には、ファンクタの潜在的な　``init``/``join``/``final``　メンバ関数もタグ付けされる必要があります。
* ``ReducerArgument`` (または ``ReducerArgumentNonConst``)　が

  - スカラー型の場合には、 ``ReducerValueType``は、同型である必要があります。
  - rank-0 ``Kokkos::View``である場合、  ``ReducerArgument::non_const_value_type`` は、 ``ReducerValueType``　に一致する必要があります。
  - satisfies the ``Reducer`` 概念を満たす場合、 ``ReducerArgument::value_type``　は、must ``ReducerValueType``　に一致する必要があります。
  -  ``Kokkos::View``　の配列またはランク1である場合には、以下の通り :
    +　配列またはViewの要素型である場合、 + ReducerValueType　は、``T[]``　である必要があります。
    +　``ReducerArgument``　が配列である場合には、 静的にサイズを指定する必要があります。
    + ファンクタについては、ReducerValueType と同じ FunctorType::value_type を定義する必要があります。t
    + ファンクタは、配列の長さである公開メンバー変数　``int value_count``　を宣言する必要があります。
    + ファンクタは、関数 ``void init( ReducerValueType dst[] ) const``　を実装する必要があります。
    + ファンクタは、関数 ``void join( ReducerValueType dst[], ReducerValueType src[] ) const``　を実装する必要があります。
    + ファンクタが ``final`` 関数を実装する場合、 引数もまた　init のそれに一致し、加わる必要があります。

セマンティクス
---------

* ``policy``で定義された反復空間の各要素に対して、ファンクターの呼び出し演算子は正確に1回呼び出されますが、ただし、それぞれチームの各ベクトルレーンおよびスレッドによって呼び出し演算子が呼び出される　``TeamPolicy`` および ``TeamThreadRange`` については除外します。
* 並行性または実行順序は、保証されません。
* ``ReducerArgument`` がスカラー型でない場合には、呼び出しは非同期である可能性があります。
* ``ReducerArgument``　の内容は上書きされます。つまり、値を還元中立要素に初期化する必要はありません。
* 演算子への入力値には部分的な削減結果が含まれる可能性があり、Kokkosはスレッドローカルな寄与を最終段階で結合するのみである場合があります。 演算子は、要求された削減タイプに応じて入力削減値を変更しなければならない。

例
--------

そのほかの例は、`Custom Reductions <../../../ProgrammingGuide/Custom-Reductions.html>`_ and `ExecutionPolicy <../policies/ExecutionPolicyConcept.html>`_ documentation　に示されています。

.. code-block:: cpp

    #include <Kokkos_Core.hpp>
    #include <cstdio>

    int main(int argc, char* argv[]) {
        Kokkos::initialize(argc, argv);

        int N = atoi(argv[1]);
        double result;
        Kokkos::parallel_reduce("Loop1", N, KOKKOS_LAMBDA (const int& i, double& lsum) {
            lsum += 1.0*i;
        }, result);

        printf("Result: %i %lf\n", N, result);
        Kokkos::finalize();
    }

.. code-block:: cpp

    #include <Kokkos_Core.hpp>
    #include <cstdio>

    int main(int argc, char* argv[]) {
        Kokkos::initialize(argc, argv);

        int N = atoi(argv[1]);
        double sum, min;
        Kokkos::parallel_reduce("Loop1", N, KOKKOS_LAMBDA (const int& i, double& lsum, double& lmin) {
            lsum += 1.0*i;
            lmin = lmin < 1.0*i ? lmin : 1.0*i;
        }, sum, Kokkos::Min<double>(min));

        printf("Result: %i %lf %lf\n", N, sum, min);
        Kokkos::finalize();
    }

.. code-block:: cpp

    #include <Kokkos_Core.hpp>
    #include <cstdio>

    struct TagMax {};
    struct TagMin {};

    struct Foo {
        KOKKOS_INLINE_FUNCTION
        void operator() (const TagMax, const Kokkos::TeamPolicy<>::member_type& team, double& lmax) const {
            if (team.league_rank() % 17 + team.team_rank() % 13 > lmax)
                lmax = team.league_rank() % 17 + team.team_rank() % 13;
        }
        KOKKOS_INLINE_FUNCTION
        void operator() (const TagMin, const Kokkos::TeamPolicy<>::member_type& team, double& lmin) const {
            if (team.league_rank() % 17 + team.team_rank() % 13 < lmin)
                lmin = team.league_rank() % 17 + team.team_rank() % 13;
        }
    };

    int main(int argc, char* argv[]) {
        Kokkos::initialize(argc, argv);

        int N = atoi(argv[1]);

        Foo foo;
        double max, min;
        Kokkos::parallel_reduce(Kokkos::TeamPolicy<TagMax>(N,Kokkos::AUTO), foo, Kokkos::Max<double>(max));
        Kokkos::parallel_reduce("Loop2", Kokkos::TeamPolicy<TagMin>(N,Kokkos::AUTO), foo, Kokkos::Min<double>(min));
        Kokkos::fence();

        printf("Result: %lf %lf\n", min, max);

        Kokkos::finalize();
    }
