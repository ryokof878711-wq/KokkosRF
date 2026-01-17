
.. role:: cpp(code)
	:language: cpp

``UnorderedMap``
================

ヘッダーファイル: ``<Kokkos_UnorderedMap.hpp>``

Kokkos　の無順序マップは、数万件の同時挿入を効率的に処理するよう設計されています。
そのため、APIは、標準　unordered_map　とは大きく異なります。
2つの主な違いは、*固定容量*　および　*指数ベース*　です。

- *固定容量*: 並列アルゴリズム内では、unordered_map　の容量は固定されます。
  つまり、マップの容量を超えると、インサートが失敗する可能性があるということです。
  マップの容量はホストから変更(リハッシュ)可能です。

- *インデックスベース*: ポインタやイテレーターを返す代わりに(メモリ空間間の移動時に動作しません)、整数インデックスを使用します。 これにより、マップはキャッシュに適した方法でデータを保存できます。 インデックスの利用可能性は、``uint32_t``　に基づく内部の原子ビットセットによって管理されます。


ディスクリプション
-----------

.. cpp:class:: template <typename Key, typename Value, typename Device = Kokkos::DefaultExecutionSpace> UnorderedMap

   :tparam Key: POD　である必要があります (POD)

   :tparam Value: `void` は順序のない集合を示します。 そうでなければ、 :cpp:any:`Value` は簡単にコピー可能でなければなりません。 マップが :cpp:any:`SequentialHostInit` プロパティで作成された場合、:cpp:any:`Value` は :cpp:class:`View`　である可能性があります。
   
     .. versionchanged:: 4.7
           :cpp:any:`Value` can now be :cpp:class:`View`

   :tparam Device: デバイスとは、以下のパブリックな型定義または型エイリアスを持つクラスまたは構造体を指します　: `execution_space`、 `memory_space`、および `device_type`

   .. rubric:: コンストラクタ

   .. cpp:function:: UnorderedMap(uint32_t capacity_hint);

      少なくとも　capacity_hint　数のオブジェクトを収容できる十分なスペースを確保するマップを作成します。

      .. warning:: ホストのみ

   .. cpp:function:: UnorderedMap(const ALLOC_PROP &prop, uint32_t capacity_hint);

      少なくとも　capacity_hint　数のオブジェクトを収容できる十分なスペースをを持つプロパティを使用しているマップを作成します。

      .. warning:: Host Only

      .. versionadded:: 4.2
      
      .. versionchanged:: 4.7
              :cpp:any:`prop` は、現在 :cpp:type:`SequentialHostInit`　を含むことが可能です。
   .. rubric:: Public Member Functionsパブリックメンバー関数

   .. cpp:function:: clear();

      マップをクリアします。

      .. warning:: ホストのみ

   .. cpp:function:: bool rehash(uint32_t requested_capacity);

      指定された容量に合わせてマップを再構成します。現在のサイズは、下限０　（容量）として使用されます。

      .. warning:: ホストのみ

   .. cpp:function:: uint32_t size() const;

      マップの現在のサイズ、０（容量）

      .. warning:: ホストのみ

   .. cpp:function:: KOKKOS_INLINE_FUNCTION uint32_t capacity() const;

       マップの容量、 O(1)

   .. cpp:function:: KOKKOS_INLINE_FUNCTION UnorderedMapInsertResult insert(key) const;

      Insert the given key into the map with a default constructed value

   .. cpp:function:: KOKKOS_INLINE_FUNCTION UnorderedMapInsertResult insert(Key key, Value value, Insert op = NoOp) const;

      Insert the given key/value pair into the map and optionally specify
      the operator, op, used for combining values if key already exists

   .. cpp:function:: KOKKOS_INLINE_FUNCTION uint32_t find(Key key) const

      Return the index of the key if it exist, otherwise return invalid_index

   .. cpp:function:: KOKKOS_INLINE_FUNCTION bool exists(Key key) const;

      Does the key exist in the map

   .. cpp:function:: KOKKOS_INLINE_FUNCTION bool valid_at(uint32_t index) const;

      Is the current index a valid key/value pair

   .. cpp:function:: KOKKOS_INLINE_FUNCTION Key key_at(uint32_t index) const;

      Return the current key at the index

   .. cpp:function:: KOKKOS_INLINE_FUNCTION Value value_at(uint32_t index) const;

      Return the current value at the index

   .. cpp:function:: KOKKOS_INLINE_FUNCTION constexpr bool is_allocated() const;

      Return true if the internal views (keys, values, hashmap) are allocated

   .. cpp:function:: create_copy_view(UnorderedMap<SKey, SValue, SDevice, Hasher, EqualTo> const &src);

      For the calling ``UnorderedMap``, allocate views to have the same capacity as ``src``, and copy data from ``src``.

   .. cpp:function:: allocate_view(UnorderedMap<SKey, SValue, SDevice, Hasher, EqualTo> const &src);

      Allocate views of the calling ``UnorderedMap`` to have the same capacity as ``src``.

   .. cpp:function:: deep_copy_view(UnorderedMap<SKey, SValue, SDevice, Hasher, EqualTo> const &src);

      Copy data from ``src`` to the calling ``UnorderedMap``.

   .. rubric:: Non-Member Functions

   .. cpp:function:: inline void deep_copy(UnorderedMap<DKey, DT, DDevice, Hasher, EqualTo> &dst, const UnorderedMap<SKey, ST, SDevice, Hasher, EqualTo> &src);

      Copy an ``UnorderedMap`` from ``src`` to ``dst``.

      .. warning::  From Kokkos 4.4, ``src.capacity() == dst.capacity()`` is required

   .. cpp:function:: UnorderedMap<Key, ValueType, Device, Hasher, EqualTo>::HostMirror create_mirror(const UnorderedMap<Key, ValueType, Device, Hasher, EqualTo> &src);

      Create a ``HostMirror`` for an ``UnorderedMap``.

.. cpp:class:: UnorderedMapInsertResult

   .. rubric:: Public Methods

   .. cpp:function:: KOKKOS_INLINE_FUNCTION bool success() const;

      Was the key/value pair successfully inserted into the map

   .. cpp:function:: KOKKOS_INLINE_FUNCTION bool existing() const;

      Is the key already present in the map

   .. cpp:function:: KOKKOS_INLINE_FUNCTION bool failed() const;

      Did the insert fail?

   .. cpp:function:: KOKKOS_INLINE_FUNCTION uint32_t index() const;

      Index where the key exists in the map as long as failed() == false

.. cpp:struct:: template <class ValueTypeView, class ValuesIdxType> UnorderedMapInsertOpTypes

   :tparam ValueTypeView: The UnorderedMap value array type.

   :tparam ValuesIdxType: The index type for lookups in the value array.

   .. rubric:: *Public* Insertion Operator Types

   .. cpp:struct:: NoOp

        Insert the given key/value pair into the map

   .. cpp:struct:: AtomicAdd

       Duplicate key insertions sum values together.


.. _unordered_map_insert_op_types_noop:

Insertion using default ``UnorderedMapInsertOpTypes::NoOp``
-----------------------------------------------------------

There are 3 potential states for every insertion which are reported by the ``UnorderedMapInsertResult``:

- ``success``: implies that the current thread has successfully inserted its key/value pair

- ``existing``: implies that the key is already in the map and its current value is unchanged

- ``failed`` means that either the capacity of the map was exhausted or that a free index was not found
  with a bounded search of the internal atomic bitset. A ``failed`` insertion requires the user to increase
  the capacity (``rehash``) and restart the algorithm.

.. code-block:: cpp

    // use the default NoOp insert operation
    using map_op_type = Kokkos::UnorderedMapInsertOpTypes<value_view_type, size_type>;
    using noop_type   = typename map_op_type::NoOp;
    noop_type noop;
    parallel_for(N, KOKKOS_LAMBDA (uint32_t i) {
      map.insert(i, values(i), noop);
    });
    // OR;
    parallel_for(N, KOKKOS_LAMBDA (uint32_t i) {
      map.insert(i, values(i));
    });
  
Insertion using ``UnorderedMapInsertOpTypes::AtomicAdd``
--------------------------------------------------------

The behavior from :ref:`unordered_map_insert_op_types_noop` holds true with the
exception that the ``UnorderedMapInsertResult``:

- ``existing`` implies that the key is already in the map and the existing value at key was summed
  with the new value being inserted.

.. code-block:: cpp

    // use the AtomicAdd insert operation
    using map_op_type     = Kokkos::UnorderedMapInsertOpTypes<value_view_type, size_type>;
    using atomic_add_type = typename map_op_type::AtomicAdd;
    atomic_add_type atomic_add;
    parallel_for(N, KOKKOS_LAMBDA (uint32_t i) {
      map.insert(i, values(i), atomic_add);
    });


Iteration
---------

Iterating over Kokkos' ``UnorderedMap`` is different from iterating over a standard container. The pattern is to iterate over the capacity of the map and check if the current index is valid.

Example
~~~~~~~

.. code-block:: cpp

    // assume umap is an existing Kokkos::UnorderedMap
    parallel_for(umap.capacity(), KOKKOS_LAMBDA (uint32_t i) {
        if( umap.valid_at(i) ) {
            auto key   = umap.key_at(i);
            auto value = umap.value_at(i);
            ...
        }
    });
