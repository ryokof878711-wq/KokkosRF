# カバナを用いた構造体の配列および配列の構造

[Cabana](https://github.com/ECP-copa/Cabana) は、粒子ベースのシミュレーション向けに設計された、MPI+Kokkos 対応のパフォーマンスポータブルライブラリです。 本ソフトウェアは、マルチコアアーキテクチャおよび　GPU　を含む、様々なプラットフォーム上でのシミュレーションを有効化するため、粒子データ構造、アルゴリズム、およびユーティリティを提供します。

本使用事例では、Cabana　が提供する　SoA　クラスおよび　AoSoA　クラスについて説明します。 その目的は、良好なアラインメントを示す連続した要素から構成される、ベクトル化に適したコレクションを提供することです。

## 配列の構造 (SoA)

### `Cabana::SoA`
ヘッダー [`<Cabana_SoA.hpp>`](https://github.com/ECP-copa/Cabana/blob/master/core/src/Cabana_SoA.hpp)　に定義。
```C++
テンプレート <typename DataTypes, int VectorLength>
構造体 SoA;
```

`Cabana::SoA` は、指定されたベクトル長を持つ異種配列の固定サイズのコレクションを格納する方法を提供する構造体テンプレートです。

概念的には、`Cabana::SoA<Cabana::MemberTypes<float[3], char>, 8>` は `std::tuple<float[3][8], char[8]>` と同等です。

データ構造は、各粒子フィールドごとに独立した均質なデータ配列を保持し、それぞれが同じ要素数を有します。そのモチベーションは、コンパイラによるベクトル化を容易にすることです。

#### テンプレートパラメータ
`データ型`
: SoAが固定サイズの配列として格納する要素の型。
`Cabana::MemberTypes` の特殊化であることが求められ、それは、```C++　と定義されます。

テンプレート <typename... Types>
MemberTypes<Types...>;
```

`ベクトル長`
: 各データ型ごとに連続したメモリ位置に格納される要素の数。

#### 非メンバー関数
`Cabana::get`
: SoA　の指定された要素にアクセス

## 配列構造の配列 (AoSoA)

### Cabana::AoSoA
ヘッダー [`<Cabana_AoSoA.hpp>`](https://github.com/ECP-copa/Cabana/blob/master/core/src/Cabana_AoSoA.hpp)　に定義。

```C++
テンプレート <class DataTypes, class MemorySpace,
          int VectorLength = DEDUCED-FROM-MEMORY-SPACE,
          class MemoryTraits = Kokkos::MemoryManaged>
クラス AoSoA;
```

#### テンプレートパラメータ
`データ型`
: 基盤となる `Cabana::SoA` に格納される要素の型。

`メモリ空間`
: ストレージをどこに割り当てるかについての情報を保持する　Kokkos のメモリ空間。

`ベクトル長`
: 配列構造のベクトル長（任意）。 指定がない場合、デフォルト値（メモリ空間ごとに定義されたもの）が使用されます; この値は、最適なパフォーマンスを得るために変更する必要があるかもしれません。

`メモリ特性`
: Kokkos　のメモリ特性は、メモリの割り当てと割り当て解除を管理する主体を示します（オプション）。

#### 非メンバー関数
`スライス`
: 粒子データフィールドにアクセス。
