.. include:: ../mydefs.rst

設定ガイド
###################

.. 注意事項::
   ``ccmake``のグラフィカルユーザーインターフェースは、利用可能なCMakeオプションとその現在の値を確認する便利な方法を提供します. 　　使用中のKokkosバージョンによりますが、より最新の状態になっている可能性があります。
   **警告の文言:** ``IMPL``という名前を含む変数は、実装の詳細を扱うプライベートな変数です。これらの設定については、その影響について深く理解した上で予告なく変更される可能性があることを認識している場合を除き、変更することは控えてください。 


本ページは、四つのセクションに分かれています:

- :ref:`keywords_backends`
- :ref:`keywords_enable_options`
- :ref:`keywords_tpls`
- :ref:`keywords_arch`

.. _keywords_backends:

バックエンドセクション
=================

**デフォルト状態:**
すべてのバックエンドはデフォルトで無効化されます。 これにより、
お客様の特定のハードウェア構成に必要なバックエンドを
明示的に選択できることが保証されます。

明示的にバックエンドが有効化されていない場合、シリアルバックエンドが有効となります。

**バックエンドの有効化:**
バックエンドは、``-DKokkos_ENABLE_<BACKEND>=ON``
フラグで設定することで有効にできますが、
そこでは、``<BACKEND>``　は有効化することを望む特定のバックエンドに置き換えてください。
（例：CUDAの場合は``-DKokkos_ENABLE_CUDA=ON``）

**制約:**
  相互排斥: 同時に有効にできるデバイスバックエンド（例：CUDA、HIP、SYCL）とホスト並列バックエンド（例：OpenMP、C++スレッド）　　は、それぞれ1つずつに限られます。なぜなら、これらのバックエンドが並列処理を
　潜在的な競合が生じる方法で管理するためです。

  ホストバックエンド要件: 少なくとも、常に1つのホストを有効化する必要があります。
  これは、Kokkosのコード実行が通常、ホスト（CPU）上で開始され、
その後、デバイス（GPU、アクセラレータ）へオフロードされる可能性があるためです。Kokkos　のコード実行が通常、ホスト（CPU）上で開始され、その後、デバイス（GPU、アクセラレータ）へオフロードされる可能性があるためです。 If you
  ホストバックエンドを明示的に有効化しない場合、Kokkos　は自動的にシリアルバックエンドを有効化しますが、それは順次実行モデルを提供します。

シリアルバックエンド
--------------

.. リスト表::
    :幅: 25 65
    :ヘッダー列: 1
    :配列: left

    * -
      - Description/info

    * - ``Kokkos_ENABLE_SERIAL``
      - CPUs　を対象としたシリアルバックエンドの構築

ホスト並列バックエンド
----------------------

.. リスト表::
    :幅: 25 65
    :ヘッダー列: 1
    :配列: 左

    * -
      - ディスクリプション/情報

    * - ``Kokkos_ENABLE_OPENMP``
      - CPUs　を対象とした　OpenMP バックエンドの構築

    * - ``Kokkos_ENABLE_THREADS``
      - C++ スレッドバックエンドの構築

    * - ``Kokkos_ENABLE_HPX``
      - :red:`[Experimental]`  HPX バックエンドの構築

デバイスバックエンド
---------------

.. リスト表::
    :幅: 25 65
    :ヘッダー列: 1
    :配列: 左

    * -
      - ディスクリプション/情報

    * - ``Kokkos_ENABLE_CUDA``
      -  NVIDIA GPUs　対象とした　CUDA　バックエンドを構築

    * - ``Kokkos_ENABLE_HIP``
      -  AMD GPUs　を対象とした　HIP バックエンドを構築

    * - ``Kokkos_ENABLE_SYCL``
      -  GPUs を対象とした　SYCL バックエンドを構築

    * - ``Kokkos_ENABLE_OPENMPTARGET``
      - :red:`[Experimental]` アクセラレータデバイスへのオフロードのための OpenMP ターゲットバックエンドを構築

    * - ``Kokkos_ENABLE_OPENACC``
      - :red:`[Experimental]` アクセラレータデバイスへのオフロードのための OpenACC バックエンドを構築


.. _keywords_enable_options:

オプション
=======

一般オプション
---------------

.. リスト表::
    :幅: 25 65 35
    :ヘッダー列: 1
    :配列: 左

    * -
      - ディスクリプション/情報
      - Defaultデフォルト

    * * ``Kokkos_ENABLE_BENCHMARKS``
      * ベンチマークを構築
      * ``オフ``

    * * ``Kokkos_ENABLE_EXAMPLES``
      * 例を構築
      * ``オフ``

    * * ``Kokkos_ENABLE_TESTS``
      * テストを構築
      * ``オフ``

    * * ``Kokkos_ENABLE_DEPRECATED_CODE_3``
      * Kokkos 3.x シリーズにおける非推奨のコードの有効化 :red:`[バージョン　4.3において削除]`
      * ``オフ``

    * * ``Kokkos_ENABLE_DEPRECATED_CODE_4``
      * Kokkos 4.x シリーズにおける非推奨のコードの有効化
      * ``オン``

    * * ``Kokkos_ENABLE_DEPRECATED_CODE_5``
      * Kokkos 5.x seriesKokkos 5.x シリーズにおける非推奨のコードの有効化
      * ``オフ``

    * * ``Kokkos_ENABLE_DEPRECATION_WARNINGS``
      * 非推奨のKokkos機能を使用する際、コンパイル時に警告を表示するかどうか
      * ``オン``

    * * ``Kokkos_ENABLE_TUNING``
      * チューニングツール用のバインディングを作成
      * ``オフ``

    * * ``Kokkos_ENABLE_AGGRESSIVE_VECTORIZATION``
      * 積極的にループをベクトル化
      * ``オフ``

デバッグ
---------
.. リスト表::
    :幅: 25 65 35
    :header-rowsヘッダー列: 1
    :配列: 左

    * -
      - ディスクリプション/情報
      - デフォルト

    * * ``Kokkos_ENABLE_DEBUG``
      * 追加のデバッグ機能を有効化 - コンパイル時間が長くなる可能性があります
      * ``CMAKE_BUILD_TYPE`` が ``デバッグ`` の場合 ``オン``、それ以外の場合は ``オフ``

    * * ``Kokkos_ENABLE_DEBUG_BOUNDS_CHECK``
      * 境界チェックを使用 - これにより実行時間が長くなります
      * ``オフ``

    * * ``Kokkos_ENABLE_DEBUG_DUALVIEW_MODIFY_CHECK`` :red:`[Deprecated since 4.7]`
      * デュアルビューのデバッグチェック
      * (以下の [#dual_view_modify_check]_　参照)


.. [#dual_view_modify_check] ``Kokkos_ENABLE_DEBUG_DUALVIEW_MODIFY_CHECK`` デフォルト値は、以下の通り:
  
  * ``CMAKE_BUILD_TYPE`` が ``デバッグ`` の場合 ``オン``、それ以外の場合は ``オフ`` ( Kokkos 4.7まで)
  * 常に ``オン`` ( Kokkos 4.7以降)

.. _keywords_enable_backend_specific_options:

バックエンド特有のオプション
------------------------
.. リスト表::
    :幅: 25 65 35
    :ヘッダー列: 1
    :配列: 左

    * -
      - ディスクリプション/情報
      - デフォルト

    * * ``Kokkos_ENABLE_CUDA_CONSTEXPR``
      * 実験的リラックス型　constexpr　関数を有効化
      * ``オフ``

    * * ``Kokkos_ENABLE_CUDA_LAMBDA`` :red:`[バージョン 4.1以降非推奨]`
      * 実験的ラムダ型機能を有効化
      * (以下の　[#cuda_lambda]_　参照)

    * * ``Kokkos_ENABLE_CUDA_RELOCATABLE_DEVICE_CODE``
      * CUDA [#rdc_with_shared_libs]_　のためのリロケータブルデバイスコード（RDC）を有効化
      * ``オフ``

    * * ``Kokkos_ENABLE_CUDA_UVM`` :red:`[4.0以降非推奨]` `代替手段<../usecases/Moving_from_EnableUVM_to_SharedSpace.html>への移行`_　参照。
      * CUDA　については、デフォルトで統一メモリ（UM）を使用
      * ``オフ``

    * * ``Kokkos_ENABLE_IMPL_CUDA_MALLOC_ASYNC``
      * ``cudaMallocAsync`` (requires CUDA Toolkit version 11.2 or higher)　を使用。 この
最適化により、デバイスあたり複数のCUDAストリームを使用するアプリケーションにおいて、パフォーマンスが向上する可能性がありますが、 MPI　ディストリビューションは、古いバージョンの　UCX　および多くの　Cray MPICH　インスタンスに基づいて構築されたものとは互換性がないことは広く認識されています。 `既知の課題 <../known-issues.html#cuda>`_　を参照してください。
      * (以下の [#cuda_malloc_async]_　を参照)

    * * ``Kokkos_ENABLE_HIP_MULTIPLE_KERNEL_INSTANTIATIONS``
      * コンパイル時に複数のカーネルをインスタンス化 - それによってパフォーマンスは向上しますが、コンパイル時間は増加します
      * ``オフ``

    * * ``Kokkos_ENABLE_HIP_RELOCATABLE_DEVICE_CODE``
      * HIP [#rdc_with_shared_libs]_　向けにリロケータブルデバイスコード（RDC）を有効化します 
      * ``オフ``

    * * ``Kokkos_ENABLE_SYCL_RELOCATABLE_DEVICE_CODE``
      * Enable relocatable device code (RDC) for SYCL [#rdc_with_shared_libs]_ 向けにリロケータブルデバイスコード（RDC）を有効化します (Kokkos 4.5以降)。
      * ``オフ``

    * * ``Kokkos_ENABLE_ATOMICS_BYPASS``
      * シリアル専用ビルドにおいて、ホスト並列処理もデバイスバックエンドも有効化されていない場合、アトミック操作を無効化します (Kokkos 4.3以降)
      * ``オフ``

    * * ``Kokkos_ENABLE_IMPL_HPX_ASYNC_DISPATCH``
      * HPX　バックエンドの非同期ディスパッチを有効化します
      * ``オン``

    * * ``Kokkos_ENABLE_COMPILE_AS_CMAKE_LANGUAGE``
      * CMake　言語機能を使用して構築してください（CUDAまたはHIPのみ）[#cmake_language]_
      * ``オフ``

    * * ``Kokkos_ENABLE_MULTIPLE_CMAKE_LANGUAGES``
      * CXXおよびバックエンド互換言語（CUDAまたはHIP）において、Kokkos　のインストールが利用可能となるようにします。 [#multiple_languages]_ (Kokkos 5.0以降)
      * ``オフ``


.. [#cuda_lambda] ``Kokkos_ENABLE_CUDA_LAMBDA`` デフォルト値 は、 3.7まで``オフ`` および 4.0以降　``オン`` 

.. [#cuda_malloc_async] ``Kokkos_ENABLE_IMPL_CUDA_MALLOC_ASYNC`` デフォルト値 は、4.2、4.3、 および 4.4以外で　``オフ``  

.. [#rdc_with_shared_libs] ``Kokkos_ENABLE_<CUDA/HIP/SYCL>_RELOCATABLE_DEVICE_CODE`` は、静的ライブラリのビルドが必要です。
  RDC　は共有ライブラリと互換性がありません。従って、このオプションは``BUILD_SHARED_LIBS``　変数が、偽の場合にのみ有効化可能です。

.. [#cmake_language] ``Kokkos_ENABLE_COMPILE_AS_CMAKE_LANGUAGE`` CMake　の言語機能を使用してビルドを行うと、下流のライブラリやアプリケーションで問題が発生する可能性があります。
  CMake　はファイルの拡張子を用いて、そのファイルがどの言語でコンパイルされるべきかを決定します。Kokkos　のファイルは、``.cpp``　および``.hpp``　という名前であるため、CMake　では``CXX``に関連付けられています。
  これは、Kokkosを使用するソースファイルやヘッダーファイルが、別の言語として扱われるよう再定義する必要があるかもしれないことを意味します。それ以外の場合は、ファイルの拡張子に基づいて言語が検出されます。これにより、ファイルがコンパイルできなくなる可能性があります。(例えば、``.cu``　ファイルではなく　``.cpp``　ファイルで　Kokkos　を使用すると、CMake　が　``CUDA``　ではなく　``CXX``　を検出することになります)。
  言語を指定しない場合、``CXX``　コンパイラがデバイスコードをコンパイルする能力によっては、コンパイルが失敗に終わる可能性があります。
  さらに、各ターゲットのアーキテクチャは、``CMAKE_<LANG>_ARCHITECTURES``　を使用してではなく、``Kokkos_ARCH_<...>``　の設定に基づいて指定する必要があります。これはまた、アクティブなアーキテクチャは一つだけであることを意味します。 これはまた、活動可能なアーキテクチャは、一つだけであることを意味します。

  ファイルを適切にマークする例は、``example/build_cmake_installed_kk_as_language``　において、見られます。

.. [#multiple_languages] ``Kokkos_ENABLE_MULTIPLE_CMAKE_LANGUAGES`` このオプションにより、インストール済みの　Kokkos　ライブラリを複数の　CMake　言語（``CXX``　および対応するバックエンド言語（``CUDA``　または　``HIP``））で使用することが可能となります。
  このオプションを有効にすると、Kokkos　は、コンパイラランチャースクリプトを使用して、``separable_compilation``　コンポーネントが要求されない限り、``CXX``　コンパイラをリダイレクトします。
  コンポーネントを使用する場合、Kokkos　にリンクするターゲット/プロジェクト/ディレクトリは、CMake　関数　 ``kokkos_compilation``　を用いて手動でマークする必要があります。
  Kokkos　は、単一のアーキテクチャに限定されているため、``CMAKE_<LANG>_ARCHITECTURES``　はKokkos　で有効化されたアーキテクチャに対応している必要があります。

  複数の言語でのKokkosの使用例は、``example/build_cmake_installed_multilanguage``　で確認できます。

開発
-----------
これらは、Kokkos　の開発者向けです。 ユーザーであれば、おそらくこれらの設定は行うべきではないでしょう。


.. リスト表::
    :幅: 25 65 35
    :ヘッダー列: 1
    :配列: 左

    * -
      - ディスクリプション/情報
      - デフォルト

    * * ``Kokkos_ENABLE_COMPILER_WARNINGS``
      * すべてのコンパイラ警告をプリント
      * ``オフ``

    * * ``Kokkos_ENABLE_HEADER_SELF_CONTAINMENT_TESTS``
      * ヘッダーが自己完結していることを確認
      * ``オフ``

    * * ``Kokkos_ENABLE_LARGE_MEM_TESTS``
      * 大規模な追加メモリテストを実施
      * ``オフ``

.. _keywords_tpls:

サードパーティーライブラリ (TPLs)
============================

以下のオプションは、TPL　を有効化して以下をコントロールします:

.. リスト表::
    :幅: 30 40 10 20
    :ヘッダー列: 1
    :配列: 左

    * -
      - ディスクリプション/情報
      - デフォルト
      - 注意事項

    * * ``Kokkos_ENABLE_HWLOC``
      * HWLOC　ライブラリを有効化するかどうか
      * ``オフ``
      *
    * * ``Kokkos_ENABLE_LIBDL``
      * LIBDL ライブラリを有効化するかどうか
      * ``オン``
      *
    * * ``Kokkos_ENABLE_LIBQUADMATH``
      * GCC　のクワッド精度数学ライブラリによる、128ビット浮動小数点型のサポートを有効化するかどうか 
      * ``オフ``
      *
    * * ``Kokkos_ENABLE_ONEDPL``
      * SYCLバックエンドを使用する際、oneDPL　ライブラリを有効化するかどうか
      * ``オン``
      *
    * * ``Kokkos_ENABLE_ROCTHRUST``
      * HIPバックエンドを使用する際、rocThrust　ライブラリを有効にするかどうか
      * ``オン``
      * ( Kokkos 4.3以降)

以下のオプションは、CMake以外のテンプレート言語（TPL）の検索と設定を制御します:

.. リスト表::
    :widths: 35 45 20
    :ヘッダー列: 1
    :配列: 左

    * -
      - ディスクリプション/情報
      - デフォルト

    * * ``Kokkos_CUDA_DIR`` または ``CUDA_ROOT``
      * ライブラリ用　CUDA　インストールプリフィックスの場所
      * PATH デフォルト:

    * * ``Kokkos_HWLOC_DIR`` または ``HWLOC_ROOT``
      * HWLOC インストールプレフィックスの場所
      * PATH デフォルト:

    * * ``Kokkos_LIBDL_DIR`` または ``LIBDL_ROOT``
      * LIBDL インストールプレフィックスの場所
      * PATH デフォルト:

以下のオプションは、CMake　ベースの TPL用の ``find_package`` パスを制御します:

.. リスト表::
    :幅: 35 60 25
    :ヘッダー列: 1
    :配列: 左

    * -
      - ディスクリプション/情報
      - デフォルト

    * * ``HPX_DIR`` or ``HPX_ROOT``
      * HPX　プレフィックス（ROOT）または　CMake　設定ファイル（DIR）の場所
      * PATH デフォルト:

.. _keywords_arch:

アーキテクチャ
=============

CPU アーキテクチャ
-----------------

Kokkos　は、特定の　CPU　アーキテクチャ向けに最適化するため、コンパイラフラグを自動的に追加または必要とすることはありません。
しかしながら、特定のアーキテクチャを対象とすることで、コンパイラは、CPU　上で　SIMD　命令を利用することが可能となります。
コードが実行されるマシン上でコンパイルする場合、CPUコード最適化ための最も簡単な方法は、ネイティブオプションを使用することです。

.. リスト表::
    :幅: 25 75
    :ヘッダー列: 1
    :配列: 左

    * -
      - ディスクリプション/情報

    * - ``Kokkos_ARCH_NATIVE``
      -  コンパイルする　CPU (``-march=native``)　のアーキテクチャを対象とします（``-march=native``）

クロスコンパイルを行う場合、またはより詳細な説明を望む場合には、CPUアーキテクチャは　Kokkos　に手動で渡すことが可能です。　利用可能なアーキテクチャについては、以下のリストを参照してください。

.. リスト表:: AMD CPU アーキテクチャ
    :幅: 30 30 30 30
    :ヘッダー列: 1
    :配列: 左

    * - CMake キーワード
      - アーキテクチャ/インストラクションセット
      - 例
      - 注意事項

    * - ``Kokkos_ARCH_ZEN5``
      - Zen 5/amd64
      -
      - (Kokkos 4.7以降)

    * - ``Kokkos_ARCH_ZEN4``
      - Zen 4/amd64
      - Epyc Genoa @ LLNL El Capitan
      - (Kokkos 4.6以降)

    * - ``Kokkos_ARCH_ZEN3``
      - Zen 3/amd64
      - Epyc 7713 @ ORNL Frontier
      -

    * - ``Kokkos_ARCH_ZEN2``
      - Zen 2/amd64
      - Epyc 7742 @ NOAA
      -

    * - ``Kokkos_ARCH_ZEN``
      - Zen/amd64
      - Epyc @ ANL Selene
      -

    * - ``Kokkos_ARCH_AMDAVX``
      - Bullozer/amd64
      -
      -

.. リスト表:: ARM CPU アーキテクチャ
    :幅: 30 30 30 30
    :ヘッダー列: 1
    :配列: 左

    * - CMake キーワード
      - アーキテクチャ/インストラクションセット
      - 例
      - 注意事項

    * - ``Kokkos_ARCH_ARMV9_GRACE``
      - ARMv9-A/A64/neoverse-v2
      - GH200 @ CSCS ALPS
      - (since Kokkos 4.4.1)

    * - ``Kokkos_ARCH_A64FX``
      - ARMv8.2/A64
      - A64FX @ Fugaku
      -

    * - ``Kokkos_ARCH_ARMV8_THUNDERX2``
      - ARMv8/A64
      - ThunderX2 @ SNL Astra
        ThunderX2 @ CEA BullSequana
      -

    * - ``Kokkos_ARCH_ARMV81``
      - ARMv8.1/A64,A32
      -
      -

    * - ``Kokkos_ARCH_ARMV8_THUNDERX``
      - ARMv8/A64
      -
      -

    * - ``Kokkos_ARCH_ARMV80``
      - ARMv8.0/A64,A32
      -
      -

.. リスト表:: IBM CPU アーキテクチャ
    :幅: 30 30 30
    :ヘッダー列: 1
    :配列: 左

    * - CMake キーワード
      - アーキテクチャ/インストラクションセット
      - 例

    * - ``Kokkos_ARCH_POWER9``
      - Power9/Power ISA
      - POWER9 @ ORNL Summit
        POWER9 @ LLNL Sierra

    * - ``Kokkos_ARCH_POWER8``
      - Power8/パワー ISA
      -

.. リスト表:: Intel CPU アーキテクチャ
    :幅: 30 30 30
    :ヘッダー列: 1
    :配列: 左

    * - CMake キーワード
      - アーキテクチャ/インストラクションセット
      - 例

    * - ``Kokkos_ARCH_SPR``
      - Sapphire Rapids/x86-64
      - Xeon 9470C @ ANL Aurora
        Xeon @ LANL Crossroads

    * - ``Kokkos_ARCH_SKX``
      - Skylake/x86-64
      - 6130 @ OSU Pete

    * - ``Kokkos_ARCH_HSW``
      - Haswell/x86-64
      - 2680v3 @ NASA Pleiades

    * - ``Kokkos_ARCH_BDW``
      - Broadwell/x86-64
      - 2680v4 @ NASA Pleiades

    * - ``Kokkos_ARCH_KNL``
      - Knights Landing/x86-64
      - 31S1P @ Tianhe-2

    * - ``Kokkos_ARCH_KNC``
      - Knights Corner/x86-64
      -

    * - ``Kokkos_ARCH_SNB``
      - Sandy Bridge/x86-64
      -

.. リスト表:: RISC-V CPU architectures
    :幅: 30 30 30 30
    :ヘッダー列: 1
    :配列: 左

    * - CMake キーワード
      - アーキテクチャ/インストラクションセット
      - 例
      - 注意事項

    * - ``Kokkos_ARCH_RISCV_RVA22V``
      - RVA22V/RISC-V ISA
      - SpacemiT K1
      - (Kokkos 5.0以降)

    * - ``Kokkos_ARCH_RISCV_SG2042``
      - SG2042/RISC-V ISA
      - Milk-V パイオニア
      - (Kokkos 5.0以降)

    * - ``Kokkos_ARCH_RISCV_U74MC``
      - U74MC/RISC-V ISA
      - SiFive Unmatched
      - (Kokkos 5.0以降)

GPU アーキテクチャ
-----------------

NVIDIA GPUs
~~~~~~~~~~~

Kokkos　の命名規則は、NVIDIA GPU　マイクロアーキテクチャの名称と、関連する　CUDA　コンピュートキャパビリティを組み合わせたものです。

``Kokkos_ARCH_<MICROARCHITECTURE><COMPUTE_CAPABILITY>``

CUDAバックエンドが有効化されており、NVIDIA GPUアーキテクチャが指定されていない場合、
Kokkos　は。設定時にアーキテクチャフラグの自動検出を試みます。

.. リスト表::
    :幅: 20 15 15 25 30
    :ヘッダー列: 1
    :配列: 左

    * - **NVIDIA GPUs**
      - アーキテクチャ
      - コンピュートキャパビリティ
      - モデル
      - 注意事項

    * * ``Kokkos_ARCH_BLACKWELL120``
      * Blackwell
      * 12.0
      * RTX 5080
      * (Kokkos 4.7以降)

    * * ``Kokkos_ARCH_BLACKWELL100``
      * Blackwell
      * 10.0
      * B200, B100
      * (Kokkos 4.7以降)

    * * ``Kokkos_ARCH_HOPPER90``
      * Hopper
      * 9.0
      * H100
      * (Kokkos 4.0以降)

    * * ``Kokkos_ARCH_ADA89``
      * Ada Lovelace
      * 8.9
      * L4, L40
      * (Kokkos 4.1以降)

    * * ``Kokkos_ARCH_AMPERE87``
      * Ampere
      * 8.7
      * Jetson Orin
      * (Kokkos 4.7以降)

    * * ``Kokkos_ARCH_AMPERE86``
      * Ampere
      * 8.6
      * A40, A10, A16, A2
      *

    * * ``Kokkos_ARCH_AMPERE80``
      * Ampere
      * 8.0
      * A100, A30
      *

    * * ``Kokkos_ARCH_TURING75``
      * Turing
      * 7.5
      * T4
      *

    * * ``Kokkos_ARCH_VOLTA72``
      * Volta
      * 7.2
      *
      *

    * * ``Kokkos_ARCH_VOLTA70``
      * Volta
      * 7.0
      * V100
      *

    * * ``Kokkos_ARCH_PASCAL61``
      * Pascal
      * 6.1
      * P40, P4
      *

    * * ``Kokkos_ARCH_PASCAL60``
      * Pascal
      * 6.0
      * P100
      *

    * * ``Kokkos_ARCH_MAXWELL53``
      * Maxwell
      * 5.3
      *
      *

    * * ``Kokkos_ARCH_MAXWELL52``
      * Maxwell
      * 5.2
      * M60, M40
      *

    * * ``Kokkos_ARCH_MAXWELL50``
      * Maxwell
      * 5.0
      *
      *

    * * ``Kokkos_ARCH_KEPLER37``
      * Kepler
      * 3.7
      * K80
      * (Kokkos 5.0において削除)

    * * ``Kokkos_ARCH_KEPLER35``
      * Kepler
      * 3.5
      * K40, K20
      * (Kokkos 5.0において削除)

    * * ``Kokkos_ARCH_KEPLER32``
      * Kepler
      * 3.2
      *
      * (Kokkos 5.0において削除)

    * * ``Kokkos_ARCH_KEPLER30``
      * Kepler
      * 3.0
      * K10
      * (Kokkos 5.0において削除)


AMD GPUs
~~~~~~~~

Kokkos　の命名規則は、AMD\_　とアーキテクチャフラグを結合するものです。

``Kokkos_ARCH_AMD_<ARCHITECTURE_FLAG>``

HIP　バックエンドが有効化されており、AMD GPU　アーキテクチャが指定されていない場合、
Kokkos　は設定時にアーキテクチャフラグの自動検出を試みます。

.. リスト表::
    :幅: 30 15 25 30
    :ヘッダー列: 1
    :配列: 左

    * - **AMD GPUs**
      - アーキテクチャフラッグ
      - モデル
      - 注意事項

    * * ``Kokkos_ARCH_AMD_GFX942_APU``
      * GFX942
      * MI300A
      * (Kokkos 4.5以降)

    * * ``Kokkos_ARCH_AMD_GFX942``
      * GFX942
      * MI300A, MI300X
      * (Kokkos 4.2以降、 Kokkos 4.5以降、これは、 MI300X用のみに使用されるべきです)

    * * ``Kokkos_ARCH_AMD_GFX940``
      * GFX940
      * MI300A (pre-production)
      * (Kokkos 4.2.1以降)

    * * ``Kokkos_ARCH_AMD_GFX90A``
      * GFX90A
      * MI200 series
      * (Kokkos 4.2以降)

    * * ``Kokkos_ARCH_AMD_GFX908``
      * GFX908
      * MI100
      * (Kokkos 4.2以降)

    * * ``Kokkos_ARCH_AMD_GFX906``
      * GFX906
      * MI50, MI60
      * (Kokkos 4.2以降)

    * * ``Kokkos_ARCH_AMD_GFX1201``
      * GFX1201
      * Radeon AI PRO R9700, Radeon RX 9070 XT
      * (Kokkos 5.0以降)

    * * ``Kokkos_ARCH_AMD_GFX1103``
      * GFX1103
      * Ryzen 8000G Phoenix series APU
      * (Kokkos 4.5以降)

    * * ``Kokkos_ARCH_AMD_GFX1100``
      * GFX1100
      * 7900xt
      * (Kokkos 4.2以降)

    * * ``Kokkos_ARCH_AMD_GFX1030``
      * GFX1030
      * V620, W6800
      * (Kokkos 4.2以降)

    * * ``Kokkos_ARCH_VEGA90A``
      * GFX90A
      * MI200 series
      * ``Kokkos_ARCH_AMD_GFX90A``　優先

    * * ``Kokkos_ARCH_VEGA908``
      * GFX908
      * MI100
      * ``Kokkos_ARCH_AMD_GFX908``　優先

    * * ``Kokkos_ARCH_VEGA906``
      * GFX906
      * MI50, MI60
      * ``Kokkos_ARCH_AMD_GFX906``　優先

    * * ``Kokkos_ARCH_VEGA900``
      * GFX900
      * MI25
      * 4.0において削除

Intel GPUs
~~~~~~~~~~

.. リスト表::
    :幅: 15 25 35 25
    :ヘッダー列: 1
    :配列: 左

    * - CMake オプション
      - アーキテクチャ
      - モデル
      - 注意事項

    * * ``Kokkos_ARCH_INTEL_PVC``
      * Xe-HPC (Ponte Vecchio)
      * Intel データセンター GPU Max 1550
      *

    * * ``Kokkos_ARCH_INTEL_XEHP``
      * Xe-HP
      *
      *

    * * ``Kokkos_ARCH_INTEL_DG2``
      * Intel DG2
      * Intel Flex, Intel Arc
      * (Kokkos 4.7以降)

    * * ``Kokkos_ARCH_INTEL_DG1``
      * Iris Xe MAX (DG1)
      *
      *

    * * ``Kokkos_ARCH_INTEL_GEN12LP``
      * Gen12LP
      * Intel UHD Graphics 770
      *

    * * ``Kokkos_ARCH_INTEL_GEN11``
      * Gen11
      * Intel UHD Graphics
      *

    * * ``Kokkos_ARCH_INTEL_GEN9``
      * Gen9
      * Intel HD Graphics 510, Intel Iris Pro Graphics 580
      *

    * *
      *
      *
      *

    * * ``Kokkos_ARCH_INTEL_GEN``
      * 特に、Intel GPU向けに、ジャストインタイムコンパイル [#arch_intel_gen]_ 
      *
      *

.. [#arch_intel_gen] ``Kokkos_ARCH_INTEL_GEN`` は、Intel GPU　向けにはジャストインタイムコンパイルを有効にし、
一方、Intelコンパイラ向けのその他のフラグはすべて
アヘッドオブタイムコンパイルを要求します

  ジャストインタイム（JIT）コンパイルとは、生成されたバイナリが実際に実行される際にコンパイラが再度呼び出され、その時点で初めてコンパイル対象のアーキテクチャが決定されることを意味します

  一方、アヘッドオブタイム（AOT）コンパイルは、標準モデルを指し、コンパイラはバイナリを生成するために一度だけ呼び出され、
コンパイル対象のアーキテクチャは、プログラムの実行前に決定されます。
