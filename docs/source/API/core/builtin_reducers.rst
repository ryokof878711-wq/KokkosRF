組み込みリデューサー
=================

`ReducerConcept <builtinreducers/ReducerConcept.html>`__ は、リデューサーの概念を提供します。

`parallel_reduce <parallel-dispatch/parallel_reduce.html>`__ と組み合わせて使用されるリデューサーオブジェクト。

.. リスト表::
   :幅: 25 75
   :ヘッダー列: 1

   * - リデューサー
     - Description
   * - `BAnd <builtinreducers/BAnd.html>`__
     - Binary 'And' reduction
   * - `BOr <builtinreducers/BOr.html>`__
     - Binary 'Or' reduction
   * - `LAnd <builtinreducers/LAnd.html>`__
     - Logical 'And' reduction
   * - `LOr <builtinreducers/LOr.html>`__
     - Logical 'Or' reduction
   * - `Max <builtinreducers/Max.html>`__
     - Maximum reduction
   * - `MaxLoc <builtinreducers/MaxLoc.html>`__
     - Reduction providing maximum and an associated index
   * - `Min <builtinreducers/Min.html>`__
     - Minimum reduction
   * - `MinLoc <builtinreducers/MinLoc.html>`__
     - Reduction providing minimum and an associated index
   * - `MinMax <builtinreducers/MinMax.html>`__
     - Reduction providing both minimum and maximum
   * - `MinMaxLoc <builtinreducers/MinMaxLoc.html>`__
     - Reduction providing both minimum and maximum and associated indices
   * - `Prod <builtinreducers/Prod.html>`__
     - Multiplicative reduction
   * - `Sum <builtinreducers/Sum.html>`__
     - Sum reduction


:cpp:struct:`reduction_identity` defines the neutral elements (identity values)
for various reduction operations. Specializing it is crucial for enabling
built-in reducers to work with user-defined types.

`Reduction Scalar Types <builtinreducers/ReductionScalarTypes.html>`__ are template classes for storage for reducers.

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
