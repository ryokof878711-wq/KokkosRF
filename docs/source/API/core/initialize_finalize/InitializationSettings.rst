``InitializationSettings``
==========================

.. role:: cpp(code)
   :language: cpp

ヘッダー　``<KokkosCore.cpp>``　に定義。

使用例:

.. code-block:: cpp

    自動設定 = Kokkos::InitializationSettings()
                    .set_num_threads(8)
                    .set_device_id(0)
                    .set_disable_warnings(false);

.. versionadded:: 3.7
   ``InitializationSettings``  `Kokkos::initialize() <initialize.html#kokkosinitialize>`_　
の2つのパラメータ形式　(``argc`` および ``argv``) を呼び出すことなく、 
Kokkosをプログラムで初期化する設定を定義するために使用可能なクラスです。
   それは、`Kokkos::InitArguments <InitArguments.html#kokkosInitArguments>`_ structure　の代わりに導入されました。

インターフェイス
---------

.. cpp:class:: InitializationSettings

   .. cpp:function:: InitializationSettings();

      どの設定に対しても値を一切含まない、新しいオブジェクトを構築します。

   .. cpp:function:: InitializationSettings(InitArguments const& 引数);

      **DEPRECATED** は、非推奨の構造体を新しいオブジェクトに変換します。構造体のデータメンバーでデフォルト値と等しいものは、設定されていないものとみなされます。 ``PARAMETER-NAME``を、以下の表で定義される　``PARAMETER-TYPE``　の有効な設定とします。

   .. cpp:function:: InitializationSettings& set_PARAMETER_NAME(PARAMETER_TYPE value);

      ``PARAMETER_NAME``　設定の内容を　``value``で置き換え、オブジェクトへの参照を返します。 ``value`` は、 ``PARAMETER_NAME``　について有効な値である必要があります。

   .. cpp:function:: bool has_PARAMETER_NAME() const;

      オブジェクトが、``PARAMETER_NAME``　設定の値を含むかどうかを確認します。 それが値を含む場合は、 ``true`` を返し、そうでない場合には、 ``false`` を返します。

   .. cpp:function:: PARAMETER_TYPE get_PARAMETER_NAME() const;

      ``PARAMETER_NAME`` 設定に含まれる値にアクセスします。 オブジェクトが、``PARAMETER_NAME``　設定の値を含まない場合には、ビヘイビアは定義されません。

以下の表は、利用可能な設定を概説しています。

=======================        ==================    ===========
**PARAMETER_NAME**             **PARAMETER_TYPE**    ディスクリプション
=======================        ==================    ===========
``num_threads``                ``int``               ホスト並列バックエンドで使用するスレッド数。  0よりも大であることが必須。
``device_id``                  ``int``               デバイス並列バックエンドで使用するデバイス。 有効な ID は、 0 から、実行に利用可能な GPU の数から 1 を引いた値までです。
``map_device_id_by``           ``std::string``       実行可能なGPUからデバイスを自動的に選択するストラテジー。ローカルMPIランクに基づくラウンドロビン割り当てについての　``"mpi_rank"``、または　``"random"``　のいずれかである必要があります。
``disable_warnings``           ``bool``              警告メッセージを無効にできるかどうか。
``print_configuration``        ``bool``              初期化後に設定を印刷するかどうか。
``tune_internals``             ``bool``              ヒューリスティックを使用する代わりに、内部の自動調整を許可するかどうか。
``tools_libs``                 ``std::string``       どのツールのダイナミックライブラリをロードするか。ライブラリの完全なパス、またはランタイムライブラリ検索パス（例: ``LD_LIBRARY_PATH``）にパスが存在する場合のライブラリ名である必要があります。
``tools_help``                 ``bool``              読み込まれたツールのコマンドラインオプションサポートを照会。
``tools_args``                 ``std::string``       コマンドライン引数としてロードされたツールに渡すオプション。
=======================        ==================    ===========

例
~~~~~~~

.. code-block:: cpp

    #include <Kokkos_Core.hpp>

    int main() {
        Kokkos::initialize(Kokkos::InitializationSettings()
                                .set_print_configuration(true)
                                .set_map_device_id_by("random")
                                .set_num_threads(1));
        // ...
        Kokkos::finalize();
    }

以下も参照
~~~~~~~~

* `Kokkos::initialize <initialize.html#kokkosinitialize>`_:　は、Kokkos 実行環境を初期化します。
