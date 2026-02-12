実行ポリシー
##################

トップレベル実行ポリシー
============================

`ExecutionPolicyConcept <policies/ExecutionPolicyConcept.html>`__ は、並列パターンの実行が、 “どのように” 行われるかを表すための基本的な抽象化です。

.. リスト表::
    :幅: 35 65
    :ヘッダー列: 1
    :配列: 左

    * - ポリシー
      - ディスクリプション

    * * `RangePolicy <policies/RangePolicy.html>`__
      * 各イテレートは、連続範囲内での整数です。

    * * `MDRangePolicy <policies/MDRangePolicy.html>`_
      * 各ランクについての各イテレートは、連続範囲内での整数です。

    * * `TeamPolicy <policies/TeamPolicy.html>`__
      * 連続した範囲の各イテレート処理にスレッドのチームを割り当てます。

ネストされた実行ポリシー
============================

ネストされた実行ポリシーは、既に実行中の並列領域内で、`TeamPolicy <policies/TeamPolicy.html>`__ またはタスクポリシーによって、配置された並列作業を配置するために使用されます。 `NestedPolicies <policies/NestedPolicies.html>`__ summary。

.. list表::
    :幅: 25 75
    :ヘッダー列: 1
    :配列: 左

    * - ポリシー
      - ディスクリプション

    * * `TeamThreadMDRange <policies/TeamThreadMDRange.html>`__
      * TeamPolicy　カーネル内で使用され、チームの各スレッドに分割された多次元範囲に対してネストされた並列ループを実行します。

    * * `TeamThreadRange <policies/TeamThreadRange.html>`__
      * チームの各スレッドに分割されたネストされた並列ループを実行するために、TeamPolicy　カーネル内で使用されます。

    * * `TeamVectorMDRange <policies/TeamVectorMDRange.html>`__
      * チームのスレッドとそのベクトルレーンに分割された多次元範囲に対して、ネストされた並列ループを実行するために、TeamPolicy　カーネル内で使用されます。

    * * `TeamVectorRange <policies/TeamVectorRange.html>`__
      * チームのスレッドとそのベクトルレーンに分割された、ネストされた並列ループを実行するために、TeamPolicy　カーネル内で使用されます。

    * * `ThreadVectorMDRange <policies/ThreadVectorMDRange.html>`__
      * スレッドのベクトルレーンを用いて多次元範囲に対して、ネストされた並列ループを実行するために、TeamPolicy　カーネル内で使用されます。

    * * `ThreadVectorRange <policies/ThreadVectorRange.html>`__
      * スレッドのベクトルレーンを用いてネストされた並列ループを実行するために、TeamPolicy　カーネル内で使用されます。

すべての実行ポリシーのための共通引数
===========================================

実行ポリシーは通常、テンプレート引数を通じてコンパイル時の引数を受け入れ、コンストラクタ引数またはセッター関数を通じて実行時の引数を受け入れます。

.. ヒント::

    テンプレート引数は任意の順序で指定できます。

.. リスト表::
    :幅: 30 30 40
    :ヘッダー列: 1
    :配列: 左

    * - 引数
      - オプション
      - 目的

    * * ExecutionSpace
      * ``Serial``, ``OpenMP``, ``Threads``, ``Cuda``, ``HIP``, ``SYCL``, ``HPX``
      * カーネルを実行する実行スペースを指定します。デフォルトは ``Kokkos::DefaultExecutionSpace`` です。

    * * スケジュール
      * ``Schedule<Dynamic>``, ``Schedule<Static>``
      * Specify scheduling policy for work items.作業項目のスケジューリングポリシーを指定します。 ``Dynamic`` スケジューリングは、作業窃取キューを通じて実装されます。デフォルトは、マシンおよびバックエンド固有です。

    * * IndexType
      * 例えば、 ``IndexType<int>``
      * イテレーション空間を走査するために使用する整数型を指定します。 デフォルトでは、`ExecutionSpaceConcept <execution_spaces.html#typedefs>`__ の ``size_type`` が使用されます。 バックエンドによっては、パフォーマンスに影響を与える可能性があります。

    * * LaunchBounds
      * ``LaunchBounds<MaxThreads, MinBlocks>``
      * CUDA/HIPの起動境界に関するコンパイラへのヒントを指定します。

    * * WorkTag
      * ``SomeClass``
      * ファンクタ演算子を呼び出す際に、使用する作業タグのタイプを指定します。 任意のタグタイプにすることが可能です (つまり、 an [empty](https://en.cppreference.com/w/cpp/types/is_empty) 構造体またはクラス )。デフォルトは、``void``　です。


.. toctree::
   :hidden:
   :maxdepth: 1

   ./policies/ExecutionPolicyConcept
   ./policies/MDRangePolicy
   ./policies/NestedPolicies
   ./policies/RangePolicy
   ./policies/TeamHandleConcept
   ./policies/TeamPolicy
   ./policies/TeamThreadMDRange
   ./policies/TeamThreadRange
   ./policies/TeamVectorMDRange
   ./policies/TeamVectorRange
   ./policies/ThreadVectorMDRange
   ./policies/ThreadVectorRange
