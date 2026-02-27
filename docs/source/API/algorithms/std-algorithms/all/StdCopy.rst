
``copy``
========

ヘッダー: ``<Kokkos_StdAlgorithms.hpp>``

ディスクリプション
-----------

ソース範囲またはランク1の　``ビュー``から、要素を宛先範囲またはランク1にコピー。

インターフェイス
---------

.. 警告:: これは、現在 ``Kokkos::Experimental`` 名前空間内部にあります。

実行空間を受け入れるオーバーロードセット
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: cpp

  テンプレート <class ExecutionSpace, class InputIteratorType, class OutputIteratorType>
  OutputIteratorType copy(const ExecutionSpace& exespace,                      (1)
                          InputIteratorType first_from,
                          InputIteratorType last_from,
                          OutputIteratorType first_to);

  テンプレート <class ExecutionSpace, class InputIteratorType, class OutputIteratorType>
  OutputIteratorType copy(const std::string& label,                            (2)
                          const ExecutionSpace& exespace,
                          InputIteratorType first_from,
                          InputIteratorType last_from,
                          OutputIteratorType first_to);
  テンプレート <
    クラス ExecutionSpace,
    クラス DataType1, class... Properties1,
    クラス DataType2, class... Properties2
  >
  自動 copy(const ExecutionSpace& exespace,                                    (3)
            const Kokkos::View<DataType1, Properties1...>& view_from,
            const Kokkos::View<DataType2, Properties2...>& view_to);
  テンプレート <
    クラス ExecutionSpace,
    クラス DataType1, class... Properties1,
    クラス DataType2, class... Properties2
  >
   copy(const std::string& label, const ExecutionSpace& exespace,          (4)
            const Kokkos::View<DataType1, Properties1...>& view_from,
            const Kokkos::View<DataType2, Properties2...>& view_to);

チームハンドルを受け入れるオーバーロードセット
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. versionadded:: 4.2

.. code-block:: cpp

  テンプレート <class TeamHandleType, class InputIteratorType, class OutputIteratorType>
  KOKKOS_FUNCTION
  OutputIteratorType copy(const TeamHandleType& teamHandle,                    (5)
                          InputIteratorType first_from,
			  InputIteratorType last_from,
			  OutputIteratorType first_to);

  テンプレート <
    クラス TeamHandleType, class DataType1, class... Properties1,
    クラス DataType2, class... Properties2>
  KOKKOS_FUNCTION
  自動 copy(const TeamHandleType& teamHandle,                                  (6)
            const ::Kokkos::View<DataType1, Properties1...>& view_from,
            ::Kokkos::View<DataType2, Properties2...>& view_to);

パラメータおよび要件
~~~~~~~~~~~~~~~~~~~~~~~~~~~

- ``exespace``: 実行空間インスタンス

- ``teamHandle``: TeamPolicyを使用する際、並列領域内で指定されたチームハンドルインスタンス

- ``label``:  デバッグ目的で内部の並列カーネルに名付けるために使用

  - 1　について、 デフォルト文字列は、: "Kokkos::copy_iterator_api_default"

  - 3　について、 デフォルト文字列は、: "Kokkos::copy_view_api_default"

  - 注意事項: チームハンドルを受け取るオーバーロードは、内部でラベルを使用しません。

- ``first_from, last_from``: コピー元の要素範囲

  - *ランダムアクセスイテレータ*　でなければなりません。

  - 有効な範囲、つまり、``last_from >= first_from`` を表さなければなりません。

  - 必ず　``exespace``　またはチームハンドルに関連付けられた実行空間からアクセス可能である必要があります。

- ``first_to``: コピー先への範囲の初め

  - *ランダムアクセスイテレータでなければならず、かつチームハンドルに関連付けられた実行空間からアクセス可能でなければなりません。

- ``view_from``, ``view_to``: 要素のコピー元およびコピー先である、ソースおよび宛先

  - 必ずランク-1であり、``LayoutLeft``　、  ``LayoutRight``　、または ``LayoutStride``　を持たなければなりません。

  - 必ずランク-1であり、``LayoutLeft``、 ``LayoutRight``、または ``LayoutStride``を持たなければなりません。

返し値
~~~~~~~~~~~~

後の要素がコピーされた *後* の宛先へのイテレータ。
