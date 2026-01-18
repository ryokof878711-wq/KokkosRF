``atomic_fetch_[op]``
=====================

.. role:: cpp(code)
   :language: cpp

ヘッダーファイル: <Kokkos_Core.hpp>

使用例
-----

.. code-block:: cpp

   old_value =  atomic_fetch_[op](ptr_to_value, update_value);

``ptr_to_value`` で与えられたアドレスの変数を、関連する操作　``op``　に従って、原子的に``update_value``　で更新し、
そのアドレスで見つけた前の値を返します。

ディスクリプション
-----------

.. cpp:function:: template<class T> T atomic_fetch_add(T* const ptr_to_value, const T value);

   ``tmp = *ptr_to_value; *ptr_to_value += value; return tmp;``　を原子的に実行します。

   :param ptr_to_value: 更新対象の値のアドレス
   :param value: 付加価値

.. cpp:function:: template<class T> T atomic_fetch_and(T* const ptr_to_value, const T value);

   Atomically executes ``tmp = *ptr_to_value; *ptr_to_value &= value; return tmp;``　を原子的に実行します。

   :param ptr_to_value: 更新対象の値のアドレス
   :param value: 元の値を組み合わせるための価値。

.. cpp:function:: template<class T>  T atomic_fetch_div(T* const ptr_to_value, const T value);

   ``tmp = *ptr_to_value; *ptr_to_value /= value; return tmp;``　を原子的に実行します。

   :param ptr_to_value: 更新対象の値のアドレス
   :param value: 元の値を割る値..

.. cpp:function:: template<class T> T atomic_fetch_lshift(T* const ptr_to_value, const unsigned shift);

   Atomically executes ``tmp = *ptr_to_value; *ptr_to_value << shift; return tmp;``　を原子的に実行します。　

   :param ptr_to_value: 更新対象の値のアドレス
   :param shift: value by which to shift the original variable.元の変数をシフトする値

.. cpp:function:: template<class T> T atomic_fetch_max(T* const ptr_to_value, const T value);

   Atomically executes ``tmp = *ptr_to_value; *ptr_to_value = max(*ptr_to_value, value); return tmp;``　を原子的に実行します。

   :param ptr_to_value: 更新対象の値のアドレス
   :param value: 最大値を取るべき値。

.. cpp:function:: template<class T> T atomic_fetch_min(T* const ptr_to_value, const T value);

   Atomically executes ``tmp = *ptr_to_value; *ptr_to_value = min(*ptr_to_value, value); return tmp;``　を原子的に実行します。

   :param ptr_to_value: 更新対象の値のアドレス
   :param value: 最小値を取るべき値。

.. cpp:function:: template<class T> T atomic_fetch_mul(T* const ptr_to_value, const T value);

   Atomically executes ``tmp = *ptr_to_value; *ptr_to_value *= value; return tmp;``　を原子的に実行します。

   :param ptr_to_value: 更新対象の値のアドレス
   :param value: 元の値に乗じる値。

.. cpp:function:: template<class T> T atomic_fetch_mod(T* const ptr_to_value, const T value);

   ``tmp = *ptr_to_value; *ptr_to_value %= value; return tmp;``　を原子的に実行します。

   :param ptr_to_value: 更新対象の値のアドレス
   :param value: 元の値を組み合わせるための値。

.. cpp:function:: template<class T> T atomic_fetch_or(T* const ptr_to_value, const T value);

   ``tmp = *ptr_to_value; *ptr_to_value |= value; return tmp;``　を原子的に実行します。

   :param ptr_to_value: address of the value to be updated更新対象の値のアドレス
   :param value: 元の値を組み合わせるための値。

.. cpp:function:: template<class T> T atomic_fetch_rshift(T* const ptr_to_value, const unsigned shift);

   ``tmp = *ptr_to_value; *ptr_to_value >> shift; return tmp;``　を原子的に実行します。

   :param ptr_to_value: 更新対象の値のアドレス
   :param shift: value by which to shift the original variable.

.. cpp:function:: template<class T> T atomic_fetch_sub(T* const ptr_to_value, const T value);

   Atomically executes ``*ptr_to_value -= value``　を原子的に実行します。

   :param ptr_to_value: address of the value to be updated
   :param value: value to be subtracted..

.. cpp:function:: template<class T> T atomic_fetch_xor(T* const ptr_to_value, const T value);

   ``tmp = *ptr_to_value; *ptr_to_value ^= value; return tmp;``　を原子的に実行します。

   :param ptr_to_value: address of the value to be updated
   :param value: value with which to combine the original value.
