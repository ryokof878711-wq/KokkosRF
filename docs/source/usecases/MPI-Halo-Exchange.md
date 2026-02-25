# MPI Halo Exchange

Kokkos　と　MPI　は相互補完的なプログラミングモデルであり、Kokkos　は共有メモリ空間内での並列プログラミングを扱うために設計されており、MPI　は複数の分散メモリ空間間での並列プログラミングを扱うために設計されています。
完全に拡張性のある並列プログラムを作成するためには、
多くの場合、Kokkos　と　MPI　を併用する必要がある場合がしばしばあります。
This Use Case document walks through an example of how MPI and Kokkos can work together.本使用事例文書では、MPI　とKokkos　が連携して動作する例について説明しています。

## 単一メッセージ送信

MPI はメッセージパッシングセマンティクスに基づいており、MPIにおける最も単純な演算の一つが、
単一メッセージ送信です。
通常、このメッセージは単一の連続したメモリ領域に格納され、
同じ型の値（例えば、倍精度浮動小数点値）が複数個含まれています。
このメッセージの定義は、`Kokkos::View` の定義と非常に似ていることに注意してください：
同じ型の値の集合であり、多くの場合連続しています。
したがって、`Kokkos::View` の内容を単一の MPI メッセージとして送信することは、多くの場合、非常に簡単です。
これを行う方法は、MPIが必要とするものを取得ですることです：メッセージ割り当ての先頭へのポインタを `Kokkos::View::data()` 経由で、メッセージ内の項目の数を `Kokkos::View::size()` 経由で取得します。
以下は、あるランクから別のランクへ`double`値を送信する例です。

```c++
int source_rank = 1;
int destination_rank = 1;
int number_of_doubles = 12;
int tag = 42;
Kokkos::View<double*> buffer("buffer", number_of_doubles);
int my_rank;
MPI_Comm comm = MPI_COMM_WORLD;
MPI_Comm_rank(comm, &my_rank);
if (my_rank == source_rank) {
  MPI_Send(buffer.data(), int(buffer.size()), MPI_DOUBLE, destination_rank, tag, comm);
} else if (my_rank == destination_rank) {
  MPI_Recv(buffer.data(), int(buffer.size()), MPI_DOUBLE, destination_rank, tag, comm);
}
```

## CUDA　対応　MPI

One common concern for programmers who are using CUDA GPU parallelism through Kokkos as well as MPI is
how to use MPI to communicate between two ranks which are each using CUDA parallelism.
The current path forward is quite simple: just pass your allocation pointers as before and it should
"just work".
In particular, the MPI libraries installed on GPU clusters should be compiled with what we call
CUDA-aware support.
This means that the MPI library is aware of CUDA and will call CUDA functions which determine whether
the user's allocation pointer points to device memory, host memory, or managed (UVM) memory.
If the pointer points to device or managed memory, the MPI library can optimize the communication by
calling CUDA functions to copy the relevant memory from one place to another on the same GPU,
or from one GPU to another through PCIe or NVIDIA NVLINK if available.
As such, the example above continues to work even if `Kokkos::DefaultExecutionSpace` is `Kokkos::Cuda`.

## Separating out messages

There is often a need in unstructured MPI-based codes to determine what subsets of data need to be
packed into which messages and sent to which other ranks, based on less structured information.
For example, assume we have a simulation composed of thousands of "elements", and each element is
owned by one MPI rank.
Other ranks may have redundant copies of that element, but some fundamental decision-making related
to that element must be done by the MPI rank that owns it.
Suppose further that on each MPI rank there exists an array that maps each element, whether owned
or redundantly copied, to the (possibly different) MPI rank which owns that element.
We can filter out the subset of these elements that are associated with a given owner using
[`Kokkos::parallel_scan`](../API/core/parallel-dispatch/parallel_scan) and subsequently pack messages using [`Kokkos::parallel_for`](../API/core/parallel-dispatch/parallel_for).

## Identifying subset indices

For the filter-out step, we simply need to identify which ranks (keys) are the same as some
known destination, and if they are then we number them consecutively in the order they appear
(which is a scan operation).
Here is an example which filters out a subset of items:

```c++
// an exclusive scan functor, which during the final pass will
// assign into the compressed subset indices
class subset_scanner {
public:
  using execution_space = Kokkos::DefaultExecutionSpace;
  using value_type = int;
  using size_type = int;
  subset_scanner(
      Kokkos::View<int*, execution_space> keys_in,
      int desired_key_in,
      Kokkos::View<int*, execution_space> subset_indices_in)
    :m_keys(keys_in)
    ,m_desired_key(desired_key_in)
    ,m_subset_indices(subset_indices_in)
  {}
  KOKKOS_INLINE_FUNCTION void operator()(int i, int& update, const bool final_pass) const {
    bool is_in = (m_keys[i] == m_desired_key);
    if (final_pass && is_in) {
      m_subset_indices[update] = i;
    }
    update += (is_in ? 1 : 0);
  }
  KOKKOS_INLINE_FUNCTION void init(int& update) const {
    update = 0;
  }
  KOKKOS_INLINE_FUNCTION void join(int& update, const int& input) const {
    update += input;
  }
private:
  Kokkos::View<int*, execution_space> m_keys;
  int m_desired_key;
  Kokkos::View<int*, execution_space> m_subset_indices;
};

Kokkos::View<int*> find_subset(Kokkos::View<int*> keys, int desired_key) {
  int subset_size = 0;
  Kokkos::parallel_reduce(keys.size(), KOKKOS_LAMBDA(int i, int& local_sum) {
    return keys[i] == desired_key ? 1 : 0;
  }, subset_size);
  Kokkos::View<int*> subset_indices("subset indices", subset_size);
  Kokkos::parallel_scan(keys.size(), subset_scanner(keys, desired_key, subset_indices));
  return subset_indices;
}
```

## Extracting subset message

Once we are able to produce a list of subset indices (those indices of elements which will be transmitted in one message),
we can use that list of indices to extract a subset of the simulation data to send.
Here, let us assume that we have a [`Kokkos::View`](../API/core/view/view) which stores one floating-point value per element, and we want
to extract a message containing only the floating-point values for the relevant subset.

```c++
Kokkos::View<double*> pack_message(Kokkos::View<double*> all_element_data, Kokkos::View<int*> subset_indices) {
  Kokkos::View<double*> message("message", subset_indices.size());
  Kokkos::parallel_for(subset_indices.size(), KOKKOS_LAMBDA(int i) {
    message[i] = all_element_data[subset_indices[i]];
  });
  return message;
}
```
