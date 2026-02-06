
.. role:: cpp(code)
   :language: cpp

``partition_space``
===================

ヘッダーファイル: ``<Kokkos_Core.hpp>``

.. warning::

   現在、``partition_space`` はまだ 名前空間　``Kokkos::Experimental`` にあります。

使用例
-----

.. code-block:: c++

   自動インスタンス = Kokkos::partition_space(Kokkos::DefaultExecutionSpace(),1,1,1);

インターフェイス
---------

.. cpp:function:: template<class ExecSpace, class ... Args> std::array<ExecSpace, sizeof...(Args)> partition_space(const ExecSpace& space, Args...args);

.. cpp:function:: template<class ExecSpace, class T> std::vector<ExecSpace> partition_space(const ExecSpace& space, std::vector<T> const& weights);

   既存の実行空間インスタンスと同じ基盤となるハードウェアリソースにディスパッチする
   新しい実行空間インスタンスを作成します。
   新しく作成されたインスタンスと既存のインスタンスの間には、暗示的同期関係は存在しません。

   :param space: 実行空間インスタンス (../execution_spaces.html　参照)

   :param args: 作成されたインスタンス数は、``sizeof...(Args)``　に等しい.
		``args``　の相対的な重みは、新しく作成される各インスタンスに関連付ける
		``space``　のハードウェアリソースの割合に関するヒントです。

   :param weights:  算術型 ``T``  の ``std::vector`` で、新しく作成される、``space``　の各インスタンスに関連付ける。
                   ハードウェアリソースの割合のヒントを提供します

必要要件
~~~~~~~~~~~~

- ``(std::is_arithmetic_v<Args> && ...)`` is ``true``.

- ``std::is_arithmetic_v<T>`` is ``true``.

- ``ExecutionSpace().concurrency() >= N_PARTITIONS``


セマンティクス
~~~~~~~~~

- いずれのインスタンス間にも、暗示的同期関係は存在せず、特に:
  - ``instance[i]`` は、``space.fence()``　により囲まれていない、
  - ``instance[i]`` は、is not fenced by ``instance[j].fence()``　により囲まれておらず、そして
  - ``space``　は、 ``instance[i].fence()``　により囲まれていません。
  しかしながら、実際には、これらのインスタンスは、同じハードウェアリソースにディスパッチされるため、互いにブロックし合う可能性があります。

- `args``（または　``weights``　要素）の相対的な重みは、望ましいリソース配分に関するヒントとして使用されます。
  例えば、個別のスレッドを使用するバックエンドの場合、 ``{1,2}`` の重みの結果、2つのインスタンスが生じ、
  その1つ目は、元のインスタンスのスレッドの約3分の1であり、
  そして2番目は、3分の2を伴います。しかしながら、一部のバックエンドについては、それぞれ返されたインスタンスは、元のもののコピーである場合があります。

.. important::

   ``Cuda``　については、 それぞれ新たにインスタスを作成した　``HIP`` および ``SYCL``は、それ自身の *stream*/*queue*　と関連します。


例
--------

同時実行カーネルで使用するための既存インスタンスを分割。

.. code-block:: cpp

   template<class ExecSpace, class ... OtherParams>
   void foo(const ExecSpace& space, OtherParams...params) {
     auto [instance0, instance1] = Kokkos::partition_space(space,1,2);
     // dispatch two kernels, F1 needs less resources then F2
     // F1 and F2 may now execute concurrently
     Kokkos::parallel_for("F1",
       Kokkos::RangePolicy<ExecSpace>(instance0,0,N1),
       Functor1(params...));
     Kokkos::parallel_for("F2",
       Kokkos::RangePolicy<ExecSpace>(instance1,0,N2),
       Functor2(params...));

     // 両方を待機
     // 注意事項: space.fence() は、インスタンス実行をブロックしません。
     instance0.fence();
     instance1.fence();
     Kokkos::parallel_for("F3",
       Kokkos::RangePolicy<ExecSpace>(space,0,N3),
       Functor3(params...));
   }
