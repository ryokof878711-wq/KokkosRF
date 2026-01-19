``atomic_fetch_[op]``
=====================

.. role:: cpp(code)
   :language: cpp

ヘッダーファイル: <Kokkos_Core.hpp>

使用例
-----

.. code-block:: cpp

   old_value =  atomic_fetch_[op](ptr_to_value, update_value);

``ptr_to_value`` で与えられたアドレスの変数を、関連する操作　``op``　に従って、原子的に　``update_value``　で更新し、
そのアドレスで見つけた前の値を返します。

ディスクリプション
-----------

.. cpp:function:: template<class T> T atomic_fetch_add(T* const ptr_to_value, const T value);

   ``tmp = *ptr_to_value; *ptr_to_value += value; return tmp;``　を原子的に実行します。

   :param ptr_to_value: 更新対象の値のアドレス
   :param value: 追加する値。

.. cpp:function:: template<class T> T atomic_fetch_and(T* const ptr_to_value, const T value);

   ``tmp = *ptr_to_value; *ptr_to_value &= value; return tmp;``　を原子的に実行します。

   :param ptr_to_value: 更新対象の値のアドレス
   :param value: 元の値を組み合わせるための値。

.. cpp:function:: template<class T>  T atomic_fetch_div(T* const ptr_to_value, const T value);

   ``tmp = *ptr_to_value; *ptr_to_value /= value; return tmp;``　を原子的に実行します。

   :param ptr_to_value: 更新対象の値のアドレス
   
.. cpp:function:: template<class T>  T atomic_fetch_div(T* const ptr_to_value, const T value);

 ``tmp = *ptr_to_value; *ptr_to_value /= value; return tmp;``　を原子的に実行します。

 　:param ptr_to_value: 更新対象の値のアドレス
   :param value: 元の値を割るための値。

.. cpp:function:: template<class T>  T atomic_fetch_inc(T* const ptr_to_value);

　``tmp = *ptr_to_value; (*ptr_to_value)++; return tmp;``　を原子的に実行します。
　
 　:param ptr_to_value: 更新対象の値のアドレス

.. cpp:function:: template<class T> T atomic_fetch_lshift(T* const ptr_to_value, const unsigned shift);

   ``tmp = *ptr_to_value; *ptr_to_value << shift; return tmp;``　を原子的に実行します。　

   :param ptr_to_value: 更新対象の値のアドレス
   :param shift: 元の変数をシフトする値。

.. cpp:function:: template<class T> T atomic_fetch_max(T* const ptr_to_value, const T value);

   ``tmp = *ptr_to_value; *ptr_to_value = max(*ptr_to_value, value); return tmp;``　を原子的に実行します。

   :param ptr_to_value: 更新対象の値のアドレス
   :param value: 最大値を取るべき値。

.. cpp:function:: template<class T> T atomic_fetch_min(T* const ptr_to_value, const T value);

   ``tmp = *ptr_to_value; *ptr_to_value = min(*ptr_to_value, value); return tmp;``　を原子的に実行します。

   :param ptr_to_value: 更新対象の値のアドレス
   :param value: 最小値を取るべき値。

.. cpp:function:: template<class T> T atomic_fetch_mul(T* const ptr_to_value, const T value);

   ``tmp = *ptr_to_value; *ptr_to_value *= value; return tmp;``　を原子的に実行します。

   :param ptr_to_value: 更新対象の値のアドレス
   :param value: 元の値に乗じる値。

.. cpp:function:: template<class T> T atomic_fetch_mod(T* const ptr_to_value, const T value);

   ``tmp = *ptr_to_value; *ptr_to_value %= value; return tmp;``　を原子的に実行します。

   :param ptr_to_value: 更新対象の値のアドレス
　 :param value:　モジュラスとして使用する値。

.. cpp:function:: template<class T> T atomic_fetch_nand(T* const ptr_to_value, const T value);

 ``tmp = *ptr_to_value; *ptr_to_value = ~(*ptr_to_value & val); return tmp;``　を原子的に実行します。

   :param ptr_to_value: 更新対象の値のアドレス
   :param value: 元の値を組み合わせるための値。

.. cpp:function:: template<class T> T atomic_fetch_or(T* const ptr_to_value, const T value);

   ``tmp = *ptr_to_value; *ptr_to_value |= value; return tmp;``　を原子的に実行します。

   :param ptr_to_value: 更新対象の値のアドレス
   :param value: 元の値を組み合わせるための値。

.. cpp:function:: template<class T> T atomic_fetch_rshift(T* const ptr_to_value, const unsigned shift);

   ``tmp = *ptr_to_value; *ptr_to_value >> shift; return tmp;``　を原子的に実行します。

   :param ptr_to_value: 更新対象の値のアドレス
   :param shift: 元の変数をシフトする値。

.. cpp:function:: template<class T> T atomic_fetch_sub(T* const ptr_to_value, const T value);

   ``*ptr_to_value -= value``　を原子的に実行します。

   :param ptr_to_value: 更新対象の値のアドレス
   :param value: 差し引かれる値..

.. cpp:function:: template<class T> T atomic_fetch_xor(T* const ptr_to_value, const T value);

   ``tmp = *ptr_to_value; *ptr_to_value ^= value; return tmp;``　を原子的に実行します。

   :param ptr_to_value: 更新対象の値のアドレス
   :param value: 元の値を組み合わせるための値。
