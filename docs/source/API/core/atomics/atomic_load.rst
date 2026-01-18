``atomic_load``
===============

.. role:: cpp(code)
    :language: cpp

``<Kokkos_Atomic.hpp>`` which is included from ``<Kokkos_Core.hpp>``から含まれている、ヘッダー <Kokkos_Atomic.hpp> に定義されています。

使用例
-----

.. code-block:: cpp

    auto current = atomic_load(&obj);

原子的に、``obj``　の現在の値を取得します。

ディスクリプション
-----------

.. cpp:function:: template<class T> T atomic_load(T* ptr);

   ``*ptr``　の内容を原子的に読み取り、それを返します。

   ``{ T val = *ptr; return val; }``

   :param ptr: 現在の値を返すべきオブジェクトのアドレス。
   :returns: the value that is held by the object pointed to by ``ptr``

See also
--------
* `atomic_store <atomic_store.html>`_: atomically replaces the value of the referenced object with a non-atomic argument
* `atomic_exchange <atomic_exchange.html>`_: atomically replaces the value of the referenced object and obtains the value held previously
