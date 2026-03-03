
``any_of``
==========

ヘッダー: ``<Kokkos_StdAlgorithms.hpp>``

ディスクリプション
-----------

範囲またはランク1の　ビュー　内の全要素が 一項述語を満たす場合、`true`　を返します。

インターフェイス
---------

.. 警告:: これは、現在 ``Kokkos::Experimental`` 名前空間内部にあります。

実行空間を受け入れるオーバーロードセット
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: cpp

   テンプレート <class ExecutionSpace, class InputIterator, class Predicate>
   ブール any_of(const ExecutionSpace& exespace,                                (1)
               InputIterator first, InputIterator last,
	       Predicate predicate);

   テンプレート <class ExecutionSpace, class InputIterator, class Predicate>
   ブール any_of(const std::string& label, const ExecutionSpace& exespace,      (2)
	       InputIterator first, InputIterator last,
	       Predicate predicate);

   テンプレート <class ExecutionSpace, class DataType, class... Properties,
	     class Predicate>
   ブール any_of(const ExecutionSpace& exespace,                                (3)
	       const ::Kokkos::View<DataType, Properties...>& v,
	       Predicate predicate);

   テンプレート <class ExecutionSpace, class DataType, class... Properties,
	     class Predicate>
   ブール any_of(const std::string& label, const ExecutionSpace& exespace,      (4)
	       const ::Kokkos::View<DataType, Properties...>& v,
	       Predicate predicate);

チームハンドルを受け入れるオーバーロードセット
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. versionadded:: 4.2

.. code-block:: cpp

   テンプレート <class TeamHandleType, class InputIterator, class Predicate>
   KOKKOS_FUNCTION
   ブール any_of(const TeamHandleType& teamHandle,                              (5)
               InputIterator first, InputIterator last,
	       Predicate predicate);

   テンプレート <class TeamHandleType, class DataType, class... Properties,
	     class Predicate>
   KOKKOS_FUNCTION
   ブール any_of(const TeamHandleType& teamHandle,                              (6)
	       const ::Kokkos::View<DataType, Properties...>& v,
	       Predicate predicate);


パラメータおよび要件
~~~~~~~~~~~~~~~~~~~~~~~~~~~

- ``exespace``: 実行空間インスタンス

- ``teamHandle``: TeamPolicyを使用する際、並列領域内で指定されたチームハンドルインスタンス


- ``label``: デバッグ目的で内部の並列カーネルに転送された文字列

  - 1: デフォルト文字列は、 "Kokkos::any_of_iterator_api_default".

  - 3: デフォルト文字列は、 "Kokkos::any_of_view_api_default".

  - 注意事項: チームハンドルを受け取るオーバーロードは、内部でラベルを使用しません。

- ``first, last``: 検索対象となる要素の範囲。

  - *ランダムアクセスイテレータ*　である必要があり、例えば、 ``Kokkos::Experimental::(c)begin/(c)end``から返されなければなりません。

  - 有効な範囲、つまり、 ``last >= first``　を表す必要があります。

  - 必ず　`exespace`` またはチームハンドルに関連付けられた実行空間からアクセス可能である必要があります。

- ``view``:

  - 必ずランク-1であり、``LayoutLeft``　、  ``LayoutRight``　、または ``LayoutStride``　を持たなければなりません。

  - 必ず　`exespace`` またはチームハンドルに関連付けられた実行空間からアクセス可能である必要があります。

- ``pred``:  *二項関数*　で、引数が望ましい条件を満たす場合に　``真``　を返します。

  ``pred(v)`` は、引数として渡された実行空間から呼び出されるために、有効でなければならず、またはチームハンドルに関連付けられた実行空間でなければならず、そして 型　value_type　の引数　``v``　のすべてのペアについて、bool型に変換可能で、そこでは、``value_type``が、``IteratorType``　の値型、または ``view``であり、  ``v``　を変更してはいけません。

  - 以下に一致しなければなりません:

  .. code-block:: cpp

     構造体 CustomPredicate
     {
       KOKKOS_INLINE_FUNCTION
       ブール operator()(const value_type & v) const {
         返し /* vが望ましい条件　*/　を満たす場合に真 */;
       }
     };


返し値
~~~~~~~~~~~~

範囲または　``ビュー``　内の全要素について、一項述語が　少なくとも1つの要素について、範囲内または　``ビュー``　内において、``真``　を返します。そのような要素が認められない場合、または範囲またはまたは　``ビュー``が空の場合、``偽``　を返します。
