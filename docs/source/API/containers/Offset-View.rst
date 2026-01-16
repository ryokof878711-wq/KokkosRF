
``OffsetView``
==============

ヘッダーファイル: ``<Kokkos_OffsetView.hpp>``

配列の添字が0以外から始まる場合、``OffsetView``　を使用できます。

.. warning::

   OffsetView は、Experimental namespace　にあります。


構築
------------

OffsetView　にはラベルと、少なくとも1つのディメンションが必要です。実行時範囲のみがサポートされますが、それ他では　OffsetView　のセマンティクスはViewのそれと同様です。 

.. code-block:: cpp

   const size_t min0 = ...;
   const size_t max0 = ...;
   const size_t min1 = ...;
   const size_t max1 = ...;
   const size_t min2 = ...;
   const size_t max2 = ...;

   OffsetView<int***> a("someLabel", {min0, max0}, {min1, max1},{min2, max2});

レイアウトからの構築も認められます。

.. code-block:: cpp

    OffsetView<int***> a("someLabel", LayoutLeft, {min0, min1, min2});

OffsetView　は、同じ基底型を持つ　View　から作成することもできます。 ビューには既に範囲が設定されているため、開始インデックスをコンストラクタに渡す必要があります。

.. code-block:: cpp

   View<double**> b("somelabel", 10, 20);
   Array<int64_t, 2> begins = {-10, -20};
   OffsetView<double**> ov(b, begins);

OffsetView ov は b と同じ範囲を持ち、[-10,-1] および [-20,-11] からインデックス付けされる必要があります。

配列の代わりに　std::initializer_list　を使用することもできます。

.. code-block:: cpp

    OffsetView<double**> ov(b, {-10, -20});

インターフェイス
---------

開始インデックスは配列として取得可能です。 各ランクについて、イテレーションの開始位置と終了位置を特定できます。

.. code-block:: cpp

   OffsetView<int***> ov("someLabel", {-1,1}, {-2,2}, {-3,3});
    Array<int64_t, 3> a = ov.begins();

   const int64_t begin0 = ov.begin(0);
   const int64_t end0= ov.end(0);

以下にご注意ください

.. code-block:: cpp

   OffsetView::end(const size_t i)は、

正当なインデックスではない値を返します:  それは、既定の次元 i に対する最大許容インデックスを正に 1 つ上回る値です。

 サブビューはサポートされており、OffsetViewのサブビューを取得した結果は、別のOffsetViewです。 サブビュー関数に ALL() が渡された場合、そのランクのオフセットは保持され、そうでない場合は破棄されます。

.. code-block:: cpp

   OffsetView<Scalar***> sliceMe("offsetToSlice", {-10,20}, {-20,30}, {-30,40});
   auto offsetSubview = subview(sliceMe,0, Kokkos::ALL(), std::make_pair(-30, -21));

   ASSERT_EQ(offsetSubview.Rank, 2);
   ASSERT_EQ(offsetSubview.begin(0) , -20);
   ASSERT_EQ(offsetSubview.end(0) , 31);
   ASSERT_EQ(offsetSubview.begin(1) , 0);
   ASSERT_EQ(offsetSubview.end(1) , 9);

以下のディープコピーもサポートされています: 定数値から　OffsetView へ; 互換性のある　OffsetView　から別のOffsetViewへ; from a compatible View to an OffsetView; 互換性のある　View から　OffsetViewへ; 互換性のある OffsetView から Viewへ。

同じラベルを持つ互換性のあるビューは、view()メソッドから取得されます。

.. code-block:: cpp

   OffsetView<int***> ov("someLabel", {-1,1}, {-2,2}, {-3,3});
   View<int***> v = ov.view();

ビューから　OffsetView　へのコピーコンストラクタと代入演算子も提供されています。

等価演算子　"==" および "!="　が定義されています。 OffsetView と View を前提とすれば、それらは 2 つの View が同等であるのと同様の意味で同等です。　同様に、れらの開始位置も一致する場合、2つのOffsetView　は、同様の意味で同等です。

ミラーもサポートされています。
