# Fortran相互運用性使用事例

## Operations on multidimensional fortran allocated arrays using Kokkos 

本例は、単純な　Fortran　プログラムから　Kokkos　を用いてDAXPY（倍精度浮動小数点演算 A * X + Y）を実行する際に、`Fortran　言語互換レイヤー（FLCL）`の使用方法を示しています。 このような使用事例は、Fortran　アプリケーション内でパフォーマンスの移植性を実現するために　Kokkos　を使用する場合に発生します。 

## プログラミング構成
本例では、[FLCL](https://github.com/kokkos/kokkos-fortran-interop) に含まれる Kokkos Fortran 相互運用ユーティリティを使用しています。
これには、Fortranで割り当てられた配列を　ndarray　に変換するための一連の　Fortran　ルーチンと、ndarray　を　Kokkos　の管理対象外ビューに変換するための一連の　C++　関数が含まれます。

ndarray　型（flcl_ndarray_t）は、ランク、次元、ストライド（dopeベクトルに相当）および平坦化されたデータを保持するシンプルな構造体です。これは　[flcl-cxx.hpp](https://github.com/kokkos/kokkos-fortran-interop/blob/master/src/flcl-cxx.hpp)　で定義および実装されています。

```c++ 
typedef struct _flcl_nd_array_t {
    size_t rank;
    size_t dims[FLCL_NDARRAY_MAX_RANK];
    size_t strides[FLCL_NDARRAY_MAX_RANK];
    void *data;
} flcl_ndarray_t;
```
これには、[flcl-f.f90](https://github.com/kokkos/kokkos-fortran-interop/blob/master/src/flcl-f.f90) に位置する Fortran 相当の型があります。

``` fortran
型, bind(C) :: nd_array_t
    integer(c_size_t) :: rank
    integer(c_size_t) :: dims(ND_ARRAY_MAX_RANK)
    integer(c_size_t) :: strides(ND_ARRAY_MAX_RANK)
    type(c_ptr) :: data
エンド型 nd_array_t
```

Fortran　で割り当てられた配列を　ndarray　に変換するには、[flcl-f.f90](https://github.com/kokkos/kokkos-fortran-interop/blob/master/src/flcl-f.f90)　で定義されている一連の手続き（インターフェースの背後で動作します）を使用します。

```fortran
インターフェイス to_nd_array
    ! 1D スペシャリゼーション
　　モジュールプロシージャ　to_nd_array_l_1d
    モジュールプロシージャ　to_nd_array_i32_1d
    モジュールプロシージャ　to_nd_array_i64_1d
    モジュールプロシージャ　to_nd_array_r32_1d
    モジュールプロシージャ　to_nd_array_r64_1d
    
    ! 2D スペシャリゼーション
    モジュールプロシージャ to_nd_array_l_2d
    モジュールプロシージャ to_nd_array_i32_2d
    モジュールプロシージャ to_nd_array_i64_2d
    モジュールプロシージャ to_nd_array_r32_2d
    モジュールプロシージャ to_nd_array_r64_2d

    ! 3D スペシャリゼーション
    モジュールプロシージャ　to_nd_array_l_3d
    モジュールプロシージャ　to_nd_array_i32_3d
    モジュールプロシージャ　to_nd_array_i64_3d
    モジュールプロシージャ　to_nd_array_r32_3d
    モジュールプロシージャ　to_nd_array_r64_3d
```

ndarray　を　Kokkos::View　に変換するには、[flcl-cxx.hpp](https://github.com/kokkos/kokkos-fortran-interop/blob/master/src/flcl-cxx.hpp)　に定義されている　view_from_ndarray　を使用します。
``` c++ 
テンプレート <typename DataType>
  Kokkos::View<DataType, Kokkos::LayoutStride, Kokkos::HostSpace, Kokkos::MemoryUnmanaged>
  view_from_ndarray(flcl_ndarray_t const &ndarray) 
```

これらは、当社の　DAXPY　の例で使用される主なユーティリティです。

まず、[axpy-ndarray-main.f90](https://github.com/kokkos/kokkos-fortran-interop/blob/master/examples/01-axpy-ndarray/axpy-ndarray-main.F90) に定義されている Fortran プログラムから始めます。

最初に、flclモジュールを読み込みます: 
``` fortran
:: flcl_mod　を使用
```
次に、2つの「Y」配列を含む配列を定義し、一方の配列は　Fortran　で　daxpy　の結果を計算するために使用され、もう一方の配列は　kokkos　で計算するために使用されます。
``` fortran 
  real(c_double), dimension(:), allocatable :: f_y
  real(c_double), dimension(:), allocatable :: c_y
  real(c_double), dimension(:), allocatable :: x
  real(c_double) :: alpha
``` 
FortranでのDAPPYの実装は、単に以下の通りです: 
``` fortran 
ii = 1, mm　を実行
    f_y(ii) = f_y(ii) + alpha * x(ii)
end do
``` 

Kokkos　における　DAXPY　の実行は、axpyの呼び出しから始まります: 
``` fortran 
axpy(c_y, x, alpha) を呼び出し
``` 

これは　[axpy-ndarray-f.f90](https://github.com/kokkos/kokkos-fortran-interop/blob/master/examples/01-axpy-ndarray/axpy-ndarray-f.f90)　で定義されています。
``` fortran 
サブルーチン axpy( y, x, alpha )
   使用、 intrinsic :: iso_c_binding
   use :: flcl_mod
   暗黙型宣言無効化
   real(c_double), dimension(:), intent(inout) :: y
   real(c_double), dimension(:), intent(in) :: x
   real(c_double), intent(in) :: alpha

   f_axpy(to_nd_array(y), to_nd_array(x), alpha)　呼び出し
エンドサブルーチン axpy
```
サブルーチン f_axpy を呼び出しますが、その前に Fortran の配列を nd_array に変換します。
f_axpy は先に定義されており、f_axpy が C ルーチン 'c_axpy' にバインドされている点に注意してください。

``` fortran
インターフェイス
    サブルーチン　f_axpy( nd_array_y, nd_array_x, alpha ) &
        & bind(c, name='c_axpy')
        使用, intrinsic :: iso_c_binding
        :: flcl_mod　を使用
        type(nd_array_t) :: nd_array_y
        type(nd_array_t) :: nd_array_x
        real(c_double) :: alpha
    エンドサブルーチン f_axpy
エンドインターフェイス
```

c_axpy は、計算に Kokkos を利用している箇所であり、[axpy-ndarray-cxx.cc](https://github.com/kokkos/kokkos-fortran-interop/blob/master/examples/01-axpy-ndarray/axpy-ndarray-cxx.cc) に定義されています。

```c++ 
void c_axpy( flcl_ndarray_t *nd_array_y, flcl_ndarray_t *nd_array_x, double *alpha ) {
  flcl::view_from_ndarray　を使用;

  auto y = view_from_ndarray<double*>(*nd_array_y);
  auto x = view_from_ndarray<double*>(*nd_array_x);

  Kokkos::parallel_for( "axpy", y.extent(0), KOKKOS_LAMBDA( const size_t idx)
  {
    y(idx) += *alpha * x(idx);
  });

  戻し;
}
```

この関数では、まず2つの　nd_array　を[`Kokkos::View`](../API/core/view/view)　に変換し、その後、単純な　DAXPY　ラムダを用いた[`Kokkos::parallel_for`](../API/core/parallel-dispatch/parallel_for)　を使用します。 

この使用事例は、Kokkos　を　Fortran　アプリケーションで使用する能力を示しており、Fortran　配列と　[`Kokkos::View`](../API/core/view/view)　の相互運用性を、FLCLで提供される　ndarray　型および変換ルーチンを通じて実現しています。
