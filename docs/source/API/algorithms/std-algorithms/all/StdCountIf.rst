
``count_if``
============

ヘッダー: ``<Kokkos_StdAlgorithms.hpp>``

ディスクリプション
-----------

指定された一項述語を満たす範囲またはランク1の　``ビュー``　内の要素数を返します。

インターフェイス
---------

.. 警告:: これは、現在 ``Kokkos::Experimental`` 名前空間内部にあります。

実行空間を受け入れるオーバーロードセット
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: cpp

   テンプレート <class ExecutionSpace, class IteratorType, class Predicate>
   型名 IteratorType::difference_type count_if(const ExecutionSpace& exespace,
						   IteratorType first,
						   IteratorType last,                   (1)
						   Predicate pred);


   テンプレート <class ExecutionSpace, class IteratorType, class Predicate>
   型名 IteratorType::difference_type count_if(const std::string& label,
						   const ExecutionSpace& exespace,
						   IteratorType first,                  (2)
						   IteratorType last,
						   Predicate pred);

   テンプレート <class ExecutionSpace, class DataType, class... Properties,
	     class Predicate>
   自動 count_if(const ExecutionSpace& exespace,
		 const ::Kokkos::View<DataType, Properties...>& view,                   (3)
		 Predicate pred);

   テンプレート <class ExecutionSpace, class DataType, class... Properties,
	     class Predicate>
    count_if(const std::string& label, const ExecutionSpace& exespace,
		 const ::Kokkos::View<DataType, Properties...>& view,                   (4)
		 Predicate pred);


チームハンドルを受け入れるオーバーロードセット
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. versionadded:: 4.2

.. code-block:: cpp

   テンプレート <class TeamHandleType, class IteratorType, class Predicate>
   KOKKOS_FUNCTION
   型名 IteratorType::difference_type count_if(const TeamHandleType& teamHandle,
						   IteratorType first,
						   IteratorType last,                   (5)
						   Predicate pred);

   テンプレート <class TeamHandleType, class DataType, class... Properties,
	     class Predicate>
   KOKKOS_FUNCTION
   自動 count_if(const TeamHandleType& teamHandle,
		 const ::Kokkos::View<DataType, Properties...>& view,                   (6)
		 Predicate pred);

パラメータおよび要件
~~~~~~~~~~~~~~~~~~~~~~~~~~~

- ``exespace``: 実行空間インスタンス

- ``teamHandle``: TeamPolicyを使用する際、並列領域内で指定されたチームハンドルインスタンス

- ``ラベル``: デバッグ目的で内部の並列カーネルに転送された文字列

  - 1: デフォルト文字列は、  "Kokkos::count_if_iterator_api_default".

  - 3: デフォルト文字列は、  "Kokkos::count_if_view_api_default".

  - 注意事項: チームハンドルを受け取るオーバーロードは、内部でラベルを使用しません。

- ``first, last``: 検索対象となる要素の範囲

  - *ランダムアクセスイテレータ*　である必要があり、例えば、 ``Kokkos::Experimental::(c)begin/(c)end``　から返されなければなりません。

  - 有効な範囲を表す必要があり、つまり、 ``last >= first``　でなければなりません。

  - 必ず　`exespace`` またはチームハンドルに関連付けられた実行空間からアクセス可能である必要があります。

- ``view``:

  - 必ずランク-1であり、``LayoutLeft``　、  ``LayoutRight``　、または ``LayoutStride``　を持たなければなりません。

  - 必ず　`exespace`` またはチームハンドルに関連付けられた実行空間からアクセス可能である必要があります。

- ``pred``: *二項関数*　で、引数が望ましい条件を満たす場合に　``真``　を返します。

  ``pred(v)`` は、引数として渡された実行空間から呼び出されるためには、有効でなければならない、またはチームハンドルに関連付けられた実行空間でなければならず、そして 型　value_type　の引数　``v``　のすべてのペアについて、bool型に変換可能で、そこでは、``value_type``が、``IteratorType``　の値型、または ``view``であり、  ``v``　を変更してはいけません。

  - 以下に一致しなければなりません:

  .. code-block:: cpp

     構造体 CustomPredicate{
       KOKKOS_INLINE_FUNCTION
       bool operator()(const value_type & v) const {
         return /* true if v satisfies your desired condition */;
       }
     };

返し値
~~~~~~~~~~~~

範囲 first, last または 値　に等しい ビュー の中、または述語が真である ``ビュー``　内にある要素数を返します。
