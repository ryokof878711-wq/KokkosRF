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
``num_threads``                ``int``               Number of threads to use with the host parallel backend.  Must be greater than zero.
``device_id``                  ``int``               Device to use with the device parallel backend.  Valid IDs are zero to number of GPU(s) available for execution minus one.
``map_device_id_by``           ``std::string``       Strategy to select a device automatically from the GPUs available for execution. Must be either ``"mpi_rank"`` for round-robin assignment based on the local MPI rank or ``"random"``.
``disable_warnings``           ``bool``              Whether to disable warning messages.
``print_configuration``        ``bool``              Whether to print the configuration after initialization.
``tune_internals``             ``bool``              Whether to allow autotuning internals instead of using heuristics.
``tools_libs``                 ``std::string``       Which tool dynamic library to load. Must either be the full path to library or the name of library if the path is present in the runtime library search path (e.g. ``LD_LIBRARY_PATH``)
``tools_help``                 ``bool``              Query the loaded tool for its command-line options support.
``tools_args``                 ``std::string``       Options to pass to the loaded tool as command-line arguments.
=======================        ==================    ===========

Example
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

See also
~~~~~~~~

* `Kokkos::initialize <initialize.html#kokkosinitialize>`_: initializes the Kokkos execution environment
