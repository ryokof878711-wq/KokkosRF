コードを、``Kokkos_ENABLE_CUDA_UVM`` を必要とする状態から　``SharedSpace` を使用する状態へ移行
==============================================================================

.. role:: cpp(code)
   :language: cpp

.. _SharedSpace: ../API/core/memory_spaces.html#kokkos-sharedspace
.. |SharedSpace| replace:: ``SharedSpace``

.. _ExecutionSpace: ../API/core/execution_spaces.html#kokkos-executionspaceconcept
.. |ExecutionSpace| replace:: ``ExecutionSpace``

.. _SharedHostPinnedSpace: ../API/core/memory_spaces.html#kokkos-sharedhostpinnedspace
.. |SharedHostPinnedSpace| replace:: ``SharedHostPinnedSpace``

.. _KokkosSharedSpace: ../API/core/memory_spaces.html#kokkos-sharedspace
.. |KokkosSharedSpace| replace:: ``Kokkos::SharedSpace``

Kokkos 4.0 では、``Kokkos_ENABLE_CUDA_UVM`` は非推奨となり、``Kokkos_ENABLE_DEPRECATED_CODE_4`` と組み合わせてのみ使用可能となります。　非推奨となった主な理由は、このオプションの使用により　Cuda　実行空間のmemory_spaceを変更したことです。 これにより、いくつかの問題が生じました。 例えば：ドライバは、アクセス状況に応じて、予告なしにいつでもこのメモリの領域をデバイスまたはホストに移動することが認められています。
``parallel_for``、``parallel_reduce``、または``parallel_scan``におけるアクセスは、いかなる保証された順序でも発生せず、さらに同じ　GPU　上で実行される他のカーネルに依存します。 これによりデバッグ作業が煩雑になります。特に、割り当てられたメモリがどの領域にあるかが明確ではなく、``cmake``　の実行時のオプションに依存している場合には、さらに煩雑になります。

代替方法
---------------

Kokkos 4.0 では、新しいエイリアス「|SharedSpace|_」を導入しています。これは常に、すべての　|ExecutionSpace|_　からアクセス可能なメモリを指示し、ユーザー演算を必要とせず、要求に応じてアクセス元の　|ExecutionSpace|_　へ自動的に移行されます。移行後は、そのメモリは、局所的にアクセスされます。
別名を使用することで、例えば、　``Views``　において表現力に富み、それゆえ読みやすくなります。さらに、これは　``ExecutionSpaces``　間でメモリを自動的に移行できるあらゆるバックエンドに対して移植性があります。
さらに、すべての有効な「実行スペース」からアクセス可能でありながら、常にホストのメモリ内に存在するメモリを指すエイリアス　|SharedHostPinnedSpace|_　を導入しています。

移行
--------------

基本的には、すべての割り当てにおいて、最終的には、テンプレート引数として、|KokkosSharedSpace|_ を指定することとなります。
以下に遷移の例を示します。
以下が以降の一例です:

* 設定時に　``Kokkos_ENABLE_CUDA_UVM``　を必要とするコード (4.0まで)

.. code-block:: cpp

   #include <Kokkos_Core.hpp>

   int main (){
     Kokkos::initialize();
     {
       未署名 int N = 100;
       Kokkos::View<int*> myView("myView",N);
       void* c_style_memory = Kokkos::kokkos_malloc("c_style_alloc",N*sizeof(double));

       ...

       Kokkos::kokkos_free(c_style_memory);
     }
     Kokkos::finalize();
     返し 0;
   }

* ``SharedSpace`` を使用するコード　(4.0以降)

.. code-block:: cpp

   #include <Kokkos_Core.hpp>

   int main (){
     Kokkos::initialize();
     {
       static_assert(Kokkos::has_shared_space,"code only works on backends with SharedSpace");

       未署名 int N = 100;
       Kokkos::View<int*,Kokkos::SharedSpace> myView("myView",N);
       void* c_style_memory = Kokkos::kokkos_malloc<Kokkos::SharedSpace>("c_style_alloc",N*sizeof(double));

       ...

       Kokkos::kokkos_free<Kokkos::SharedSpace>(c_style_memory);
     }
     Kokkos::finalize();
     返し 0;
   }
