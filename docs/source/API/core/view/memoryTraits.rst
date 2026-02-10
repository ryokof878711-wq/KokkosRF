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

  A boolean that indicates whether the RandomAccess trait is enabled.

.. cpp:member::  static constexpr bool is_atomic

  A boolean that indicates whether the Atomic trait is enabled.

.. cpp:member::  static constexpr bool is_restrict

  A boolean that indicates whether the Restrict trait is enabled.

.. cpp:member::  static constexpr bool is_aligned

  A boolean that indicates whether the Aligned trait is enabled.

.. _MemoryAccessTraits: ../../../ProgrammingGuide/View.html#memory-access-traits

.. |MemoryAccessTraits| replace:: memory access traits

.. _UnmanagedViews: ../../../ProgrammingGuide/View.html#unmanaged-views

.. |UnmanagedViews| replace:: unmanaged views

Non-Member Enums
^^^^^^^^^^^^^^^^

The following enumeration values are used to specify the memory access traits. Check the sub-section on |MemoryAccessTraits|_ in the Programming Guide for further information about how these traits can be used in practice.

.. cpp:enum:: MemoryTraitsFlags

  The following enumeration values are defined in this enumeration type.

.. cpp:enumerator:: Unmanaged

  This traits means that Kokkos does neither reference counting nor automatic deallocation for such Views. This trait can be associated with memory allocated in any memory space. For example, an *unmanaged view* can be created by wrapping raw pointers of allocated memory, while also specifying the execution or memory space accordingly.

.. cpp:enumerator:: RandomAccess

  Views that are going to be accessed irregularly (e.g., non-sequentially) can be declared as random access. 

.. cpp:enumerator:: Atomic

  In such a view, every access (read or write) to any element will be atomic. 

.. cpp:enumerator:: Restrict

  This trait indicates that the memory of this view doesn't alias/overlap with another data structure in the current scope. 

.. cpp:enumerator:: Aligned

  This trait provides additional information to the compiler that the memory allocation in this ``View`` has an alignment of 64. 

Non-Member Type aliases
^^^^^^^^^^^^^^^^^^^^^^^

The following type aliases are also available in the ``Kokkos`` namespace.

.. cpp:type:: MemoryManaged = Kokkos::MemoryTraits<>;
.. cpp:type:: MemoryUnmanaged = Kokkos::MemoryTraits<Kokkos::Unmanaged>;
.. cpp:type:: MemoryRandomAccess = Kokkos::MemoryTraits<Kokkos::Unmanaged | Kokkos::RandomAccess>;

.. deprecated:: 4.7
  Managed memory as an explicit memory trait (i.e., ``using MemoryManaged = Kokkos::MemoryTraits<>;``) has been deprecated in Kokkos 4.7. Also, in earlier versions of Kokkos, the enumeration value of ``0`` had to be explicitly mentioned, i.e., ``Kokkos::MemoryTraits<0>``. Check the sub-section on |UnmanagedViews|_ for a discussion about this.

Note that in order to use a managed View in a random access manner, the memory trait should be specified as ``Kokkos::MemoryTraits<Kokkos::RandomAccess>`` and not ``Kokkos::MemoryRandomAccess``.

Examples
^^^^^^^^

.. code-block:: cpp

   Kokkos::View<DayaType, LayoutType, MemorySpace, Kokkos::MemoryTraits<SomeFlag | SomeOtherFlag> > my_view;

Example MemoryTraits type: ``Kokkos::MemoryTraits<Kokkos::Unmanaged | Kokkos::RandomAccess>``

