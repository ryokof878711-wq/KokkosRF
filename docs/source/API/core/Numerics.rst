数値計算
=========

.. toctree::
   :maxdepth: 1

   numerics/mathematical-constants.md

ヘッダー ``<Kokkos_MathematicalConstants.hpp>`` is a backport of the C++20 標準ライブラリヘッダー ``<numbers>`` のバックポートであり、 ``pi`` または ``sqrt2``　等いくつかの数学定数を提供します。

.. toctree::
   :maxdepth: 1

   numerics/mathematical-functions.md

The header ``<Kokkos_MathematicalFunctions.hpp>`` provides a consistent and portable overload set for standard C
library mathematical functions, such as ``fabs``, ``sqrt``, and ``sin``.

.. toctree::
   :maxdepth: 1

   numerics/numeric-traits.md

ヘッダー ``<Kokkos_NumericTraits.hpp>`` は、 C++23 標準ライブラリに追加されている新たなファシリティを実装し、 ``std::numeric_limits``　の代替えを目的とします。

.. toctree::
   :maxdepth: 1

   numerics/bit-manipulation.md

ヘッダー ``<Kokkos_BitManipulation.hpp>`` は、is a backport of the C++20 標準ライブラリヘッダー ``<bit>`` のバックポートであり、
個々のビットおよびビットシーケンスにアクセス、操作および処理するためのいくつかの関数テンプレートを提供します。

.. toctree::
   :maxdepth: 1

   複素数演算 <numerics/complex>

ヘッダー　``<Kokkos_Complex.hpp>``　は、 複素数の　Kokko　互換実装を提供し、``std::complex``　の機能性を反映します。
