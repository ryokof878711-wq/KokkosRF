``atomic_compare_exchange_strong``
==================================

.. warning::
   Kokkos 4.5　以降は非推奨なので、
   代わりに　`atomic_compare_exchange <atomic_compare_exchange.html>`_ をお使いください。

.. role:: cpp(code)
   :language: cpp

``<Kokkos_Core.hpp>`` から含まれている、ヘッダー ``<Kokkos_Atomic.hpp>``  に定義されています。

使用例
-----

.. code-block:: cpp

   bool was_exchanged = atomic_compare_exchange_strong(&obj, expected, desired);

原子的に、 ``obj`` の現在値をwith ``expected``　と比較し、
そして、等しければその値を　``desired``　値に置き換えます。
交換が起こった場合には、関数は　``true``　を返しますが、そうでなければ、``false`` を返します。

ディスクリプション
-----------

.. cpp:function:: template<class T> bool atomic_compare_exchange_strong(T* ptr, std::type_identity_t<T> expected, std::type_identity_t<T> desired);

  replaces the former with ``desired``.原子的に、 ``*ptr``　を ``expected``　と比較し、 それらがビット単位で等しい場合には、 前者を ``desired``　と置換し、そして、呼び出し前に　``ptr``が指していた実際の値を常に返します。

   If ``desired`` is written into ``*ptr`` then ``true`` is returned.

   `` (*ptr == expected) { *ptr = desired; return true; } であれば、false　を返します。;``

   :param ptr: テストし、変更するオブジェクトのアドレス
   :param expected: オブジェクト内で見つかると予想される値
   :param desired: オブジェクトに格納する値が予想通りである場合
   :returns: the result of the comparison, ``true`` if ``*ptr`` was equal to ``expected``, ``false`` otherwise

   .. deprecated:: 4.5
      Prefer :cpp:expr:`expected == atomic_compare_exchange(&obj, expected, desired)`
