乱数
=============

.. role:: cpp(code)
    :language: cpp

Rand
----

ヘッダーファイル: ``<Kokkos_Core.hpp>``, ``<Kokkos_Random.hpp>``

.. code-block:: cpp

   template<class Generator>
   struct rand<Generator, gen_data_type>
   {
     KOKKOS_INLINE_FUNCTION
     static gen_func_type max(){
       return type_value;
     }

     KOKKOS_INLINE_FUNCTION
     static gen_func_type draw(Generator& gen){
       return gen_data_type((gen.rand()&gen_return_value)
     }

     KOKKOS_INLINE_FUNCTION
     static gen_func_type draw(Generator& gen,
                               const gen_data_type& range){
       return gen_data_type((gen.rand(range));
     }

     KOKKOS_INLINE_FUNCTION
     static gen_func_type draw(Generator& gen,
                               const gen_data_type& start,
			       const gen_data_type& end){
       return gen_data_type(gen.rand(start,end));
     }

``gen_data_type``、 ``gen_func_type``および``type_value``は、機能仕様です。
``Kokkos::`` namespace.この一覧にあるすべての関数およびクラスは、 Kokkos:: ``Kokkos::`` の一部です

+-------------------+-------------------+---------------------------+-----------------------+
| gen_data_type     | gen_func_type     | type_value                | gen_return_value      |
+===================+===================+===========================+=======================+
| char              | short             | 127                       | (&0xff+256)%256       |
+-------------------+-------------------+---------------------------+-----------------------+
| short             | short             | 32767                     | (&0xffff+65536)%32768 |
+-------------------+-------------------+---------------------------+-----------------------+
| int               | int               | MAX_RAND                  |  ?                    |
+-------------------+-------------------+---------------------------+-----------------------+
| uint              | uint              | MAX_URAND                 |  ?                    |
+-------------------+-------------------+---------------------------+-----------------------+
| long              | long              | MAX_RAND or MAX_RAND64    |  ?                    |
+-------------------+-------------------+---------------------------+-----------------------+
| ulong             | ulong             | MAX_RAND or MAX_RAND64    |  ?                    |
+-------------------+-------------------+---------------------------+-----------------------+
| long long         | long long         | MAX_RAND64                |  ?                    |
+-------------------+-------------------+---------------------------+-----------------------+
| ulong long        | ulong long        | MAX_URAND64               |  ?                    |
+-------------------+-------------------+---------------------------+-----------------------+
| float             | float             | 1.0f                      |  ?                    |
+-------------------+-------------------+---------------------------+-----------------------+
| double            | double            | 1.0                       |  ?                    |
+-------------------+-------------------+---------------------------+-----------------------+
| complex<float>    | complex<float>    | 1.0,1.0                   |  ?                    |
+-------------------+-------------------+---------------------------+-----------------------+
| complex<double>   | complex<double>   | 1.0,1.0                   |  ?                    |
+-------------------+-------------------+---------------------------+-----------------------+

XorShift関数の値の最大値が、以下の列挙型によって与えられる場合。

* enum {MAX_URAND = 0xffffffffU};
* enum {MAX_URAND64 = 0xffffffffffffffffULL-1};
* enum {MAX_RAND = static_cast<int>(0xffffffffU/2)};
* enum {MAX_RAND64 = static_cast<int64_t>(0xffffffffffffffffULL/2-1)};

ジェネレータ
=========

ヘッダーファイル: ``<Kokkos_Core.hpp>`` ``<Kokkos_Random.hpp>``

概要
--------

Kokkos_Randomは擬似乱数ジェネレータに必要な構造を提供します。これらのジェネレータは、Vigna, Sebastiano (2014) に基づいています。[*"マルサーリアのXORシフト生成器に関する実験的探求、スクランブル処理されています。" 以下を参照: http://arxiv.org/abs/1402.6246*]  

乱数生成器自体には、ステートプールと実際のジェネレータの二つの構成要素があります。:
ステートプールは複数のジェネレータを管理し、各アクティブなスレッドが自身のジェネレータを取得できるようにします。
各アクティブなスレッドが自身のジェネレータを取得できるようにします。
これにより、スレッド間で独立した乱数を生成することが可能になります。
生成することが可能になります。**CuRAND**とは対照的に、
プール（またはジェネレータ）の関数はいずれも集合関数ではないことに注意してください。
つまり、すべての関数は条件式内で呼び出すことができます。

.. code-block:: cpp

    template<class DeviceType>
    class Pool {
      public:

    　device_type使用 = DeviceType;
      generator_type使用 = Generator<DeviceType>;

      Pool();
      Pool(uint64_t seed);
      Pool(uint64_t seed, uint64_t num_states);
      Pool(const typename DeviceType::execution_space& exec, uint64_t seed);
      Pool(const typename DeviceType::execution_space& exec, uint64_t seed, uint64_t num_states);

      void init(uint64_t seed, uint64_t num_states);  // deprecated since Kokkos 5.0
      generator_type get_state();
      void free_state(generator_type Gen);
    }

構築および初期化
-------------------------------

ジェネレータのプールでは、開始シードを用い、num_states のプールサイズが設定して、初期化されます。
Random_XorShift64 ジェネレータはすべての状態を初期化するために、シリアルで使用され、
初期化プロセスをプラットフォームに依存せず
かつ決定論的にします。
ジェネレータを要求するとその状態がロックされ、
各スレッドが専用の（独立した）ジェネレータを保証されます。
（Cudaデバイス上で状態を取得するにはアトミック操作が必要であり、非決定的になることにご注意ください！）
完了後、ジェネレータはstateプールに戻され、それを解除し、
その状態のアップデート時に再び、
プール内で利用可能になります。

実行空間インスタンスを選択しないプールジェネレータは同期的であり、指定された`DeviceType`のデフォルトの実行スペースインスタンスを使用します。
実行空間インスタンスを選択するプールコンストラクタは非同期的です

使用例
---

プールおよびそのプール内でのジェネレータの選択を前提とすれば、
次の段階で、ジェネレータを使用して、所望する型の乱数を生成するファンクターを
開発します。

.. code-block:: cpp

    template<class Device>
    class Generator {
      public:

      typedef DeviceType device_type;

      //Max return values of respective [X]rand[S]() functions (XorShift).
      enum {MAX_URAND = 0xffffffffU};
      enum {MAX_URAND64 = 0xffffffffffffffffULL-1};
      enum {MAX_RAND = static_cast<int>(0xffffffffU/2)};
      enum {MAX_RAND64 = static_cast<int64_t>(0xffffffffffffffffULL/2-1)};

      //Init with a state and the idx with respect to pool. Note: in serial the
      //Generator can be used by just giving it the necessary state arguments
      KOKKOS_INLINE_FUNCTION
      Generator (STATE_ARGUMENTS, int state_idx = 0);

      //Draw a equidistributed uint32_t in the range [0,MAX_URAND)
      KOKKOS_INLINE_FUNCTION
      uint32_t urand();

      //Draw a equidistributed uint32_t in the range [0,range)
      KOKKOS_INLINE_FUNCTION
      uint32_t urand(const uint32_t& range);

      //Draw a equidistributed uint32_t in the range [start,end)
      KOKKOS_INLINE_FUNCTION
      uint32_t urand(const uint32_t& start, const uint32_t& end );
    }

選択された32ビット符号なし整数型に対して、3つの範囲オプションが表示されます: [0,MAX_URAND), 
[0,range) および [start,end)。
最初の（デフォルトの）オプションは、そのデータ型の最大可能な範囲を超える符号なし整数を選択します。MAX_URAND の定義値は、上記の列挙型として示されています 。（また64ビット符号なし整数用のmaX_URANDも示されてます。) 後者の2つのオプションは、ユーザー定義の整数の範囲をカバーします。

他のデータ型についての詳細情報: Scalar, uint64_t, int, int32_t, int64_t, float, double; また正規分布と、[0, range) および [start, end) オプション用のView-fillオプション。

例
-------

.. code-block:: cpp

    #include <Kokkos_Core.hpp>
    #include <Kokkos_Random.hpp>

    int main(int argc, char *argv[]) {
        Kokkos::ScopeGuard guard(argc, argv);

        Kokkos::Random_XorShift64_Pool<> random_pool(/*seed=*/12345);

        int total = 1000000;
        int count;
        Kokkos::parallel_reduce(
            "approximate_pi", total,
            KOKKOS_LAMBDA(int, int& local_count) {
                // acquire the state of the random number generator engine
                auto generator = random_pool.get_state();

                double x = generator.drand(0., 1.);
                double y = generator.drand(0., 1.);

                // do not forget to release the state of the engine
                random_pool.free_state(generator);

                if (x * x + y * y <= 1.) {
                    ++local_count;
                }
            },
            count);

        printf("pi = %f\n", 4. * count / total);
    }
