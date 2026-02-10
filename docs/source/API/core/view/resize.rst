``resize``
==========

.. role:: cpp(code)
   :language: cpp

ヘッダーファイル: <Kokkos_Core.hpp>

使用例
-----

.. code-block:: cpp

    resize(view,n0,n1,n2,n3);
    resize(view,layout);

新たな次元を持つために、ビューを再割り当てします｡ 拡大または縮小でき、共通サブテクストの内容は保持されません。


ディスクリプション
-----------

* .. code-block:: cpp

     template <class T, class... P>
     void resize(View<T, P...>& v,
		 const size_t n0 = KOKKOS_IMPL_CTOR_DEFAULT_ARG,
		 const size_t n1 = KOKKOS_IMPL_CTOR_DEFAULT_ARG,
		 const size_t n2 = KOKKOS_IMPL_CTOR_DEFAULT_ARG,
		 const size_t n3 = KOKKOS_IMPL_CTOR_DEFAULT_ARG,
		 const size_t n4 = KOKKOS_IMPL_CTOR_DEFAULT_ARG,
		 const size_t n5 = KOKKOS_IMPL_CTOR_DEFAULT_ARG,
		 const size_t n6 = KOKKOS_IMPL_CTOR_DEFAULT_ARG,
		 const size_t n7 = KOKKOS_IMPL_CTOR_DEFAULT_ARG);

  古いビューおよび新たなビューの共通サブビューの内容を保持しながら、 ``v`` を、新しい次元にサイズ変更します。

  * ``v``: 既存のビューであり、デフォルトコンストラクタで生成されたものになる可能性があります。
  * ``n[X]``: 範囲　X　の新たな長さ。

  制約:

  * ``View<T, P...>::array_layout`` は、  ``LayoutLeft`` または　``LayoutRight``　です。

* .. code-block:: cpp

      テンプレート <class I, class T, class... P>
      void resize(const I& arg_prop, Kokkos::View<T, P...>& v,
		  const size_t n0 = KOKKOS_IMPL_CTOR_DEFAULT_ARG,
		  const size_t n1 = KOKKOS_IMPL_CTOR_DEFAULT_ARG,
		  const size_t n2 = KOKKOS_IMPL_CTOR_DEFAULT_ARG,
		  const size_t n3 = KOKKOS_IMPL_CTOR_DEFAULT_ARG,
		  const size_t n4 = KOKKOS_IMPL_CTOR_DEFAULT_ARG,
		  const size_t n5 = KOKKOS_IMPL_CTOR_DEFAULT_ARG,
		  const size_t n6 = KOKKOS_IMPL_CTOR_DEFAULT_ARG,
		  const size_t n7 = KOKKOS_IMPL_CTOR_DEFAULT_ARG);

  古いビューおよび新たなビューの共通サブビューの内容を保持しながら、 ``v`` を、新しい次元にサイズ変更します。
　新たな ``Kokkos::View`` は、 ビューコンストラクタ特性 ``arg_prop``、
  例えば、 Kokkos::WithoutInitializing　を使って、構築されます。

  * ``v``: 既存のビューであり、デフォルトコンストラクタで生成されたものになる可能性があります。

  * ``n[X]``: 範囲　X　の新たな長さ。

  * ``arg_prop``: ビューコンストラクタ特性、例えば、 ``Kokkos::WithoutInitializing``。

  制約:

  * ``View<T, P...>::array_layout`` は、 ``LayoutLeft` または `LayoutRight``。

* .. code-block:: cpp

     テンプレート <class T, class... P, class ALLOC_PROP>
     void resize(const ALLOC_PROP& arg_prop,
		 Kokkos::View<T, P...>& v,
		 const size_t n0 = KOKKOS_IMPL_CTOR_DEFAULT_ARG,
		 const size_t n1 = KOKKOS_IMPL_CTOR_DEFAULT_ARG,
		 const size_t n2 = KOKKOS_IMPL_CTOR_DEFAULT_ARG,
		 const size_t n3 = KOKKOS_IMPL_CTOR_DEFAULT_ARG,
		 const size_t n4 = KOKKOS_IMPL_CTOR_DEFAULT_ARG,
		 const size_t n5 = KOKKOS_IMPL_CTOR_DEFAULT_ARG,
		 const size_t n6 = KOKKOS_IMPL_CTOR_DEFAULT_ARG,
		 const size_t n7 = KOKKOS_IMPL_CTOR_DEFAULT_ARG);

  古いビューおよび新たなビューの共通サブビューの内容を保持しながら、 ``v`` を、新しい次元にサイズ変更します。
  新たな ``Kokkos::View`` は、ビューコンストラクタ特性 ``arg_prop``、
  例えば、``Kokkos::view_alloc(Kokkos::WithoutInitializing)``　を使って、構築されます。
  ``arg_prop`` が、実行空間を含む場合には、 最終フェンスを使用せずに、 メモリ配分およびコピー要素のために使用されます。

  * ``v``: 既存のビューであり、デフォルトコンストラクタで生成されたものになる可能性があります。
  * ``n[X]``: 範囲　X　の新たな長さ。
  * ``arg_prop``: ビューコンストラクタ特性、例えば、  ``Kokkos::view_alloc(Kokkos::WithoutInitializing)``。

  制約:

  * ``View<T, P...>::array_layout`` は、 ``LayoutLeft`` または　``LayoutRight``　です。
  * ``arg_prop`` は、メモリ、ラベルまたはメモリ空間へのポインタを含んではなりません。

* .. code-block:: cpp

     テンプレート <class T, class... P>
     void resize(Kokkos::View<T, P...>& v,
                 const typename Kokkos::View<T, P...>::array_layout& layout);

  古いビューおよび新たなビューの共通サブビューの内容を保持しながら、 ``v`` を、新しい次元にサイズ変更します。

  * ``v``: 既存のビューであり、デフォルトコンストラクタで生成されたものになる可能性があります。
  * ``layout``: 新たな次元を含むレイアウトインスタンス。

* .. code-block:: cpp

     テンプレート　<class T, class... P>
     void resize(const I& arg_prop, Kokkos::View<T, P...>& v,
	         const typename Kokkos::View<T, P...>::array_layout& layout);

  古いビューおよび新たなビューの共通サブビューの内容を保持しながら、 ``v`` を、新しい次元にサイズ変更します。
　新たな ``Kokkos::View`` は、ビューコンストラクタ特性 ``arg_prop``、
  例えば、Kokkos::WithoutInitializing　を使って、構築されます。


  * ``v``: 既存のビューであり、デフォルトコンストラクタで生成されたものになる可能性があります。
  * ``layout``: 新たな次元を含むレイアウトインスタンス。
  * ``arg_prop``: ビューコンストラクタ特性、例えば、 ``Kokkos::WithoutInitializing``。

* .. code-block:: cpp

     テンプレート <class T, class... P, class ALLOC_PROP>
     void resize(const ALLOC_PROP& arg_prop,
	         Kokkos::View<T, P...>& v,
	         const typename Kokkos::View<T, P...>::array_layout& layout);

  古いビューおよび新たなビューの共通サブビューの内容を保持しながら、 ``v`` を、新しい次元にサイズ変更します。
  新たな ``Kokkos::View`` は、ビューコンストラクタ特性 ``arg_prop``、
  例えば、　``Kokkos::view_alloc(Kokkos::WithoutInitializing)``を使って、構築されます。
  ``arg_prop`` が、実行空間を含む場合には、 最終フェンスを使用せずに、 メモリ配分およびコピー要素のために使用されます。

  * ``v``: 既存のビューであり、デフォルトコンストラクタで生成されたものになる可能性があります。
  * ``layout``: 新たな次元を含むレイアウトインスタンス。
  * ``arg_prop``: ビューコンストラクタ特性、例えば、  ``Kokkos::view_alloc(Kokkos::WithoutInitializing)``。

  制約:

  * ``arg_prop`` は、メモリ、ラベルまたはメモリ空間へのポインタを含んではなりません。

例:
--------

* .. code-block:: cpp

    Kokkos::resize(v, 2, 3);

Resize a ``Kokkos::View`` with dynamic rank 2 to have dynamic extent 2 and 3 respectively preserving previous content.

* .. code-block:: cpp

    Kokkos::resize(Kokkos::WithoutInitializing, v, 2, 3);

Resize a ``Kokkos::View`` with dynamic rank 2 to have dynamic extent 2 and 3 respectively preserving previous content. After this call, the new content is uninitialized.
