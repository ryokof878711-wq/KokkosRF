``view_alloc``
==============

.. role:: cpp(code)
   :language: cpp

ヘッダーファイル: ``<Kokkos_Core.hpp>``

使用例
-----

.. code-block:: cpp

    Kokkos::view_alloc(exec_space, Kokkos::WithoutInitializing, "ViewString");
    Kokkos::view_wrap(pointer_to_wrapping_memory);

引数リストからビュー代入パラメータバンドルを、作成します。 有効な引数リストメンバーは以下の通りです:

*  ``C``-string または ``std::string``　としてのラベル。

*  ``View::memory_space`` 型のメモリ空間インスタンス。

*  ``View::memory_space``　にアクセス可能な実行スペースインスタンス。

*  要素の初期化および破棄を回避するための　``Kokkos::WithoutInitializing``。

* 　ホスト上で要素の初期化と破棄を順次実行するための　``Kokkos::SequentialHostInit`` (　4.4.01より　)。

* 　メモリアライメントのためのパディング次元への割り当てを可能にするための　``Kokkos::AllowPadding`` 。

* 　そのポインタをラップする、管理対象外ビューを作成するためのポインタ。


ディスクリプション
-----------

.. cpp:function:: template <class... Args> ALLOC_PROP view_alloc(Args const&... args)

   引数リストから、ビュー割り当てパラメータバンドルを作成します。

   ``args`` : Cannot contain a pointer to memory.メモリへのポインタを含有不可能。

.. cpp:function:: テンプレート <class... Args> ALLOC_PROP view_wrap(Args const&... args)

   引数リストから、ビュー割り当てパラメータバンドルを作成します。

   ``args`` : メモリへのポインタでのみあることが可能。


.. cpp:type:: ALLOC_PROP

   :cpp:type:`ALLOC_PROP` は、 :cpp:func:`view_alloc`
   および :cpp:func:`view_wrap`　により返される、特別な、スペルが不可能な実装定義型です。それは、ビューラベル、メモリ空間インスタンス、実行空間インスタンス、メモリを初期化するかどうか、パディングを可能にするかどうかを含む、アロケータパラメータのバンドル、および生ポインタ値　（ラッピングされた管理対象外ビューについて）を表します。 

.. cpp:type:: WithoutInitializing

   :cpp:type:`WithoutInitializing` は、関連実行空間における `View` 要素　のデフォルト構築が、必要でない、または成立しない状況における使用を目的とします。 特に、In particular, it may not be viable in situations such as 仮想関数を持つオブジェクトの構築等の状況において、または、デフォルトコンストラクタを持たない要素の `Views` について、 成立しない場合があります。そのような状況においては、このオプションは、オブジェクトの手動インプレース　`new`　構築や要素の手動破棄と組み合わせて使用されることが多いです。

.. cpp:type:: SequentialHostInit

   :cpp:type:`SequentialHostInit` は、　Kokkos　並列領域内で呼び出せるデフォルトコンストラクタやデストラクタを持たない要素を初期化するための使用を目的とします。特に、これは、以下のコンストラクタおよびデストラクタを含みます:

   * メモリの割り当てまたは解放。
   * 管理下にある `Kokkos::View` オブジェクトの作成または破棄。
   * call Kokkos 並列演算の呼び出し。

   この割り当てオプションを使用する場合、`View`　コンストラクタ/デストラクタは、ホスト上でシリアルループ内で、要素を作成/破棄します。

   .. 警告::

     `SequentialHostInit` は、メモリ空間としての　`View`s with `HostSpace`、 `SharedSpace`、
     または `SharedHostPinnedSpace` 等、ホストアクセス可能な　`View`　作成時にのみ使用可能です。

   .. versionadded:: 4.4.01
