最小/最大 演算子
==========================

.. role:: cpp(code)
    :language: cpp

.. _StandardLibraryHeaderAlgorithm: https://en.cppreference.com/w/cpp/header/algorithm

.. |StandardLibraryHeaderAlgorithm| replace:: ``<algorithm>``

ヘッダー ``<Kokkos_Core.hpp>``　に定義。

標準ライブラリヘッダー |StandardLibraryHeaderAlgorithm|_ から最小値/最大値および関連する操作を提供します。

min/max および clamp 関数テンプレートは、Kokkos 3.7 以降、``Kokkos::`` 名前空間で定義されています。

.. _min: https://en.cppreference.com/w/cpp/algorithm/min

.. |min| replace:: ``min``

.. _max: https://en.cppreference.com/w/cpp/algorithm/max

.. |max| replace:: ``max``

.. _minmax: https://en.cppreference.com/w/cpp/algorithm/minmax

.. |minmax| replace:: ``minmax``

.. _clamp: https://en.cppreference.com/w/cpp/algorithm/clamp

.. |clamp| replace:: ``clamp``


========== ============================================================
|min|_     規定値の小さい方を返します。
|max|_     規定値の大きい方を返します。
|minmax|_  規定値の小さい方と大きい方を返します。
|clamp|_   値を一対の境界値の間に収めます。
========== ============================================================

注意事項
-----

.. _KokkosClamp: https://github.com/kokkos/kokkos/blob/4.3.00/core/src/Kokkos_Clamp.hpp

.. |KokkosClamp| replace:: ``<Kokkos_Clamp.hpp>``

.. _KokkosMinMax: https://github.com/kokkos/kokkos/blob/4.3.00/core/src/Kokkos_MinMax.hpp

.. |KokkosMinMax| replace:: ``<Kokkos_MinMax.hpp>``

* バージョン4.3より、 これらの関数を利用可能にするには、それぞれ |KokkosClamp|_ および |KokkosMinMax|_ が含まれる場合があります。

----

以下も参照
--------

.. _min_element: ../../algorithms/std-algorithms/all/StdMinElement.html

.. |min_element| replace:: ``min_element``

.. _max_element: ../../algorithms/std-algorithms/all/StdMaxElement.html

.. |max_element| replace:: ``max_element``

.. _minmax_element: ../../algorithms/std-algorithms/all/StdMinMaxElement.html

.. |minmax_element| replace:: ``minmax_element``

|min_element|_: 範囲内で、最小要素を返します。

|max_element|_: 範囲内で、最大要素を返します。

|minmax_element|_: 範囲内で、最小及び最大要素を返します。
