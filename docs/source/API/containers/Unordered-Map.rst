
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

      指定されたキーをデフォルト値で構築された値と共にマップに挿入します。

   .. cpp:function:: KOKKOS_INLINE_FUNCTION UnorderedMapInsertResult insert(Key key, Value value, Insert op = NoOp) const;

      指定されたキー/値のペアをマップに挿入し、オプションで
      キーが既に存在する場合に値を結合するために使用する演算子 op を指定します。
     
   .. cpp:function:: KOKKOS_INLINE_FUNCTION uint32_t find(Key key) const

      キーが存在する場合、そのインデックスを返し、存在しない場合はinvalid_indexを返します。

   .. cpp:function:: KOKKOS_INLINE_FUNCTION bool exists(Key key) const;

      マップ内にキーが存在しますか。

   .. cpp:function:: KOKKOS_INLINE_FUNCTION bool valid_at(uint32_t index) const;

      現在のインデックスは有効なキー/値ペアですか？

   .. cpp:function:: KOKKOS_INLINE_FUNCTION Key key_at(uint32_t index) const;

      インデックスの現在のキーを返します。

   .. cpp:function:: KOKKOS_INLINE_FUNCTION Value value_at(uint32_t index) const;

      インデックスの現在の値を返します。

   .. cpp:function:: KOKKOS_INLINE_FUNCTION constexpr bool is_allocated() const;

      内部ビュー（キー、値、ハッシュマップ）が割り当てられている場合に true を返します。

   .. cpp:function:: create_copy_view(UnorderedMap<SKey, SValue, SDevice, Hasher, EqualTo> const &src);

      ``UnorderedMap``　を呼び出すために、 ビューを割り当てて、　``src``　と同じ容量を持たせ、``src``　からデータをコピーします。

   .. cpp:function:: allocate_view(UnorderedMap<SKey, SValue, SDevice, Hasher, EqualTo> const &src);

      ``src``　と同じ容量を持たせるために、 ``UnorderedMap`` の呼び出しを配分します。

   .. cpp:function:: deep_copy_view(UnorderedMap<SKey, SValue, SDevice, Hasher, EqualTo> const &src);

      ``src`` から ``UnorderedMap``　呼び出しにデータをコピーします。

   .. rubric:: Non-Member Functions

   .. cpp:function:: inline void deep_copy(UnorderedMap<DKey, DT, DDevice, Hasher, EqualTo> &dst, const UnorderedMap<SKey, ST, SDevice, Hasher, EqualTo> &src);

      ``src`` から ``dst``　に　``UnorderedMap``　をコピーします。

      .. warning::  Kokkos 4.4　から ``src.capacity() == dst.capacity()`` が必要です。

   .. cpp:function:: UnorderedMap<Key, ValueType, Device, Hasher, EqualTo>::HostMirror create_mirror(const UnorderedMap<Key, ValueType, Device, Hasher, EqualTo> &src);

      ``UnorderedMap``のために、``HostMirror``　を作成します。

.. cpp:class:: UnorderedMapInsertResult

   .. rubric:: Public Methodsパブリックメソッド

   .. cpp:function:: KOKKOS_INLINE_FUNCTION bool success() const;

     キーと値のペアはマップに正常に挿入されていましたか？

   .. cpp:function:: KOKKOS_INLINE_FUNCTION bool existing() const;

      キーは既にマップ内に存在しますか？

   .. cpp:function:: KOKKOS_INLINE_FUNCTION bool failed() const;

      挿入は出来ませんでしたか・

   .. cpp:function:: KOKKOS_INLINE_FUNCTION uint32_t index() const;

      failed() == falsefailed() == false である限りにおいて、キーがマップ内に存在するインデックス

.. cpp:struct:: template <class ValueTypeView, class ValuesIdxType> UnorderedMapInsertOpTypes

   :tparam ValueTypeView: UnorderedMap 値配列型。

   :tparam ValuesIdxType: The index type for lookups in the value array.

   .. rubric:: *Public* 挿入演算子の型

   .. cpp:struct:: NoOp

        既定のキー/値をマップ内に挿入します。
   .. cpp:struct:: AtomicAdd

       キー挿入合計値を、合わせて複写転送します。


.. _unordered_map_insert_op_types_noop:

デフォルト　``UnorderedMapInsertOpTypes::NoOp``　使用による挿入。
-----------------------------------------------------------

``UnorderedMapInsertResult``:　によって報告されるすべての挿入に対して、3つの状態の可能性が報告されます:

- ``success``: 現在のスレッドがキー/値ペアを正常に挿入したことを意味します。

- ``existing``: キーはすでにマップ上にあり、現在の値が変わっていないことを意味します。

- ``failed`` は、マップの容量が使い果たされたか、内部原子　bitset　の有界探索で自由インデックスが見つからなかったことを意味します。 挿入が、``failed``　の場合、ユーザーは容量を増やして(rehash)し、アルゴリズムを再起動する必要があります。 

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
  
Insertion using ``UnorderedMapInsertOpTypes::AtomicAdd``を用いた挿入 
--------------------------------------------------------

:ref:`unordered_map_insert_op_types_noop` からの挙動は :ref:'unordered_map_insert_op_types_noop' は、 
``UnorderedMapInsertResult``を除き真実です :

- ``existing`` は、キーがすでにマップに存在し、
  キーの既存の値を新しい値と合計したことを意味します。 

.. code-block:: cpp

    // AtomicAdd 挿入演算を使用
    using map_op_type     = Kokkos::UnorderedMapInsertOpTypes<value_view_type, size_type>;
    using atomic_add_type = typename map_op_type::AtomicAdd;
    atomic_add_type atomic_add;
    parallel_for(N, KOKKOS_LAMBDA (uint32_t i) {
      map.insert(i, values(i), atomic_add);
    });


イテレーション
---------

 Kokkosの　``UnorderedMap``　を反復することは、標準的なコンテナに対するイテレーションとは異なります。パターンとしては、マップの容量を反復し、現在のインデックスが有効かどうかを確認します。

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
