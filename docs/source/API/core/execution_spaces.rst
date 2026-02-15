.. _api-execution-spaces:

Execution Spaces実行空間
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

``ExecutionSpace`` の概念は、Kokkos　における実行の　"場所"　と "方法"　を表すための、基本的な抽象化です。Kokkos　を使用するコードの大半は、特定のインスタンスよりもむしろ、「実行空間」という　*汎用的な概念*　について、記述される必要があります。 このページでは、Kokkos　の実行空間の一般的な機能を　実際にどのように*使用*　するかを説明しています；より正式で理論的な処理については、 |KokkosConcepts|_　を参照してください。

  *免責事項*: 　C++における　"概念"　という用語に目新しい点はありません; C++でテンプレートを使ったことがある人は
知っているいようといまいと、コンセプトを使っています。「概念」という言葉自体に惑わされないでください。
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

*Returns:* a value convertible to ``const char*`` that is guaranteed to be unique to a given ``ExecutionSpace`` instance type.
*Note:* the pointer returned by this function may not be accessible from the ``ExecutionSpace`` itself (for instance, on a device); use with caution.

.. code-block:: cpp

    ex.fence(str);

*Effects:* Upon return, all parallel patterns and deep_copy calls executed on the instance ``ex`` are guaranteed to have completed, and their effects are guaranteed visible to the calling thread. The optiopnal ``str`` argument allows customizing the event reported to Kokkos Tools.
*Returns:* Nothing.
*Note:* This *cannot* be called from within a parallel pattern.  Doing so will lead to unspecified effects (i.e., it might work, but only for some execution spaces, so be extra careful not to do it).

.. code-block:: cpp

    ex.print_configuration(ostr);
    ex.print_configuration(ostr, detail);

where ``ostr`` is a ``std::ostream`` (like ``std::cout``, for instance) and ``detail`` is a boolean indicating whether a detailed description should be printed.

*Effects:* Outputs the configuration of ``ex`` to the given ``std::ostream``.
*Returns:* Nothing.
*Note:* This *cannot* be called from within a parallel pattern.

Additionally, the following type aliases (a.k.a. ``typedef`` s) will be defined by all execution space types:

* ``Ex::memory_space``: the default |MemorySpace|_ to use when executing with ``Ex``. Kokkos guarantees that ``Kokkos::SpaceAccessibility<Ex, Ex::memory_space>::accessible`` will be ``true`` (see |KokkosSpaceAccessibility|_)

* ``Ex::array_layout``: the default ``ArrayLayout`` recommended for use with ``View`` types accessed from ``Ex``.

* ``Ex::scratch_memory_space``: the ``ScratchMemorySpace`` that parallel patterns will use for allocation of scratch memory (for instance, as requested by a |KokkosTeamPolicy|_). Only unmanaged Views can be created using this memory space.

Default Constructibility, Copy Constructibility
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

In addition to the above functionality, all ``ExecutionSpace`` types in Kokkos are default
constructible (you can construct them as ``Ex ex()``) and copy constructible (you can construct them as ``Ex ex2(ex1)``).
All default constructible instances of an ``ExecutionSpace`` type are guaranteed to have equivalent behavior,
and all copy constructed instances are guaranteed to have equivalent behavior to the instance they were copied from.

Detection
^^^^^^^^^

Kokkos provides the convenience type trait ``Kokkos::is_execution_space<T>`` which has a ``value`` compile-time
accessible value (usable as ``Kokkos::is_execution_space<T>::value``) that is ``true`` if and only if a type ``T``
meets the requirements of the ``ExecutionSpace`` concept. Any ``ExecutionSpace`` type ``T`` will also
have the expression ``Kokkos::is_space<T>::value`` evaluate to ``true`` as a compile-time constant.

Synopsis
~~~~~~~~

.. code-block:: cpp

    // This is not an actual class, it just describes the concept in shorthand
    class ExecutionSpaceConcept {
    public:
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

Typedefs
~~~~~~~~

* ``execution_space``: The self type;

* ``memory_space``: The default |MemorySpace|_ to use when executing with |ExecutionSpaceConcept|_. Kokkos guarantees that ``Kokkos::SpaceAccessibility<Ex, Ex::memory_space>::accessible`` will be ``true`` (see |KokkosSpaceAccessibility|_)

* ``device_type``: ``DeviceType<execution_space,memory_space>``.

* ``array_layout``: The default ``ArrayLayout`` recommended for use with ``View`` types accessed from |ExecutionSpaceConcept|_.

* ``scratch_memory_space``: The ``ScratchMemorySpace`` that parallel patterns will use for allocation of scratch memory (for instance, as requested by a |KokkosTeamPolicy|_). Only unmanaged Views can be created using this memory space.

* ``size_type``: The default integer type associated with this space. Signed or unsigned, 32 or 64 bit integer type, used as preferred type for indexing.

Constructors
~~~~~~~~~~~~

* ``ExecutionSpaceConcept()``: Default constructor.

* ``ExecutionSpaceConcept(const ExecutionSpaceConcept& src)``: Copy constructor.

Functions
~~~~~~~~~

* ``const char* name() const;``: *Returns* the label of the execution space instance.

* ``int concurrency() const;`` *Returns* the maximum amount of concurrently executing work items in a parallel setting, i.e. the maximum number of threads utilized by an execution space instance.

* ``void fence(const std::string& label = unspecified-default-value) const;`` *Effects:* Upon return, all parallel patterns executed on the instance |ExecutionSpaceConcept|_ are guaranteed to have completed, and their effects are guaranteed visible to the calling thread. *Note:* This *cannot* be called from within a parallel pattern. Doing so will lead to unspecified effects (i.e., it might work, but only for some execution spaces, so be extra careful not to do it). The optional ``label`` argument allows customizing the event reported to Kokkos Tools.

* ``void print_configuration(std::ostream ostr) const;``: *Effects:* Outputs the configuration of ``ex`` to the given ``std::ostream``. *Note:* This *cannot* be called from within a parallel pattern.

Non Member Facilities
~~~~~~~~~~~~~~~~~~~~~

* ``template<class MS> struct is_execution_space;``: typetrait to check whether a class is a execution space.

* ``template<class S1, class S2> struct SpaceAccessibility;``: typetraits to check whether two spaces are compatible (assignable, deep_copy-able, accessible). (see |KokkosSpaceAccessibility|_)

* ``bool operator==(const execution_space& lhs, const execution_space& rhs)``: tests whether the two space instances (of the same type) are identical.
