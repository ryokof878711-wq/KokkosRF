``Kokkos::kokkos_swap``
=======================

.. role:: cpp(code)
    :language: cpp

 ``<Kokkos_Core.hpp>``　から含まれるヘッダー ``<Kokkos_Swap.hpp>``:sup:`(4.3より)`　に定義。

.. code-block:: cpp

    template <class T>
    KOKKOS_FUNCTION constexpr void
    kokkos_swap(T& a, T& b) noexcept(std::is_nothrow_move_constructible_v<T> &&
                                     std::is_nothrow_move_assignable_v<T>);  // (1) (4.3より)

    template <class T2, std::size_t N>
    KOKKOS_FUNCTION constexpr void
    kokkos_swap(T2 (&a)[N], T2 (&b)[N]) noexcept(noexcept(*a, *b));  // (2) (4.3より)


　1) 　値　``a`` および　``b``　を入れ替えます。 ``std::is_move_constructible_v<T> && std::is_move_assignable_v<T>``　が　
``true``　でなければ、このオーバーロードは、
オーバーロード解決に関与しません。

　2) 　配列 ``a`` および　``b``　を入れ替えます。 ``T2`` が入れ替え可能でなければ、
   このオーバーロードは、オーバーロード解決に関与しません。

注意事項
-----
.. _std_swap: https://en.cppreference.com/w/cpp/algorithm/swap

.. |std_swap| replace:: ``std::swap``

``kokkos_swap`` は、 |std_swap|_　と同じ機能性を提供します。  `ADL
<https://en.cppreference.com/w/cpp/language/adl>`_　が理由で、 単に
``swap``　と呼ぶことはできない、または、一部の状況において、オーバーロード解決における曖昧性が生じます。
