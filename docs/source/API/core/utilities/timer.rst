``Timer``
=========

.. role:: cpp(code)
    :language: cpp

``<Kokkos_Core.hpp>``　から含まれる、ヘッダー ``<Kokkos_Timer.hpp>``　に定義。
 
使用例
-----

.. code-block:: cpp

    Kokkos::Timer timer;
    ダブルタイム = タイマー.seconds();
    timer.reset();

インターフェイス
---------

.. cpp:class:: タイマー

   経過時間を計測するための高精度タイマークラスです。

   .. note::

       　このクラスは「手っ取り早く大まかな」タイミング測定だけでなく、
        タイミング測定を「常時有効」とする状況も想定しています。
    　　 本格的なパフォーマンスプロファイリングには、
        **Kokkos Tools** APIの使用が推奨されます。
        Kokkos Toolsは、アプリケーションを変更することなく、
        実行時にプロファイリングを有効化または無効化する柔軟性を提供し
        明示的なタイマーオブジェクトでコードを煩雑にする
        必要性を回避します。

   .. cpp:function:: Timer()

      新しいTimerインスタンスを構築し、直ちにタイマーを開始します。

      タイマーは、現在時刻で初期化され、
      測定期間の開始が記録されます。

   .. cpp:function:: double seconds() const

      :returns: タイマーが開始または最後にリセットされてから
        経過した秒数。

   .. cpp:function:: void reset()

      タイマーをリセットし、開始時刻を現在の時刻に設定します。

      この関数は、新しいタイマーオブジェクトを作成せずに、
　　　測定期間を効果的に再起動します

   .. cpp:function:: Timer(Timer const&&) = 削除
   .. cpp:function:: Timer(Timer&&) = 削除

      タイマークラスは、コピーコンストラクタもコピー代入もできません。


例
-------

.. code-block:: cpp

    タイマー　タイマー;
    // ...
    ダブルタイム1 = timer.seconds();
    timer.reset();
    // ...
    ダブルタイム2 = timer.seconds();
