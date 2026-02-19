初期化
==============

 Kokkos　使用のためには、初期化呼び出しが必要です。 その呼び出しは、内部オブジェクトの初期化とスレッドなどのハードウェアリソースの取得を担当します。　通常、この呼び出しはプログラムの開始直後に行う必要があります。 MPI　と　Kokkos　の両方を使用する場合、プログラムは、`MPI_Init`　を呼び出した直後に、Kokkos　を初期化する必要があります。 そうすれば、MPI　がプロセスバインディングマスクを設定した場合、Kokkos　はその情報を取得し、最高のパフォーマンスのために活用します。 プログラムは、Kokkos　の使用が終了したら、ハードウェアリソースを解放するために必ず　_finalize_　する必要があります。

ヘッダー挿入
---------------

Kokkos　の主要な機能は、すべて　`Kokkos_Core.hpp`　ヘッダーファイルによって提供されます。
一部の機能、具体的には　`containers`　サブパッケージのデータ構造体と　`algorithms`　サブパッケージのアルゴリズム機能は、個別のヘッダーファイルを介して組み込まれています。
具体的な機能については、その　API　リファレンスを確認してください:

- `API: コア <../API/core-index.html>`_
- `API: コンテナ <../API/containers-index.html>`_
- `API: アルゴリズム <../API/algorithms-index.html>`_
- `アルファベット順の　API <../API/alphabetical.html>`_

コマンドライン引数による初期化
----------------------------------------

Kokkos　を初期化する最も簡単な方法は、以下の関数を呼び出すことです:

.. code-block:: cpp

    Kokkos::initialize(int& argc, char* argv[]);

`MPI_Init`と同様に、この関数は、コマンドライン引数を解釈して要求された設定を決定します。 また、　`MPI_Init`　と同様に、入力リストからコマンドライン引数を削除する権利を留保します。 これが、`argc`を値ではなく参照で受け取る理由です。

初期化の間に、 1つ以上の実行スペースが初期化され、以下の別名のいずれかに割り当てられます。

.. code-block:: cpp

    Kokkos::DefaultExecutionSpace;

.. code-block:: cpp

    Kokkos::DefaultHostExecutionSpace;

 `DefaultExecutionSpace` は、ポリシーやビューで明示的に指定されていない場合に使用される実行スペースです。 主に、Kokkosはビルド構成で有効化されている場合、異種バックエンド（CUDA、HIP、OpenACC、OpenMPTarget、SYCL）のいずれかをデフォルト実行空間として初期化します。 さらに、Kokkosでは、　`DefaultHostExecutionSpace`　が必要です。 `DefaultHostExecutionSpace` は、ホスト演算が必要な場合に使用される、デフォルト実行空間です。 ビルド環境で並列ホスト実行スペースのいずれか一つが有効化されている場合、`Kokkos::Serial` はビルド構成で明示的に有効化されている場合にのみ初期化されます。 ビルド構成で並列ホスト実行空間が有効化されていない場合、`Kokkos::Serial` が `DefaultHostExecutionSpace` として初期化されます。
Kokkos　は、以下のリストを使用して2つのスペースを選択します:

1. `Kokkos::Cuda`
2. `Kokkos::Experimental::HPX`
3. `Kokkos::Experimental::OpenACC`
4. `Kokkos::Experimental::OpenMPTarget`
5. `Kokkos::SYCL`
6. `Kokkos::HIP`
7. `Kokkos::OpenMP`
8. `Kokkos::Threads`
9. `Kokkos::Serial`

有効化されているリスト内の最高位の実行空間は、Kokkosのデフォルト実行空間であり、有効化されたホスト実行空間の中で最上位のものは、Kokkos　のデフォルトホスト実行空間です。 例えば、 `Kokkos::Cuda`, `Kokkos::OpenMP`, および　`Kokkos::Serial` が有効かされていれば、 `Kokkos::Cuda`　がデフォルトの実行空間であり、`Kokkos::OpenMP`　がデフォルトのホスト実行空間です\ :sup:`1`。  有効なバックエンドの中で最上位のものが、ホスト並列実行空間である場合、 `DefaultExecutionSpace` および `DefaultHostExecutionSpace` は、同じになります。

 `Kokkos::initialize <../API/Initialize-and-Finalize.html#kokos-initialize>`_ は、`--kokkos-` で始まるフラグをコマンドラインから解析し、認識されたフラグのすべてを削除します。 引数のオプションは等号（`=`）で指定します。 同じ引数が複数回指定された場合、最後のものが使用されます。例えば、引数

    --kokkos-threads=4 --kokkos-threads=3

は、スレッド数を3に設定します。 :ref:`表 4.1 <Table_cli-opts>` は、コマンドラインオプションの完全なリストを提供します。

.. _Table_cli-opts:

表 4.1:  Kokkos::initialize　のコマンドラインオプション

.. リスト表::

  * - 引数
    - ディスクリプション
  * - :code:`--kokkos-help`
    - 本メッセージをプリント
  * - :code:`--kokkos-disable-warnings`     
    -  kokkos 警告メッセージを無効化
  * - :code:`--kokkos-print-configuration` 
    - 設定をプリント
  * - :code:`--kokkos-tune-internals`      
    - Kokkosがポリシーを自動調整し、調整システムを通じて調整機能を宣言できるようにします。無効にした場合、Kokkos　はヒューリスティックを使用します。
  * - :code:`--kokkos-num-threads=INT`     
    - ホスト上の並列領域で使用するスレッドの総数を指定
  * - :code:`--kokkos-device-id=INT`
    - Kokkos　で使用するデバイスIDを指定
  * - :code:`--kokkos-map-device-id-by=(random\|mpi_rank)`, default: :code:`mpi_rank`
    - 利用可能なデバイスからデバイスIDを自動的に選択するストラテジー: ランダムまたは　mpi_rank\ :sup:`2`
  * - :code:`--kokkos-tools-libs=STR`      
    - 使用するツールを指定。ライブラリの完全なパス、またはランタイムライブラリ検索パス（例: LD_LIBRARY_PATH）にパスが存在する場合のライブラリ名でなければなりません。
  * - :code:`--kokkos-tools-help`          
    - (読み込まれた) kokkos-tool のコマンドラインオプションサポートを参照（その後、--kokkos-tools-args="..."　 経由で渡される必要があります）
  * - :code:`--kokkos-tools-args=STR`      
    - オプションの単一（引用符付き）文字列で、これは空白で区切られ、ロードされた　kokkos-tool　にコマンドライン引数として渡されます。例： :code:`<EXE> --kokkos-tools-args="-c input.txt"` は、 :code:`<EXE> -c input.txt` を argc/argv としてツールに渡します。

ブール値を文字列として渡す場合、許容される値は次の通りです:
 - 真, はい, 1
 - 偽, いいえ, 0

値は、大文字小文字を区別しません。


:sup:`1` これは、CUDA　と　OpenMP　が有効な場合の推奨デフォルト設定です。 スレッド並列ホスト実行空間を使用する場合、Kokkos　の　OpenMP　バックエンドを推奨します。これにより、Kokkos　のスレッドとアプリケーションが直接使用する　OpenMP　スレッドとの互換性が保証されます。 Kokkos　は、そのスレッドバックエンドがアプリケーションによるオペレーティングシステムスレッドの直接使用と競合しないことを保証できません。

:sup:`2` 2つのデバイスIDマッピング戦略は以下の通りです:
- random: 利用可能なデバイスからランダムに1つ選択。
- mpi_rank: ローカルMPIランクのラウンドロビン割り当てに基づいてデバイスを選択。 OpenMPI、MVAPICH、SLURM、およびその派生した実装と連携。  MPICH　のサポートを  Kokkos 4.0において追加

環境変数による初期化
--------------------------------------

コマンドライン引数を使用する代わりに、環境変数を使用することもできます。 環境変数は、 :ref:`Table 4.1 <Table_cli-opts>` における引数と同一ですが、それらは、それらは大文字であり、ダッシュはアンダースコアに置換されます。　例えば、例えば、スレッド数を3に設定したい場合、 以下を使用できます

.. code-block:: sh

  KOKKOS_NUM_THREADS=3


構造体による初期化
------------------------

`Kokkos::initialize() <../API/core/initialize_finalize/initialize.html>`_ command-line 引数を提供する代わりに、  `Kokkos::InitializationSettings` 構造体を使用して、初期化パラメータにおいて直接引き渡すことができます。  構造体を使用してオプションを設定したい場合、ダッシュがアンダースコアに置換されている  :ref:`Table 4.1 <Table_cli-opts>`　における引数に、`xxx`　が同一である関数 `set_xxx` を使用できます。 変数が設定されているかどうかを確認するには、`has_xxx`関数を使用できます。最終的に、設定された値を取得するには、`get_xxx`　関数を使用できます。


`num_threads`　を設定しない場合、Kokkos　は可能な限りデフォルト値を決定しようと試み、それが不可能な場合には、1に設定します。 特に、Kokkos　は、　`hwloc`　ライブラリを使用して、プロセスバインディングマスクが一意である、つまりつまり、本プロセスは、別のプロセスとコアを共有しないという仮定のもとで、デフォルト設定を決定できます。各パラメータのデフォルト値が、 -1であることに注意してください。」

以下に、構造体の使用方法の一例を示します。

.. code-block:: cpp

    Kokkos::InitializationSettings 設定;
    // 8 (CPU) スレッド
    settings.set_num_threads(8);
    // Kokkos　が、CUDA　有効状態で構築された場合、デバイスID 1の　GPU　を使用してください。
    settings.set_device_id(1);

    Kokkos::initialize(settings);

最終処理完了
------------

各プログラム終了時に、 Kokkos は、リソースを解放するために、シャットダウンする必要があります; `Kokkos::finalize() <../API/core/initialize_finalize/finalize.html>`_　を呼び出すことにより、これを行ってください。 `MPI_Finalize` で自動的に呼び出されるようにするために、`atexit` フックを設定するか、または　`MPI_COMM_SELF` に関数をアタッチすることにより、 プログラム終了時に自動的に呼び出されるように設定することを推奨します。

コードの例
------------

最小限の　Kokkos　コードは、以下のようになります:

.. code-block:: cpp

    #include<Kokkos_Core.hpp>
    
    int main(int argc, char* argv[]) {
      Kokkos::initialize(argc,argv);
    
      Kokkos::finalize();
    }
