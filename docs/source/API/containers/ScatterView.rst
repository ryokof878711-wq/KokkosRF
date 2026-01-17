``ScatterView``
===============

.. role:: cpp(code)
	:language: cpp

ヘッダーファイル: ``<Kokkos_ScatterView.hpp>``

.. warning::

   ``ScatterView`` はKokkos::Experimental　という名前空間にまだ存在しています。


.. _parallelReduce: ../core/parallel-dispatch/parallel_reduce.html

.. |parallelReduce| replace:: :cpp:func:`parallel_reduce`

.. _View: ../core/view/view.html

.. |View| replace:: ``View``

.. |reset| replace:: ``reset()``

.. |access| replace:: ``access()``

.. |contribute| replace:: ``contribute()``

.. |create_scatter_view| replace:: ``create_scatter_view()``

使用例
-----

Kokkos ScatterView　は、標準的な　Kokkos::|View|_　のインターフェースとして機能し、原子またはデータレプリケーションを通じて散乱加算パターンを実装します。

ScatterView　の構築は  コストがかかることがあるため、可能であれば同じものを再利用するようにし、その場合は  使用の間に　|reset|_　を呼び出してください。

ScatterView　を直接アドレスすることはできません: 並列領域内の各スレッドは　|access|　に呼び出しを行い、|access|_　の返却値を通じて基礎となるViewにアクセスする必要があります。

並列領域の後、最終還元を行うために自由関数　Kokkos::Experimental::contribute()　を呼び出します。

インターフェイス
---------
.. code-block:: cpp

    template <typename DataType [, typename Layout [, typename ExecSpace [, typename Operation [, typename Duplication [, typename Contribution]]]]]>
    class ScatterView

パラメータ
~~~~~~~~~~
``DataType`` 以外のテンプレートパラメータは任意ですが、指定されている場合は、前のパラメータも指定しなければなりません。
つまり、例えば動作は省略できますが、指定されている場合は、　``Layout`` and ``ExecSpace`` も指定しなければなりません。

*  ``DataType``, ``Layout`` および ``ExecSpace``　は、Kokkos　のものと同じ型である必要があります::このScatterViewはインターフェースされています。

* ``Operation``:
  次の値を選択可能です:

  - ``Kokkos::Experimental::ScatterSum``: は、Sum　を実行します。それがデフォルトの値です。

  - ``Kokkos::Experimental::ScatterProd``: は、乗算を実行します。

  - ``Kokkos::Experimental::ScatterMin``: が、minを選択します。

  - ``Kokkos::Experimental::ScatterMax``: が、maxを選択します。

* ``Duplication``:
  Whether to duplicate the grid or not; defaults to ``Kokkos::Experimental::ScatterDuplicated``, other option is ``Kokkos::Experimental::ScatterNonDuplicated``.

* ``Contribution``:
  アトミック使用に貢献するかどうか；グリッドを複製するかどうか; デフォルトは、Kokkos::Experimental::ScatterDuplicated、もう一つの選択肢は、Kokkos::Experimental::ScatterNonDuplicated　です。

このインターフェースを使って、デフォルトでない``Operation``, ``Duplication`` または　``Contribution``　を持つ　ScatterView　を作成するのは複雑になることがあります。なぜなら、DataType、Layout、ExecSpaceの正確なタイプを指定する必要があるからです。 そのため、代わりに　Kokkos::Experimental::|create_scatter_view|_　という関数を使うことをお勧めします。

ディスクリプション
-----------

.. cpp:class:: template <typename DataType, typename Layout, typename ExecSpace, typename Op, typename Duplication, typename Contribution> ScatterView

    .. rubric:: パブリックメンバー変数

    .. cpp:type:: original_view_type

        コンストラクタに渡されたビュー型。

    .. cpp:type:: original_value_type

        Value type of the original_view_type　の値型。

    .. cpp:type:: original_reference_type

       original_view_type　の参照型。

    .. cpp:type:: data_type_info

        DuplicatedDataType で、指定されたビューデータ型から新たなランタイム次元を持つ、新規作成されたデータ型。この新たなディメンションが最大ストライドディメンションとなります。
    .. cpp:type:: internal_data_type

        data_type_info　の値型。

    .. cpp:type:: internal_view_type

        A View type created from the internal_data_type　から作成したビュー型。

    .. rubric:: コンストラクタ

    .. cpp:function:: ScatterView()

        デフォルトコンストラクタで、デフォルトがメンバーを構築します。

    .. cpp:function:: ScatterView(View<RT, RP...> const&)

        ``Kokkos::View``　からのコンストラクタ。 ``internal_view`` メンバーは、この入力ビューから構築されたコピーです。

    .. cpp:function:: ScatterView(std::string const& name, Dims ... dims)

        可変長のディメンション引数パックからのコンストラクタ。 ``internal_view`` member　を構築します。

    .. cpp:function:: ScatterView(ALLOC_PROP const& arg_prop, Dims... dims)

        可変長のディメンション引数パックからのコンストラクタ。 ``internal_view`` memberを構築します。
        このコンストラクタは、最初の引数として、``Kokkos::view_alloc``　で作成されたオブジェクトを渡すことを可能にします。例えば、実行空間を指定するために　``Kokkos::view_alloc(exec_space, 「label」)``　のように使用します。

    .. rubric:: パブリックメソッド

    .. cpp:function:: constexpr bool is_allocated() const

        :return: true if the ``internal_view`` points to a valid memory location. This function works for both managed and unmanaged views. With the unmanaged view, there is no guarantee that referenced address is valid, only that it is a non-null pointer.

    .. _access:

    .. cpp:function:: access() const

       use within a kernel to return a ``ScatterAccess`` member; this member accumulates a given thread's contribution to the reduction.

    .. cpp:function:: subview() const

        :return: a subview of a ``ScatterView``

    .. cpp:function:: contribute_into(View<DT, RP...> const& dest) const

       contribute ``ScatterView`` array's results into the input View ``dest``

    .. _reset:

    .. cpp:function:: reset()

       performs reset on destination array

    .. cpp:function:: reset_except(View<DT, RP...> const& view)

       tbd

    .. cpp:function:: resize(const size_t n0 = 0, const size_t n1 = 0, const size_t n2 = 0, const size_t n3 = 0, const size_t n4 = 0, const size_t n5 = 0, const size_t n6 = 0, const size_t n7 = 0)

       resize a view with copying old data to new data at the corresponding indices

    .. cpp:function:: realloc(const size_t n0 = 0, const size_t n1 = 0, const size_t n2 = 0, const size_t n3 = 0, const size_t n4 = 0, const size_t n5 = 0, const size_t n6 = 0, const size_t n7 = 0)

       resize a view with discarding old data


    .. rubric:: *Private* Members

    :member: typedef original_view_type internal_view_type;
    :member: internal_view_type internal_view;


.. rubric:: Free Functions

.. _create_scatter_view:

.. cpp:function:: template <typename Operation, typename Duplication, typename Contribution> create_scatter_view(const View<DT1, VP...>& view)

   create a new ScatterView interfacing the View ``view``.
   Default value for ``Operation`` is ``Kokkos::Experimental::ScatterSum``, ``Duplication`` and ``Contribution`` are chosen to make the ScatterView as efficient as possible when running on its ``ExecSpace``.

.. _contribute:

.. cpp:function:: contribute(View<DT1, VP...>& dest, Kokkos::Experimental::ScatterView<DT2, LY, ES, OP, CT, DP> const& src)

   convenience function to perform final reduction of ScatterView
   results into a resultant View; may be called following |parallelReduce|_.


Example
-------

.. code-block:: cpp


    #include <Kokkos_Core.hpp>
    #include <Kokkos_ScatterView.hpp>

    KOKKOS_INLINE_FUNCTION int foo(int i) { return i; }
    KOKKOS_INLINE_FUNCTION double bar(int i) { return i*i; }

    int main (int argc, char* argv[]) {
        Kokkos::ScopeGuard guard(argc, argv);

        Kokkos::View<double*> results("results", 1);
        auto scatter = Kokkos::Experimental::create_scatter_view(results);
        Kokkos::parallel_for(1, KOKKOS_LAMBDA(int input_i) {
            auto access = scatter.access();
            auto result_i = foo(input_i);
            auto contribution = bar(input_i);
            access(result_i) += contribution;
        });
        Kokkos::Experimental::contribute(results, scatter);
    }
