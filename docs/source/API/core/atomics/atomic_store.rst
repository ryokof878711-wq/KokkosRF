``atomic_store``
================

.. role:: cpp(code)
    :language: cpp

``<Kokkos_Core.hpp>``　から含まれている、ヘッダー <Kokkos_Atomic.hpp> に定義されています。

使用例
-----

.. code-block:: cpp

    atomic_store(&obj, desired);

原子的に、 ``obj`` の現在の値を ``desired``　に置き換えます。

ディスクリプション
-----------

.. cpp:function:: template<class T> void atomic_store(T* ptr, std::type_identity_t<T> val);

   原子的に、``*ptr``　に　``val``　を挿入します。

   ``{ *ptr = val; }``

   :param ptr: 置き換える対象のオブジェクトのアドレス。
   :param val: 参照対象オブジェクトに格納する値。
   :returns: (無し)


以下も参照
--------
* `atomic_load <atomic_load.html>`_: 原始的に、参照対象オブジェクトの値を取得します。
* `atomic_exchange <atomic_exchange.html>`_: 原子的に、参照対象の値を置き換え、以前保持していた値を取得します。
