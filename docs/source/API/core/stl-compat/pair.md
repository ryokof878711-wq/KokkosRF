# `Kokkos::pair`

ヘッダーファイル: `Kokkos_Pair.hpp`

 `Kokkos::pair` がデバイス上で機能する場合を除き、完全に互換性を持つことを目的とした `std::pair` の実装。また、`std::pair` からの変換、および `std::pair` への変換のための、ユーティリティ関数も提供します。

使用例: 
```c++
std::pair<int, float> std_pair = std::make_pair(1,2.0f); 
Kokkos::pair<int_float> kokkos_pair = Kokkos::make_pair(1,2.0f);
Kokkos::pair<int, float> converted_std_pair(std_pair);
std::pair<int,float> converted_kokkos_pair = kokkos_pair.to_std_pair();
```

## シノプシス 
```c++
template <class T1, class T2>
構造体ペア {

    typedef T1 first_type;
    typedef T2 second_type;

    first_type first;
    second_type second;
  
    KOKKOS_DEFAULTED_FUNCTION constexpr pair() = default;
    KOKKOS_FORCEINLINE_FUNCTION constexpr pair(first_type const& f,
                                               second_type const& s);
  
    template <class U, class V>
    KOKKOS_FORCEINLINE_FUNCTION constexpr pair(const pair<U, V>& p);

    template <class U, class V>
    KOKKOS_FORCEINLINE_FUNCTION pair<T1, T2>& operator=(const pair<U, V>& p);
  
    template <class U, class V>
    pair(const std::pair<U, V>& p);
  
    std::pair<T1, T2> to_std_pair() const;
};
```

### パブリックメンバー関数

  * `first`: ペア内の1番目の要素
  * `second`: ペア内の2番目の要素

### Typedefs
   
  * `first_type`: ペア内の1番目の型
  * `second_type`: ペア内の2番目の型

### コンストラクタ

  * 
  ```c++ 
  KOKKOS_DEFAULTED_FUNCTION constexpr pair() = default;
  ```

  デフォルトコンストラクタ。 そのデフォルトを使って、データメンバーを初期化します。

  * 
  ```c++
  KOKKOS_FORCEINLINE_FUNCTION constexpr pair(first_type const& f,
                                  second_type const& s);
  ```

  要素ごとのコンストラクタ。 `first` に `f`　の値、 `second` にthe value of `s` の値を割り当てます。

  * 
  ```c++
  template <class U, class V>
  KOKKOS_FORCEINLINE_FUNCTION constexpr pair(const pair<U, V>& p);
  ``` 
      
   `std::pair`　から変換。ペアの各要素を、  `p`　内の対応する要素に、割り当てます。

### 割り当ておよび変換

  * 
  ```c++
  template <class U, class V>
  KOKKOS_FORCEINLINE_FUNCTION pair<T1, T2>& operator=(const pair<U, V>& p);
  ```

   `first` を `p.first` に、and `second` を `p.second` に設定します。
  ### 関数

  * 
  ```c++
  std::pair<T1, T2> to_std_pair() const;
  ```

  コンテンツが、  `Kokkos::pair`　のコンテンツに一致する  `std::pair` を返します。明示的に `std::pair` のみを受け入れるライブラリの処理に有用です。
