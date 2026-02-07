(KokkosALL)=
# `Kokkos::ALL`

ヘッダー `<Kokkos_Core.hpp>`　に定義。

```c++
namespace Kokkos{
  constexpr UNSPECIFIED-TYPE ALL = IMPLEMENTATION-DETAIL;
}
```

`Kokkos::ALL` は、指定されていない型の定数であり、次元の全要素を選択するために使用されます。


## 例

```c++
Kokkos::View<double**[5]> a("A",N0,N1);

auto s  = Kokkos::subview(a,
              5,
              Kokkos::ALL,
              Kokkos::ALL);
```
