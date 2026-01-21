``reduction_identity``
======================

.. role:: cpp(code)
    :language: cpp

ヘッダー ``<Kokkos_Core.hpp>``　に定義

.. cpp:namespace:: Kokkos

.. cpp:struct:: template <typename ScalarType> reduction_identity
   :no-index-entry:

   ``reduction_identity`` クラステンプレートは、
様々な還元型に対する恒等値を返す静的メンバ関数を提供します。

   .. rubric:: 整数型および浮動小数点型の両方で利用可能な静的メンバ:

   .. cpp:function:: KOKKOS_FUNCTION static ScalarType sum()

      :returns: 組み込みリデューサー用の中立要素 :cpp:class:`Sum`

   .. cpp:function:: KOKKOS_FUNCTION static ScalarType prod()

      :returns: 組み込みリデューサー用の中立要素 :cpp:class:`Prod`

   .. cpp:function:: KOKKOS_FUNCTION static ScalarType min()

      :returns: 組み込みリデューサー用の中立要素 :cpp:class:`Min`

   .. cpp:function:: KOKKOS_FUNCTION static ScalarType max()

      :returns: 組み込みリデューサー用の中立要素 :cpp:class:`Max`
   
   .. rubric:: Static members available for integral types:

   .. cpp:function:: KOKKOS_FUNCTION static ScalarType land()

      :returns: 組み込みリデューサー用の中立要素 :cpp:class:`LAnd` (Logical AND)

   .. cpp:function:: KOKKOS_FUNCTION static ScalarType lor()

      :returns: 組み込みリデューサー用の中立要素 :cpp:class:`LOr` (Logical OR)

   .. cpp:function:: KOKKOS_FUNCTION static ScalarType band()

      :returns: 組み込みリデューサー用の中立要素 :cpp:class:`BAnd` (Bitwise AND)

   .. cpp:function:: KOKKOS_FUNCTION static ScalarType bor()

      :returns: 組み込みリデューサー用の中立要素 :cpp:class:`BOr` (Bitwise OR)

ディスクリプション
-----------

``reduction_identity``構造体は、様々な一般的な還元演算における  
単位元（中立要素とも呼ばれる）を提供します。並列還元において、恒等要素は各スレッドにおける
還元変数の開始値となる。
他の値と還元演算を用いて組み合わせた場合、
他の値を変化させません。
For example, for a sum　reduction, the identity is :math:`0`, because :math:`x+0=x`. For a product
reduction, the identity is :math:`1`, because :math:`x \times 1 = x`.

Kokkos' built-in reducers (e.g., :doc:`Sum`, :doc:`Prod`, :doc:`Min`,
:doc:`Max`) implicitly use specializations of ``reduction_identity`` to
initialize the thread-local reduction accumulators.

Kokkos provides specializations for all arithmetic types (i.e. integral and
floating-point types) as well as for :cpp:class:`complex\<T\> <complex>`.

.. note::

   ``Kokkos::reduction_identity`` is not intended for direct use in your
   application code. Instead, it serves as a customization point within the
   Kokkos framework. You should only specialize reduction_identity when you
   need to enable Kokkos's built-in reducers (like ``Kokkos::Sum``,
   ``Kokkos::Min``, ``Kokkos::Max``, etc.) to work seamlessly with your own
   user-defined data types.  This allows Kokkos to correctly determine the
   initial "identity" value for your custom type during parallel reduction
   operations.

Custom Scalar Types
-------------------

For custom (user-defined) ``ScalarType``\s to be used with Kokkos' built-in
reducers, a template specialization of
``Kokkos::reduction_identity<CustomType>`` must be defined.  This
specialization must provide static member functions corresponding to the
desired reduction operations. These functions should return an instance of
``CustomType`` initialized with the appropriate identity value.

Example: Specializing ``reduction_identity`` for a Custom Array Type
--------------------------------------------------------------------

Consider a custom struct ``array_type`` that holds an an array of integers, for
which we want to perform a sum reduction.

.. code-block:: cpp

    #include <Kokkos_Core.hpp>
 
    namespace sample {
    template <class ScalarType, int N>
    struct array_type {
      ScalarType the_array[N] = {};
 
      KOKKOS_FUNCTION
      array_type& operator+=(const array_type& src) {
        for (int i = 0; i < N; ++i) {
          the_array[i] += src.the_array[i];
        }
        return *this;
      }
    };
 
    using ValueType = array_type<int, 4>;
    } // namespace sample
 
    // Specialization of Kokkos::reduction_identity for sample::ValueType
    template <>
    struct Kokkos::reduction_identity<sample::ValueType> {
      KOKKOS_FUNCTION static sample::ValueType sum() {
        return sample::ValueType(); // Default constructor initializes to zeros
      }
      // If other reduction types were needed (e.g., min, max, prod),
      // their respective identity functions would also be defined here.
    };
 
    int main(int argc, char* argv[]) {
      Kokkos::initialize(argc, argv);
      {
        const int E = 100;
        sample::ValueType tr; // Result will be stored here
 
        Kokkos::parallel_reduce(
            "SumArray", E,
            KOKKOS_LAMBDA(const int i, sample::ValueType& lval) {
              lval.the_array[0] += 1;
              lval.the_array[1] += i;
              lval.the_array[2] += i * i;
              lval.the_array[3] += i * i * i;
            },
            Kokkos::Sum<sample::ValueType>(tr));
 
        printf("Computed result for %d is {%d, %d, %d, %d}\n", E,
               tr.the_array[0], tr.the_array[1], tr.the_array[2],
               tr.the_array[3]);
 
        // Expected values:
        // [0]: E = 100
        // [1]: sum(0..99) = 99*100/2 = 4950
        // [2]: sum(0..99) of i*i = 99*(99+1)*(2*99+1)/6 = 328350
        // [3]: sum(0..99) of i*i*i = (99*100/2)^2 = 4950^2 = 24502500
 
        printf("Expected result for %d is {%d, %d, %d, %d}\n", E,
               100, 4950, 328350, 24502500);
      }
      Kokkos::finalize();
      return 0;
    }

