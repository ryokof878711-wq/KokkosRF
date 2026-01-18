``atomic_assign``
=================

.. warning::
   Kokkos 4.5より、非推奨なので、
   代わりに、:cpp:func:`atomic_store` をご使用ください。

.. role:: cpp(code)
    :language: cpp

<Kokkos_Core.hpp>から含まれているヘッダー　``<Kokkos_Atomic.hpp>``　により、定義されています。

使用例
-----

.. code-block:: cpp

    atomic_assign(&obj, desired);

原子レベルでは、``obj``　の現在の値を、``desired``　に置き換えます

ディスクリプション
-----------

.. cpp:function:: template<class T> void atomic_assign(T* ptr, std::type_identity_t<T> val);

   原子レベルでは、``val`` をinto ``*ptr``　に挿入します。

   ``{ *ptr = val; }``

   :param ptr: 値が置換されるオブジェクトのアドレス
   :param val: 参照されるオブジェクトに保存する値
   :returns: (無し)

   .. 4.5より、非推奨なので、
      代わりに、:cpp:func:`atomic_store` をご使用ください。
