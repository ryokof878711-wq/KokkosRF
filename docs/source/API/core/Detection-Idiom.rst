検出イディオム
===============

.. role:: cpp(code)
    :language: cpp

検出イディオムは、SFINAE　に配慮した方法で、あらゆる　C++　式が有効かどうかを認識するために使用されます。

ヘッダーファイル: ``<Kokkos_DetectionIdiom.hpp>``

Kokkos 検出イディオムは、ISO/IEC TS 19568:2017、ライブラリ基礎のためのC++拡張のバージョン2の検出イディオムに基づいており、
そのドラフトは、`here <https://cplusplus.github.io/fundamentals-ts/v2.html#meta.detect>`　に見られます。

元の C++ プロポーザルは、 `here <https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2015/n4436.pdf>`　に見られます。

API
---

.. code-block:: cpp

    // VOID_T および DETECTOR は説明用であり、直接使用を目的としていません。

    // SFINAE　を効果的に活用する便利なメタ関数
    template<class...>
    VOID_T = void;　を使用

    // 典型的な Op<Args...> をサポートしない型のためのプライマリテンプレート
    template<class Default, class /* AlwaysVoid */, template<class...> class /* Op */, class... /* Args */>
    構造体 DETECTOR {
        value_t = std::false_type;　を使用
        type    = Default;　を使用
    };

    // Specialization for types supporting the archetypal Op<Args...>
    template<class Default, template<class...> class Op, class... Args>
    構造体 DETECTOR<Default, VOID_T<Op<Args...>>, Op, Args...> {
        value_t = std::true_type;　を使用
        type    = Op<Args...>;　を使用
    };

.. code-block:: cpp

    名前空間 Kokkos {

    // 提供されたアーキタイプをサポートしない型について、detected_t が返す型の簡略化
    構造体 nonesuch {
        nonesuch(nonesuch&&) = delete;
        ~nonesuch() = delete;
    };

    // is_detected は、Op<Args...> が有効な型である場合に std::true_type の別名となります。
    // そうでない場合、std::false_type の別名です。

    テンプレート <template <class...> class Op, class... Args>
    is_detected =
        typename DETECTOR<nonesuch, void, Op, Args...>::value_t;　を使用

    // detected_t は、Op<Args...> が有効な型である場合に Op<Args...> の別名となります。
    //  そうでない場合、 Kokkos::nonesuch　の別名です。

    テンプレート <template <class...> class Op, class... Args>
    detected_t = typename DETECTOR<nonesuch, void, Op, Args...>::type;　を使用

    // detected_or_t は、Op<Args...> が有効な型である場合に  Op<Args...> の別名となります。
    //  そうでない場合、 Default の別名です。

    テンプレート <class Default, template <class...> class Op, class... Args>
    using detected_or_t = typename DETECTOR<Default, void, Op, Args...>::type;　を使用

    // is_detected_exact は、Op<Args...> が、 Expected と同じ型である場合に std::true_type の別名となります。
    //  そうでない場合、std::false_type　の別名です。

    テンプレート <class Expected, template <class...> class Op, class... Args>
    is_detected_exact = std::is_same<Expected, detected_t<Op, Args...>>;　を使用

    // is_detected_convertible は、Op<Args...> が To へ変換可能な場合に std::true_type の別名となります。
    //  そうでない場合、std::false_type　の別名です。

    テンプレート <class To, template <class...> class Op, class... Args>
    is_detected_convertible =
        std::is_convertible<detected_t<Op, Args...>, To>;　を使用

    // C++17　またはそれ以降の便利変数

    テンプレート <template <class...> class Op, class... Args>
    inline constexpr bool is_detected_v = is_detected<Op, Args...>::value;

    テンプレート <class Expected, template <class...> class Op, class... Args>
    inline constexpr bool is_detected_exact_v =
        is_detected_exact<Expected, Op, Args...>::value;

    テンプレート <class Expected, template <class...> class Op, class... Args>
    inline constexpr bool is_detected_convertible_v =

    } // Kokkos namespace

例
--------

式を検出
~~~~~~~~~~~~~~~~~~~~~~~

ある型 ``T`` がコピー代入可能かどうかを検出する型特性を記述する必要があると仮定します。 まず、アーキタイプヘルパーエイリアスを記述します:

.. code-block:: cpp

    template<class T>
    copy_assign_t = decltype(std::declval<T&>() = std::declval<T const&>());　を使用

その後、その特性は簡単に次のように表現できます:

.. code-block:: cpp

    template<class T>
    is_copy_assignable = Kokkos::is_detected<copy_assign_t, T>;　を使用

コピー代入の戻り値の型が ``T&`` であることを確認したい場合、以下を使用します:

.. code-block:: cpp

    template<class T>
    is_canonical_copy_assignable = Kokkos::is_detected_exact<T&, copy_assign_t, T>;　を使用

ネストされたtypedefを検出
~~~~~~~~~~~~~~~~~~~~~~~~~~

ネストされた``MyType::difference_type``　が存在する場合には、それを使用したいと仮定し、そうでない場合には、``std::ptrdiff_t``　の使用を望みます:

まず、アーキタイプヘルパーエイリアスを記述します:

.. code-block:: cpp

    template<class T>
    diff_t = typename T::difference_type;　を使用

その後、型を宣言することができます:

.. code-block:: cpp

    our_difference_type = Kokkos::detected_or_t<std::ptrdiff_t, diff_t, MyType>;　を使用
