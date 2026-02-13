Kokkos 概念
===============

序章
------------

Kokkos は、創業当初から本質的にコンセプト主導のデザインを採用してきた。
最近まで、これらの概念をコードで表現する手軽な手段は存在しませんでした。
(実際、概念を　*文書化する*　手法についてさえ、一般的に合意されたアプローチは存在していなかった。)
C++コンセプトが言語機能として作業草案に統合され、複数のコンパイラで実装され、
そして最初のコンセプト駆動型ライブラリ機能、範囲が基準に採用されたことに伴い、 Kokkos が汎用コードで使用する概念を正式に定義し文書化する時期が到来しています。 さらに、最近の新たな　Kokkos　バックエンドの急速な拡大（およびそれに対応する需要）により、基本的な　Kokkos　概念に対する堅牢で正式な仕様の欠如がこれまで以上に深刻化しています。
近い将来に、SYCL、レジリエンス、そして　HPX　など、複数の新しいバックエンドプロジェクトが控えており、いくつか例を挙げると、今こそが、　Kokkos　の中核概念を正式に定義し、汎用コードでKokkosが使用する概念セットを反復的に強化し始める最適な時期です。
さらに、Kokkos 3.0 へのアップグレードには、特定の実行空間における多くの非汎用機能の廃止計画が含まれているため、今こそ、実行空間の概念と、それが相互作用する概念に対して、より包括的なアプローチを取る良い機会です。

アプローチ
--------

過去数年にわたるISO-C++実行器仕様策定活動で得た経験に加え、サットンとストラウストルプによる『C++のための概念ライブラリ設計』（2011年）のアプローチを指針とすべきです。
特に、*概念 = 制約 + 公理* という哲学に基づいて設計すべきでありますが、これは
使用の柔軟性と習得の容易さのバランスに焦点を当てたものです。 これは私たちが既に採用している設計思想からは、大きく変わるものではありませんー例えば、 純粋な制約ベースのアプローチでは、これら2つの　``parallel_for`` および ``parallel_reduce``　を同一にする必要性を見出さないかもしれませんが、これら2つに与えられる範囲ポリシーについて、実際には別個の「概念」は存在しません。（アルゴリズムは、結局のところそれらに対してほんのわずかに異なる処理を行います）
"similar enough" 制約集合を組み合わせることで  (これまで常にしてきたように)、
ユーザーへの認知的負荷を最小限に抑えつつ、必要な柔軟性を維持することができます。

概念
--------

.. _ExecutionSpace: execution_spaces.html

.. |ExecutionSpace| replace:: ``ExecutionSpace``

.. _MemorySpace: memory_spaces.html

.. |MemorySpace| replace:: ``MemorySpace``

.. _ExecutionPolicy: Execution-Policies.html

.. |ExecutionPolicy| replace:: ``ExecutionPolicy``

.. _RangePolicy: policies/RangePolicy.html

.. |RangePolicy| replace:: ``RangePolicy``

.. _TeamMember: policies/TeamHandleConcept.html

.. |TeamMember| replace:: ``TeamMember``

認知負荷に関して言えば、概念の総数を制限することよりも、おそらくさらに重要なのは、概念の　*subsumption hierarchies*　の数を制限することです。　C++　の範囲に関する経験からも、
これらの階層構造における分岐幅を制限することが学習の容易さを高めることが示されています。
おおまかに言えば、高レベルな視点から見た場合、　Kokkos　が現在使用している主要なユーザー可視の概念階層は次の通りです:

* |ExecutionSpace|_
* |MemorySpace|_
* |ExecutionPolicy|_ (includes, for instance, |RangePolicy|_)
* |TeamMember|_
* ``Functor``

いくつかの小規模な階層には以下が含まれます:

* ``MemoryLayout``
* ``Reducer``
* ``TaskScheduler``

現在（　``Kokkos_Concepts.hpp`` によると）概念として扱われているものの、実際には列挙型（または類似のもの）に近い性質を持つものは以下の通りです:

* ``MemoryTraits``
* Several things used only with execution policies:
    - ``Schedule``
    - The ``IndexType<>`` tag
    - The ``LaunchBounds<>`` tag
    - ``IterationPattern`` (a.k.a. ``Kokkos::Iterate``)

.. _Kokkos_View: view/view.html

.. |Kokkos_View| replace:: ``Kokkos::View``

Kokkos　の外部に存在する　``DualView``　および　``OffsetView``　のような、動作が類似したクラス型を前提とすれば、
また、　|Kokkos_View|_（および関連クラス）を単なるクラス・テンプレートではなく、概念として提示すべきかどうかについても、疑問が投げかけられています。

``ExecutionSpace`` 概念
------------------------------

.. _ExecutionSpaceTwo: execution_spaces.html#executionspaceconcept

.. |ExecutionSpaceTwo| replace:: ``ExecutionSpace``

現在　``Serial``、 ``Cuda``、 ``OpenMP``、 ``Threads``、``HIP``　および ``OpenMPTarget``　に共通する機能性を基盤として、　Kokkos |ExecutionSpaceTwo|_ 概念の現状は以下のようなものとなります:

.. code-block:: cpp

    テンプレート <typename Ex>
    概念 ExecutionSpace =
        std::regular<Ex> &&
        // Member type requirements:
        requires() {
            typename Ex::execution_space;
            std::is_same_v<Ex, typename Ex::execution_space>;
            typename Ex::memory_space;
            is_memory_space<typename Ex::memory_space>::value;
            typename Ex::size_type;
            std::is_integral_v<typename Ex::size_type>;
            typename Ex::array_layout;
            is_array_layout<typename Ex::array_layout>::value;
            typename Ex::scratch_memory_space;
            is_memory_space<typename Ex::scratch_memory_space>::value;
            typename Ex::device_type;
        } &&
        // 必要なメソッド:
        requires(Ex ex, std::string str, std::ostream& ostr, bool detail) {
            { ex.fence(str) };
            { ex.fence() };
            { ex.name() } -> const char*;
            { ex.concurrency() } -> int;
            { ex.print_configuration(ostr) };
            { ex.print_configuration(ostr, detail) };
        } &&

ここでは、実行スペースインスタンスに関する最近の進捗から、現在スタティックメソッドとして実装されている多くのメソッドが、一般的なケースでは最終的にインスタンスメソッドのみを必要とすることが推測されます。

実装必要要件
~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. _Kokkos_parallel_for: parallel-dispatch/parallel_for.html

.. |Kokkos_parallel_for| replace:: ``Kokkos::parallel_for``

追加の概念によって制約された追加の型なしでは、さらなる要件を表現することはできません
（これは、C++　の概念機構におけるよく知られた制限であり、型システムの決定可能性を維持するために必要です）。
この問題を回避するためにアーキタイプパターンを使用すべきだと主張する人もいますが　(　これにより、追加の概念の要件を満たすように設計された実装非公開の名前を持つアーキタイプが制約の定義で使用されます）、
実践の状況は、関連する全ての型をテンプレート化した追加の命名済み概念を作成し、それらをまとめて制約する戦略に収束しつつあるように見え、その後、関連する呼び出し元で使用可能となります。
多くの人は、これは言語機能の必然的な副産物であると主張しますが、このような方法で概念を一緒に制約することは、認知負荷評価の目的において、"追加の"　概念とは考えられません。
このアプローチを適用し、|Kokkos_parallel_for|_　のようなものがカスタマイズポイントではなくアルゴリズムとして残ることを意図していると仮定すると、 ``Kokkos::Impl`` 名前空間からいくつかの追加要件を取得します:

.. code-block:: cpp

    テンプレート <typename Ex, typename ExPol, typename F, typename ResultType = int>
    概念 ExecutionSpaceOf =
        ExecutionSpace<Ex> &&
        ExecutionPolicyOf<ExPol, Ex> && // defined below
        Functor<F> && // defined below
        // Requirements imposed by Kokkos_Parallel.hpp
        requires(Ex ex, ExPol const& policy, F f, ResultType& total) {
            // This is technically not exactly correct, since an rvalue reference qualified
            // execute() method would meet these requirements and wouldn't work with Kokkos,
            // but for brevity:
            { Impl::ParallelFor<F, ExPol, Ex>(f, policy).execute(); }
            { Impl::ParallelScan<F, ExPol, Ex>(f, policy).execute(); }
            { Impl::ParallelScanWithTotal<F, ExPol, Ex>(f, policy, total).execute(); }
        }

    テンプレート <typename Ex, typename ExPol, typename F, typename Red>
    concept ExecutionSpaceOfReduction =
        ExecutionSpaceOf<Ex, ExPol, F> &&
        Reducer<Red> &&
        // Requirements imposed by Kokkos_Parallel_Reduce.hpp
        requires(
            Ex ex, ExPol const& policy, F f, Red red,
            Impl::ParallelReduce<F, ExPol, Red, Ex>& closure
        ) {
            { Impl::ParallelReduce<F, ExPol, Red, Ex>(f, policy); }
            { closure.execute(); }
        }

ただし、おそらく、これらは内部の概念の一部であるべきかもしれません (例えば、　``ImplExecutionSpaceOf``　)。
そして、ユーザーに表示されるコンセプトからは、これらの要件を除外すべきです。

 ``UniqueToken`` のサポートには、以下の要件が追加されます:

.. code-block:: cpp

    template <typename Ex>
    concept UniqueTokenExecutionSpace =
        requires(
            Experimental::UniqueToken<Ex, Experimental::UniqueTokenScope::Instance> const& itok,
            Experimental::UniqueToken<Ex, Experimental::UniqueTokenScope::Global> const& gtok,
            typename Ex::size_type size
        ) {
            typename Experimental::UniqueToken<Ex, Experimental::UniqueTokenScope::Global>::size_type;
            std::is_same_v<Ex, typename Experimental::UniqueToken<Ex, Experimental::UniqueTokenScope::Global>::execution_space>;
            { itok.size() } -> typename Ex::size_type;
            { gtok.size() } -> typename Ex::size_type;
            { itok.acquire() } -> typename Ex::size_type;
            { gtok.acquire() } -> typename Ex::size_type;
            { itok.release(size) };
            { gtok.release(size) };
        }
        && CopyConstructible<Experimental::UniqueToken<Ex, Experimental::UniqueTokenScope::Instance>>
        && DefaultConstructible<Experimental::UniqueToken<Ex, Experimental::UniqueTokenScope::Instance>>
        && CopyConstructible<Experimental::UniqueToken<Ex, Experimental::UniqueTokenScope::Global>>
        && DefaultConstructible<Experimental::UniqueToken<Ex, Experimental::UniqueTokenScope::Global>>;

一部の　*事実上*　の要件
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

``Impl::TeamPolicyInternal``　等、具体的な実行空間を使用して部分的な特殊化を提供している他の箇所も存在します。 これらも``ExecutionSpace``上の「要件」として扱われ、これは``Impl::ParallelFor<...>``と同様です。 こうしたケースの多くでは、部分クラス型テンプレートの特化という、　"すべてか無か"　的なカスタマイズ手法よりも、より柔軟なアプローチを採用できるようリファクタリングできれば理想的です。

 ``MemorySpace`` 概念
---------------------------

Looking at the common functionality in the current implementations of現在の実装における ``CudaSpace``、 ``CudaUVMSpace``、``HostSpace`` および ``OpenMPTargetSpace``　の共通機能を見れば、 ``MemorySpace`` の現在の概念は、以下の様に見えます:

.. code-block:: cpp

    テンプレート <typename Mem>
    concept MemorySpace =
        CopyConstructible<Mem> &&
        DefaultConstructible<Mem> &&
        Destructible<Mem> &&
        // Member type requirements:
        requires() {
            std::is_same_v<Mem, typename Mem::memory_space>;
            Kokkos::is_execution_space<typename Mem::execution_space>::value;
            typename Mem::device_type;
        }
        // 必要なメソッド:
        requires(Mem m, size_t size, void* ptr) {
            { m.name() } -> const char*;
            { m.allocate(size) } -> void*;
            { m.deallocate(ptr, size) };
        };

実装必要要件
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Kokkos　が汎用的な文脈で、　``MemorySpace``　概念を使用するほとんどの方法が、``Impl``　名前空間内で行われています。

.. code-block:: cpp

    テンプレート <typename Mem>
    concept ImplMemorySpace =
        MemorySpace<Mem> &&
        DefaultConstructible<Impl::SharedAllocationRecord<Mem, void>> &&
        Destructible<Impl::SharedAllocationRecord<Mem, void>>
        requires(
            Mem mem, std::string label, size_t size,
            void* ptr, std::ostream& ostr, bool detail,
            Impl::SharedAllocationRecord<Mem, void> record,
            void (*dealloc)(Impl::SharedAllocationRecord<void, void>*)
        ) {
            { Impl::SharedAllocationRecord<Mem, void>(mem, label, size) };
            { Impl::SharedAllocationRecord<Mem, void>(mem, label, size, dealloc) };
            { record.get_label() } -> std::string;
            { Impl::SharedAllocationRecord<Mem, void>::allocate_tracked(mem, label, size) }
            -> void*;
            { Impl::SharedAllocationRecord<Mem, void>::reallocate_tracked(ptr, size) }
            -> void*;
            { Impl::SharedAllocationRecord<Mem, void>::deallocate_tracked(ptr) };
            { Impl::SharedAllocationRecord<Mem, void>::print_records(ostr, mem) };
            { Impl::SharedAllocationRecord<Mem, void>::print_records(ostr, mem, detail) };
            { Impl::SharedAllocationRecord<Mem, void>::get_record(ptr) }
            -> Impl::SharedAllocationRecord<Mem, void>*
        };

    テンプレート <typename Mem1, typename Mem2, typename Ex>
    概念　ImplRelatableMemorySpaces =
        ImplMemorySpace<Mem1> &&
        ImplMemorySpace<Mem2> &&
        ExecutionSpace<Ex> &&
        requires(const void* ptr) {
            { Impl::MemorySpaceAccess<Mem1, Mem2>::assignable } -> bool;
            { Impl::MemorySpaceAccess<Mem1, Mem2>::accessible } -> bool;
            { Impl::MemorySpaceAccess<Mem1, Mem2>::deepcopy } -> bool;
            { Impl::VerifyExecutionCanAccessMemorySpace<Mem1, Mem2>::value } -> bool;
            { Impl::VerifyExecutionCanAccessMemorySpace<Mem1, Mem2>::verify() };
            { Impl::VerifyExecutionCanAccessMemorySpace<Mem1, Mem2>::verify(ptr) };
        } &&
        requires(Ex ex, void* dst, const void* src, size_t n) {
            { Impl::DeepCopy<Mem1, Mem2, Ex>(dst, src, n) };
            { Impl::DeepCopy<Mem1, Mem2, Ex>(exec, dst, src, n) };
        }

 ``ExecutionPolicy`` 概念
-------------------------------

これが、私たちが最も力を入れるべき点だと考えます。例えば、``RangePolicy<...>``　および　``ThreadVectorRange<...>``　といった異なるインターフェースを単一の階層構造に統合することで、大幅な複雑性の削減を実現できます。

``RangePolicy<...>``、 ``MDRangePolicy<...>``、 ``TeamPolicy``、
``Impl::TeamThreadRangeBoundariesStruct``、 および ``Impl::TeamVectorRangeBoundariesStruct``　の現在の実装について見てみると、 共通点として認められるものは以下の通りです:

.. code-block:: cpp

    テンプレート <typename ExPol>
    concept BasicExecutionPolicy =
        CopyConstructible<ExPol> &&
        Destructible<ExPol> &&
        requires(ExPol ex) {
            ExPol::index_type;
        }

それは、もちろん、有用な概念ではありません。
 ``Impl::TeamThreadRangeBoundariesStruct`` および ``Impl::TeamVectorRangeBoundariesStruct``　を除外すれば、 また以下のタグを取得します:

.. code-block:: cpp

    テンプレート <typename ExPol>
    concept ExecutionPolicy =
        BasicExecutionPolicy<ExPol> &&
        requires(ExPol ex) {
            std::is_same_v<ExPol, typename ExPol::execution_policy>;
        }

それは、他のアルゴリズム内部で並列アルゴリズムに与えられるポリシーが、他のものと同じ概念の一部となることを目的としていなかったことを示しています　(ただし、私はおそらくそれらが一部となることを目的としているべきであると主張していますが)。 ``TeamPolicy`` および　``RangePolicy`` 双方とも、チャンクサイズ管理関数を持ちます:

.. code-block:: cpp

    テンプレート <typename ExPol>
    concept ChunkedExecutionPolicy =
        ExecutionPolicy<ExPol> &&
        requires(ExPol ex, typename ExPol::index_type size) {
            { ex.chunk_size() } -> typename ExPol::index_type;
            { ex.set_chunk_size(size) } -> ExPol&
        }

チャンクサイズは、もちろん　``MDRangePolicy``　では少し複雑になりますが、各次元におけるチャンクへの一般化は、かなり単純明快であるため、ここで概念を少し統一できました。 
``IterateTile`` 抽象化は、かなり優れたもので、 ``impl/Kokkos_OpenMP_Parallel.hpp``のような場所での重複コード量を削減するために、これらの概念を統合できる可能性があるように思われます。

それぞれ、ユーザーが　``RangePolicy`` をランク1の ``MDRangePolicy`` の特殊ケースとして考えることを可能にすることで、およびユーザーが ``RangePolicy`` を、各チームが1つの　``N``　個のチームから成る　``TeamPolicy`` の特殊なケースとして考えることを可能にすることにより、概念的な表面積を縮小する方法があれば理想的です。もちろん、 現在のインターフェースはショートカットとして引き続き提供し、
おそらく現在の方法で教えることになりますが、ユーザーがこれらのすべてを使いこなす段階に進むと、 三つの異なる事柄ではなく、一つの事柄を二つの異なる軸で考えられるようにすることが、理想的です。

最後に、なぜ　``TeamThreadRange`` および ``ThreadVectorRange``という別々の概念が必要なのか、私は完全には理解できていません。
複数のレベルで入れ子化された並列処理は、実行ポリシーの概念を拡張するための単なる別の軸に過ぎないものであると考えており、
そして、その階層構造における特定のポイントを記述するために、なぜ余分な概念的なオーバーヘッドを消費する必要があるのか、完全には理解できていません。
(繰り返しますが、名前そのものには特に異論はあるわけではなく、ただ、余計な認知負荷がかかるだけであると申し上げます。)

ここには、大幅な単純化を行う余地が全くないという可能性があります。
おそらく現在の関心の分離が、考え得る限り最も単純な形であるのでしょう。
しかしながら、Kokkos の概念を強化する方向で検討している以上、少なくともこの領域を探求する必要があります。

 ``TeamMember`` 概念
--------------------------

TODO

 ``Functor`` 概念
-----------------------

TODO

実装委任に関する注意事項
-----------------------------------

TODO
