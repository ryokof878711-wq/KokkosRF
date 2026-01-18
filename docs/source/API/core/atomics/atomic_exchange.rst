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

  原子レベルでは、``val`` をinto ``*ptr``　に挿入し、``*ptr`` のもとの値を返します。

   ``{ auto old = *ptr; *ptr = val; return old; }``

   :param ptr: 値を変更するオブジェクトのアドレス
   :param val: 参照されるオブジェクトに保存する値
   :returns: ``ptr``　が指すオブジェクトが以前に保持していた値


以下も参照
--------
* `atomic_load <atomic_load.html>`_: 参照対象の値を、原子的に取得します
* `atomic_store <atomic_store.html>`_:、原子的に、置き換えます
* `atomic_compare_exchange <atomic_compare_exchange.html>`_: 原子的に、非原子的引数と 参照対象の値を比較し、等しければ原始的交換を実行し、等しくなければ原始的負荷を実行します 
