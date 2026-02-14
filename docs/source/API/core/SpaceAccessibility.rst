空間アクセシビリティ
===================

.. role::cpp(code)
    :language: cpp

.. _ExecutionSpace: execution_spaces.html#executionspaceconcept

.. |ExecutionSpace| replace:: ``ExecutionSpace``

.. _MemorySpace: memory_spaces.html#memoryspaceconcept

.. |MemorySpace| replace:: ``MemorySpace``

``Kokkos::SpaceAccessibility<>`` は、最初のテンプレート引数として |ExecutionSpace|_ 型または |MemorySpace|_ 型を、2番目の型として |MemorySpace|_ 型を取る特性クラステンプレートで、それらのエンティティ間の関係について詳細を表現しています。メモリ空間型　``MSp1`` および ``MSp2`` 、および実行空間型　``Ex``　を前提とすれば、以下の式は指定された意味において有効となります:

------------

.. code-block:: cpp
    
    Kokkos::SpaceAccessibility<Ex, MSp1>::accessible

``MSp1``　のインスタンスによって割り当てられたメモリに、　``Ex``のインスタンスによって提供される実行スレッドからアクセスが、*すべての* ``Ex`` インスタンスおよび　*すべての* ``MSp1`` インスタンスにおいて、Kokkosプログラミングモデル内で明確に定義され有効である場合に限り（例えば、``Ex``　のインスタンス上で起動された並列パターン内で）、コンパイル時に、`bool``　に変換可能で、必ず　``true``　となる値。

------------

.. code-block:: cpp
    
    Kokkos::SpaceAccessibility<MSp1, MSp2>::accessible

 ``Kokkos::SpaceAccessibility<MSp1::execution_space, MSp2>::accessible``　と同等。

------------

.. code-block:: cpp
    
    Kokkos::SpaceAccessibility<MSp1, MSp2>::割り当て可能

.. _KokkosView: view/view.html

.. |KokkosView| replace:: ``Kokkos::View``

``V2``　型　のいずれか　（そうでない場合には、有効な）インスタンスから ( ``true``　に等しい``std::is_same<V2::memory_space, MSp2>::value``　を持つ) ``View`` ``V1``　型 ( ``true``　に等しい　``std::is_same<V1::memory_space, MSp1>::value`` を持つ)　のいずれか（そうでない場合には、有効な）インスタンスから取得する参照データに割り当てることが、 Kokkos プログラミングモデル内で有効である場合に限り、コンパイル時に、`bool``　に変換可能で、必ず　``true``　となる値。  

------------

.. code-block:: cpp
    
    Kokkos::SpaceAccessibility<Ex, MSp1>::assignable

``Kokkos::SpaceAccessibility<Ex::memory_space, MSp1>::assignable``　と同等。

------------

.. code-block:: cpp
    
    Kokkos::SpaceAccessibility<MSp1, MSp2>::deepcopy

.. _KokkosDeepCopy: view/deep_copy.html

.. |KokkosDeepCopy| replace:: ``Kokkos::deep_copy``

``V2``　型　のいずれか　（そうでない場合には、有効な）インスタンスから ( ``true``　に等しい``std::is_same<V2::memory_space, MSp2>::value``　を持つ) ``View`` ``V1``　型 ( ``true``　に等しい　``std::is_same<V1::memory_space, MSp1>::value`` を持つ)　のいずれか（そうでない場合には、有効な）インスタンスへ、Kokkosプログラミングモデル内で、Kokkos::deep_copy　することが有効である場合に限り、コンパイル時に、`bool``　に変換可能で、必ず　``true``　となる値。換言すれば、 ``v2`` が、``V2``　の有効なインスタンスであり、 ``v1`` が ``V1`` の有効なインスタンス(形および他の属性を持ち、そうでない場合には、 ``v2``と互換性がある)　である場合、以下の式が  Kokkos プログラミングモデル内で明確に定義され、有効となります:

.. code-block:: cpp
    
    Kokkos::deep_copy(v1, v2);

------------

.. code-block:: cpp
    
    Kokkos::SpaceAccessibility<Ex, MSp1>::deepcopy

``Kokkos::SpaceAccessibility<Ex::memory_space, MSp1>::deepcopy``　と同等。

------------

さらに、以下のネスト型名が、定義されます:

.. code-block:: cpp
    
    Kokkos::SpaceAccessibility<Ex, MSp1>::space

``Ex``　のあらゆるインスタンスがアクセスするためにメモリをディープコピーするために使用されるべき "仲介" メモリ空間。 正式には、コンパイル時に以下の式がすべて ``true``　となる ``Kokkos::Device``　の要件を満たす型:

* ``Kokkos::SpaceAccessibility<Ex, Kokkos::SpaceAccessibility<Ex, MSp1>::space::memory_space>::accessible``
* ``Kokkos::SpaceAccessibility<Kokkos::SpaceAccessibility<Ex, MSp1>::space::memory_space, MSp1>::deepcopy``
* ``Kokkos::SpaceAccessibility<Ex, Kokkos::SpaceAccessibility<Ex, MSp1>::space::memory_space>::deepcopy``

------------

.. code-block:: cpp
    
    Kokkos::SpaceAccessibility<MSp1, MSp2>::space

 ``Kokkos::SpaceAccessibility<MSp1::execution_space, MSp2>::space``　と同等。
