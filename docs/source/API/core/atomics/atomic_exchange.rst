``atomic_exchange``
===================

.. role:: cpp(code)
   :language: cpp

 ``<Kokkos_Core.hpp>``　から含まれているヘッダー　``<Kokkos_Atomic.hpp>``　により、定義されています。

使用例
-----

.. code-block:: cpp

   auto old = atomic_exchange(&obj, desired);

原子レベルでは、``obj``　の現在の値を、``desired``　に置き換え、呼び出し前に値を返します。

ディスクリプション
-----------

.. cpp:function:: template<class T> T atomic_exchange(T* ptr, std::type_identity_t<T> val);

   Atomically writes ``val`` into ``*ptr`` and returns the original value of ``*ptr``.

   ``{ auto old = *ptr; *ptr = val; return old; }``

   :param ptr: address of the object to modify
   :param val: the value to store in the referenced object
   :returns: the value held previously by the object pointed to by ``ptr``


See also
--------
* `atomic_load <atomic_load.html>`_: atomically obtains the value of the referenced object
* `atomic_store <atomic_store.html>`_: atomically replaces the value of the referenced object with a non-atomic argument
* `atomic_compare_exchange <atomic_compare_exchange.html>`_: atomically compares the value of the referenced object with non-atomic argument and performs atomic exchange if equal or atomic load if not
