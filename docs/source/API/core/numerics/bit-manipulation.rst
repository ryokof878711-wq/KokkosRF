ビット操作
================

.. ロール::cpp(code)
    :language: cpp

.. ロール:: ストライク
    :クラス: ストライク

.. _KokkosBitManipulation: https://github.com/kokkos/kokkos/blob/4.1.00/core/src/Kokkos_BitManipulation.hpp

.. |KokkosBitManipulation| 置換:: ``<Kokkos_BitManipulation.hpp>``

.. _StandardLibraryHeaderBit: https://en.cppreference.com/w/cpp/header/bit

.. |StandardLibraryHeaderBit| 置換:: ``<bit>``

 ``<Kokkos_Core.hpp>``に含まれる ヘッダー　 |KokkosBitManipulation|_ に定義。

標準ライブラリヘッダー |StandardLibraryHeaderBit|_ から関数テンプレートを提供します(C++20　以降)。

 Kokkos 4.1ビット演算関数テンプレートは、Kokkos 4.1以降、Kokkos:: 名前空間で定義されています。

.. _bit_cast: https://en.cppreference.com/w/cpp/numeric/bit_cast

.. |bit_cast| replace:: ``bit_cast``

.. _byteswap: https://en.cppreference.com/w/cpp/numeric/byteswap

.. |byteswap| replace:: ``byteswap``

.. _has_single_bit: https://en.cppreference.com/w/cpp/numeric/has_single_bit

.. |has_single_bit| replace:: ``has_single_bit``

.. _bit_ceil: https://en.cppreference.com/w/cpp/numeric/bit_ceil

.. |bit_ceil| replace:: ``bit_ceil``

.. _bit_floor: https://en.cppreference.com/w/cpp/numeric/bit_floor

.. |bit_floor| replace:: ``bit_floor``

.. _bit_width: https://en.cppreference.com/w/cpp/numeric/bit_width

.. |bit_width| replace:: ``bit_width``

.. _rotl: https://en.cppreference.com/w/cpp/numeric/rotl

.. |rotl| replace:: ``rotl``

.. _rotr: https://en.cppreference.com/w/cpp/numeric/rotr

.. |rotr| replace:: ``rotr``

.. _countl_zero: https://en.cppreference.com/w/cpp/numeric/countl_zero

.. |countl_zero| replace:: ``countl_zero``

.. _countl_one: https://en.cppreference.com/w/cpp/numeric/countl_one

.. |countl_one| replace:: ``countl_one``

.. _countr_zero: https://en.cppreference.com/w/cpp/numeric/countr_zero

.. |countr_zero| replace:: ``countr_zero``

.. _countr_one: https://en.cppreference.com/w/cpp/numeric/countr_one

.. |countr_one| replace:: ``countr_one``

.. _popcount: https://en.cppreference.com/w/cpp/numeric/popcount

.. |popcount| replace:: ``popcount``

================== ============================================================
|bit_cast|_        あるタイプのオブジェクト表現を別のタイプのものとして再解釈します　(下記の注参照)
|byteswap|_        与えられた整数値のバイトを反転します
|has_single_bit|_  数が2の整数乗であるかどうかを検証します
|bit_ceil|_        与えられた値より小さい2の最小積分べき乗を求めます
|bit_floor|_       与えられた値より大きくない2の最大の積分冪を求めます
|bit_width|_       与えられた値を表現するために必要な最小ビット数を見つけます
|rotl|_            ビットごとに左回転した結果を計算します。
|rotr|_            ビットごとに右回転した結果を計算します。
|countl_zero|_     最上位ビットから連続した0ビットの数を数えます
|countl_one|_      最上位ビットから連続した1ビットの数を数えます
|countr_zero|_     連続した0ビットの数を、下位ビットから数えます
|countr_one|_      counts the number of consecutive 1 bits, starting from the least significant bit下位ビットから連続した1ビットの数を数えます
|popcount|_        counts the number of 1 bits in an unsigned integer
================== ============================================================

----

Notes
-----

* For all the above template functions, a non-``constexpr`` counterpart ending
  with the ``*_builtin`` suffix is provided in the ``Kokkos::Experimental::``
  namespace to make up for some compiler intrinsics that cannot appear in
  constant expressions.
* In contrast to its counterpart in the C++ standard library,
  ``Kokkos::bit_cast`` is not usable in constant expressions (not a
  ``constexpr`` function) as it is not implementable as a library facility
  and requires compiler magic which is not available to us.
