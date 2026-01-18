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
   :returns: ``ptr`` で指し示されたオブジェクトにより保持される値。

以下も参照
--------
* `atomic_store <atomic_store.html>`_: 参照対象の値を、非原子的な引数に、原子的に置き換えます。
* `atomic_exchange <atomic_exchange.html>`_: 原子的に参照対象の値を置き換え、以前保持していた値を取得します。
