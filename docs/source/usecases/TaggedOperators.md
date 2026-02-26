# タグ付き演算

## C++17以前のバージョンへの対応

HPC分野の多くのアプリケーションやライブラリは、オブジェクト指向設計を用いて作成されています。
これには、コードの整理および再利用の可能性という点で、多くの利点があります。
残念ながら、C++17　が利用できない場合、　Kokkos　の使用に関して重大な欠点が生じます。

粒子系の時間発展をシミュレートするアプリケーションについて検討します。
オブジェクト指向設計においては、おそらく`ParticleInteractions`　クラスのようなものが含まれる可能性が高く、
このクラスは、粒子および相互作用ポテンシャルのデータ構造への参照を保持します。
また、メンバー関数　`compute`　も含まれており、この関数は粒子をループ処理し
相互作用を計算します:

```c++
クラス ParticleInteractions {
  ParticlePositions pos;
  ParticleForces forces;
  ParticleNeighbors neighbors;
  PairForce force;

パブリック:
  void compute() {
    for(int i=0; i<pos.extent(0); i++) {
      for(int j=0; j<neighbors.count(i); j++) {
        forces(i) = force(pos(i),pos(neighbors(i,j)));
      }
    }
  }
};
```

この処理を並列化する簡単な方法は、メンバ関数 `compute` の内部に [`parallel_for`](../API/core/parallel-dispatch/parallel_for) を追加することです。

```c++
クラス ParticleInteractions {
  ...
  void compute() {
    parallel_for("Compute", KOKKOS_LAMBDA (const int i) {
      for(int j=0; j<neighbors.count(i); j++) {
        forces(i) = force(pos(i),pos(neighbors(i,j)));
      }
    });
  }
};
```

実際、非アクセラレータベースのシステムでは、これは機能するでしょう。しかし、[`parallel_for`](../API/core/parallel-dispatch/parallel_for)　がアクセラレータに作業を分散させるシステムでは、この手法はアクセスエラーで失敗する可能性が高いです。

その理由は、クラスメンバ関数内のラムダ式は他のクラスメンバを個別にキャプチャせず、クラス全体をキャプチャするためです。
より正確に言えば、C++17　以前のバージョンでは、`compute`のスコープ内で　`this`　ポインタをキャプチャします。

実際には、以下の通りに書かれたかのようです：
```c++
クラス ParticleInteractions {
  ...
  void compute() {
    parallel_for("Compute", KOKKOS_LAMBDA (const int i) {
      for(int j=0; j<this->neighbors.count(i); j++) {
        this->forces(i) = this->force(this->pos(i),this->pos(this->neighbors(i,j)));
      }
    });
  }
};
```
実行空間の範囲内で、`this`　の参照解除ができない場合、実行は失敗します。

C++17　では、`[*this]` キャプチャ句を使用することでこの状況を修正できます。 その場合、
ディスパッチの一環として、クラスインスタンス全体がキャプチャされ、アクセラレータにコピーされます。

C++17　以前の状況に対処する一つの方法としては、単純に
compute　関数に対応する演算子を作成し、クラス自体をファンクタとしてディスパッチすることがあります。:
```c++
クラス ParticleInteractions {
  ...
  void compute() {
    parallel_for("Compute", *this);
  }
  KOKKOS_FUNCTION
  void operator() (const int i) const {
    for(int j=0; j<neighbors.count(i); j++) {
      forces(i) = force(pos(i),pos(neighbors(i,j)));
    }
  }
};
```

実際、並列操作の作業項目を、ディスパッチ自体およびそれに伴う事前および事後処理作業から分離するため、この手法には明瞭さを高める特性さえあるかもしれません。

では、クラス内に複数の並列処理が存在する場合は、どうでしょうか？
ここで、Kokkos のタグ付きディスパッチが役立ちます。
クラスが複数の演算子を持つ場合、Kokkosではこれらの演算子に、オーバーロード解決に使用される、未使用の追加パラメータを持たせることが可能です。
本パラメータは、ディスパッチ時に、実行ポリシーへの追加テンプレートパラメータとして指定されます。
推奨される方法は、元のオブジェクト内にネストされた定義として`タグクラス`を作成することです:

```c++
クラス ParticleInteractions {
  クラス TagPhase1 {};
  クラス TagPhase2 {};
  ...

  void compute() {
    parallel_for("Compute1", RangePolicy<TagPhase1>(0,N), *this);
    parallel_for("Compute2", RangePolicy<TagPhase2>(0,N), *this);
  }
  KOKKOS_FUNCTION
  void operator() (TagPhase1, const int i) const {
    ...
  }
  KOKKOS_FUNCTION
  void operator() (TagPhase2, const int i) const {
    ...
  }
};
```

## テンプレート演算子

タグ付きインターフェースのもう一つの有用な応用例は、演算子をテンプレート化する機会です。
特定の使用事例として、実行時ループパラメータをコンパイル時パラメータに変換することが挙げられます:
