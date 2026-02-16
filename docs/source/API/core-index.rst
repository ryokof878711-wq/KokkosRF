コア API
########

.. リスト表::
   :幅: 20 80
   :ヘッダー列: 1

   * - リデューサ
     - ディスクリプション
   * - `初期化及び最終処理完了 <core/Initialize-and-Finalize.html>`__
     - Kokkos　の初期化及び最終処理完了。
   * - `ビューおよびその関連 <core/View.html>`__
     - Kokkos　多次元ビュークラスおよび関連するフリー関数
   * - `Parallel Execution/Dispatch <core/ParallelDispatch.html>`__
     - 並列実行ディスパッチ。
   * - `組み込みリデューサ <core/builtin_reducers.html>`__
     - 組み込みリデューサ。
   * - `実行ポリシー <core/Execution-Policies.html>`__
     - 実行ポリシー。
   * - `Spaces <core/Spaces.html>`__
     - メモリ空間及び実行空間のディスクリプション。
   * - `タスク並列処理 <core/Task-Parallelism.html>`__
     - タスクグラフの作成と配信。
   * - `MultiGPU サポート <core/MultiGPUSupport.html>`__
     - 単一プロセスから複数のGPU上でカーネルを起動。
   * - `アトミック <core/atomics.html>`__
     - アトミック。
   * - `数値計算 <core/Numerics.html>`__
     - 一般的な数学関数、数学定数、数値特性。
   * - `Cスタイルメモリ管理 <core/c_style_memory_management.html>`__
     - Cスタイルメモリ管理
   * - `特性 <core/Prod.html>`__
     - 特性。
   * - `Kokkos 概念 <core/KokkosConcepts.html>`__
     - Kokkos 概念
   * - `STL 互換性問題 <core/STL-Compatibility.html>`__
     - 標準　C++　の機能の移植版であり、それらが様々なハードウェアプラットフォームでは動作しない場合あり。
   * - `Utilities <core/Utilities.html>`__
     -  Kokkos コアのユーティリティ機能性部分。
   * - `検出イディオム <core/Detection-Idiom.html>`__
     - SFINAE　に配慮した方法で、任意の　C++　式が有効かどうかを認識するために使用。
   * - `マクロス　<core/Macros.html>`__
     - グローバルマクロであり、アーキテクチャや一般的な設定等に使用。
   * - `半精度型 <core/Half-precision-types.html>`__
     - 半精度型へのポータブルなアクセスを実現する補助関数。

.. toctree::
   :hidden:
   :maxdepth: 1

   ./core/Initialize-and-Finalize
   ./core/View
   ./core/ParallelDispatch
   ./core/builtin_reducers
   ./core/Execution-Policies
   ./core/Spaces
   ./core/Task-Parallelism
   ./core/MultiGPUSupport
   ./core/atomics
   ./core/Numerics
   ./core/c_style_memory_management
   ./core/Traits
   ./core/KokkosConcepts
   ./core/STL-Compatibility
   ./core/Utilities
   ./core/Detection-Idiom
   ./core/Macros
   ./core/Profiling
   ./core/Half-precision-types
