``atomic_[op]``
===============

.. role:: cpp(code)
    :language: cpp

ヘッダーファイル: ``<Kokkos_Core.hpp>``

使用例
-----

.. code-block:: cpp

    atomic_[op](ptr_to_value,update_value);

``ptr_to_value`` と　``update_value``　で与えられたアドレスの　``value``　を、関連する操作に従って、原子的に更新します。

ディスクリプション
-----------

.. cpp:function:: template<class T> void atomic_add(T* const ptr_to_value, const T value);

    ``*ptr_to_value += value``　を原子的に実行します。

   * ``ptr_to_value``: 更新対象の値のアドレス

   * ``value``: 追加する値。

.. cpp:function:: template<class T> void atomic_and(T* const ptr_to_value, const T value);

   ``*ptr_to_value &= value``を原子的に実行します。

   * ``ptr_to_value``: 更新対象の値のアドレス。

   * ``value``: 元の値を組み合わせるための値。

.. cpp:function:: template<class T> void atomic_dec(T* ptr_to_value);

   ``(*ptr_to_value)--`` or calls ``atomic_fetch_sub(ptr_to_value, T(-1))``　を原子的に実行します。

   * ``ptr_to_value``: 更新対象の値のアドレス。

.. cpp:function:: template<class T> void atomic_decrement(T* const ptr_to_value);

   ``(*ptr_to_value)--`` or calls ``atomic_fetch_sub(ptr_to_value, T(-1))``　を原子的に実行します。

   * ``ptr_to_value``: address of the to be updated value.

   .. 4.5　より廃止
      代わりに :cpp:func:'atomic_dec' を使ってください。

.. cpp:function:: template<class T> void atomic_inc(T* ptr_to_value);

   ``(*ptr_to_value)++`` or calls ``atomic_fetch_add(ptr_to_value, T(1))``　を原子的に実行します。

   * ``ptr_to_value``: 更新対象の値のアドレス。

.. cpp:function:: template<class T> void atomic_increment(T* const ptr_to_value);

   ``(*ptr_to_value)++`` or calls ``atomic_fetch_add(ptr_to_value, T(1))``　を原子的に実行します。

   * ``ptr_to_value``: 更新対象の値のアドレス。

   .. 4.5より廃止
      代わりに　:cpp:func:`atomic_inc` を使ってください。

.. cpp:function:: template<class T> void atomic_max(T* const ptr_to_value, const T value);

    ``if (value > *ptr_to_value) *ptr_to_value = value``　を原子的に実行します。

   * ``ptr_to_value``: 更新対象の値のアドレス。

   * ``value``: value which to take the maximum with.

.. cpp:function:: template<class T> void atomic_min(T* const ptr_to_value, const T value);

   Atomically executes ``if (value < *ptr_to_value) *ptr_to_value = value``.

   * ``ptr_to_value``: address of the to be updated value.

   * ``value``: value which to take the minimum with.

.. cpp:function:: template<class T> void atomic_or(T* const ptr_to_value, const T value);

   Atomically executes ``*ptr_to_value |= value``.

   * ``ptr_to_value``: address of the to be updated value.

   * ``value``: value with which to combine the original value.

.. cpp:function:: template<class T> void atomic_sub(T* const ptr_to_value, const T value);

   Atomically executes ``*ptr_to_value -= value``.

   * ``ptr_to_value``: address of the to be updated value.

   * ``value``: value to be subtracted.
