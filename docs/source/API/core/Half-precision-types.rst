.. _api-Half-precision-types:

半精度型
====================

.. 警告::
   ``half_t`` および ``bhalf_t`` は、未だ  ``Kokkos::Experimental`` 名前空間内にあります。

型
-----
Kokkos offers portable half precision types under the name .Kokkosはポータブルな半精度型を、``Kokkos::Experimental::half_t`` および ``Kokkos::Experimental::bhalf_t``　の名称で提供します。
 - ``half_t`` は、標準の半精度浮動小数点数を表し、1ビットの符号ビット、5ビットの指数部、10ビットの小数部で構成されます。
 - ``bhalf_t`` は、'brain half'　として知られる型に対応し、1ビットの符号ビット、8ビットの指数部、7ビットの小数部を持ちます。

これらの型は、現在のバックエンド固有の型（例：　Cuda上の``__half``　）にマッピングされるか、そのような型が存在しない場合は、``float``　にマッピングされます。

マクロ ``KOKKOS_HALF_T_IS_FLOAT`` および ``KOKKOS_BHALF_T_IS_FLOAT`` は、``half_t`` および ``bhalf_t`` が 、``float`` にマップされる場合に、 ``true`` に設定され、それ自身の独立した型にマップされる場合には、 ``false`` に設定されます。

関数
---------
以下の表は、``half_t`` および ``bhalf_t`` 型で、使用可能な標準数学関数を、一覧表示します。

さらに、Cuda　および　SYCL　バックエンドでは、マークされた関数は特定の半精度関数を使用して実行されるため、より高いパフォーマンスを発揮する可能性があります。
特定の関数が存在しない場合のデフォルト動作は、半精度浮動小数点数を　``float``　にキャストし、標準関数で演算を実行した後、結果を半精度に再キャストすることです。

.. csv表::
   :ヘッダー: "Function", "``half_t`` Cuda", "``bhalf_t`` Cuda", "``half_t`` SYCL", "``bhalf_t`` SYCL"
   :幅: 自動

   "abs", "X", "X", "X", 
   "fabs","X", "X", "X", "X"
   "fmod", , , "X", 
   "remainder", , , "X", 
   "fmax","X¹", "X¹", "X", "X"
   "fmin","X¹", "X¹", "X", "X"
   "fdim", "X", "X", "X", 
   "exp", "X", "X", "X", "X"
   "exp2", "X", "X", "X", "X"
   "expm1", , , "X", 
   "log", "X", "X", "X", "X"
   "log10", "X", "X", "X", "X"
   "log2", "X", "X", "X", "X"
   "log1p", , , "X", 
   "pow", , , "X", 
   "sqrt", "X", "X", "X", "X"
   "cbrt", , , "X", 
   "hypot", , , "X", 
   "sin", "X", "X", "X", "X"
   "cos", "X", "X", "X", "X"
   "tan", , , "X", 
   "asin", , , "X", 
   "acos", , , "X", 
   "atan", , , "X", 
   "atan2", , , "X", 
   "sinh", , , "X", 
   "cosh", , , "X", 
   "tanh", , , "X", 
   "asinh", , , "X", 
   "acosh", , , "X", 
   "atanh", , , "X", 
   "erf", , , "X", 
   "erfc", , , "X", 
   "tgamma", , , "X", 
   "lgamma", , , "X", 
   "ceil", "X", "X", "X", "X"
   "floor", "X", "X", "X", "X"
   "trunc", "X", "X", "X", "X"
   "round", , , "X", 
   "nearbyint", "X", "X", , 
   "logb", , , "X", 
   "nextafter", , , "X", 
   "copysign", , , "X", 
   "isfinite", , , "X", 
   "isinf", "X²", "X²", "X", 
   "isnan", "X", "X", "X", "X"
   "signbit", , , "X", 

¹GPU_ARCH >= 80　の場合のみ

² --std=c++20 (https://docs.nvidia.com/cuda/archive/12.3.2/cuda-toolkit-release-notes/index.html#cuda-math-release-12-3)を使ってコンパイルする際、nvcc-12.2　に対して、ではありません。

例
~~~~~~~
.. code-block:: cpp

    #include<Kokkos_Core.hpp>
    #include<iostream>

    int main(int argc, char* argv[]) {
        Kokkos::ScopeGuard guard(argc, argv);
        const int N = 10;

        half_type = Kokkos::Experimental::bhalf_t;　を使用

        Kokkos::View<half_type*> view("half view", N);

        Kokkos::parallel_for("parallel region",
          N,
          KOKKOS_LAMBDA(const int i) {
            // `bhalf`　型に対して実行される指数関数で、利用可能であれば、　`float` に対して、そうでなければ、 
            view (i) = Kokkos::exp(half_type(i));
          });
    }

数値的特性
--------------

以下の標準数値特性は ``half_t`` および ``bhalf_t`` を使って、使用可能です:
 - infinity
 - finite_min
 - finite_max
 - epsilon
 - round_error
 - norm_min
 - quiet_NaN
 - signaling_NaN
 - digits
 - digits10
 - radix
 - min_exponent
 - max_exponent

例
~~~~~~~
.. code-block:: cpp

    #include<Kokkos_Core.hpp>
    #include<iostream>

    int main(int argc, char* argv[]) {
        Kokkos::ScopeGuard guard(argc, argv);

        // KOKKOS_HALF_T_IS_FLOAT　の値に依存する、プリント　24　または　11。
        std::cout << Kokkos::Experimental::digits_v<Kokkos::Experimental::half_t> << std::endl;
    }
