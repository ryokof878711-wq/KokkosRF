``atomic_[op]_fetch``
=====================

.. role:: cpp(code)
    :language: cpp

ヘッダーファイル: ``<Kokkos_Core.hpp>``

使用例
-----

.. code-block:: cpp

    new_value =  atomic_[op]_fetch(ptr_to_value,update_value);

 ``ptr_to_value`` で与えられたアドレスの変数を、関連する演算に従って、原子的に　``update_value``　で更新し、 そのアドレスで見つけた前の値を返します。

ディスクリプション
-----------

.. cpp:function:: template<class T> T atomic_add_fetch(T* const ptr_to_value, const T value);

   ``*ptr_to_value += value; return *ptr_to_value;``　を原子的に実行します。

   * ``ptr_to_value``: 更新対象の値のアドレス

   * ``value``: 追加する値。

.. cpp:function:: template<class T> T atomic_and_fetch(T* const ptr_to_value, const T value);

   ``*ptr_to_value &= value; return *ptr_to_value;``　を原子的に実行します。

   * ``ptr_to_value``: 更新対象の値のアドレス

   * ``value``: 元の値を組み合わせるための値。

.. cpp:function:: template<class T> T atomic_div_fetch(T* const ptr_to_value, const T value);

   ``*ptr_to_value /= value; return *ptr_to_value;``　を原子的に実行します。

   * ``ptr_to_value``: 更新対象の値のアドレス。

   * ``value``: 元の値を割るための値..

.. cpp:function:: template<class T> T atomic_lshift_fetch(T* const ptr_to_value, const unsigned shift);

   ``*ptr_to_value << shift; return *ptr_to_value;``　を原子的に実行します。

   * ``ptr_to_value``: 更新対象の値のアドレス。

   * ``shift``: 元の変数をシフトする値。

.. cpp:function:: template<class T> T atomic_max_fetch(T* const ptr_to_value, const T value);

   Atomically executes ``*ptr_to_value = max(*ptr_to_value, value); return *ptr_to_value;``.

   * ``ptr_to_value``: 更新対象の値のアドレス。

   * ``value``: value which to take the maximum with.

.. cpp:function:: template<class T> T atomic_min_fetch(T* const ptr_to_value, const T value);

   Atomically executes ``*ptr_to_value = min(*ptr_to_value, value); return *ptr_to_value;``.

   * ``ptr_to_value``: 更新対象の値のアドレス。

   * ``value``: value which to take the minimum with.

.. cpp:function:: template<class T> T atomic_mul_fetch(T* const ptr_to_value, const T value);

   Atomically executes ``*ptr_to_value *= value; return *ptr_to_value;``.

   * ``ptr_to_value``: 更新対象の値のアドレス。

   * ``value``: value by which to multiply the original value.

.. cpp:function:: template<class T> T atomic_mod_fetch(T* const ptr_to_value, const T value);

   Atomically executes ``*ptr_to_value %= value; return *ptr_to_value;``.

   * ``ptr_to_value``: 更新対象の値のアドレス。

   * ``value``: value with which to combine the original value.

.. cpp:function:: template<class T> T atomic_or_fetch(T* const ptr_to_value, const T value);

   Atomically executes ``*ptr_to_value |= value; return *ptr_to_value;``.

   * ``ptr_to_value``: 更新対象の値のアドレス。

   * ``value``: value with which to combine the original value.

.. cpp:function:: template<class T> T atomic_rshift_fetch(T* const ptr_to_value, const unsigned shift);

   Atomically executes ``*ptr_to_value >> shift; return *ptr_to_value;``.

   * ``ptr_to_value``: 更新対象の値のアドレス。

   * ``shift``: value by which to shift the original variable.

.. cpp:function:: template<class T> T atomic_sub_fetch(T* const ptr_to_value, const T value);

   Atomically executes ``*ptr_to_value -= value; return *ptr_to_value;``.

   * ``ptr_to_value``: 更新対象の値のアドレス。

   * ``value``: value to be subtracted.

.. cpp:function:: template<class T> T atomic_xor_fetch(T* const ptr_to_value, const T value);

   Atomically executes ``*ptr_to_value ^= value; return *ptr_to_value;``.

   * ``ptr_to_value``: 更新対象の値のアドレス。

   * ``value``: value with which to combine the original value.
