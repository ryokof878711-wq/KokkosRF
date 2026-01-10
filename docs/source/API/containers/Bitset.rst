
.. role:: cpp(code)
   :language: cpp

``Bitset``
==========

ヘッダーファイル: ``<Kokkos_Bitset.hpp>``

クラスインターフェース
---------------

.. cpp:class:: template <typename Device> Bitset

  :cpp:`Kokkos::Bitset` は、固定サイズ（実行時）のNビットシーケンスへのスレッドセーフビューを表します。

  :tparam Device: ビットを物理的に収容するデバイス。

  .. rubric:: 静的定数

  .. cpp:member:: static constexpr符号なしBIT_SCAN_REVERSE = 1u

    :cpp:`BIT_SCAN_REVERSE` : スキャン方向ビットマスク

  .. cpp:member:: static constexpr符号なしMOVE_HINT_BACKWARD = 2u

    :cpp:`MOVE_HINT_BACKWARD` : ヒント方向ビットマスク

  .. cpp:member:: static constexpr符号なしBIT_SCAN_FORWARD_MOVE_HINT_FORWARD = 0u

    :cpp:`BIT_SCAN_FORWARD_MOVE_HINT_FORWARD` : When passed as :cpp:`scan_direction` to :cpp:`find_any_set_near(...)`又は :cpp:`find_any_reset_near(...)`は、 前方（増加するインデックス）方向のビットをスキャンします。ビットが見つからなかった場合、現在のヒントの先にある新しいヒントを選択します。

  .. cpp:member:: static constexpr符号なしBIT_SCAN_REVERSE_MOVE_HINT_FORWARD = BIT_SCAN_REVERSE

    :cpp:`BIT_SCAN_REVERSE_MOVE_HINT_FORWARD`: When passed as :cpp:`scan_direction` to :cpp:`find_any_set_near(...)`又は :cpp:`find_any_reset_near(...)`は、逆（減少するインデックス）方向のビットをスキャンします。ビットが見つからなかった場合、現在のヒントの先にある新しいヒントを選択します。

  .. cpp:member:: static constexpr符号なしBIT_SCAN_FORWARD_MOVE_HINT_BACKWARD = MOVE_HINT_BACKWARD

    :cpp:`BIT_SCAN_FORWARD_MOVE_HINT_BACKWARD`: When passed as :cpp:`scan_direction` to :cpp:`find_any_set_near(...)` 又は、:cpp:`find_any_reset_near(...)`は、 前方（増加するインデックス）方向のビットをスキャンします。ビットが見つからなかった場合、現在のヒントの前にある新しいヒントを選択します。

  .. cpp:member:: static constexpr符号なしBIT_SCAN_REVERSE_MOVE_HINT_BACKWARD = BIT_SCAN_REVERSE | MOVE_HINT_BACKWARD

    :cpp:`BIT_SCAN_REVERSE_MOVE_HINT_BACKWARD`: When passed as :cpp:`scan_direction` to :cpp:`find_any_set_near(...)`又は :cpp:`find_any_reset_near(...)`は、 逆（減少するインデックス）方向のビットをスキャンします。ビットが見つからなかった場合、現在のヒントの前にある新しいヒントを選択します。

  .. rubric:: コンストラクタ

  .. cpp:function:: Bitset(符号なしarg_size = 0u)

    ホスト/デバイス: :cpp:`arg_size` bitsを持つビットセットを構築します。

  .. rubric:: データアクセス機能

  .. cpp:function:: 符号なしsize() const

    ホスト/デバイス: ビット数を返します。

  .. cpp:function:: 符号count() const

    ホスト: ``1``に設定されたビット数を返します。

  .. cpp:function:: void set()

    ホスト: すべてのビットを``1``に設定します。

  .. cpp:function:: void reset();
  .. cpp:function:: void clear();

    ホスト/デバイス: すべてのビットを``0``に設定します。

  .. cpp:function:: void set(符号なし i)

    デバイス: ``i``\ 'th bitを``1``に設定します。

  .. cpp:function:: void reset(符号なし i)

    デバイス: ``i``\ 'th bitを to ``0``に設定します。

  .. cpp:function:: bool test(unsigned i) const

    デバイス: return :cpp:`true` if and only if the ``i``\ 'th bit is set to ``1``.

  .. cpp:function:: unsigned max_hint() const

    ホスト/デバイス: used with :cpp:`find_any_set_near(...)` & :cpp:`find_any_reset_near(...)` functions.

    利用可能なビットを検索する際に、それらの関数を呼び出すべき最大回数を返します。

  .. cpp:function:: Kokkos::pair<bool, unsigned> find_any_set_near(符号なしhint, 符号なしscan_direction = BIT_SCAN_FORWARD_MOVE_HINT_FORWARD) const

    ホスト/デバイス: starting at the :cpp:`hint` position, find the first bit set to ``1``.

    :cpp:`pair<bool, unsigned>`を返します。

    :cpp:`result.first` が :cpp:`true` の場合、:cpp:`result.second` は検出されたビット位置です。

    :cpp:`result.first` が :cpp:`false` の場合、:cpp:`result.second` は新しいヒント位置です。

    :cpp:`scan_direction & BIT_SCAN_REVERSE`\の場合、その次に、ビットのスキャンはインデックスの降順で行われます。 then the scanning for the bit happens in decreasing index order;
    otherwise, it happens in increasing index order.

    If :cpp:`scan_direction & MOVE_HINT_BACKWARDS`\ , then the new hint position occurs at a smaller index than :cpp:`hint`\ ;
    otherwise, it occurs at a larger index than :cpp:`hint`.

  .. cpp:function:: Kokkos::pair<bool, unsigned> find_any_unset_near(unsigned hint, unsigned scan_direction = BIT_SCAN_FORWARD_MOVE_HINT_FORWARD) const;

    Host/Device: starting at the :cpp:`hint` position, find the first bit set to ``0``.

    Returns a :cpp:`pair<bool, unsigned>`.

    When :cpp:`result.first` is :cpp:`true` then :cpp:`result.second` is the bit position found.

    When :cpp:`result.first` is :cpp:`false` then :cpp:`result.second` is a new hint position.

    If :cpp:`scan_direction & BIT_SCAN_REVERSE`\ , then the scanning for the bit happens in decreasing index order; otherwise, it happens in increasing index order.

    If :cpp:`scan_direction & MOVE_HINT_BACKWARDS`\ , then the new hint position occurs at a smaller index than :cpp:`hint`\ ; otherwise, it occurs at a larger index than :cpp:`hint`.

  .. cpp:function:: constexpr bool is_allocated() const

    Host/Device: the bits are allocated on the device.

``ConstBitset``
===============

Class Interface
---------------

.. cpp:class:: template <typename Device> ConstBitset

  :tparam Device: Device that physically contains the bits.

  .. rubric:: Constructors / assignment

  .. cpp:function:: ConstBitset()

    Host/Device: Construct a bitset with no bits.

  .. cpp:function:: ConstBitset(ConstBitset const& rhs) = default
  .. cpp:function:: ConstBitset& operator=(ConstBitset const& rhs) = default

    Copy constructor/assignment operator.

  .. cpp:function:: ConstBitset(Bitset<Device> const& rhs)
  .. cpp:function:: ConstBitset& operator=(Bitset<Device> const& rhs)

    Host/Device: Copy/assign a :cpp:`Bitset` to a :cpp:`ConstBitset`.

  .. cpp:function:: unsigned size() const

    Host/Device: return the number of bits.

  .. cpp:function:: unsigned count() const

     Host/Device: return the number of bits which are set to ``1``.

  .. cpp:function:: bool test(unsigned i) const

    Host/Device: Return ``true`` if and only if the ``i``\ 'th bit set to ``1``.

Non-Member Functions
--------------------

  .. cpp:function:: template <typename DstDevice, typename SrcDevice> void deep_copy(Bitset<DstDevice>& dst, Bitset<SrcDevice> const& src)

    Copy a ``Bitset`` from ``src`` on ``SrcDevice`` to ``dst`` on ``DstDevice``.

  .. cpp:function:: template <typename DstDevice, typename SrcDevice> void deep_copy(Bitset<DstDevice>& dst, ConstBitset<SrcDevice> const& src)

    Copy a ``ConstBitset`` from ``src`` on ``SrcDevice`` to a ``Bitset`` ``dst`` on ``DstDevice``.
