
``equal``
=========

ヘッダー: ``<Kokkos_StdAlgorithms.hpp>``

ディスクリプション
-----------

2つの範囲または2つのランク-1 ``ビュー`` 　が等しい場合、真を返します。

インターフェイス
---------

.. 警告:: これは、現在 ``Kokkos::Experimental`` 名前空間内部にあります。


実行空間を受け入れるオーバーロードセット
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: cpp

   テンプレート <class ExecutionSpace, class IteratorType1, class IteratorType2>
   ブール equal(const ExecutionSpace& exespace,                                        (1)
              IteratorType1 first1, IteratorType1 last1,
	      IteratorType2 first2);

   テンプレート <class ExecutionSpace, class IteratorType1, class IteratorType2>
   ブール equal(const std::string& label, const ExecutionSpace& exespace,              (2)
	      IteratorType1 first1, IteratorType1 last1,
	      IteratorType2 first2);

   テンプレート <class ExecutionSpace, class IteratorType1, class IteratorType2,
	     class BinaryPredicateType>
   ブール equal(const ExecutionSpace& exespace,                                        (3)
              IteratorType1 first1, IteratorType1 last1,
	      IteratorType2 first2,
	      BinaryPredicateType predicate);

   テンプレート <class ExecutionSpace, class IteratorType1, class IteratorType2,
	     class BinaryPredicateType>
   ブール equal(const std::string& label, const ExecutionSpace& exespace,              (4)
	      IteratorType1 first1, IteratorType1 last1,
	      IteratorType2 first2,
	      BinaryPredicateType predicate);

   テンプレート <class ExecutionSpace, class IteratorType1, class IteratorType2>
   ブール equal(const ExecutionSpace& exespace, IteratorType1 first1,                  (5)
              IteratorType1 last1, IteratorType2 first2,
	      IteratorType2 last2);

   テンプレート <class ExecutionSpace, class IteratorType1, class IteratorType2>
   ブール equal(const std::string& label, const ExecutionSpace& exespace,              (6)
	      IteratorType1 first1, IteratorType1 last1,
	      IteratorType2 first2, IteratorType2 last2);

   テンプレート <class ExecutionSpace, class IteratorType1, class IteratorType2,
	     class BinaryPredicateType>
   ブール equal(const ExecutionSpace& exespace,                                        (7)
	      IteratorType1 first1, IteratorType1 last1,
	      IteratorType2 first2, IteratorType2 last2,
	      BinaryPredicateType predicate);

   テンプレート <class ExecutionSpace, class IteratorType1, class IteratorType2,
	     class BinaryPredicateType>
   ブール equal(const std::string& label, const ExecutionSpace& exespace,              (8)
	      IteratorType1 first1, IteratorType1 last1,
	      IteratorType2 first2, IteratorType2 last2,
	      BinaryPredicateType predicate);

   テンプレート <class ExecutionSpace, class DataType1, class... Properties1,
	     class DataType2, class... Properties2>
   ブール equal(const ExecutionSpace& exespace,                                        (9)
	      const Kokkos::View<DataType1, Properties1...>& view1,
              const Kokkos::View<DataType2, Properties2...>& view2);

   テンプレート <class ExecutionSpace, class DataType1, class... Properties1,
	     class DataType2, class... Properties2>
   ブール equal(const std::string& label, const ExecutionSpace& exespace,             (10)
	      const Kokkos::View<DataType1, Properties1...>& view1,
	      const Kokkos::View<DataType2, Properties2...>& view2);

   テンプレート <class ExecutionSpace, class DataType1, class... Properties1,
	     class DataType2, class... Properties2, class BinaryPredicate>
   ブール equal(const ExecutionSpace& exespace,                                       (11)
	      const Kokkos::View<DataType1, Properties1...>& view1,
	      const Kokkos::View<DataType2, Properties2...>& view2,
	      BinaryPredicate pred);

   テンプレート <class ExecutionSpace, class DataType1, class... Properties1,
	     class DataType2, class... Properties2, class BinaryPredicate>
   ブール equal(const std::string& label, const ExecutionSpace& exespace,             (12)
	      const Kokkos::View<DataType1, Properties1...>& view1,
	      const Kokkos::View<DataType2, Properties2...>& view2,
	      BinaryPredicate pred);

チームハンドルを受け入れるオーバーロードセット
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. versionadded:: 4.2

.. code-block:: cpp

   テンプレート <class TeamHandleType, class IteratorType1, class IteratorType2>
   KOKKOS_FUNCTION
   ブール equal(const TeamHandleType& teamHandle,                                     (13)
              IteratorType1 first1, IteratorType1 last1,
	      IteratorType2 first2);

   テンプレート <class TeamHandleType, class IteratorType1, class IteratorType2,
	     class BinaryPredicateType>
   KOKKOS_FUNCTION
   ブール equal(const TeamHandleType& teamHandle,                                     (14)
              IteratorType1 first1, IteratorType1 last1,
	      IteratorType2 first2,
	      BinaryPredicateType predicate);

   テンプレート <class TeamHandleType, class IteratorType1, class IteratorType2>
   KOKKOS_FUNCTION
   ブール equal(const TeamHandleType& teamHandle,                                     (15)
              IteratorType1 first1, IteratorType1 last1,
	      IteratorType2 first2, IteratorType2 last2);

   テンプレート <class TeamHandleType, class IteratorType1, class IteratorType2,
	     class BinaryPredicateType>
   KOKKOS_FUNCTION
   ブール equal(const TeamHandleType& teamHandle,                                     (16)
              IteratorType1 first1, IteratorType1 last1,
	      IteratorType2 first2, IteratorType2 last2,
	      BinaryPredicateType predicate);

   テンプレート <class TeamHandleType, class DataType1, class... Properties1,
	     class DataType2, class... Properties2>
   KOKKOS_FUNCTION
   ブール equal(const TeamHandleType& teamHandle,                                     (17)
	      const Kokkos::View<DataType1, Properties1...>& view1,
	      const Kokkos::View<DataType2, Properties2...>& view2);

   テンプレート <class TeamHandleType, class DataType1, class... Properties1,
	     class DataType2, class... Properties2, class BinaryPredicate>
   KOKKOS_FUNCTION
   ブール equal(const TeamHandleType& teamHandle,                                     (18)
	      const Kokkos::View<DataType1, Properties1...>& view1,
	      const Kokkos::View<DataType2, Properties2...>& view2,
	      BinaryPredicate pred);


オーバーロードセット詳細ディスクリプション
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

- (1,2,3,4,13,14): 範囲　``[first1, last1)`` が
  範囲 ``[first2, first2 + (last1 - first1))``　に等しい場合には、真を返し、そうでない場合には、偽を返します。

- (5,6,7,8,15,16): 範囲 ``[first1, last1)`` が 範囲 ``[first2, last2)``　に等しい場合には、真を返し、そうでない場合には、偽を返します。

- (9,10,11,12,17,18):  ``view1`` および ``view2`` が等しい場合には、真を返し、そうでない場合には、偽を返します。

- 該当する場合、二項述語　``pred``　は、二つの要素間の等価性を確認するために使用されます。そうでない場合には、``operator ==`` が使用されます。

パラメータおよび要件
~~~~~~~~~~~~~~~~~~~~~~~~~~~

- ``exespace``: 実行空間インスタンス

- ``teamHandle``: TeamPolicyを使用する際、並列領域内で指定されたチームハンドルインスタンス

- ``ラベル``: デバッグ目的で内部の並列カーネルに転送された文字列

  - (1,3,5,7): デフォルト文字列は、 "Kokkos::equal_iterator_api_default"

  - (9,11): デフォルト文字列は、 "Kokkos::equal_view_api_default"

  - 注意事項: チームハンドルを受け取るオーバーロードは、内部でラベルを使用しません。

- ``first1``, ``last1``, ``first2``, ``last2``: 読み取りおよび比較を行う範囲を定義するイテレータ

  - *ランダムアクセスイテレータ*　である必要があり、例えば、 ``Kokkos::Experimental::(c)begin/(c)end``　から返されなければなりません。

  - 有効な範囲を表す必要があり、つまり、 ``last >= first``　でなければなりません。

  - 必ず　`exespace`` またはチームハンドルに関連付けられた実行空間からアクセス可能である必要があります。

- ``view1``, ``view2``: 比較するためのビュー

  - 必ずランク-1であり、``LayoutLeft``　、  ``LayoutRight``　、または ``LayoutStride``　を持たなければなりません。

  - must be accessible from ``exespace`` or from the execution space associated with the team handle

- ``pred``: *binary* functor returning ``true`` if two arguments should be considered "equal".

  ``pred(a,b)`` must be valid to be called from the execution space passed, or
  the execution space associated with the team handle, and convertible to bool
  for every pair of arguments ``a,b`` of type ``ValueType1`` and ``ValueType2``,
  respectively, ``ValueType1`` and ``ValueType{1,2}`` are the value types of
  ``IteratorType{1,2}`` or ``view{1,2}``, and must not modify ``a,b``.

  - must conform to:

  .. code-block:: cpp

     template <class ValueType1, class ValueType2 = ValueType1>
     struct IsEqualFunctor {
      KOKKOS_INLINE_FUNCTION
      bool operator()(const ValueType1& a, const ValueType2& b) const {
        return (a == b);
      }
     };

Return Value
~~~~~~~~~~~~

If the elements of the two ranges or Views are equal, returns ``true``, otherwise ``false``.

Corner cases when ``false`` is returned:

- if ``view1.extent(0) != view2.extent(1)`` for all overloads accepting Views

- if the length of the range ``[first1, last)`` is not equal to length of ``[first2,last2)``


Example
-------

.. code-block:: cpp

   namespace KE = Kokkos::Experimental;

   template <class ValueType1, class ValueType2 = ValueType1>
   struct IsEqualFunctor {
     KOKKOS_INLINE_FUNCTION
     bool operator()(const ValueType1& a, const ValueType2& b) const {
       return (a == b);
     }
   };

   auto exespace = Kokkos::DefaultExecutionSpace;
   using view_type = Kokkos::View<exespace, int*>;
   view_type a("a", 15);
   view_type b("b", 15);
   // fill a,b somehow

   // create functor
   IsEqualFunctor<int,int> p();

   bool isEqual = KE::equal(exespace, KE::begin(a), KE::end(a),
                            KE::begin(b), KE::end(b) p);

   // To run explicitly on the host (assuming a and b are accessible on Host)
   bool isEqual = KE::equal(Kokkos::DefaultHostExecutionSpace(), KE::begin(a), KE::end(a),
                            KE::begin(b), KE::end(b), p);
