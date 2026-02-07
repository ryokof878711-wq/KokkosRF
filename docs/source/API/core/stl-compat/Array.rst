``Array``
==============

.. role:: cpp(code)
    :language: cpp

..
 ユーザーがそのコードを含む (パブリックヘッダー) ファイル 

``<Kokkos_Core.hpp>``　から含まれる、ヘッダー ``<Kokkos_Array.hpp>`` に定義。

..
  物事が行うこと、そして可能であれば、それがなぜ存在するかについての短い説明についての、高レベルの、人間の言語の概要 (2 - 3 文、最大);

ディスクリプション
-----------

``Array`` は、連続した集合を所有するコンテナで、固定サイズのオブジェクト列（正確にN個の要素を持つモデル）を格納します。

* これは、``std::array<T, N>``　の代替として設計されています。
* このコンテナは所有コンテナです（データはコンテナ自体に埋め込まれています）。
* このコンテナは、この場合 ``N > 0`` であるため、Cスタイル配列 ``T[N]`` を唯一の非静的データメンバーとして保持する構造体と同一の意味論を持つ集合型です；そうでない場合には、それは空のコンテナです。
* Cスタイルの配列とは異なり、自動的に　``T*``　に型変換されることはありません。
* 集合型として、それは、 ``T``: ``Kokkos::Array<int, 3> a = { 1, 2, 3 };``　に変換可能な、最大で　``N``　個の初期化子による集合初期化で初期化できます。

..
  エンティティのAPI。

インターフェイス
---------

.. versionchanged:: 4.4.0

.. cpp:struct:: template <class T, size_t N> Array

  ..
    テンプレートパラメータ (該当する場合)
    特殊化専用に使用される／演鐸される／および／またはユーザーに公開すべきでないテンプレートパラメータは省略します。

  .. rubric:: テンプレートパラメータ

  :tparam T: 格納されているエレメント型。
  :tparam N: 格納されているエレメント数。

  .. rubric:: パブリックタイプ

  .. cpp:type:: value_type = T
  .. cpp:type:: pointer = T*
  .. cpp:type:: const_pointer = const T*
  .. cpp:type:: reference = T&
  .. cpp:type:: const_reference = const T&
  .. cpp:type:: size_type = size_t
  .. cpp:type:: difference_type = ptrdiff_t

  .. rubric:: パブリックメンバー関数

  .. cpp:function:: 静的定数コンストラクタ付きブール値　empty()　noexcept　

    :return: ``N == 0``
    :since: 5.0　より　``noexcept`` 

  .. cpp:function:: 静的　constexpr size_type size() noexcept
  .. cpp:function:: constexpr size_type max_size() const noexcept

    :return: ``N``
    :since: 5.0より ``noexcept``

  .. cpp:function:: constexpr　参照 operator[](size_t i)
  .. cpp:function::  const_reference operator[](size_t i) const

    :return: 配列の　``i``　番目のエレメントへの参照。
    :since: 引数が整数型またはスコープ外列挙型である必要がなくなりました (5.1より)。

  .. cpp:function:: constexpr ポインタ data() noexcept
  .. cpp:function:: constexpr const_pointer data() const noexcept

    :return: 配列の1番目の要素へのポインタ 　``N == 0``　である場合、 戻り値が特定されず、間接参照できません。
    :since:  5.0より、``noexcept``

  .. cpp:function:: constexpr ポインタ begin() noexcept
  .. cpp:function:: constexpr const_pointer begin() const noexcept
  .. cpp:function:: constexpr const_pointer cbegin() const  noexcept

    :return: ``data()``
    :since: since 5.0

  .. cpp:function:: constexpr ポインタ end() noexcept
  .. cpp:function:: constexpr const_pointer end() const noexcept
  .. cpp:function:: constexpr const_pointer cend() const noexcept

    :return: ``data() + size()``. 戻り値は、間接参照できません。 ``N == 0``　である場合、 戻り値は、 ``begin()``　に等しくなります。
    :since: 5.0より


演鐸ガイド
----------------

.. cpp:function:: template<class T, class... U> Array(T, U...) -> Array<T, 1 + sizeof...(U)>

非メンバー関数
--------------------

..
  これらは密接に関連している場合にのみ、ここに記載されるべきです。例えば、 フレンド演算子。しかしながら、
  view_alloc　のようなものは、 ビューのためにはここに存在すべきではありません。

.. cpp:function:: template<class T, size_t N> constexpr bool operator==(const Array<T, N>& l, const Array<T, N>& r) noexcept

   :return: ``l`` および　``r`` の要素がすべて等しい場合に限り、　``true``  

.. cpp:function:: template<class T, size_t N> constexpr bool operator!=(const Array<T, N>& l, const Array<T, N>& r) noexcept

   :return: ``!(l == r)``

.. cpp:function:: template<class T, size_t N> constexpr kokkos_swap(Array<T, N>& l, Array<T, N>& r) noexcept(N == 0 || is_nothrow_swappable_V<T>)

   :return: If ``T``　が交換可能、または  ``N == 0``　の場合、  `l` および `r` の中のそれぞれの要素は、 ``kokkos_swap``　を通じて交換されます。

.. cpp:function:: template<class T, size_t N> constexpr Array<remove_cv_t<T>, N> to_array(T (&a)[N])
.. cpp:function:: template<class T, size_t N> constexpr Array<remove_cv_t<T>, N> to_array(T (&&a)[N])

   :return:  ``a``　からコピー/移動した要素を含む ``Array`` 

.. cpp:function:: template<size_t I, class T, size_t N> constexpr T& get(Array<T, N>& a) noexcept
.. cpp:function:: template<size_t I, class T, size_t N> constexpr const T& get(const Array<T, N>& a) noexcept

   :return: ``a[I]`` for (tuple protocol / structured binding support)

.. cpp:function:: template<size_t I, class T, size_t N> constexpr T&& get(Array<T, N>&& a) noexcept
.. cpp:function:: template<size_t I, class T, size_t N> constexpr const T&& get(const Array<T, N>&& a) noexcept

   :return: ``std::move(a[I])`` (タプルプロトコル / 構造化バインディングサポートについて)

.. cpp:function:: template<class T, size_t N> constexpr T* begin(Array<T, N>& a) noexcept
.. cpp:function:: template<class T, size_t N> constexpr const T* begin(const Array<T, N>& a) noexcept

   :return: ``a.data()``

.. cpp:function:: template<class T, size_t N> constexpr T* end(Array<T, N>& a) noexcept
.. cpp:function:: template<class T, size_t N> constexpr const T* end(const Array<T, N>& a) noexcept

   :return: ``a.data() + a.size()``

非推奨インターフェイス
--------------------
.. deprecated:: 4.4.00

.. cpp:struct:: template<class T = void, size_t N = KOKKOS_INVALID_INDEX, class Proxy = void> Array

* 一次テンプレートは、型 ``T``　の要素　``N`` 個からなる完全な連続集合を保持するコンテナでした。
* このコンテナは、移動セマンティクスをサポートしていませんでした。

.. cpp:struct:: template<class T, class Proxy> Array<T, 0, Proxy>

* このコンテナは、空のコンテナでした。

.. cpp:struct:: template<class T> Array<T, KOKKOS_INVALID_INDEX, Array<>::contiguous>

* このコンテナは、非所有コンテナです。
* このコンテナのサイズは、構築時に決定されました。
* このコンテナは、任意の　``Array<T, N , Proxy>``　から割り当てられる可能性があります。
* 割り当てにより、このコンテナーのサイズは変更されませんでした。
* このコンテナは、移動セマンティクスをサポートしていませんでした。

.. cpp:struct:: template<class T> Array<T, KOKKOS_INVALID_INDEX, Array<>::strided>

* このコンテナは、非所有コンテナです。
* このコンテナのサイズは、構築時に決定されました。
* このコンテナは、任意の　``Array<T, N , Proxy>``　から割り当てられる可能性があります。
* 割り当てにより、このコンテナーのサイズは変更されませんでした。
* このコンテナは、移動セマンティクスをサポートしていませんでした。

.. cpp:struct:: template<> Array<void, KOKKOS_INVALID_INDEX, void>

   .. rubric:: パブリックタイプ

   .. cpp:type:: 隣接
   .. cpp:type:: 一定間隔

* この仕様は、は、埋め込みタグの種類を定義した型: ``contiguous`` および　``strided``　。

例
________

.. code-block:: cpp

 #include "Kokkos_Core.hpp"
 #include <algorithm>
 #include <iostream>
 #include <iterator>
 #include <memory>
 #include <string>
 #include <string_view>
 #include <type_traits>
 #include <utility>

 // creates a constexpr array of string_view's
 constexpr auto w1n = Kokkos::to_array<std::string_view>(
     {"Mary", "Patricia", "Linda", "Barbara", "Elizabeth", "Jennifer"});
 static_assert(
     std::is_same_v<decltype(w1n), const Kokkos::Array<std::string_view, 6>>);
 static_assert(w1n.size() == 6 and w1n[5] == "Jennifer");

 extern int Main(int /* argc */, char const *const /* argv */[]);
 int Main(int /* argc */, char const *const /* argv */[]) {
   Kokkos::ScopeGuard _;

   // Construction uses aggregate initialization
   [[maybe_unused]] Kokkos::Array<int, 3> a1{
       {1, 2, 3}}; // C++11　に必要とされたダブルブレース
                   // および　C++14　その後にもまだ認められています。

   Kokkos::Array<int, 3> a2 = {1, 2, 3}; // Double braces never required after =

   // 出力は、 3 2 1
   std::reverse_copy(std::data(a2), end(a2),
                     std::ostream_iterator<int>(std::cout, " "));
   std::cout << '\n';

   // ループの範囲指定はサポートされています。
   // 出力は、 E Ǝ
   Kokkos::Array<std::string, 2> a3{"E", "\u018E"};
   for (const auto &s : a3)
     std::cout << s << ' ';
   std::cout << '\n';

   // 配列作成のための演鐸ガイド
   [[maybe_unused]] Kokkos::Array a4{3.0, 1.0, 4.0}; // Kokkos::Array<double, 3>

   // 特定されていない要素のビヘイビアは、組み込み配列と同様です。
   [[maybe_unused]] Kokkos::Array<int, 2> a5; // init　のリストはなく, a5[0] および a5[1]
                                              // デフォルト初期化されています。
   [[maybe_unused]] Kokkos::Array<int, 2>
       a6{}; // init　をリストし、両方の要素が値です。 
             // 初期化されており、 a6[0] = a6[1] = 0
   [[maybe_unused]] Kokkos::Array<int, 2> a7{
       1}; // init　をリストし、 特定されていない要素は値です。
           // initialized, a7[0] = 1, a7[1] = 0

   // copies a string literal
   auto t1 = Kokkos::to_array("foo");
   static_assert(t1.size() == 4);

   // 要素の型および長さ両方を演鐸します。
   auto t2 = Kokkos::to_array({0, 2, 1, 3});
   static_assert(std::is_same_v<decltype(t2), Kokkos::Array<int, 4>>);

   // 特定された要素の型を使って、長さを演鐸します。
   // 暗示的変換が起こります。
   auto t3 = Kokkos::to_array<long>({0, 1, 3});
   static_assert(std::is_same_v<decltype(t3), Kokkos::Array<long, 3>>);

   auto t4 = Kokkos::to_array<std::pair<int, float>>(
       {{3, 0.0f}, {4, 0.1f}, {4, 0.1e23f}});
   static_assert(t4.size() == 3);

   // コピー不可能な　Kokkos::Array　を作成します。
   auto t5 = Kokkos::to_array({std::make_unique<int>(3)});
   static_assert(t5.size() == 1);

   // エラー: 多次元配列のコピーは、サポート対象外です。
   // char s[2][6] = {"nice", "thing"};
   // auto t6 = Kokkos::to_array(s);

   return 0;
 }
