組み込みリデューサー
=================

`ReducerConcept <builtinreducers/ReducerConcept.html>`__ は、リデューサーの概念を提供します。

`parallel_reduce <parallel-dispatch/parallel_reduce.html>`__ と組み合わせて使用されるリデューサーオブジェクト。

.. リスト表::
   :幅: 25 75
   :ヘッダー列: 1

   * - リデューサー
     - ディスクリプション
   * - `BAnd <builtinreducers/BAnd.html>`__
     - バイナリ 'And' 還元
   * - `BOr <builtinreducers/BOr.html>`__
     - バイナリ 'Or' 還元
   * - `LAnd <builtinreducers/LAnd.html>`__
     - 論理的 'And' 還元
   * - `LOr <builtinreducers/LOr.html>`__
     - 論理的' Or' 還元
   * - `Max <builtinreducers/Max.html>`__
     - 最大還元
   * - `MaxLoc <builtinreducers/MaxLoc.html>`__
     - 最大値および関連インデックスを提供する還元
   * - `Min <builtinreducers/Min.html>`__
     - 最小還元
   * - `MinLoc <builtinreducers/MinLoc.html>`__
     - 最小値および関連インデックスを提供する還元
   * - `MinMax <builtinreducers/MinMax.html>`__
     - 最小値および最大値の両方を提供する還元
   * - `MinMaxLoc <builtinreducers/MinMaxLoc.html>`__
     - 最小値および最大値の両方並びに関連インデックスを提供する還元
   * - `Prod <builtinreducers/Prod.html>`__
     - 乗法的還元
   * - `Sum <builtinreducers/Sum.html>`__
     - 和の還元


:cpp:struct:`reduction_identity` は、様々な還元演算における中和元（恒等値）を定義します。 特化処理は、組み込みリデューサーがユーザー定義型と連動するために不可欠です。

`還元スカラー型 <builtinreducers/ReductionScalarTypes.html>`__ は、リデューサーのストレージ用テンプレートクラスです。

.. toctree::
   :hidden:
   :maxdepth: 1

   ./builtinreducers/ReducerConcept
   ./builtinreducers/BAnd
   ./builtinreducers/BOr
   ./builtinreducers/LAnd
   ./builtinreducers/LOr
   ./builtinreducers/Max
   ./builtinreducers/MaxLoc
   ./builtinreducers/Min
   ./builtinreducers/MinLoc
   ./builtinreducers/MinMax
   ./builtinreducers/MinMaxLoc
   ./builtinreducers/Prod
   ./builtinreducers/Sum
   ./builtinreducers/ReductionScalarTypes
   ./builtinreducers/reduction_identity
