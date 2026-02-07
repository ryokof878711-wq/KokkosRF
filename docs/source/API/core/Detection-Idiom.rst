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

    // is_detected is an alias for std::true_type if Op<Args...> is a valid type
    // otherwise, an alias for std::false_type

    template <template <class...> class Op, class... Args>
    is_detected =
        typename DETECTOR<nonesuch, void, Op, Args...>::value_t;　を使用

    // detected_t is an alias for Op<Args...> if Op<Args...> is a valid type
    //  otherwise, an alias for Kokkos::nonesuch

    template <template <class...> class Op, class... Args>
    using detected_t = typename DETECTOR<nonesuch, void, Op, Args...>::type;

    // detected_or_t is an alias for Op<Args...> if Op<Args...> is a valid type
    //  otherwise, an alias for Default

    template <class Default, template <class...> class Op, class... Args>
    using detected_or_t = typename DETECTOR<Default, void, Op, Args...>::type;

    // is_detected_exact is an alias for std::true_type if Op<Args...> is the same type as Expected
    //  otherwise, an alias for std::false_type

    template <class Expected, template <class...> class Op, class... Args>
    using is_detected_exact = std::is_same<Expected, detected_t<Op, Args...>>;

    // is_detected_convertible is an alias for std::true_type if Op<Args...> is convertible to To
    //  otherwise, an alias for std::false_type

    template <class To, template <class...> class Op, class... Args>
    using is_detected_convertible =
        std::is_convertible<detected_t<Op, Args...>, To>;

    // C++17 or later convenience variables

    template <template <class...> class Op, class... Args>
    inline constexpr bool is_detected_v = is_detected<Op, Args...>::value;

    template <class Expected, template <class...> class Op, class... Args>
    inline constexpr bool is_detected_exact_v =
        is_detected_exact<Expected, Op, Args...>::value;

    template <class Expected, template <class...> class Op, class... Args>
    inline constexpr bool is_detected_convertible_v =

    } // Kokkos namespace

Examples
--------

Detecting an expression
~~~~~~~~~~~~~~~~~~~~~~~

Suppose we needed to write a type trait to detect if a given type ``T`` is copy assignable. First we write an archetype helper alias:

.. code-block:: cpp

    template<class T>
    using copy_assign_t = decltype(std::declval<T&>() = std::declval<T const&>());

Then the trait can be easily expressed as:

.. code-block:: cpp

    template<class T>
    using is_copy_assignable = Kokkos::is_detected<copy_assign_t, T>;

If we also wanted to check that the return type of the copy assignment is ``T&``, we would use:

.. code-block:: cpp

    template<class T>
    using is_canonical_copy_assignable = Kokkos::is_detected_exact<T&, copy_assign_t, T>;

Detecting a nested typedef
~~~~~~~~~~~~~~~~~~~~~~~~~~

Suppose we want to use a nested ``MyType::difference_type`` if it exists, otherwise, we want to use ``std::ptrdiff_t``:

First we write an archetype helper alias:

.. code-block:: cpp

    template<class T>
    using diff_t = typename T::difference_type;

Then we can declare our type:

.. code-block:: cpp

    using our_difference_type = Kokkos::detected_or_t<std::ptrdiff_t, diff_t, MyType>;
