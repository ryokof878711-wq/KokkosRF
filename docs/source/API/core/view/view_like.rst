 View-like 型
===============

View-like 型は、 インターフェースの観点から、`Kokkos::View <view.html>`__　のように振る舞うクラステンプレートの集合として大まかに定義されます。 この意味について、まだ完全な正式な定義は存在しませんが、それは、ユーザーがこのリストに追加する方法は存在せず、新たに追加されたクラスが　View　のようなものに対して動作するKokkosの機能によって認識されることを意味します。 Kokkos では、これらのクラステンプレートは、　View　のようなものと考えられます:

* `Kokkos::View <view.html>`_
* `Kokkos::DynRankView <../../containers/DynRankView.html>`_
* `Kokkos::OffsetView <../../containers/Offset-View.html>`_
* `Kokkos::DynamicView <../../containers/DynamicView.html>`_

特に、 `Kokkos::DualView <../../containers/DualView.html>`_ and `Kokkos::ScatterView <../../containers/ScatterView.html>`_ は、  このカテゴリーには、含まれ　**ません**。
