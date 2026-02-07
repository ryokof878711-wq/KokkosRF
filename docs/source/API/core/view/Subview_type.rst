``Kokkos::Subview``
===================

.. role:: cpp(code)
   :language: cpp

.. _subviewfunc: subview.html

.. |subviewfunc| replace:: ``Kokkos::subview()``

ヘッダーファイル: ``Kokkos_Core.hpp``

ディスクリプション
-----------

|subviewfunc|_　関数を指定された引数で呼び出した際に返される型の演鐸を行うエイリアステンプレート。

インターフェイス
---------

.. code-block:: cpp

   テンプレート　<class ViewType, class... Args>
   using Subview = IMPL_DETAIL; を使用　// ソースビューの特性からサブビューのタイプを演鐸します。

 ``Kokkos::subview(ViewType view_arg, Args .... args)``　の結果の型。

必要要件
------------

以下を必要とします:

- ``ViewType`` は、 ``Kokkos::View``　の仕様です。

- ``Args...``  は、 |subviewfunc|_ で定義されているスライス指定子です。

- ``sizeof... (Args) == ViewType::rank()``.


例
--------

.. code-block:: cpp

   view_type = Kokkos::View<double ***[5]>;　を使用
   view_type a("A",N0,N1,N2);

   構造体 subViewHolder {
   Kokkos::Subview<view_type,
                   std::pair<int,int>,
                   int,
                   decltype(Kokkos::ALL),
                   int> s;
   } subViewHolder;

   subViewHolder.s  = Kokkos::subview(a,
                                      std::pair<int,int>(3,15),
                                      5,
                                      Kokkos::ALL,
                                      3);
