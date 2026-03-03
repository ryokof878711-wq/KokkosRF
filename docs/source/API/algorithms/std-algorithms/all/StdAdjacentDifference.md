
# `adjacent_difference`

ヘッダーファイル: `Kokkos_StdAlgorithms.hpp`

```c++
namespace Kokkos{
namespace Experimental{

テンプレート <class ExecutionSpace, class InputIteratorType, class OutputIteratorType>
OutputIteratorType adjacent_difference(const ExecutionSpace& exespace,                    (1)
                                       InputIteratorType first_from,
                                       InputIteratorType last_from,
                                       OutputIteratorType first_dest);

テンプレート <class ExecutionSpace, class InputIteratorType, class OutputIteratorType, class BinaryOp>
OutputIteratorType adjacent_difference(const ExecutionSpace& exespace,                    (2)
                                       InputIteratorType first_from,
                                       InputIteratorType last_from,
                                       OutputIteratorType first_dest,
                                       BinaryOp bin_op);

テンプレート <class ExecutionSpace, class InputIteratorType, class OutputIteratorType>
OutputIteratorType adjacent_difference(const std::string& label,                          (3)
                                       const ExecutionSpace& exespace,
                                       InputIteratorType first_from,
                                       InputIteratorType last_from,
                                       OutputIteratorType first_dest);

テンプレート <class ExecutionSpace, class InputIteratorType, class OutputIteratorType, class BinaryOp>
OutputIteratorType adjacent_difference(const std::string& label,                          (4)
                                       const ExecutionSpace& exespace,
                                       InputIteratorType first_from,
                                       InputIteratorType last_from,
                                       OutputIteratorType first_dest,
                                       BinaryOp bin_op);

テンプレート <
  クラス ExecutionSpace,
  クラス DataType1, class... Properties1,
  クラス DataType2, class... Properties2>
自動 adjacent_difference(const ExecutionSpace& exespace,                                  (5)
                         const ::Kokkos::View<DataType1, Properties1...>& view_from,
                         const ::Kokkos::View<DataType2, Properties2...>& view_dest);

テンプレート <
  クラス ExecutionSpace,
  クラス DataType1, class... Properties1,
  クラス DataType2, class... Properties2,
  クラス BinaryOp>
自動 adjacent_difference(const ExecutionSpace& exespace,                                  (6)
                         const ::Kokkos::View<DataType1, Properties1...>& view_from,
                         const ::Kokkos::View<DataType2, Properties2...>& view_dest,
                         BinaryOp bin_op);

テンプレート <
  クラス ExecutionSpace,
  クラス DataType1, class... Properties1,
  クラス DataType2, class... Properties2>
自動 adjacent_difference(const std::string& label,                                        (7)
                         const ExecutionSpace& exespace,
                         const ::Kokkos::View<DataType1, Properties1...>& view_from,
                         const ::Kokkos::View<DataType2, Properties2...>& view_dest);

テンプレート <
  クラス ExecutionSpace,
  クラス DataType1, class... Properties1,
  クラス DataType2, class... Properties2,
  クラス BinaryOp>
自動 adjacent_difference(const std::string& label,                                        (8)
                         const ExecutionSpace& exespace,
                         const ::Kokkos::View<DataType1, Properties1...>& view_from,
                         const ::Kokkos::View<DataType2, Properties2...>& view_dest,
                         BinaryOp bin_op);

} //エンド 名前空間 実験的
} //エンド 名前空間 Kokkos
```

## ディスクリプション

- (1,3,5,7): 第一に、 `*first_from` のコピーは、(1,3)　について`*first_dest` に書き込まれ、
  または、 `view_from(0)` のコピーは、 (5,7)　について `view_dest(0)` に書き込まれます。
  第二に、それは、(1,3)　について、または、 (5,7)　について `view_from` において、範囲`[first_from, last_from)` の要素の各隣接ペアの二番目及び一番目の *差* を計算し、それらを (1,3)　について、または  (5,7)　について `view_dest``first_dest + 1`　において範囲の初めに書き込みます。 

- (2,4,6,8): 第一に、 `*first_from`  のコピーは、 (2,4)　について　`*first_dest` に書き込まれ、
  または、`view_from(0)` のコピーは、 (6,8)　について `view_dest(0)` に書き込まれます。
  第二に、 (2,4)　について範囲　`[first_from, last_from)` の、または、 (6,8)　について  `view_from`　において、要素の各隣接ペアの二番目及び一番目を使ったバイナリファンクタを呼び出し、それらを　`first_dest + 1`  (2,4)　について、または (6,8)　について `view_dest`　において、範囲の初めに書き込みます。 



## パラメータおよび要件

- `exespace`:
  - 実行空間インスタンス
- `ラベル`:
  - デバッグ目的で実装カーネルに名前を付けるために使用されます
  - 1,2　について、 デフォルトストリングは、以下の通りです: "Kokkos::adjacent_difference_iterator_api_default"
  - 5,6　について、 デフォルトストリングは、以下の通りです: "Kokkos::adjacent_difference_view_api_default"
- `first_from`, `last_from`, `first_dest`:
  -  `*_from` から読み取り、`first_dest`　に書き込むための要素の範囲
  -  *ランダムアクセスイテレータ*　でなければなりません
  - 有効な範囲、すなわち、`last_from >= first_from` を表す必要があります。（デバッグモードで確認済み）
  - `exespace`　からアクセス可能でなければなりません。
- `view_from`, `view_dest`:
  - `view_from` から読み取り、`view_dest`　に書き込むためのビュー
  - ランク-1であり、`LayoutLeft`, `LayoutRight`, または `LayoutStride`　を持たなければなりません。
  - `exespace`　からアクセス可能でなければなりません。
- `bin_op`:
  - 各要素のペアに対して適用する演算を表す　*バイナリ* ファンクタ。
  渡された実行空間より呼び出され、そこでは `value_type`が
  `InputIteratorType`の価値型 (1,2,3,4について) または、 `view_from`　の価値型 (5,6,7,8について)　であり、 `a,b`　を修正しなければなりません。
  - 以下に一致しなければなりません:
  ```c++
  構造体 BinaryOp
  {
     KOKKOS_INLINE_FUNCTION
     return_type operator()(const value_type & a,
	                        const value_type & b) const {
       返し /* ... */;
     }

     // または、また有効
     return_type operator()(value_type a,
	                        value_type b) const {
       返し /* ... */;
     }
  };
  ```
  返し型 `return_type` は、 (1,2,3,4)　については、型のオブジェクト `OutputIteratorType` 
  または、 `value_type` が、 (5,6,7,8)　については、 `view_dest`　の価値型である型のオブジェクトが、参照解除可能であり、
  型 `return_type`　の値が割り当て可能であるようになければなりません。

## 返し

書き込まれた最後の要素の　*後の*　要素へのイテレータ。
