.. _kokkos_if_on_host_device_macros:

=================================================
``KOKKOS_IF_ON_HOST`` および ``KOKKOS_IF_ON_DEVICE``
=================================================

.. role:: cpp(code)
   :language: cpp

概要
========
``KOKKOS_IF_ON_HOST`` 及び``KOKKOS_IF_ON_DEVICE`` マクロは、 
3.6 で導入された関数のようなマクロのペアであり、
**single ``KOKKOS_FUNCTION`` body** 内で、**portable　conditional compilation**を可能にします。
それらにより関数がホスト（CPU）上で実行されるか、
デバイス（GPUなど）上で実行されるかに基づいて、コンパイルおよび実行されるコードの選択が可能になります。 
これらのマクロは、#ifdef ``__CUDA_ARCH__`` のような
移植性のないプリプロセッサ構文に代わる手段を提供します。

モチベーション
==========
従来のプリプロセッサ指令（例：``#ifdef __CUDA_ARCH__``）は、ホストコードとデバイスコードが
別々のパスでコンパイルされる分割コンパイルモデルに依存しています。
このモデルは一部のコンパイラ（　``nvcc``　など）でサポートされていますが、
OpenACC　や　OpenMPTarget　をサポートするコンパイラ等、GPUアクセラレーションコード向けのその他の現代的なコンパイラでは、
ホストコードとデバイスコードの両方を、
単一のパスでコンパイルする統一されたコンパイル手法を採用しています。
その結果、バックエンド固有のマクロで記述されたコードは、
異なるコンパイラやプログラミングモデル間において、移植性がありません。

``KOKKOS_IF_ON_HOST`` および　``KOKKOS_IF_ON_DEVICE`` マクロは、
コンパイラが単一のパス内でコードを条件付きでコンパイルできるように
この移植性の問題を解決し、単一のコードベースをより幅広いコンパイラやバックエンドで
使用できるようにします。

使用例
=====
これらのマクロは、``KOKKOS_FUNCTION`` で装飾された関数内で使用されるように
設計されています。それらは単一の引数を受け付けますが、
それは二重括弧で囲まれたコードブロックです。 マクロの括弧内のコードは、
指定されたアーキテクチャでのみ、コンパイルおよび実行されます。


シグネチャ
---------

.. code-block:: cpp

    KOKKOS_IF_ON_HOST(( /* code to be compiled on the host */ ))
    KOKKOS_IF_ON_DEVICE(( /* code to be compiled on the device */ ))


例: ホスト/デバイス オーバーロード
--------------------------------

一般的なユースケースとして、ホスト側とデバイス側での実行用に、関数の異なる実装を提供する
ことが挙げられます。

.. code-block:: cpp

    struct MyS { int i; };

    KOKKOS_FUNCTION MyS MakeStruct() {
      // This return statement is only compiled for the host target.
      KOKKOS_IF_ON_HOST((
        return MyS{0};
      ))

      // This return statement is only compiled for the device target.
      KOKKOS_IF_ON_DEVICE((
        return MyS{1};
      ))
    }

重要考慮事項
========================

範囲
-----

各 ``KOKKOS_IF_ON_*`` マクロは、新たな範囲を導入します。マクロの括弧内で宣言された変数は、
そのスコープ内でローカルとなり、
その範囲外からはアクセスできません。

.. code-block:: cpp

    KOKKOS_IF_ON_HOST((
      int x = 0; // 'x' is only visible within this scope
      std::cout << x << '\n';
    )) // The scope of 'x' ends here.

``constexpr`` コンテクスト
---------------------

これらのマクロは、``constexpr``　を必要とするコンテキストでは使用できません。
(定数式).

ベストプラクティス
--------------

**可能な限りこれらのマクロの使用は避けてください。**

``KOKKOS_IF_ON_HOST`` and ``KOKKOS_IF_ON_DEVICE`` のコードの差別化の手段としては、
あくまで　**最終手段**　と考えるべきです。 Kokkos の主な目標は、
統一されたコードベースを通じて高い移植性を実現することである。 これらのマクロに依存すると、
ホスト/デバイス固有のロジックを導入することで、この目標の達成を妨げる可能性があります。

これらのマクロ使用前に、実行空間に対する　**部分テンプレート特化**　または
Kokkos　の組み込み機能の使用といった代替アプローチを検討しますが、
それは、すべてのバックエンド間で移植可能になるように設計されています。
これらのマクロの使用は、ホストとデバイスのAPI間に根本的な差異が存在し、
I/O操作や特定のバックエンド固有命令など、別々のコードパスが必要となる状況に
限定するべきです。
