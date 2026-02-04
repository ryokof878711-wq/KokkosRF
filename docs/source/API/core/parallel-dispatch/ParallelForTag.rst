``ParallelForTag``
==================

.. role::cpp(code)
    :language: cpp

ヘッダーファイル: ``<Kokkos_ExecPolicy.hpp>``

.. _parallelFor: ../parallel-dispatch/parallel_for.html

.. |parallelFor| replace:: :cpp:func:`parallel_for`

チーム規模の要求対象となるファンクターを示すための、チーム規模計算機能で使用されるタグは、|parallelFor|_に使用されています。  

使用例
-----

.. code-block:: cpp

    PolicyType = Kokkos::TeamPolicy<>　を使用; 
    PolicyType policy;
    int recommended_team_size = policy.team_size_recommended(
        Functor, Kokkos::ParallelForTag());

シノプシス 
--------

.. code-block:: cpp

    構造体　ParallelForTag{};

パブリッククラスメンバー
--------------------

無し

Typedefs
~~~~~~~~
   
無し

コンストラクタ
~~~~~~~~~~~~
 
デフォルト

関数
~~~~~~~~~

無し
