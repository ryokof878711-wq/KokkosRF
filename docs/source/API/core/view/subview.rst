``subview``
===========

.. role:: cpp(code)
    :language: cpp

ヘッダーファイル: ``<Kokkos_Core.hpp>``

使用例
-----

.. code-block:: cpp

    auto s = subview(view,std::pair<int,int>(5,191),Kokkos::ALL,1);

別の ``Kokkos::View``　のサブセットを表す ``Kokkos::View`` を作成します。


.. _KokkosAll: ../utilities/all.html#kokkosall

.. |KokkosAll| replace:: :cpp:func:`Kokkos::ALL`

ディスクリプション
-----------

.. cpp:function:: template<class ViewType, class ... Args> IMPL_DETAIL subview(const ViewType& v, Args ... args)

　``args...``　が特定する　``v``　のサブセットを表す、新たな  ``Kokkos::View`` ``s`` を返します。The return type of subview　の返す型は、 実装の詳細であり、  ``Args...``　における型により決定されます。

   .. rubric:: サブセット選択:

   *  ``args...`` におけるすべての整数引数に対して、返されたビューのランクは、``v``のランクより一つ小さく、 
``s`` により参照された値は、``v`` へのインデックス付けの間に、
対応する位置で整数引数を使用することに関連する値に対応します。  

   *  ``r``\ th 引数として KokkosAll_ を渡すことは、　``r``\ th  引数として pair<ptrdiff_t,ptrdiff_t>(0,v.extent(r)) を渡すことに等しいです。

   *  ``r``\ th 引数 ``arg_r`` が``s.extent(d) = arg_r.second-arg_r.first``\ よりも、 引数リストにある　``d``\ th 範囲 (\ ``std::pair``\ ``Kokkos::pair`` または、|KokkosAll|_ )　である場合には、 
      ``s``　の次元　``d``　は、　``v``の次元　``r``　の範囲 ``[arg_r.first,arg_r.second)``　を参照します。

   .. rubric:: 制約:

   * ``sizeof...(args)`` は、　``ViewType::rank``　に等しいです。

   * 有効な引数は、以下の型です:

     - 真である　``std::pair<iType,iType>`` with ``std::is_integral<iType>::value`` 。

     - 真である　``Kokkos::pair<iType,iType>`` with ``std::is_integral<iType>::value`` 。

     - 真である　``iType`` with ``std::is_integral<iType>::value`` 。

     - ``std::remove_const_t< decltype(``\ |KokkosAll|_ ``)>``

   *  ``r``\ th 引数 ``arg_r`` がis of type ``std::pair<iType,iType>`` または ``Kokkos::pair<iType,iType>`` の型である場合には、以下を満たす必要があります:

     - ``arg_r.first >= 0``

     - ``arg_r.second <= v.extent(r)``

     - ``arg_r.first <= arg_r.second``

   *  ``r``\ th 引数 ``arg_r`` が整数である場合には、以下を満たす必要があります:

     - ``arg_r >= 0``

     - ``arg_r < v.extent(r)``

例
--------

.. code-block:: cpp

    Kokkos::View<double***[5]> a("A",N0,N1,N2);

    auto s  = Kokkos::subview(a,
                              std::pair<int,int>(3,15),
			      5,
			      Kokkos::ALL,
			      Kokkos::ALL);
    for(int i0 = 0; i0 < s.extent(0); i0++)
    for(int i1 = 0; i1 < s.extent(1); i1++)
    for(int i2 = 0; i2 < s.extent(2); i2++) {
        assert(s(i0,i1,i2) == a(i0+3,5,i1,i2));
    }

    auto s3415 = Kokkos::subview(a,3,4,1,5);
    assert(s3415() == a(3,4,1,5));
