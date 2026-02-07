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

      Constructs a new Timer instance and immediately starts the clock.

      The timer is initialized with the current time, marking the beginning of
      the measurement period.

   .. cpp:function:: double seconds() const

      :returns: The number of seconds that have elapsed since the timer was
        started or last reset.

   .. cpp:function:: void reset()

      Resets the timer, setting the start time to the current time.

      This function effectively restarts the measurement period without
      creating a new Timer object.

   .. cpp:function:: Timer(Timer const&&) = delete
   .. cpp:function:: Timer(Timer&&) = delete

      The Timer class is neither copy-constructible nor copy-assignable.


例
-------

.. code-block:: cpp

    Timer timer;
    // ...
    double time1 = timer.seconds();
    timer.reset();
    // ...
    ダブルタイム2 = timer.seconds();
