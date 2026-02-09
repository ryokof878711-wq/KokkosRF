``deep_copy``
=============

.. role:: cpp(code)
    :language: cpp

ヘッダーファイル: ``<Kokkos_Core.hpp>``

使用例
-----

.. code-block:: cpp

   Kokkos::deep_copy(exec_space, dest, src);
   Kokkos::deep_copy(dest, src);

データをCopies data from ``src`` から ``dest``　にコピーしますが、その中で ``src`` および ``dest``　が
特定状況下で、`Kokkos::Views <view.html>`_ またはスカラーである可能性があります。

インターフェイス
---------

.. cpp:function:: テンプレート <class ExecSpace, class ViewDest, class ViewSrc> void Kokkos::deep_copy(const ExecSpace& exec_space, const ViewDest& dest, const ViewSrc& src);

.. cpp:function:: テンプレート <class ExecSpace, class ViewDest> void Kokkos::deep_copy(const ExecSpace& exec_space, const ViewDest& dest, const typename ViewDest::value_type& src);

.. cpp:function:: テンプレート <class ExecSpace, class ViewSrc> void Kokkos::deep_copy(const ExecSpace& exec_space, ViewSrc::value_type& dest, const ViewSrc& src);

.. cpp:function:: テンプレート <class ViewDest, class ViewSrc> void Kokkos::deep_copy(const ViewDest& dest, const ViewSrc& src);

.. cpp:function:: テンプレート <class ViewDest> void Kokkos::deep_copy(const ViewDest& dest, const typename ViewDest::value_type& src);

.. cpp:function:: テンプレート <class ViewSrc> void Kokkos::deep_copy(ViewSrc::value_type& dest, const ViewSrc& src);

パラメータ
~~~~~~~~~~

* ExecSpace: An `ExecutionSpace <../execution_spaces.html>`_

* ViewDest: 非定数 ``value_type``　を伴う `view-like type <view_like.html>`_ 

* ViewSrc: `view-like type <view_like.html>`_.

必要要件
~~~~~~~~~~~~

* ``src`` および ``dest`` が `Kokkos::View <view.html>`_ である場合には、 以下のすべてが真です:

  - ``std::is_same<ViewDest::non_const_value_type, ViewSrc::non_const_value_type>::value == true``

  - ``src.rank == dest.rank`` (または  ``Kokkos::DynRankView`` については、 ``src.rank() == dest.rank()`` )

  - ``[0, dest.rank)``　内のすべての  ``k`` については、``dest.extent(k) == src.extent(k)`` (または、 ``dest.rank()``と同じ)

  -  ``SpaceAccessibility<copy_space, ViewDest::memory_space>::accessible == true`` および ``SpaceAccessibility<copy_space,ViewSrc::memory_space>::accessible == true``　両方であるための、``src.span_is_contiguous() && dest.span_is_contiguous() && std::is_same<ViewDest::array_layout,ViewSrc::array_layout>::value``、 *or* there exists an `ExecutionSpace <../execution_spaces.html>`_ ``copy_space`` (規定またはデフォルト)

* ``src`` is a `Kokkos::View <view.html>`_ and ``dest`` is a scalar, then ``src.rank == 0`` が真である場合には、

セマンティクス
---------

* `ExecutionSpace <../execution_spaces.html>`_ argument が提供されなければ、  いかなる実行空間のすべての優れた演算子　（カーネル、コピー演算子）は、コピーが実行される前に終了し、コピー演算子は、呼び出しが返される前に終了します。

* `ExecutionSpace <../execution_spaces.html>`_ argument ``exec_space`` が提供されば、 呼び出しは、同期可能ーつまり、コピー演算子が実行される前に、呼び出しは、返されます。 その場合には、コピー演算子は、``exec_space``　に既に送信された作業がすべて完了した後にのみ発生し、コピー演算子は、 ``deep_copy`` 呼び出し返しの実行後に、``exec_space``　に送信されたいずれかの作業の前に完了します。注意事項: コピー演算子は、特定実行空間インスタンスにおける作業に関してのみ、同期的ですが、必ずしも同じ型の他のインスタンスにおける作業を伴っていません。 これは、追加の同期処理なしに、特定の　CUDA　ストリームに対して、``cudaMemcpyAsync``　を発行するのと同様の動作をします

例
--------

出来る事と出来ない事
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: cpp

    #include <Kokkos_Core.hpp>
    #include <cstdio>

    int main(int argc, char* argv[]) {
        Kokkos::initialize(argc, argv);
        {
            int N = argc > 1 ? atoi(argv[1]) : 12;
            if (N < 6) N = 12;

            // 連続デバイスビュー
            Kokkos::View<int**, Kokkos::LayoutLeft> d_a("A", N, 10);
            // Deep Copy Scalar into every element of a view
            Kokkos::deep_copy(d_a, 3);

            // 非連続デバイスビュー
            auto d_a_2 = Kokkos::subview(d_a, 2, Kokkos::ALL);
            // 非連続ビューの各要素への、深部コピースカラー
            Kokkos::deep_copy(d_a_2, 5);
            // 非連続デバイスビュー
            auto d_a_5 = Kokkos::subview(d_a, 5, Kokkos::ALL);
            // 共通の実行領域を持つ2つの非連続ビュー間の深部コピー
            Kokkos::deep_copy(d_a_2, d_a_5);

            // 連続ホストビュー
            auto h_a = Kokkos::create_mirror_view(d_a);
            // 深部コピー連続ビュー
            Kokkos::deep_copy(h_a, d_a);

            // 非連続ホストビュー
            auto h_a_2 = Kokkos::subview(h_a, 2, Kokkos::ALL);
            // 共通要素が潜在的に存在しない、非連続な2つのビュー間の深部コピー
            // 実行空間 例えば、これは、Cudaでコードをコンパイルすると失敗します。
            // Kokkos::deep_copy(h_a_2, d_a_2);

            // スカラービュー
            auto d_a_2_5 = Kokkos::subview(d_a, 2, 5);
            int scalar;
            // スカラーへの深部コピースカラービュー
            Kokkos::deep_copy(scalar, d_a_2_5);
        }
        Kokkos::finalize();
    }

レイアウト互換性のないビューをコピーする方法
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: cpp

    #include<Kokkos_Core.hpp>

    int main(int argc, char* argv[]) {
        Kokkos::initialize(argc,argv);
        {
            int N = argc>1?atoi(argv[1]):1000000;
            int R = argc>2?atoi(argv[2]):10;

            // 異なるレイアウトを使って、2つのビューを作成。           
　　　　　　Kokkos::View<int**[5], Kokkos::LayoutLeft> d_view("DeviceView",N,R);
            Kokkos::View<int**[5], Kokkos::LayoutRight, Kokkos::HostSpace> h_view("HostView",N,R);

            // 例えば、CUDA　ビルドや　HIP　ビルドでは失敗するでしょう:
            // Kokkos::deep_copy(d_view,h_view);

            // 互換性のないレイアウトを持つ2つのビューをデバイス間でコピーするには、一時的に
            auto h_view_tmp = Kokkos::create_mirror_view(d_view)を必要とします;

            // これは、d_view　
            static_assert(std::is_same<decltype(h_view_tmp)::array_layout,
                                       Kokkos::LayoutLeft>::value)からのからレイアウトを継承します;

            // これは現在機能しています。なぜなら h_view_tmp および h_view の両方が
            //  HostSpace::execution_space
            Kokkos::deep_copy(h_view_tmp,h_view)からアクセス可能であるためです;

            // 現在　h_view_tmp および d_view が互換性のあるレイアウトであるため、h_view_tmp から d_view へのコピーが可能です。
            // If we just compiled for OpenMP用にコンパイルするのであれば、　について  。
            // h_view_tmp および d_view　が同じデータを参照するであろうから、これは　no-op です。
            Kokkos::deep_copy(d_view,h_view_tmp);
        }
        Kokkos::finalize();
    }
