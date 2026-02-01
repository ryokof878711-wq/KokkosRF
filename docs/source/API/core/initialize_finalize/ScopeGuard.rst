``ScopeGuard``
==============

.. role:: cpp(code)
   :language: cpp

ヘッダー　``<Kokkos_Core.hpp>``　に定義。

使用例
-----

.. code-block:: cpp

    Kokkos::ScopeGuard guard(argc, argv);
    Kokkos::ScopeGuard guard(Kokkos::InitializationSettings()  // (since 3.7)
                                .set_map_device_id_by("random")
                                .set_num_threads(1));


``ScopeGuard`` は、`RAII <https://en.cppreference.com/w/cpp/language/raii>`_ を使用して Kokkos を初期化および最終処理するためのクラスです。
それは、 コンストラクタ内の提供された引数およびデストラクタ内の　`Kokkos::finalize <finalize.html#kokkosfinalize>`_ で`Kokkos::initialize <initialize.html#kokkosinitialize>`_　を呼び出します。
正しい使用法のためには、Kokkosへの呼び出しを発行する前に、必ず``ScopeGuard``の命名済みインスタンスを作成する必要があります。


.. warning:: バージョン 3.7におけるビヘイビアの変更 (以下参照)。 ``ScopeGuard`` は、:cpp:func:`is_initialized()` または :cpp:func:`is_finalized()` のいずれかが ``true`` を返した場合に中断します。

ディスクリプション
-----------

.. cpp:class:: ScopeGuard

     そのライフサイクルの開始時に　``Kokkos::initialize``　を呼び出し、終了時に、``Kokkos::finalize``　を呼び出すクラス。

    .. rubric:: コンストラクタ

    .. cpp:function:: ScopeGuard(int& argc, char* argv[]);

       :param argc: コマンドライン引数の数。
       :param argv: コマンドライン引数を格納するヌル終端文字列への文字ポインタの配列。

       .. warning:: 3.7まで有効。

    .. cpp:function:: ScopeGuard(InitArguments const& arguments = InitArguments());

       :param arguments: 有効な初期化引数を持つ　``struct`` オブジェクト。

       .. warning:: 3.7まで有効。

    .. cpp:function:: テンプレート <class... Args> ScopeGuard(Args&&... args);

        :param args: `Kokkos::initialize <initialize.html#kokkosinitialize>`_　に引き渡す引数。

	可能な実装:

	.. code-block:: cpp

	   テンプレート <class... Args> ScopeGuard(Args&&... args){ initialize(std::forward<Args>(args)...); }

    .. cpp:function:: ~ScopeGuard();

       デストラクタ

       可能な実装:

       .. code-block:: cpp

	  ~ScopeGuard() { finalize(); }

    .. cpp:function:: ScopeGuard(ScopeGuard const&) = delete;

       コピーコンストラクタ

    .. cpp:function:: ScopeGuard(ScopeGuard&&) = delete;

       移動コンストラクタ

    .. cpp:function:: ScopeGuard& operator=(ScopeGuard const&) = delete;

       コピー代入演算子

    .. cpp:function:: ScopeGuard& operator=(ScopeGuard&&) = delete;

       移動代入演算子

注意事項
-----

- コンストラクタでは、すべてのパラメータが内部で呼び出される　``Kokkos::initialize``　に渡される。
  詳細については、 `Kokkos::initialize <initialize.html#kokkosinitialize>`_ 参照。


- Kokkos バージョン 3.7以来、 ``ScopeGuard`` は与えられた引数を無条件に`Kokkos::initialize <initialize.html#kokkosinitialize>`_　
  に転送し、それは、それらが同じ必須条件を持つことを
  意味します。  バージョン3.7まで、 ``ScopeGuard``は、
 ``Kokkos::is_initialized()`` が 　``false``　であった場合にのみ、そのコンストラクタにおいて ``Kokkos::initialize`` を呼び出しており、
 それは、そのコンストラクタにおいて、
　``Kokkos::initialize``　を呼び出した場合にのみ、``Kokkos::finalize``　をそのデストラクタにおいて呼び出していました。

  古いビヘイビアについてのサポートを停止しましが。それが実際に必要であるとお考えでしたら、そう考えて構いません:

  .. code-block:: cpp

      auto guard = std::unique_ptr<Kokkos::ScopeGuard>(
	  Kokkos::is_initialized() ? new Kokkos::ScopeGuard() : nullptr);

  又は

  .. code-block:: cpp

      auto guard = Kokkos::is_initialized() ? std::make_optional<Kokkos::ScopeGuard>()
					  : std::nullopt;

  C++17を用いて。  これは、Kokkos　バージョンに関わらず、機能します。

例
~~~~~~~

.. code-block:: cpp

    int main(int argc, char* argv[]) {
        Kokkos::ScopeGuard guard(argc, argv);
        Kokkos::View<double*> my_view("my_view", 10);
        // my_view destructor called before Kokkos::finalize
        // ScopeGuard destructor called, calls Kokkos::finalize
    }


以下も参照
~~~~~~~~

`Kokkos::initialize <initialize.html#kokkosinitialize>`_, `Kokkos::finalize <finalize.html#kokkosfinalize>`_
