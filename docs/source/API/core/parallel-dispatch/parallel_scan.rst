``parallel_scan``
=================

.. role::cpp(code)
    :language: cpp

ヘッダーファイル: ``<Kokkos_Core.hpp>``

使用例
-----

.. code-block:: cpp

    Kokkos::parallel_scan( name, policy, functor, result );
    Kokkos::parallel_scan( name, policy, functor );
    Kokkos::parallel_scan( policy, functor, result);
    Kokkos::parallel_scan( policy, functor );

``functor``　で定義された並列作業を、*ExecutionPolicy* ``policy``　に従ってディスパッチし、作業項目が提供する成果物に対して排他的または包括的なスキャンを実行します。 オプションのラベル　``name`` はプロファイリングおよびデバッグツールで使用されます。提供された場合、最終結果は、 ``result``に格納されます。

インターフェイス
---------

.. cpp:function:: template <class ExecPolicy, class FunctorType> Kokkos::parallel_scan(const std::string& name, const ExecPolicy& policy, const FunctorType& functor);

.. cpp:function:: template <class ExecPolicy, class FunctorType> Kokkos::parallel_scan(const ExecPolicy&  policy, const FunctorType& functor);

.. cpp:function:: template <class ExecPolicy, class FunctorType, class ReturnType> Kokkos::parallel_scan(const std::string& name, const ExecPolicy&  policy, const FunctorType& functor, ReturnType& return_value);

.. cpp:function:: template <class ExecPolicy, class FunctorType, class ReturnType> Kokkos::parallel_scan(const ExecPolicy&  policy, const FunctorType& functor, ReturnType& return_value);

パラメータ:
~~~~~~~~~~~

* ``name``: ユーザーが提供した文字列で、Kokkos Profiling Hooksを介してプロファイリングおよびデバッグツールで使用されます。
* ExecPolicy: 反復空間およびその他の実行プロパティを定義する  *ExecutionPolicy* 　。 有効なポリシーは以下の通り:

  - ``IntegerType``: 1D反復範囲を定義し、0からカウント値までを範囲とします。
  - `RangePolicy <../policies/RangePolicy.html>`_: 1D反復範囲を定義します。
  - `TeamThreadRange <../policies/TeamThreadRange.html>`_: チーム内のスレッドにより実行されるべき1次元反復範囲を定義します。``TeamPolicy`` または ``TaskTeam``を通じて実行される並列領域内でのみ有効です。
  - `ThreadVectorRange <../policies/ThreadVectorRange.html>`_: チーム内のスレッドを分割するベクトル並列化を通じて実行されるべき1次元反復範囲を定義します。 ``TeamPolicy`` または ``TaskTeam`` を通じて実行される並列領域内でのみ有効です。
* FunctorType: 有効なファンクタで、（少なくとも）　 ``ExecPolicy`` と縮小型との組み合わせに対応するシグネチャを持つ　`operator()`` を備えるもの。
* ReturnType: ``operator +=`` および ``operator =``　を持つPOD型　または ``Kokkos::View``.

必要要件:
~~~~~~~~~~~~~

* ``functor`` は、 ``operator() (const HandleType& handle, ReturnType& value, const bool final) const`` または ``operator() (const WorkTag, const HandleType& handle, ReturnType& value, const bool final) const``　の形式のメンバー関数を持ちます。

  - ExecPolicy が IntegerType　または　ExecPolicy::work_tag が void　である場合、 WorkTag　を持たない　``operator()``　オーバーロードが使用されます。
  - HandleType は、ExecPolicy が IntegerType の場合、IntegerType であり、そうでない場合は ExecPolicy::member_type です
*   ``functor``　の　``ReturnType`` 型は、parallel_scanの ``ReturnType``　と互換性があり、提供されていれば、 ``init`` および ``join`` 関数の引数に一致しなければなりません。 ファクタが、 ``init`` メンバー関数を持たない場合には、 スキャン演算の同一性は、値型のデフォルトコンストラクタによって与えられるものと仮定されます。（`reduction_identity <../builtinreducers/reduction_identity.html>`_ によってではありません）。
* ファクタは、``ReturnType``　と同様に、 ``FunctorType::value_type`` を定義する必要があります。

セマンティクス
---------

* Neither concurrency nor order of execution are guaranteed.
* The ``ReturnType`` content will be overwritten, i.e. the value does not need to be initialized to the reduction-neutral element.
* The input value to the operator may contain a partial result, Kokkos may only combine the thread local contributions in the end. The operator should modify the input value according to the desired scan operation.
* For every element of the iteration space defined in ``policy`` the functors call operator is invoked exactly once with ``final = true``.
* It is not guaranteed that the functor will ever be called with ``final = false``.
* The functor might be called multiple times with ``final = false`` and the user has to make sure that the behavior in this case stays the same for repeated calls.

Examples
--------

.. code-block:: cpp

    #include<Kokkos_Core.hpp>
    #include<cstdio>

    int main(int argc, char* argv[]) {
      Kokkos::initialize(argc,argv);
      {
        int N = argc>1?atoi(argv[1]):100;
        int64_t result;
        Kokkos::View<int64_t*> in_scan("inclusive_scan",N);
        Kokkos::View<int64_t*> ex_scan("exclusive_scan",N);

        Kokkos::parallel_scan("Loop1", N,
          KOKKOS_LAMBDA(int64_t i, int64_t& partial_sum, bool is_final) {
          if (is_final) ex_scan(i) = partial_sum;
          partial_sum += i;
          if (is_final) in_scan(i) = partial_sum;
        }, result);

        // exclusive scan: 0,0,1,3,6,10,...
        // inclusive scan: 0,1,3,6,10,...
        // result: N*(N-1)/2
        printf("Result: %i %li\n", N, result);
      }
      Kokkos::finalize();
    }
