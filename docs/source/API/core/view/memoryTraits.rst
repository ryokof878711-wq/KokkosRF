``MemoryTraits``
================

:cpp:struct:`MemoryTraits` は、is the last template parameter of :cpp:class:`View`　の最後のテンプレートパラメータです。

構造体インターフェイス
----------------

.. cpp:struct:: テンプレート <unsigned N> MemoryTraits

  多次元ビューに提供された場合、 ``MemoryTraits`` は、割り当て処理に関する追加情報を渡すことを許可します。 テンプレート引数は、下記の列挙型値のビット単位の論理和であることが想定されます。

.. rubric:: ネストされた型

.. cpp:type::  memory_traits

  ``N``で示されるメモリアクセス特性（複数可）を表すタグタイプ。

.. rubric:: メンバー変数

.. cpp:member:: static constexpr bool is_unmanaged

  アンマネージドトレイトが有効かどうかを示すブール値

.. cpp:member::  static constexpr bool is_random_access

  RandomAccessトレイトが有効かどうかを示すブール値。

.. cpp:member::  static constexpr bool is_atomic

  Atomicトレイトが有効かどうかを示すブール値。

.. cpp:member::  static constexpr bool is_restrict

  Restrictトレイトが有効かどうかを示すブール値。

.. cpp:member::  static constexpr bool is_aligned

  Alignedトレイトが有効かどうかを示すブール値。

.. _MemoryAccessTraits: ../../../ProgrammingGuide/View.html#memory-access-traits

.. |MemoryAccessTraits| replace:: メモリアクセストレイト

.. _UnmanagedViews: ../../../ProgrammingGuide/View.html#unmanaged-views

.. |UnmanagedViews| replace:: 管理対象外ビュー

非メンバー列挙型
^^^^^^^^^^^^^^^^

以下の列挙値は、メモリアクセス特性指定に使用されます。これらの特性が実際にどのように使用できるかについての詳細については、プログラミングガイドの　|MemoryAccessTraits|_　のサブセクションを確認してください。

.. cpp:enum:: MemoryTraitsFlags

  この列挙型では、以下の列挙値が定義されています。

.. cpp:enumerator:: 管理対象外

  この特性は、　Kokkos　がこのような　View　に対して参照カウントも自動解放も行わないことを意味します。 この特性は、任意のメモリ空間に割り当てられたメモリに関連付けることができます。例えば、 *非管理ビュー*　は、割り当てられたメモリの生ポインタをラップすることで作成でき、同時に実行領域またはメモリ空間を適切に指定することもできます。

.. cpp:enumerator:: RandomAccess

   不規則にアクセスされるビュー（例：非順次アクセス）は、ランダムアクセスとして宣言できます。

.. cpp:enumerator:: Atomic

 　このようなビューでは、あらゆる要素へのアクセス（読み取りまたは書き込み）はすべてアトミックである。

.. cpp:enumerator:: Restrict

   この特性は、このビューのメモリが現在のスコープ内の他のデータ構造の別名ではない/重複していないことを示します

.. cpp:enumerator:: Aligned

  この特性は、この　``View``　におけるメモリ割り当てが、64バイト単位でアラインメントされていることをコンパイラに追加で通知します。

非メンバー型エイリアス
^^^^^^^^^^^^^^^^^^^^^^^

以下の型エイリアスも、　``Kokkos``　名前空間で利用可能です。

.. cpp:type:: MemoryManaged = Kokkos::MemoryTraits<>;
.. cpp:type:: MemoryUnmanaged = Kokkos::MemoryTraits<Kokkos::Unmanaged>;
.. cpp:type:: MemoryRandomAccess = Kokkos::MemoryTraits<Kokkos::Unmanaged | Kokkos::RandomAccess>;

.. deprecated:: 4.7
  明示的なメモリ特性としてのマネージドメモリ（例：　``using MemoryManaged = Kokkos::MemoryTraits<>;``　）は、　Kokkos 4.7 で非推奨となりました。 また、以前のバージョンのKokkosでも、列挙値の　``0``　を明示的に指定する必要がありました。つまり、``Kokkos::MemoryTraits<0>``　のように記述する必要があったのです。 これに関する詳細は、　|UnmanagedViews|_　の項目を参照してください。

管理下にあるビューをランダムアクセス方式で使用するには、メモリ特性として、 ``Kokkos::MemoryTraits<Kokkos::RandomAccess>`` を指定する必要があり、　``Kokkos::MemoryRandomAccess`` としては指定しないことに注意してください。

例
^^^^^^^^

.. code-block:: cpp

   Kokkos::View<DayaType, LayoutType, MemorySpace, Kokkos::MemoryTraits<SomeFlag | SomeOtherFlag> > my_view;

 MemoryTraits 型の例: ``Kokkos::MemoryTraits<Kokkos::Unmanaged | Kokkos::RandomAccess>``

