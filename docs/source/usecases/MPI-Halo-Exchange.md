# MPI Halo Exchange

Kokkos　と　MPI　は相互補完的なプログラミングモデルであり、Kokkos　は共有メモリ空間内での並列プログラミングを扱うために設計されており、MPI　は複数の分散メモリ空間間での並列プログラミングを扱うために設計されています。
完全に拡張性のある並列プログラムを作成するためには、
多くの場合、Kokkos　と　MPI　を併用する必要がある場合がしばしばあります。
本使用事例文書では、MPI　と　Kokkos　が連携して動作する例について説明しています。

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

Kokkos　を介した　CUDA GPU　並列処理と　MPI　の両方を使用しているプログラマーの方々が抱える一般的な懸念の一つは、
それぞれが　CUDA　並列処理を利用している二つのランク間で、MPI　を用いてどのように通信を行うかということです。
現在の進め方は非常に簡単です：これまでと同様に割り当てポインタを渡すだけで、
"問題なく動作する"はずです。
特に、GPU　クラスタにインストールされる　MPI　ライブラリは、いわゆる
CUDA　対応サポートを使って、コンパイルされる必要があります。
これは、MPI　ライブラリがCUDA　を認識しており、ユーザーの割り当てポインタがデバイスメモリ、ホストメモリ、または管理された（UVM）メモリのいずれを指しているかを判断する　CUDA　関数を呼び出すことを意味します。
ポインタがデバイスメモリまたは管理メモリを指している場合、MPI　ライブラリは通信を最適化するために、
CUDA　関数を呼び出して関連するメモリを同一　GPU　上の別の場所へコピーする、
あるいは利用可能な場合、PCIe　または　NVIDIA NVLINK　を介してGPU間をコピーすることが可能です。
したがって、上記の例は、`Kokkos::DefaultExecutionSpace` が `Kokkos::Cuda` であっても引き続き正常に動作します。

## メッセージ分離

構造化されていないMPIベースのコードでは、多くの場合に、より構造化されていない情報に基づいて、どのデータのサブセットをどのメッセージにパックし、どの他のランクに送信する必要があるかを判断する必要が生じることがあります。
例えば、何千もの「要素」で構成されるシミュレーションがあると仮定し、各要素は
1つの　MPI　ランクによって所有されています。
他のランクは当該要素の冗長コピーを保持している可能性がありますが、当該要素に関連する基本的な決定は、その要素を所有する　MPI　ランクによって行われなければなりません。
Suppose further that on each MPI rank there exists an array that maps each element, whether owned
or redundantly copied, to the (possibly different) MPI rank which owns that element.さらに、各　MPI　ランク上に、所有されているか冗長にコピーされているかを問わず、各要素をその要素を所有する（異なる可能性のある）MPIランクにマッピングする配列が存在すると仮定します。
特定の所有者と関連付けられている要素のサブセットを、
[`Kokkos::parallel_scan`](../API/core/parallel-dispatch/parallel_scan) を使用して抽出することが可能であり、その後、[`Kokkos::parallel_for`](../API/core/parallel-dispatch/parallel_for) を使用してメッセージをパックします。

## 部分集合インデックスの特定

フィルタリング工程では、どのランク（キー）が特定の既知の宛先と一致するかを特定し、該当するランクが存在する場合には、出現順に連続した番号を割り当てます（これはスキャン演算となります）。
以下は、アイテムの一部をフィルタリングする例です:

```c++
// an exclusive scan functor, which during the final pass will排他的なスキャンファンクタであり、
// 最終パスにおいて圧縮された部分集合のインデックスに値を割り当てます
クラス subset_scanner {
パブリック:
  execution_space = Kokkos::DefaultExecutionSpace　を使用；
  using value_type = int　を使用;
  using size_type = int　を使用;
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
    更新 += (is_in ? 1 : 0);
  }
  KOKKOS_INLINE_FUNCTION void init(int& update) const {
    更新 = 0;
  }
  KOKKOS_INLINE_FUNCTION void join(int& update, const int& input) const {
    更新　+= 入力;
  }
プライベート:
  Kokkos::View<int*, execution_space> m_keys;
  int m_desired_key;
  Kokkos::View<int*, execution_space> m_subset_indices;
};

Kokkos::View<int*> find_subset(Kokkos::View<int*> keys, int desired_key) {
  int subset_size = 0;
  Kokkos::parallel_reduce(keys.size(), KOKKOS_LAMBDA(int i, int& local_sum) {
    戻し　keys[i] == desired_key ? 1 : 0;
  }, subset_size);
  Kokkos::View<int*> subset_indices("subset indices", subset_size);
  Kokkos::parallel_scan(keys.size(), subset_scanner(keys, desired_key, subset_indices));
  戻し subset_indices;
}
```

## 部分集合メッセージ抽出

部分集合インデックス一覧（1つのメッセージで送信される要素のインデックス）を生成できるようになれば、
そのインデックス一覧を用いて、送信するシミュレーションデータの部分集合を抽出することが可能となります。
ここでは、要素ごとに1つの浮動小数点値を格納する[`Kokkos::View`](../API/core/view/view)があると仮定し、
関連する部分集合の浮動小数点値のみを含むメッセージを抽出したいものとします。

```c++
Kokkos::View<double*> pack_message(Kokkos::View<double*> all_element_data, Kokkos::View<int*> subset_indices) {
  Kokkos::View<double*> message("message", subset_indices.size());
  Kokkos::parallel_for(subset_indices.size(), KOKKOS_LAMBDA(int i) {
    message[i] = all_element_data[subset_indices[i]];
  });
  返し message;
}
```
