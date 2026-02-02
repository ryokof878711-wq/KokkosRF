``initialize``
==============

.. role::cpp(code)
    :language: cpp

ヘッダー ``<Kokkos_Core.hpp>``　に定義。

使用例: 

.. code-block:: cpp

    Kokkos::initialize(argc, argv);
    Kokkos::initialize(Kokkos::InitializationSettings()  // (since 3.7)
                           .set_disable_warnings(true)
                           .set_num_threads(8)
                           .set_map_device_id_by("random"));

実行環境を初期化します。
この関数は、他の　Kokkos API　関数やコンストラクタよりも先に呼び出す必要があります。
:cpp:func:`Kokkos::is_initialized() <is_initialized()>` または
:cpp:func:`Kokkos::is_finalized() <is_finalized()>`等の、少数の例外もあります。

Kokkos は最大で1回のみ初期化が可能です; その後の呼び出しは、誤っています。

関数には、2つのオーバーロードがあります。
最初の関数は、プログラムが実行される環境からプログラムに渡されるコマンドライン引数に
対応する、``main()`` と同じ2つのパラメータを取ります。Kokkos は、
認識するフラグの引数を解析します。フラグが検出されるたびに、
それは``argv``から削除され、
``argc``が1減算される。
2番目の関数は、 `Kokkos::InitializationSettings <InitializationSettings.html#kokkosInitializationSettings>`_ class オブジェクトを選択し、
引数のプログラムによる制御を可能にします。
`Kokkos::InitializationSettings <InitializationSettings.html#kokkosInitializationSettings>`_ は、暗示的に
バージョン3.7において非推奨である ``Kokkos::InitArguments`` :sup:`　から暗示的に構築可能です。

インターフェイス
---------

.. code-block:: cpp

    Kokkos::initialize(int& argc, char* argv[]);                 //             (1)
    Kokkos::initialize(InitArguments const& arguments);          // (until 3.7) (2)
    Kokkos::initialize(InitializationSettings const& settings);  // (since 3.7) (3)
    
パラメータ
~~~~~~~~~~

* ``argc``: 非負の値で、プログラムに渡された
  コマンドライン引数の数を表します。
* ``argv``: ``argc + 1``　個のポインタの配列の最初の要素へのポインタで、
  その最後のものは、ヌルであり、その1つ前のものは、もし存在するのであれば、 プログラムに渡された引数を表す
　ヌル終端マルチバイト文字列を
  指します。
* ``arguments``: (バージョン3.7より非推奨) C-スタイル ``struct`` オブジェクトは、
  後方互換性のため ``Kokkos::InitializationSettings`` に変換されました。
* ``settings``: Kokkos　の初期化を制御する設定を含む
  ``class``　オブジェクト。

必要要件
~~~~~~~~~~~~

* ``Kokkos::finalize`` は、 ``Kokkos::initialize``　の後に呼び出される必要があります。
* ``Kokkos::initialize`` は一般的には、 Kokkos が MPI コンテクスト内で初期化される際に、 ``MPI_Init``　の後に呼び出される必要があります。
* ``Kokkos::initialize`` is called.ユーザーが開始したKokkosオブジェクトは、``Kokkos::initialize``が呼び出されるまで構築できません。
* ``Kokkos::finalize``.``Kokkos::initialize`` は ``Kokkos::finalize`` の呼び出し後に呼び出すことはできません。

セマンティクス
~~~~~~~~~

* ``Kokkos::initialize``　呼出し後に、 :cpp:func:`Kokkos::is_initialized() <is_initialized()>` は、true　を返す必要があります。

例
~~~~~~~

.. code-block:: cpp

    #include <Kokkos_Core.hpp>

    int main(int argc, char* argv[]) {
        Kokkos::initialize(argc, argv);
        {  // my_view デストラクタが Kokkos::finalize　の前に呼ばれることを確認するための範囲
            Kokkos::View<double*> my_view("my_view", 10);
        }  // my_view　の範囲は、ここで終了。 
        Kokkos::finalize();
    }    

以下も参照
--------

* `Kokkos::InitializationSettings <InitializationSettings.html#kokkosInitializationSettings>`_
* `Kokkos::ScopeGuard <ScopeGuard.html#kokkosScopeGuard>`_
