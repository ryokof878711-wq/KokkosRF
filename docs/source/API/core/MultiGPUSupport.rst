.. role:: cpp(code)
    :language: cpp

マルチ　GPU サポート
=================

Kokkos　は、単一のホストプロセス（例：単一のMPIランク）から複数の　GPU　上でカーネルを起動する機能をサポートしており、この機能は現在、CUDA、HIP、SYCL　の各バックエンドで利用可能です。

この機能を使用するには、デフォルト以外の実行スペースインスタンスを作成するためのバックエンド固有のAPI呼び出しに関する知識が必要です。実行スペースが作成されると、
ユーザーが選択したデバイス上で、実行ポリシーの作成やビューの割り当てなどを行うことができます。

実行空間構築
-----------------------------

CUDA
~~~~

CUDAバックエンドについては、ユーザーは　``cudaStream_t``　オブジェクトを作成し、それを　``Kokkos::Cuda``　コンストラクタに渡します。

.. 注意事項:: ストリームに関連するすべての　Kokkos　オブジェクト（実行空間など）のライフタイムは、
ストリーム自体を　``cudaStreamDestroy``　で無効化する前に終了させる必要があります。以下のように、そのようなオブジェクトにスコープを追加することで実現できます。


.. code-block:: cpp

    // 利用可能なデバイス数を照会
    int n_devices;
    cudaGetDeviceCount(&n_devices);

    // 0 <= N < n_devices　を選択。
    int N = ...;

    // デバイス N　上にストリームを作成。
    cudaStream_t stream;
    cudaSetDevice(N);
    cudaStreamCreate(&stream);

    // ストリームを保証するためにスコープ実行領域を特定。
    // 実行空間　*後に*　破棄。
    {
      // 実行空間を作成。
      Kokkos::Cuda exec_space(stream);

      // 実行空間を使用。
      /* ... */
    }

    // 使用後にストリームを破棄。
    cudaSetDevice(N);
    cudaStreamDestroy(stream);

HIP
~~~

HIP backend　について、  CUDAを使うなど、ユーザーは、``hipStream_t``　オブジェクトを作成し、それを　``Kokkos::HIP``　コンストラクタに渡します。

.. 注意事項:: ストリームに関連するすべての　Kokkos　オブジェクトの存続期間は　(例えば、 実行空間)  ``hipStreamDestroy``　経由で、ストリームそれ自体が無効化される前に、終了する必要があります。 これは、以下のように、そのようなオブジェクトの周囲にスコープを追加することで実現できます。

.. 警告:: Multi-GPU　は、 only supported for ROCm 5.6 以降のみをサポートしました。 ROCm 5.6前のストリームのデバイス照会のための　HIP API 関数が欠如しているため、 非デフォルトデバイス上の　``Kokkos::HIP`` インスタンス構築は、サポートされていません。


.. code-block:: cpp

    // 利用可能なデバイス数を照会
    int n_devices;
    hipGetDeviceCount(&n_devices);

    // 0 <= N < n_devices　を選択。
    int N = ...;

    // デバイス N　上にストリームを作成。
    hipStream_t stream;
    hipSetDevice(N);
    hipStreamCreate(&stream);

    // ストリームを保証するためにスコープ実行領域を特定。
    // is destroyed *after* execution space
    {
      // 実行空間を作成。
      Kokkos::HIP exec_space(stream);

      // Use execution space実行空間を使用。
      /* ... */
    }

    // 使用後ストリームを破棄。
    hipSetDevice(N);
    hipStreamDestroy(stream);

SYCL
~~~~

SYCL backendについては、 ユーザーは、``sycl::queue`` オブジェクトを作成し、それをand passes it to the ``Kokkos::SYCL`` コンストラクタに渡します。

.. code-block:: cpp

    // 利用可能なデバイスのリストを取得。
    std::vector<sycl::device> gpu_devices =
      sycl::device::get_devices(sycl::info::device_type::gpu);

    // 0 <= N < gpu_devices.size()　を選択。
    int N = ...;

    // Create a queue on デバイス N　上にキューを作成。
    // Note: Kokkos は、 SYCL キューが、to be "in_order"　であることを要求します。
    sycl::queue queue{gpu_devices[N], sycl::property::queue::in_order()};

    // 実行空間を作成。
    Kokkos::SYCL exec_space(queue);

    // Use execution space実行空間を使用
    /* ... */

 Kokkos メソッド使用
--------------------

選択したデバイス上に実行領域が作成されると、 実行空間は、選択されたデバイス上で使用することを目的としたすべての Kokkos メソッドに渡さる必要があります。実行空間が渡されなければ、 Kokkos は、 Kokkos が初期化されたデバイスと関連付けられたデフォルト実行空間を使用します。

管理対象ビューの割り当て
~~~~~~~~~~~~~~~~~~~~~~~~

デバイス上の管理対象ビューの割り当てのために、 実行空間を ``Kokkos::view_alloc()``　に渡します。

例:

.. code-block:: cpp

    ExecutionSpace = decltype(exec_space);　使用。
    Kokkos::View<int*, typename ExecutionSpace::memory_space> V(Kokkos::view_alloc("V", exec_space), 10);

カーネル起動
~~~~~~~~~~~~~~~~~

デバイス上のカーネル起動のため、実行空間をポリシーコンストラクタに渡します。

例:

.. code-block:: cpp

    Kokkos::parallel_for("inc_V", Kokkos::RangePolicy(exec_space, 0, 10),
      KOKKOS_LAMBDA (const int i) {
        V(i) += i;
    });

注意事項
-----

- チュートリアル CUDAに関する　マルチGPU　使用のための <https://github.com/kokkos/kokkos-tutorials/tree/main/Exercises/multi_gpu_cuda>`_ が、利用可能です。
