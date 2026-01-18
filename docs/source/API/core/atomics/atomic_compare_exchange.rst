``atomic_compare_exchange``
===========================

.. role:: cpp(code)
   :language: cpp

 ``<Kokkos_Core.hpp>`` から含まれているヘッダー ``<Kokkos_Atomic.hpp>``　で定義されています。

使用例
-----

.. code-block:: cpp

   auto old = atomic_compare_exchange(&obj, expected, desired);

``obj``　の現在の値  と　``expected``　を原子的に比較し、
等しい場合は望む値に置き換え、交換が行われたかどうかに関わらず、
アドレス　``&obj``　で以前に保存された値を必ず返します。

ディスクリプション
-----------

.. cpp:function:: template<class T> T atomic_compare_exchange(T* ptr, std::type_identity_t<T> expected, std::type_identity_t<T> desired);

   原子的に、 ``*ptr``　を ``expected``　と比較し、 それらがビット単位で等しい場合には、 前者をreplaces the former with ``desired``　と置換し、そして、呼び出し前に　``ptr``が指していた実際の値を常に返します。

   ``{ old = *ptr; if (old == expected) *ptr = desired; return old; }``

   :param ptr: テストし、変更するオブジェクトのアドレス
   :param expected: オブジェクト内で見つかると予想される値
   :param desired: オブジェクトに格納する値が予想通りである場合
   :returns: ``ptr``　が指すオブジェクトが以前に保持していた値


以下も参照
--------
* `atomic_exchange <atomic_exchange.html>`_: 参照対象の値を、原子的に置き換え、以前保持していた値を取得します。
