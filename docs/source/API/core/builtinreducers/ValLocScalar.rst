``ValLocScalar``
================

.. role::cpp(code)
    :language: cpp

:cpp:struct:`ValLocScalar` は、 **value** およびそれに対応する　 **location** (インデックス)  
適切な単一単位として、格納するクラステンプレートです。
それは、主に :cpp:class:`MinLoc` および :cpp:class:`MaxLoc` 組み込みリデューサーを使用する　
:cpp:func:`parallel_reduce` 演算の結果を保持するように設計されています。

一般的には、正確なテンプレートパラメータが使用されることを確証するために、`　
リデューサーの　``::value_type`　例えば、``MaxLoc<Scalar,Index,Space>::value_type``）
を使用して、この型を取得することをお勧めします。

ヘッダーファイル: ``<Kokkos_Core.hpp>``

使用例
-----

.. code-block:: cpp

    MaxLoc<T,I,S>::value_type result;
    parallel_reduce(N,Functor,MaxLoc<T,I,S>(result));
    T resultValue = result.val;
    I resultIndex = result.loc;

インターフェイス
---------

.. cpp:struct::  template<class Scalar, class Index> ValLocScalar

   :tparam Scalar: 還元されている値のデータ型 (例えば、 ``double``、 ``int``)。

   :tparam Index: 保存先またはイテレーションインデックスのデータ型 (例えば、 ``int``、 ``long long``).

   .. rubric:: データメンバー

   .. cpp:var:: スカラー変数

      還元値。

   .. cpp:var:: インデックス保存先

      還元値の保存先（イテレーションインデックス）
