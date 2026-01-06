イテレータ
=========

.. role:: cpp(code)
    :language: cpp

``Kokkos::Experimental::{begin, cbegin, end, cend}``
----------------------------------------------------

ヘッダーファイル: ``<Kokkos_StdAlgorithms.hpp>``


.. cpp:function:: template <class DataType, class... Properties> KOKKOS_INLINE_FUNCTION auto begin(const Kokkos::View<DataType, Properties...>& view);

   ``view``の先頭へKokkos **random access** イテレータを返します。

.. cpp:function:: template <class DataType, class... Properties> KOKKOS_INLINE_FUNCTION auto cbegin(const Kokkos::View<DataType, Properties...>& view);

   ``view``の先頭へKokkos const修飾**random access** イテレータを返します。

.. cpp:function:: template <class DataType, class... Properties> KOKKOS_INLINE_FUNCTION auto end(const Kokkos::View<DataType, Properties...>& view);

   ``view``の要素の末尾を過ぎた位置までKokkos **random access** イテレータを返します。

.. cpp:function:: template <class DataType, class... Properties> KOKKOS_INLINE_FUNCTION auto cend(const Kokkos::View<DataType, Properties...>& view);

   Returns a const-qualified Kokkos **random access** iterator to the element past the end of ``view``の要素の末尾を過ぎた位置までKokkos **random access** イテレータを返します。

注意事項
~~~~~

* 返されたイテレーターは、パフォーマンス上の理由からランダムアクセスです。

* ``view`` is taken as ``const`` because, within each function, we are not changing the view itself: the returned iterator operates on the view without changing its structure.

* dereferencing an iterator must be done within an execution space where ``view`` is accessible

Parameters and Requirements
~~~~~~~~~~~~~~~~~~~~~~~~~~~

* ``view``: must be a rank-1 view with ``LayoutLeft``, ``LayoutRight``, or ``LayoutStride``

Example
~~~~~~~

.. code-block:: cpp

    namespace KE = Kokkos::Experimental;
    using view_type = Kokkos::View<int*>;
    view_type a("a", 15);

    auto it = KE::begin(a);
    // if dereferenced (within a proper execution space), can modify the content of `a`

    auto itc = KE::cbegin(a);
    // if dereferenced (within a proper execution space), can only read the content of `a`

------------------

``Kokkos::Experimental::distance``
----------------------------------

.. cpp:function:: template <class IteratorType> KOKKOS_INLINE_FUNCTION constexpr typename IteratorType::difference_type distance(IteratorType first, IteratorType last);

   Returns the number of steps needed to go from ``first`` to ``last``.

Parameters and Requirements
~~~~~~~~~~~~~~~~~~~~~~~~~~~

* ``first, last``: range to calculate the distance of

Return
~~~~~~

The number of steps needed to go from ``first`` to ``last``.
The value may be negative if random-access iterators are used.


Example
~~~~~~~

.. code-block:: cpp

    namespace KE = Kokkos::Experimental;
    Kokkos::View<double*> a("a", 13);

    auto it1 = KE::begin(a);
    auto it2 = it1 + 4;
    const auto stepsA = KE::distance(it1, it2);
    // stepsA should be equal to 4

    const auto stepsB = KE::distance(it2, it1);
    // stepsB should be equal to -4

------------------

``Kokkos::Experimental::iter_swap``
-----------------------------------

.. cpp:function:: template <class IteratorType> void iter_swap(IteratorType first, IteratorType last);

   Swaps the values of the elements the given iterators are pointing to.

Parameters and Requirements
~~~~~~~~~~~~~~~~~~~~~~~~~~~

* ``first, last``: iterators to swap

Notes
~~~~~

Currently, the API does not have an execution space parameter because the operation is performed in the *default execution space*. The operation fences the default execution space.

Return
~~~~~~

None

Example
~~~~~~~

.. code-block:: cpp

    namespace KE = Kokkos::Experimental;
    Kokkos::View<double*> a("a", 13);

    auto it1 = KE::begin(a);
    auto it2 = it1 + 4;
    KE::swap(it1, it2);
