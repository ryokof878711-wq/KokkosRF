``parallel_for``
================

.. role::cpp(code)
    :language: cpp

ヘッダーファイル: ``<Kokkos_Core.hpp>``

使用例:

.. code-block:: cpp

    Kokkos::parallel_for(name, policy, functor);
    Kokkos::parallel_for(policy, functor);

.. _text: ../policies/ExecutionPolicyConcept.html

.. |text| replace:: *ExecutionPolicy*

``functor``で定義された並列作業を、|text|_ ``policy``　に従ってディスパッチします。 オプションのラベル　``name``　は、
プロファイリングおよびデバッグツールで使用されます。この呼び出しは非同期であり、呼び出し元へ直ちに返る可能性があります。 

インターフェイス
---------

.. cpp:function:: template <class ExecPolicy, class FunctorType> Kokkos::parallel_for(const std::string& name, const ExecPolicy& policy, const FunctorType& functor);

.. cpp:function:: template <class ExecPolicy, class FunctorType> Kokkos::parallel_for(const ExecPolicy& policy, const FunctorType& functor);

パラメータ:
~~~~~~~~~~~

* ``name``: ユーザーが提供した文字列で、Kokkos Profiling Hooksを介してプロファイリングおよびデバッグツールで使用されます。
* ExecPolicy: 反復空間およびその他の実行プロパティを定義する　*ExecutionPolicy* :

  - ``IntegerType``: 1D反復範囲を定義し、0からカウント値までを範囲とします。
  - `RangePolicy <../policies/RangePolicy.html>`_: 1D反復範囲を定義します。
  - `MDRangePolicy <../policies/MDRangePolicy.html>`_: 多次元反復空間を定義します。
  - `TeamPolicy <../policies/TeamPolicy.html>`_: 1次元の反復範囲を定義し、それぞれがスレッドチームに割り当てらます。
  - `TeamThreadRange <../policies/TeamVectorRange.html>`_: スレッドチームによって実行される1次元の反復範囲を定義します。  ``TeamPolicy`` または ``TaskTeam`` を通じて実行される並列領域内でのみ有効です。
  - `ThreadVectorRange <../policies/ThreadVectorRange.html>`_: チーム内のスレッドを分割するベクトル並列化を通じて実行されるべき1次元反復範囲を定義します。 TeamPolicy または TaskTeam を通じて実行される並列領域内でのみ有効です。 ``TeamPolicy`` または ``TaskTeam`` を通じて実行される並列領域内でのみ有効です。

* FunctorType:  ``ExecPolicy`` のシグネチャに一致する operator() を持つ有効なファンクタ。 詳細については以下の例を参照してください。

必要要件
~~~~~~~~~~~~

* ``ExecPolicy`` が ``IntegerType``　であれば、 ``functor`` は、メンバー関数 ``operator() (const IntegerType& i) const``　を持ちます。  
* ``ExecPolicy`` が ``MDRangePolicy`` であり、 ``ExecPolicy::work_tag`` が ``void``　であれば、 ``functor`` は、``N`` が ``ExecPolicy::rank-1``　であるメンバー関数 ``operator() (const IntegerType& i0, ... , const IntegerType& iN) const`` を持ちます。 
* ``ExecPolicy`` が ``MDRangePolicy`` であり、 ``ExecPolicy::work_tag`` が ``void``　でなければ、 ``functor`` は、``N`` が ``ExecPolicy::rank-1``　であるメンバー関数 ``operator() (const ExecPolicy::work_tag, const IntegerType& i0, ... , const IntegerType& iN) constを持ちます。
* ``ExecPolicy::work_tag`` が　``void``　であれば、 ``functor``　は、 メンバー関数 ``operator() (const ExecPolicy::member_type& handle) const``　を持ちます。
* ``ExecPolicy::work_tag`` が ``void``　でなければ、 ``functor`` は、 メンバー関数　``operator() (const ExecPolicy::work_tag, const ExecPolicy::member_type& handle) const``　を持ちます。 

セマンティクス
---------

* ``policy``で定義された反復空間の各要素に対して、ファンクターの呼び出し演算子は正確に1回呼び出されます。ただし、``TeamPolicy`` および ``TeamThreadRange`` については、それぞれチームの各ベクトルレーンおよびスレッドによって呼び出し演算子が呼び出されます。
* 並行性も、反復処理の実行順序も、保証されません
* この呼び出しは非同期になる可能性があります。 カーネルの終了を保証するには、開発者はカーネルが実行されている実行領域に対してフェンスを呼び出すべきです。

例
--------

より詳細な例は、実行ポリシーのドキュメントに記載されています。

* ラムダ式をファンクタとする　``IntegerType``　ポリシー。 KOKKOS_LAMBDA は [=] KOKKOS_FUNCTION と同じであることに注意してください。これは、ラムダ式内で使用されるすべての変数が値でキャプチャされることを意味します。 また、 KOKKOS_LAMBDA および KOKKOS_FUNCTION マクロは、ターゲット実行空間に必要なすべての関数指定子を追加します。

.. code-block:: cpp

    #include<Kokkos_Core.hpp>
    #include<cstdio> 

    int main(int argc, char* argv[]) {
        Kokkos::initialize(argc,argv);

        int N = atoi(argv[1]);

        Kokkos::parallel_for("Loop1", N, KOKKOS_LAMBDA (const int i) {
            printf("Greeting from iteration %i\n",i);
        });

        Kokkos::finalize();
    }

* C++ struct をファンクタとして使用としてする　``TeamPolicy`` ポリシー。  KOKKOS_INLINE_FUNCTION マクロは、ターゲット実行空間に必要なすべての関数指定子を追加することに注意してください。 TagA/B構造体は、同じファンクタ内で演算子を　'overload'　する機能も提供します。 ラムダ式の例と同様に、ファンクタとその内部に含まれるメンバー変数は値でキャプチャされるため、暗示的または明示的なコピーコンストラクタを持つ必要があります。

.. code-block:: cpp

    #include<Kokkos_Core.hpp>
    #include<cstdio> 

    struct TagA {};
    struct TagB {};

    struct Foo {
        KOKKOS_INLINE_FUNCTION
        void operator() (const TagA, const Kokkos::TeamPolicy<>::member_type& team) const {
            printf("TagA\n"を使った、チーム%i　のスレッド%iからの挨拶
                    team.thread_rank(),team.league_rank());
        }
        KOKKOS_INLINE_FUNCTION
        void operator() (const TagB, const Kokkos::TeamPolicy<>::member_type& team) const {
            printf("TagB\n"を使った、チーム%i　のスレッド%iからの挨拶
                    team.thread_rank(),team.league_rank());
        }
    };

    int main(int argc, char* argv[]) {
        Kokkos::initialize(argc,argv);

        int N = atoi(argv[1]);

        Foo foo;

        Kokkos::parallel_for(Kokkos::TeamPolicy<TagA>(N,Kokkos::AUTO), foo);
        Kokkos::parallel_for("Loop2", Kokkos::TeamPolicy<TagB>(N,Kokkos::AUTO), foo);
        
        Kokkos::finalize();
    }
