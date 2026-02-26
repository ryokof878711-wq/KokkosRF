
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
  第二に、それは、(1,3)　について、または、 (5,7)　について `view_from` において、範囲`[first_from, last_from)` の要素の各隣接ペアの二番目及び一番目の *差* を計算し、
   それらを (1,3)　について、または  (5,7)　について `view_dest``first_dest + 1`　において範囲の初めに書き込みます。 

- (2,4,6,8): 第一に、 `*first_from`  のコピーは、 (2,4)　について　`*first_dest` に書き込まれ、
  または、`view_from(0)` のコピーは、 (6,8)　について `view_dest(0)` に書き込まれます。
  第二に、it calls the binary functor with the second and the first elements
  of each adjacent pair of elements of the range for (2,4)　について範囲　`[first_from, last_from)` の、または、 (6,8)　について  `view_from`　において、要素の各隣接ペアの二番目及び一番目を使ったバイナリファンクタを呼び出し、それらを
   `first_dest + 1` for (2,4)　について、または (6,8)　について `view_dest`　において、範囲の初めに書き込みます。 



## Parameters and Requirements

- `exespace`:
  - execution space instance
- `label`:
  - used to name the implementation kernels for debugging purposes
  - for 1,2 the default string is: "Kokkos::adjacent_difference_iterator_api_default"
  - for 5,6 the default string is: "Kokkos::adjacent_difference_view_api_default"
- `first_from`, `last_from`, `first_dest`:
  - range of elements to read from `*_from` and write to `first_dest`
  - must be *random access iterators*
  - must represent a valid range, i.e., `last_from >= first_from` (checked in debug mode)
  - must be accessible from `exespace`
- `view_from`, `view_dest`:
  - views to read elements from `view_from` and write to `view_dest`
  - must be rank-1, and have `LayoutLeft`, `LayoutRight`, or `LayoutStride`
  - must be accessible from `exespace`
- `bin_op`:
  - *binary* functor representing the operation to apply to each pair of elements.
  Must be valid to be called from the execution space passed, and callable with
  two arguments `a,b` of type (possible const) `value_type`, where `value_type`
  is the value type of `InputIteratorType` (for 1,2,3,4) or the value type
  of `view_from` (for 5,6,7,8), and must not modify `a,b`.
  - must conform to:
  ```c++
  struct BinaryOp
  {
     KOKKOS_INLINE_FUNCTION
     return_type operator()(const value_type & a,
	                        const value_type & b) const {
       return /* ... */;
     }

     // or, also valid
     return_type operator()(value_type a,
	                        value_type b) const {
       return /* ... */;
     }
  };
  ```
  The return type `return_type` must be such that an object of type `OutputIteratorType` for (1,2,3,4)
  or an object of type `value_type` where `value_type` is the value type of `view_dest` for (5,6,7,8)
  can be dereferenced and assigned a value of type `return_type`.

## Return

Iterator to the element *after* the last element written.
