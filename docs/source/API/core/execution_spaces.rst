.. _api-execution-spaces:

実行空間
================

.. role:: cpp(code)
    :language: cpp

.. _ExecutionSpaceConceptType: #kokkos-executionspaceconcept

.. |ExecutionSpaceConceptType| 置換:: :cpp:func:`ExecutionSpace` 型

.. _DocExecutionSpaceConcept: #kokkos-executionspaceconcept

.. |DocExecutionSpaceConcept| 置換:: the documentation on the :cpp:func:`ExecutionSpace` 概念

.. _Experimental: utilities/experimental.html#experimentalnamespace

.. |Experimental| 置換:: 実験的

.. _KokkosConcepts: KokkosConcepts.html

.. |KokkosConcepts| 置換:: 本文書

.. _ExecutionSpaceS: #kokkos-executionspaceconcept

.. |ExecutionSpaceS| 置換:: :cpp:func:`ExecutionSpace` s

.. _MemorySpace: memory_spaces.html#kokkos-memoryspaceconcept

.. |MemorySpace| 置換:: :cpp:func:`MemorySpace`

.. _KokkosSpaceAccessibility: SpaceAccessibility.html

.. |KokkosSpaceAccessibility| 置換:: :cpp:func:`Kokkos::SpaceAccessibility`

.. _KokkosTeamPolicy: policies/TeamPolicy.html

.. |KokkosTeamPolicy| 置換:: :cpp:func:`Kokkos::TeamPolicy`

.. _ExecutionSpaceConcept: #kokkos-executionspaceconcept

.. |ExecutionSpaceConcept| 置換:: :cpp:func:`ExecutionSpaceConcept`

``Kokkos::Cuda``
----------------

``Kokkos::Cuda`` は、Cudaデバイス上での実行を表す |ExecutionSpaceConceptType|_ です。
ごく稀な場合を除き、直接使用すべきではなく、代わりに汎用的な実行空間として使用される必要があります。　詳細については、|DocExecutionSpaceConcept|_　を参照してください。


``Kokkos::HIP``
---------------

:sup: |Experimental|_ :sup:`バージョン4.0以降` `昇格した`　``Kokkos::HIP`` は、HIPがサポートするデバイス上での実行を表す　|ExecutionSpaceConceptType|_ です。ごく稀な場合を除き、直接使用すべきではなく、代わりに汎用的な実行空間として使用される必要があります。　詳細については、|DocExecutionSpaceConcept|_　を参照してください。

``Kokkos::SYCL``
------------------------------

:sup:`promoted from` |Experimental|_ :sup:`バージョン4.5以降`  `昇格した`　``Kokkos::SYCL`` は、 SYCLがサポートするデバイス上での実行を表す|ExecutionSpaceConceptType|_　です。

SYCL　バックエンドが有効化され、　GPU　アーキテクチャが指定されていない場合、　Kokkos　は、特定の　SYCL　デバイスタイプへ、制限なしで、Just-In-Timeコンパイルを使用します。
したがって、SYCLバックエンド（実験的、未検証、最適化されていない）で、CPU　をターゲットにする唯一の選択肢がこれです。

``Kokkos::HPX``
---------------

``Kokkos::HPX`` は、HPXランタイムシステムによる実行を表す |ExecutionSpaceConceptType|_ です。
ごく稀な場合を除き、直接使用すべきではなく、代わりに汎用的な実行空間として使用される必要があります。
詳細については、|DocExecutionSpaceConcept|_　を参照してください。

``Kokkos::OpenMP``
------------------

``Kokkos::OpenMP`` は、OpenMP　ランタイムシステムによる実行を表す　|ExecutionSpaceConceptType|_です。
ごく稀な場合を除き、直接使用すべきではなく、代わりに汎用的な実行空間として使用される必要があります。 詳細については、|DocExecutionSpaceConcept|_　を参照してください。

``Kokkos::OpenMPTarget``
------------------------

``Kokkos::OpenMPTarget`` は、OpenMP　ランタイムシステムのターゲットオフロード機能による実行を表す　|ExecutionSpaceConceptType|_　です。ごく稀な場合を除き、直接使用すべきではなく、代わりに汎用的な実行空間として使用される必要があります。 詳細については、|DocExecutionSpaceConcept|_　を参照してください。

``Kokkos::Threads``
-------------------

``Kokkos::Threads`` は、std::threads による並列実行を表す |ExecutionSpaceConceptType|_ です。
ごく稀な場合を除き、直接使用すべきではなく、代わりに汎用的な実行空間として使用される必要があります。 詳細については、|DocExecutionSpaceConcept|_　を参照してください。

``Kokkos::Serial``
------------------

``Kokkos::Serial`` は、CPU上でのシリアル実行を表す|ExecutionSpaceConceptType|_　です。
ごく稀な場合を除き、直接使用すべきではなく、代わりに汎用的な実行空間として使用される必要があります。 詳細については、|DocExecutionSpaceConcept|_　を参照してください。

``Kokkos::ExecutionSpaceConcept``
---------------------------------

ExecutionSpace`` の概念は、Kokkos　における実行の　"場所"　と "方法"　を表すための、基本的な抽象化です。Kokkos　を使用するコードの大半は、特定のインスタンスよりもむしろ、``実行空間``　という　*汎用的な概念*　について、記述される必要があります。 このページでは、Kokkos　の実行空間の一般的な機能を　実際にどのように*使用*　するかを説明しています；より正式で理論的な処理については、 |KokkosConcepts|_　を参照してください。

  *免責事項*: 　C++における　"概念"　という用語に目新しい点はありません; C++でテンプレートを使ったことがある人は
知っていようといまいと、コンセプトを使っています。「概念」という言葉自体に惑わされないでください。
この言葉は現在、C++20の新たな言語機能と結びつけられることが多くなっています。 ここで　"概念"　とは単に
"特定の場所でテンプレートパラメータとなる型に対してできること"　を意味します。

設定に基づく別名
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

``Kokkos::DefaultExecutionSpace``
---------------------------------

``Kokkos::DefaultExecutionSpace`` は、|ExecutionSpaceConceptType|_ の別名であり、Kokkos の現在の構成に基づく 
``ExecutionSpace``　を指します。これは階層　``device,host-parallel,host-serial``　で利用可能な最上位に設定されます。
また、|ExecutionSpaceConceptType|_ のオプションで指定可能なテンプレートパラメータのデフォルト値としても、機能します。

``Kokkos::DefaultHostExecutionSpace``
-------------------------------------

``Kokkos::DefaultHostExecutionSpace`` は、|ExecutionSpaceConceptType|_ の別名であり、Kokkos の現在の構成に基づく ``ExecutionSpace`` を指しています。 これは階層　``device,host-parallel,host-serial``　で利用可能な最上位に設定されます。

最も簡単な使い方：全くない？
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Kokkos　を使い始めたばかりの頃、|ExecutionSpaceS|_　が明示的に使用される場所についての（意外な）答えは、"どこにもない"　です。
多くのユーザーが最初に学ぶことの多くは、"デフォルトの実行空間を使ってこの操作を行う"　ための　"ショートカット"　であり、
それはビルドシステムフラグに基づいて定義された ``Kokkos::DefaultExecutionSpace`` という名前の型エイリアス（別名、 ``typedef``）です。 例えば、

.. code-block:: cpp

    Kokkos::parallel_for(
        42,
        KOKKOS_LAMBDA (int n) { /* ... */ }
    );は、

以下の　"ショートカット"　です。

.. code-block:: cpp

    Kokkos::parallel_for(
        Kokkos::RangePolicy<Kokkos::DefaultExecutionSpace>(
            Kokkos::DefaultExecutionSpace(), 0, 42
        ),
        KOKKOS_LAMBDA(int n) { /* ... */ }
    );

より一般的なもの
~~~~~~~~~~~~~~~~~~

しかし、中級および上級ユーザーにとっては、呼び出し元コードが、必要に応じてデフォルト以外の実行空間を渡すことができるように、実行空間に対して明示的に汎用性を備えたコードを書くことが良い慣行となる場合が多いです。
例えば、関数の簡易バージョンが以下である場合。

.. code-block:: cpp

    void my_function(Kokkos::View<double*> data, double scale) {
        Kokkos::parallel_for(
            data.extent(0),
            KOKKOS_LAMBDA (int n) {
                data(n) *= scale;
            }
        );
    }

その場合、より高度で柔軟なバージョンの関数は、以下の通りです:

.. code-block:: cpp

    テンプレート <class ExecSpace, class ViewType>
    void my_function(
    ExecSpace ex,
    ViewType data,
    double scale
    ) {
    static_assert(
        Kokkos::SpaceAccessibility<ExecSpace, typename ViewType::memory_space>::assignable,
        "Incompatible ViewType and ExecutionSpace"
    );
    Kokkos::parallel_for(
        Kokkos::RangePolicy<ExecSpace>(ex, 0, data.extent(0)),
        KOKKOS_LAMBDA (int n) {
        data(n) *= scale;
        }
    );
    }

上級ユーザーは、また後でコードを読む際に "shortcuts" を解釈するという追加の精神的負荷を避けるため、より明示的な形式を好む場合もあります。 *どこで*、*どのように*
Kokkos　の並列パターンが実行されているかを明示的に記述することは、冗長になる場合あっても、バグを減らす傾向があります。

機能性
~~~~~~~~~~~~~

すべての　``ExecutionSpace`` 型は、共通の機能セットを公開します。Kokkos　を使用する汎用コード（ほぼ全てのユーザーコード）において、
すべての実行空間タイプに共通ではない実行空間タイプのいかなる部分も、決して使用すべきではありません（そうしないと、コードの移植性が失われるリスクがあります）。いくつかの式は、あらゆる　``ExecutionSpace``　型に対して
確実に有効です。 ``ExecutionSpace``　型である型 ``Ex`` および
その型のインスタンス　``ex``　を前提とすれば、 Kokkos は、以下の式が指定された機能を提供することを保証します:

.. code-block:: cpp

    ex.name();

*戻り値:* 指定された `ExecutionSpace` インスタンス型に対して一意であることが保証された、`const char*` に変換可能な値。
*注意事項:* この関数が返すポインタは、``ExecutionSpace``　自体からはアクセスできない場合があります（例えば、デバイス上など）。; 使用には注意を払ってください。

.. code-block:: cpp

    ex.fence(str);

*効果:* 戻りの際に、インスタンス　``ex`` に対して実行されたすべての並列パターンおよび　deep_copy　呼び出しは、完了していることが保証され、その効果は呼び出し元スレッドに確実に反映されます。オプションの``str``引数により、Kokkos Tools　に報告されるイベントをカスタマイズできます。
*戻り値:* 無し。
*注意事項:* これは並列パターン内部から呼び出すことは*できません*。 そうすると不特定の影響が生じる可能性があります（つまり、動作する可能性はあるものの、特定の実行空間でのみ有効となるため、特に注意して実行しないようにしてください）。

.. code-block:: cpp

    ex.print_configuration(ostr);
    ex.print_configuration(ostr, detail);

ここで、``ostr``は ``std::ostream``　であり（例えば、 ``std::cout``　のように）、また ``detail`` は、詳細な説明を出力するかどうかを示すブール値です。

*効果:* ``ex`` の構成を与えられた既定の ``std::ostream`` への構成を出力します。
*Returns:* 無し。
*注意事項:* これは、並列パターン内から呼び出すことは、*出来ません*。

さらに、以下の型の別名 (別名 ``typedef`` ) は、すべての実行空間型により定義されます:

* ``Ex::memory_space``:  |MemorySpace|_ は ``Ex`` で実行する際に使用されます。Kokkos は ``Kokkos::SpaceAccessibility<Ex, Ex::memory_space>::accessible`` が ``true`` になることを保証します（|KokkosSpaceAccessibility|_ を参照）。

* ``Ex::array_layout``: デフォルトの　``ArrayLayout``　は、``Ex``　からアクセスされる　``View``　タイプとの併用に推奨される ``ArrayLayout``　。

* ``Ex::scratch_memory_space``: 並列パターンがスクラッチメモリの割り当てに使用する　``ScratchMemorySpace``（例：|KokkosTeamPolicy|_によって要求される場合）。このメモリ空間を使用して作成できるのは、管理対象外ビューのみです。

デフォルトコンストラクタの生成可能性、コピーコンストラクタの生成可能性
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

上記の機能に加え、Kokkos のすべての ``ExecutionSpace`` 型は、デフォルトのコンストラクタを生成可能であり（``Ex ex()`` として構築可能）、コピーのコンストラクタを生成可能（``Ex ex2(ex1)`` として構築可能）です。
``ExecutionSpace``型のデフォルトコンストラクタで生成されるすべてのインスタンスは、同等の動作を持つことが保証され、また、コピーによって構築されたすべてのインスタンスは、コピー元のインスタンスと同等の動作を持つことが保証されます。

検出
^^^^^^^^^

Kokkos　は、利便性型特性　``Kokkos::is_execution_space<T>``　を提供し、それは、型``T``が　``ExecutionSpace``　コンセプトの要件を満たす場合にのみ、``true``　である、コンパイル時にアクセス可能な値　``value``　を持ち（``Kokkos::is_execution_space<T>::value``として使用可能）、``true``を返します。任意の　``ExecutionSpace``　型 ``T`` は、式 ``Kokkos::is_space<T>::value`` がコンパイル時定数として、``true``　を評価する性質も持ちます。

シノプシス
~~~~~~~~

.. code-block:: cpp

    // これは実際のクラスではなく、概念を簡略に記述したものです。
    クラス ExecutionSpaceConcept {
    パブリック:
        typedef ExecutionSpaceConcept execution_space;
        typedef ... memory_space;
        typedef Device<execution_space, memory_space> device_type;
        typedef ... scratch_memory_space;
        typedef ... array_layout;
        typedef ... size_type;

        ExecutionSpaceConcept();
        ExecutionSpaceConcept(const ExecutionSpaceConcept& src);

        const char* name() const;
        void print_configuration(std::ostream ostr&) const;
        void print_configuration(std::ostream ostr&, bool details) const;

        int concurrency() const;

        void fence(const std::string&) const;
        void fence() const;

        friend bool operator==(const execution_space& lhs, const execution_space& rhs);
    };

    template<class MS>
    struct is_execution_space {
    enum { value = false };
    };

    template<>
    struct is_execution_space<ExecutionSpaceConcept> {
    enum { value = true };
    };

型定義
~~~~~~~~

* ``execution_space``: 自己型;

* ``memory_space``: |ExecutionSpaceConcept|_で実行する際に使用するデフォルトの |MemorySpace|_。 Kokkos　は、``Kokkos::SpaceAccessibility<Ex, Ex::memory_space>::accessible``が``true``となることを保証します（|KokkosSpaceAccessibility|_を参照）。

* ``device_type``: ``DeviceType<execution_space,memory_space>``.

* ``array_layout``: |ExecutionSpaceConcept|　からアクセスされる　|View|型　で使用することを推奨されるデフォルトの　``ArrayLayout``　。

* ``scratch_memory_space``: 並列パターンがスクラッチメモリの割り当てに使用する　``ScratchMemorySpace``（例：　|KokkosTeamPolicy|_　で要求される場合）。このメモリ空間を使用して作成できるのは、管理対象外ビューのみです。

* ``size_type``: このスペースに関連付けられたデフォルトの整数型。符号付きまたは符号なし、32ビットまたは64ビットの整数型で、インデックス付けの優先型として使用されます。

コンストラクタ
~~~~~~~~~~~~

* ``ExecutionSpaceConcept()``: デフォルトコンストラクタ。

* ``ExecutionSpaceConcept(const ExecutionSpaceConcept& src)``: コピーコンストラクタ。

関数
~~~~~~~~~

* ``const char* name() const;``: 実行空間のインスタンスのラベルを　*返します*　。

* ``int concurrency() const;`` 並列環境における同時に実行される作業項目の最大数、すなわち実行空間インスタンスが使用するスレッドの最大数を　*返します*　。

* ``void fence(const std::string& label = unspecified-default-value) const;`` *効果:* 返す際に、 インスタンス |ExecutionSpaceConcept|_ 上で実行されたすべての並列パターンは、完了が保証され、そしてそれらの効果は呼び出し元スレッドから確実に可視化されます。*注意事項:* これは、並列パターン内部から呼び出すことは、*できません*　。そうすると、特定されない影響が生じる可能性があります（つまり、動作の可能性はありますが、一部の実行空間のみで可能です。そのため、絶対に実行しないよう、特に注意してください）。 オプションの　``label``　引数により、Kokkos Tools　に報告されるイベントをカスタマイズできます。

* ``void print_configuration(std::ostream ostr) const;``: *効果:* 既定の　``std::ostream``　に　``ex``　の設定を出力します。 *注意事項:* これは、並列パターン内部から呼び出すことは、*できません*　。

非メンバーファシリティ
~~~~~~~~~~~~~~~~~~~~~

* ``template<class MS> struct is_execution_space;``: クラスが実行空間であるかどうかを確認するための型特性。

* ``template<class S1, class S2> struct SpaceAccessibility;``: は、2つのスペースが互換性があるか（割り当て可能、deep_copy可能、アクセス可能）を確認するための型特性。 ( |KokkosSpaceAccessibility|_　を参照。)

* ``bool operator==(const execution_space& lhs, const execution_space& rhs)``: 2つの空間インスタンス（同じ型）が同一であるかどうかをテスト。
