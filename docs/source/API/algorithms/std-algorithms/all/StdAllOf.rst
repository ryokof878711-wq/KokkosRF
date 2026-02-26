
``all_of``
==========

ヘッダー: ``<Kokkos_StdAlgorithms.hpp>``

ディスクリプション
-----------

範囲またはランク1の　``ビュー``　内の全要素が
一項述語を満たす場合、`true`を返します。

インターフェイス
---------

.. 警告:: これは、現在 ``Kokkos::Experimental`` 名前空間内部にあります。

実行空間を受け入れるオーバーロードセット
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: cpp

   テンプレート <class ExecutionSpace, class InputIterator, class Predicate>
   ブール all_of(const ExecutionSpace& exespace,                                       (1)
               InputIterator first, InputIterator last,
	       Predicate predicate);

   テンプレート <class ExecutionSpace, class InputIterator, class Predicate>
   ブール all_of(const std::string& label, const ExecutionSpace& exespace,             (2)
	       InputIterator first, InputIterator last,
	       Predicate predicate);

   テンプレート <class ExecutionSpace, class DataType, class... Properties,
	     class Predicate>
   ブール all_of(const ExecutionSpace& exespace,
	       const ::Kokkos::View<DataType, Properties...>& view,                  (3)
	       Predicate predicate);

   テンプレート <class ExecutionSpace, class DataType, class... Properties,
	     class Predicate>
   ブール all_of(const std::string& label, const ExecutionSpace& exespace,             (4)
	       const ::Kokkos::View<DataType, Properties...>& view,
	       Predicate predicate);

チームハンドルを受け入れるオーバーロードセット
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. versionadded:: 4.2

.. code-block:: cpp

   テンプレート <class TeamHandleType, class InputIterator, class Predicate>
   KOKKOS_FUNCTION
   ブール all_of(const TeamHandleType& teamHandle,                                     (5)
               InputIterator first, InputIterator last,
	       Predicate predicate);

   テンプレート <class TeamHandleType, class DataType, class... Properties,
	     class Predicate>
   KOKKOS_FUNCTION
   ブール all_of(const TeamHandleType& teamHandle,                                     (6)
	       const ::Kokkos::View<DataType, Properties...>& view,
	       Predicate predicate);

パラメータおよび要件
~~~~~~~~~~~~~~~~~~~~~~~~~~~

- ``exespace``: 実行空間インスタンス

- ``teamHandle``: TeamPolicyを使用する際、並列領域内で指定されたチームハンドルインスタンス

- ``label``: デバッグ目的で内部の並列カーネルに転送された文字列

  - 1: デフォルト文字列は、 "Kokkos::all_of_iterator_api_default".

  - 3: デフォルト文字列は、 "Kokkos::all_of_view_api_default".

  - 注意事項: チームハンドルを受け取るオーバーロードは、内部でラベルを使用しません。

- ``first, last``: 検索対象となる要素の範囲

  - *ランダムアクセスイテレータ*である必要があり、例えば、 ``Kokkos::Experimental::(c)begin/(c)end``から返されなければなりません。

  - 有効な範囲を表す必要があり、つまり、 ``last >= first``　でなければなりません。

  - 必ず　`exespace`` またはチームハンドルに関連付けられた実行空間からアクセス可能である必要があります。

- ``view``:

  - 必ずランク-1であり、``LayoutLeft``　、  ``LayoutRight``　、または ``LayoutStride``　を持たなければなりません。

  - 必ず　`exespace`` またはチームハンドルに関連付けられた実行空間からアクセス可能である必要があります。

- ``pred``: *二項関数*　で、引数が望ましい条件を満たす場合に　``真``　を返します。

  ``pred(v)`` must be valid to be called from the execution space passed, or the execution space
  associated with the team handle, and convertible to bool for every argument ``v``
  of type ``value_type``, where ``value_type`` is the value type of ``IteratorType`` or ``view``
  and must not modify ``v``.

  - must conform to:

  .. code-block:: cpp

     struct CustomPredicate{
       KOKKOS_INLINE_FUNCTION
       bool operator()(const value_type & v) const {
         return /* true if v satisfies your desired condition */;
       }
     };


Return Value
~~~~~~~~~~~~

Returns ``true`` if the unary predicate returns ``true`` for all elements in the range or ``view``,
or the range or ``view`` are empty. Returns ``false`` otherwise.
