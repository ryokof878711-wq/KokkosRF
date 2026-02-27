
``adjacent_find``
=================

ヘッダー: ``<Kokkos_StdAlgorithms.hpp>``

ディスクリプション
-----------

Searches a given range or rank-1 ``View`` for two consecutive equal elements.指定された範囲またはランク1の　``ビュー``　において、連続する2つの等しい要素を検索します。

インターフェイス
---------

.. 警告:: これは、現在 ``Kokkos::Experimental`` 名前空間内部にあります。


実行空間を受け入れるオーバーロードセット
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: cpp

   テンプレート <class ExecutionSpace, class IteratorType>
   IteratorType adjacent_find(const ExecutionSpace& exespace,                              (1)
		              IteratorType first, IteratorType last);

   テンプレート <class ExecutionSpace, class IteratorType>
   IteratorType adjacent_find(const std::string& label, const ExecutionSpace& exespace,    (2)
			      IteratorType first, IteratorType last);

   テンプレート <class ExecutionSpace, class DataType, class... Properties>
   自動 adjacent_find(const ExecutionSpace& exespace,                                      (3)
		      const ::Kokkos::View<DataType, Properties...>& view);

   テンプレート <class ExecutionSpace, class DataType, class... Properties>
   自動 adjacent_find(const std::string& label, const ExecutionSpace& exespace,            (4)
		      const ::Kokkos::View<DataType, Properties...>& view);

   テンプレート <class ExecutionSpace, class IteratorType, class BinaryPredicateType>
   IteratorType adjacent_find(const ExecutionSpace& exespace,                              (5)
		              IteratorType first, IteratorType last,
			      BinaryPredicateType pred);

   テンプレート <class ExecutionSpace, class IteratorType, class BinaryPredicateType>
   IteratorType adjacent_find(const std::string& label, const ExecutionSpace& exespace,    (6)
			      IteratorType first, IteratorType last,
			      BinaryPredicateType pred);

   テンプレート <class ExecutionSpace, class DataType, class... Properties,
	     クラス BinaryPredicateType>
   自動 adjacent_find(const ExecutionSpace& exespace,
		      const ::Kokkos::View<DataType, Properties...>& view,                 (7)
		      BinaryPredicateType pred);

   テンプレート <class ExecutionSpace, class DataType, class... Properties,
	     クラス BinaryPredicateType>
   自動 adjacent_find(const std::string& label, const ExecutionSpace& exespace,            (8)
		      const ::Kokkos::View<DataType, Properties...>& view,
		      BinaryPredicateType pred);

チームハンドルを受け入れるオーバーロードセット
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. versionadded:: 4.2

.. code-block:: cpp

   テンプレート <class TeamHandleType, class IteratorType>
   KOKKOS_FUNCTION
   IteratorType adjacent_find(const TeamHandleType& teamHandle,                            (9)
		              IteratorType first, IteratorType last);

   テンプレート <class TeamHandleType, class DataType, class... Properties>
   KOKKOS_FUNCTION
   自動 adjacent_find(const TeamHandleType& teamHandle,                                   (10)
		      const ::Kokkos::View<DataType, Properties...>& view);

   テンプレート　<class TeamHandleType, class IteratorType, class BinaryPredicateType>
   KOKKOS_FUNCTION
   IteratorType adjacent_find(const TeamHandleType& teamHandle,                           (11)
		              IteratorType first, IteratorType last,
			      BinaryPredicateType pred);

   テンプレート <class TeamHandleType, class DataType, class... Properties,
	     class BinaryPredicateType>
   KOKKOS_FUNCTION
   自動 adjacent_find(const TeamHandleType& teamHandle,
		      const ::Kokkos::View<DataType, Properties...>& view,                (12)
		      BinaryPredicateType pred);

パラメータおよび要件
~~~~~~~~~~~~~~~~~~~~~~~~~~~

- ``exespace``: 実行空間インスタンス

- ``teamHandle``: TeamPolicyを使用する際、並列領域内で指定されたチームハンドルインスタンス

- ``ラベル``: デバッグ目的で内部の並列カーネルに転送された文字列

  - 1,5: デフォルト文字列は、 "Kokkos::adjacent_find_iterator_api_default".

  - 3,7: デフォルト文字列は、 "Kokkos::adjacent_find_view_api_default".

  - 注意事項: チームハンドルを受け取るオーバーロードは、内部でラベルを使用しません。

- ``first, last``: 検索対象となる要素の範囲

  - *ランダムアクセスイテレータ*である必要があり、例えば、``Kokkos::Experimental::(c)begin/(c)end``から返されなければなりません。

  - 有効な範囲を表す必要があり、つまり、``last >= first`` でなければなりません。

  - 必ず　``exespace``　またはチームハンドルに関連付けられた実行空間からアクセス可能である必要があります。

- ``view``:

  - 必ずランク-1であり、``LayoutLeft``、 ``LayoutRight``、または ``LayoutStride``を持たなければなりません。

  - 必ず　``exespace``　またはチームハンドルに関連付けられた実行空間からアクセス可能である必要があります。

- ``pred``: *二項関数*　で、二つの引数が　"等しい"　と見なされる場合に　``真``　を返します。

  ``pred(a,b)`` は、引数として渡された実行空間から呼び出されるためには、有効でなければならない、またはチームハンドルに関連付けられた実行空間でなければならず、そして 型　value_type　の引数　``a,b``　のすべてのペアについて、bool型に変換可能で、そこでは、``value_type``　が、　``IteratorType``　の値型、または　``view``　であり、 　``a,b``　を変更してはいけません。

  - 以下に一致しなければなりません:

  .. code-block:: cpp

     構造体 Comparator{
       KOKKOS_INLINE_FUNCTION
       bool operator()(const value_type & a, const value_type & b) const {
         返し /*  a が b */　に等しいとみなされる場合に真;
       }
     };


返し値
~~~~~~~~~~~~

- 1,2,9:  ``*it == *(it+1)`` が真となるように、第一イテレータ ``it``　を返します。
- 5,6,11:  ``pred(*it, *it+1)`` が真を返すように、第一イテレータ ``it``　を返します。
- 3,4,10:  ``view(it) == view(it+1)`` が真となるように、第一　Kokkos　イテレータ ``it``　を返します。
- 7,8,12:  ``pred(view(it), view(it+1))`` が真を返すように、第一　Kokkos　イテレータ ``it``　を返します。

該当する要素が見つからない場合、イテレータを受け入れるすべてのオーバーロードに対しては ``last`` を返し、
ビューを受け入れるすべてのオーバーロードについては、 ``Kokkos::Experimental::end(view)`` を返します。
