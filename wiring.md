Connection
----------

你需要为面板提供一个单独的电源。有一个独立于逻辑连接器的连接器，通常是在板子中间的一个大的连接器。板子需要5V电压（仔细检查极性：印在板子上的东西是正确的--我有一次得到的板子上的电缆，红色（表示 "+"）和黑色（表示 "GND"）是相反的！）。这个电源是用来点亮LED的；计划每块32x32的面板需要3.5安培。
You need a separate power supply for the panel. There is a connector for that separate from the logic connector, typically a big one in the center of the board. The board requires 5V (double check the polarity: what is printed on the board is correct - I once got boards with supplied cables that had red (suggesting `+`) and black (suggesting `GND`) reversed!). This power supply is used to light the LEDs; plan for ~3.5 Ampere per 32x32 panel.

RGB 面板上的连接器称为 Hub75 接口。 每个面板通常有两个端口，一个是输入，另一个是输出以链接其他面板。 通常，箭头显示哪个连接器是输入。
The connector on the RGB panels is called a Hub75 interface. Each panel typically has two ports, one is the input and the other is the output to chain additional panels. Usually an arrow shows which of the connectors is the input.

在这里，您可以看到 RGB 面板底部有一个 Hub75 连接器，包括指示输入方向的箭头：
Here you see a Hub75 connector to be seen at the bottom of the RGB panel board including the arrow indicating the input direction:
![Hub 75 interface][hub75-arrow]

其他的板子非常相似, 但不是zero-indexed颜色位
Other boards are very similar, but instead of zero-indexed color bits
`R0`, `G0`, `B0`, `R1`, `G1`, `B1`, they start the index with one and name these //他们以 1 开始索引并命名这些
`R1`, `G1`, `B1`, `R2`, `G2`, `B2`; the functionality is identical.//功能是相同的
![Hub 75 interface][hub75]

游览这个文档, 我们使用the one-index为基础, 所以我们使用这些信号`R1`, `G1`, `B1`, `R2`, `G2`, `B2`
Throughout this document, we will use the one-index base, so we will call these signals `R1`, `G1`, `B1`, `R2`, `G2`, `B2` below.

选通信号有时也称为`latch`或 `lat`。 我们在这里称之为频闪。
The `strobe` signals is sometimes also called `latch` or `lat`. We'll call it `strobe` here.

如果你将一条IDC电缆插入你的RGB面板到输入连接器上, 这就是电缆另一端的信号位置（想象一下左侧图片外某处的 LED 面板）； 注意连接器右侧的凹口：
If you plug an IDC-cable into your RGB panel to the input connector, this is how the signal positions are on the other end of the cable (imagine the LED panels somewhere outside the picture on the left); note the notch on the right side of the connector:
![Hub 75 IDC connector][hub75-idc]

RPi 仅具有 3.3V 逻辑输出电平，但许多在 5V 下运行的显示器可以很好地解释这些逻辑电平，只需确保将短电缆连接到电路板即可。 如果确实遇到故障或不稳定的像素，请考虑一些行缓冲，
The RPi only has 3.3V logic output level, but many displays operated at 5V interprets these logic levels fine, just make sure to run a short cable to the board. If you do run into glitches or erratic pixels, consider some line-buffering,
e.g. using the [active adapter PCB](./adapter/).

由于我们只需要 RPi 上的输出引脚，因此我们不需要担心电平转换回来。
Since we only need output pins on the RPi, we don't need to worry about level conversion back.

对于单个 LED 面板链，我们需要 13 条 IO 线，这些线全部适合旧 Raspberry Pi 的接头座。 较新的 40 引脚 Raspberry Pi 具有更多 GPIO 线，这使我们能够连接三个并行的 RGB 面板链。
For a single chain of LED-panels, we need 13 IO lines, which fit all in the header of the old Raspberry Pis. Newer Raspberry Pis with 40 pins have more GPIO lines which allows us to connect three parallel chains of RGB panels.

作为参考，Raspberry Pi 上的编号如下所示： (refresh: 刷新)
For reference, this is how the numbering on the Raspberry Pi looks like:
<a href="img/raspberry-gpio.jpg"><img src="img/raspberry-gpio.jpg" width="600px"></a>

这与下表中使用的表示方法相同，有助于目视检查。
This is the same representation used in the table below, which helps for visual inspection.

### Chains

你将Pi连接到面板链中第一个面板的输入。每个面板都有一个输出接口，然后你可以连接到该链中的下一个面板。
You connect the Pi to the input of the first in the chain of panels. Each panel has an output connector, that you then can connect to the next panel in that chain.

IO和库支持并行运行多达三条链。
The IO and library supports to run up to three chains in parallel.

因此，你可以创建一个更大的面板。这里是一个示意图，下面在 "电源 "部分，你可以看到一个真实的面板，从后面看有三个5个面板的链。
Thus you can create a larger panel. Here a schematic view, below in the 'Power' section, you can see a real-live panel with three chains of 5 panels each seen from the back.

![Coordinate overview][coordinates]

### Wiring diagram

你可以在下表中找到Raspberry Pi上的引脚位置和相应的逻辑连接（Raspberry Pi上有更多的GND引脚，但为了简单起见，它们被排除在外）。
You find the positions of the pins on the Raspberry Pi and the corresponding logical connections in the table below (there are more GND pins on the Raspberry Pi, but they are left out for simplicity).

#### Shared connections
对于最多三条链中的每一条，你必须将`GND`、`strobe`、`clock`、`OE-`、`A`、`B`、`C`、`D`连接到**all**这些（32x32显示器需要`D`线；32x16显示器不需要）。
For each of the up to three chains, you have to connect `GND`, `strobe`, `clock`, `OE-`, `A`, `B`, `C`, `D` to **all** of these (the `D` line is needed for 32x32 displays; 32x16 displays don't need it).

如果你有一个 64x64 显示器, 这些有一个额外的 "E "线，通常在矩阵连接器的第4或第8针上。
If you have a 64x64 display, these have an additional `E` line which is typically on Pin 4 or 8 on the matrix connector.

对于这些引脚，所有的链子都接收相同的数据线，例如，如果你有三个链子，你必须用三根线把Pi上的`A`输出接到`A`输入的三个链子的输入。
For these pins, all chains receive the same data line, e.g. if you have three chains, you have to wire the `A` output on the Pi with three wires to the three chain inputs of the `A` input.

#### Connections per chain

然后，对于每一个链的第一个面板，有一组（R1, G1, B1, R2, G2, B2），你必须连接到相应的引脚。
它们被标记为"[1]"、"[2]"和"[3]"，分别代表下面的链1、2和3。

如果你只连接一个面板或只有一条链，就把它连接到`[1]`(:smile:)；如果你使用平行链，就加上其他`[2]`和`[3]`。

为了使事情在视觉上更快地导航，每条链都用一个单独的图标标记：

`[1]`=:smile:, `[2]`=:boom: and `[3]`=:droplet:; 通往所有链的信号都有所有的图标。

Then for each first panel of a chain there is a set of (R1, G1, B1, R2, G2, B2) that you have to connect to the corresponding pins.
They are marked `[1]`, `[2]` and `[3]` for chain 1, 2, and 3 below.

If you only connect one panel or have one chain, connect it to `[1]` (:smile:); if you use parallel chains, add the other `[2]` and `[3]`.

To make things quicker to navigate visually, each chain is marked with a separate icon:

`[1]`=:smile:, `[2]`=:boom: and `[3]`=:droplet: ; signals that go to all chains have all icons.

|Connection                        | Pin | Pin |  Connection
|---------------------------------:|:---:|:---:|:-----------------------------
|                             -    |   1 |   2 | -
|             :droplet: **[3] G1** |   3 |   4 | -
|             :droplet: **[3] B1** |   5 |   6 | **GND** :smile::boom::droplet:
|:smile::boom::droplet: **strobe** |   7 |   8 | **[3] R1** :droplet:
|                              -   |   9 |  10 | **E**    :smile::boom::droplet: (for 64 row matrix, 1:32)
|:smile::boom::droplet: **clock**  |  11 |  12 | **OE-**  :smile::boom::droplet:
|              :smile:  **[1] G1** |  13 |  14 | -
|:smile::boom::droplet:      **A** |  15 |  16 | **B**    :smile::boom::droplet:
|                             -    |  17 |  18 | **C**    :smile::boom::droplet:
|              :smile:  **[1] B2** |  19 |  20 | -
|              :smile:  **[1] G2** |  21 |  22 | **D**    :smile::boom::droplet: (for 32 row matrix, 1:16)
|              :smile:  **[1] R1** |  23 |  24 | **[1] R2** :smile:
|                             -    |  25 |  26 | **[1] B1** :smile:
|                             -    |  27 |  28 | -
|              :boom:   **[2] G1** |  29 |  30 | -
|              :boom:   **[2] B1** |  31 |  32 | **[2] R1** :boom:
|              :boom:   **[2] G2** |  33 |  34 | -
|              :boom:   **[2] R2** |  35 |  36 | **[3] G2** :droplet:
|              :droplet:**[3] R2** |  37 |  38 | **[2] B2** :boom:
|                              -   |  39 |  40 | **[3] B2** :droplet:

In the [adapter/](./adapter) directory, there are some boards that make
the wiring task simpler.

<a href="adapter/"><img src="img/three-parallel-panels-soic.jpg" width="300px"></a>

### Alternative Hardware Mappings

上面描述的硬件映射是 "常规 "硬件映射，它是这个库的默认值。然而，也有其他的硬件映射可以选择，比如Adafruit出售的电路板就选择了不同的映射。你可以用`--led-gpio-mapping`标志来选择。

如果你得到一个未知来源的转接板，而你没有得到任何输出：仔细检查他们使用的GPIO映射。

你有相对的自由将任何引脚分配给你选择的输出，只要在[lib/hardware-mapping.c](lib/hardware-mapping.c)中添加一个新的映射，重新编译，它将作为`--led-gpio-mapping`的一个新选项提供。
The hardware mapping described above is the 'regular' hardware mapping, which is the default for this library. However, there are alternative hardware mappings to choose from, e.g. Adafruit sells a board where they choose a different mapping. You can choose with the `--led-gpio-mapping` flag.

If you got an adapter board that is from some unknown source and you don't get any output: double check the GPIO mappings they use.

You have relative freedom to assign any pins to the output of your choosing, just add a new mapping in [lib/hardware-mapping.c](lib/hardware-mapping.c), recompile and it will be provided as a new option in `--led-gpio-mapping`.

<details><summary>Table: GPIO-pins for each hardware mapping</summary>

|         | regular | adafruit-hat | adafruit-hat-pwm | regular-pi1 | classic | classic-pi1 | compute-module |
----------|---------|--------------|------------------|-------------|---------|-------------|----------------|
Parallel chains|        3|             1|                 1|            1|        3|            1|               6|
~OE       |GPIO 18  |GPIO 4        |GPIO 18           |GPIO 18      |GPIO 27  |GPIO 0       |GPIO 18         |
Clock     |GPIO 17  |GPIO 17       |GPIO 17           |GPIO 17      |GPIO 11  |GPIO 1       |GPIO 16         |
Strobe    |GPIO 4   |GPIO 21       |GPIO 21           |GPIO 4       |GPIO 4   |GPIO 4       |GPIO 17         |
A         |GPIO 22  |GPIO 22       |GPIO 22           |GPIO 22      |GPIO 7   |GPIO 7       |GPIO 2          |
B         |GPIO 23  |GPIO 26       |GPIO 26           |GPIO 23      |GPIO 8   |GPIO 8       |GPIO 3          |
C         |GPIO 24  |GPIO 27       |GPIO 27           |GPIO 24      |GPIO 9   |GPIO 9       |GPIO 4          |
D         |GPIO 25  |GPIO 20       |GPIO 20           |GPIO 25      |GPIO 10  |GPIO 10      |GPIO 5          |
E         |GPIO 15  |GPIO 24       |GPIO 24           |GPIO 15      |        -|            -|GPIO 6          |
Chain 1/R1|GPIO 11  |GPIO 5        |GPIO 5            |GPIO 11      |GPIO 17  |GPIO 17      |GPIO 7          |
Chain 1/G1|GPIO 27  |GPIO 13       |GPIO 13           |GPIO 21      |GPIO 18  |GPIO 18      |GPIO 8          |
Chain 1/B1|GPIO 7   |GPIO 6        |GPIO 6            |GPIO 7       |GPIO 22  |GPIO 22      |GPIO 9          |
Chain 1/R2|GPIO 8   |GPIO 12       |GPIO 12           |GPIO 8       |GPIO 23  |GPIO 23      |GPIO 10         |
Chain 1/G2|GPIO 9   |GPIO 16       |GPIO 16           |GPIO 9       |GPIO 24  |GPIO 24      |GPIO 11         |
Chain 1/B2|GPIO 10  |GPIO 23       |GPIO 23           |GPIO 10      |GPIO 25  |GPIO 25      |GPIO 12         |
Chain 2/R1|GPIO 12  |             -|                 -|            -|GPIO 12  |            -|GPIO 13         |
Chain 2/G1|GPIO 5   |             -|                 -|            -|GPIO 5   |            -|GPIO 14         |
Chain 2/B1|GPIO 6   |             -|                 -|            -|GPIO 6   |            -|GPIO 15         |
Chain 2/R2|GPIO 19  |             -|                 -|            -|GPIO 19  |            -|GPIO 19         |
Chain 2/G2|GPIO 13  |             -|                 -|            -|GPIO 13  |            -|GPIO 20         |
Chain 2/B2|GPIO 20  |             -|                 -|            -|GPIO 20  |            -|GPIO 21         |
Chain 3/R1|GPIO 14  |             -|                 -|            -|GPIO 14  |            -|GPIO 22         |
Chain 3/G1|GPIO 2   |             -|                 -|            -|GPIO 2   |            -|GPIO 23         |
Chain 3/B1|GPIO 3   |             -|                 -|            -|GPIO 3   |            -|GPIO 24         |
Chain 3/R2|GPIO 26  |             -|                 -|            -|GPIO 15  |            -|GPIO 25         |
Chain 3/G2|GPIO 16  |             -|                 -|            -|GPIO 26  |            -|GPIO 26         |
Chain 3/B2|GPIO 21  |             -|                 -|            -|GPIO 21  |            -|GPIO 27         |
Chain 4/R1|        -|             -|                 -|            -|        -|            -|GPIO 28         |
Chain 4/G1|        -|             -|                 -|            -|        -|            -|GPIO 29         |
Chain 4/B1|        -|             -|                 -|            -|        -|            -|GPIO 30         |
Chain 4/R2|        -|             -|                 -|            -|        -|            -|GPIO 31         |
Chain 4/G2|        -|             -|                 -|            -|        -|            -|GPIO 32         |
Chain 4/B2|        -|             -|                 -|            -|        -|            -|GPIO 33         |
Chain 5/R1|        -|             -|                 -|            -|        -|            -|GPIO 34         |
Chain 5/G1|        -|             -|                 -|            -|        -|            -|GPIO 35         |
Chain 5/B1|        -|             -|                 -|            -|        -|            -|GPIO 36         |
Chain 5/R2|        -|             -|                 -|            -|        -|            -|GPIO 37         |
Chain 5/G2|        -|             -|                 -|            -|        -|            -|GPIO 38         |
Chain 5/B2|        -|             -|                 -|            -|        -|            -|GPIO 39         |
Chain 6/R1|        -|             -|                 -|            -|        -|            -|GPIO 40         |
Chain 6/G1|        -|             -|                 -|            -|        -|            -|GPIO 41         |
Chain 6/B1|        -|             -|                 -|            -|        -|            -|GPIO 42         |
Chain 6/R2|        -|             -|                 -|            -|        -|            -|GPIO 43         |
Chain 6/G2|        -|             -|                 -|            -|        -|            -|GPIO 44         |
Chain 6/B2|        -|             -|                 -|            -|        -|            -|GPIO 45         |

</details>


A word about power
------------------

这些显示器会消耗大量电流。 在 5V 电压下，当所有 LED 均亮起（全白）时，我的 32x32 LED 面板消耗约 3.4A 电流。 对于非常亮的户外面板，这可能是两倍。这意味着，您需要强大的电源来驱动这些面板； 一个2A USB 对于 32x32 面板来说，充电器或类似设备是不够的； 它可能适用于 16x32。

如果将多块板连接在一起，则需要一个可以的电源跟上3.5A/面板。 旧电脑电源很好，经常使用在 5V 电源轨上提供 > 20A 的电流。 或者你可以获得专用的5V大电流用于此类应用的开关电源（查看 eBay）。

目前的吸引力相当强劲。 由于 LED 的 PWM，有很多几个 100ns 到大约 1ms 的短峰值全电流消耗。通常，由于固有的原因，电源线无法支持这些非常短的尖峰电感。 这可能会导致“嘈杂”的输出，随机像素不表现正如他们应该的那样。 在这些情况下，靠近输入的低 ESR 电容器会很好。

在某些显示器上，输出的质量很快就会变得不稳定当电压低于4.5V时。 有些甚至需要更高一点的电压5.5V可靠工作。 另外，使用“--led-slowdown-gpio”标志进行调整。

当您将这些板连接到电源时，以下情况是好的指导方针：
    - 使用相当粗的电缆将电源连接到电路板。计划从源到 LED 矩阵的电压不要超过 50mV。因此，即 50mV / 3.5A = 14 mΩ。 对于两条电源线，因此 7mΩ
      每一条痕迹。1mm² 铜电缆的电阻约为 17.5mΩ/米，因此您需要 **2.5mm² 每米和面板的铜缆**。 乘以米和数量面板以获得所需的横截面。（对于美国人：对于 3 英尺和一个面板来说，这将是 ~13 规格的电线）

    - 虽然布线的星形配置是最佳的（每个面板都有来自电源的单独电线），通常就足够了使用铝安装支架或杆作为您的电源解决方案。 1mm² 铝的电阻率为
      大约 28mΩ/米，每个面板需要大约 4mm² 的横截面积和米。

      在下面的示例中，您会看到中间的结构铝条（覆盖有彩色乙烯基）兼作电源棒。 已连接60A/5V电源到中心螺栓（显示屏使用约 42A 所有 LED 灯）：
      ![电源栏][电源栏]

    - 这些板通常附带带有压接连接器的电缆。一些便宜的电缆通常太细； 你可能想把它们夹在靠近的地方连接器将合适的粗电缆焊接到其上。

    - 最好直接在面板上缓冲电流尖峰。 最多对单条线路进行 PWM 处理时会出现尖峰。假设我们想要缓冲为单条线路供电的能量，而无需下降超过50mV。 我们使用 3.5A，即 3.5 焦耳/秒。 我们的确是大约 140Hz 刷新率并将其分为 16 行，所以我们需要3.5 Joule/140/16 = ~1.6mJoule 在显示一行的时间段内。我们想从50mV的压降中获取能量； 所以与W = 1/2*C*U²，我们可以计算出所需的电容：
        C = 2 * 1.6mJoule / ((5V)² - (5V - 50mV)²) = ~6400μF。
      因此，直接并联 2 个 3300μF 低 ESR 电容器在板上是一个不错的选择（两个，因为并行 ESR 较低；也更容易安装在板下）。（实际上，我们需要的当然更少，因为最高的纹波伴随着50% 占空比，电流减半； 输入也正在重新充电时间。 但是：作为工程师的计划是最大化然后是一些； 图片中上面我每个面板使用 1x3300uF，效果很好）。

现在欢迎您精心设计的电源解决方案:)

These displays suck a lot of current. At 5V, when all LEDs are on (full white),my 32x32 LED panel draws about 3.4A. For an outdoor panel that is very bright,that can be twice as much. That means, you need a beefy power supply to drive these panels; a 2A USB charger or similar is not enough for a 32x32 panel; it might be for a 16x32.

If you connect multiple boards together, you needs a power supply that can keep up with 3.5A / panel. Good are old PC power supplies that often provide > 20A on the 5V rail. Or you can get a dedicated 5V high current switching power supply for these kind of applications (check eBay).

The current draw is pretty spiky. Due to the PWM of the LEDs, there are very short peaks of a couple of 100ns to about 1ms of full current draw. Often, the power cable can't support these very short spikes due to inherent inductance. This can result in 'noisy' outputs, with random pixels not behaving as they should. A low ESR capacitor close to the input is good in these cases.

On some displays, the quality of the output quickly gets erratic when voltage drops below 4.5V. Some even need a little bit higher voltage around 5.5V to work reliably. Also, tweak with the `--led-slowdown-gpio` flag.

When you connect these boards to a power source, the following are good
guidelines:
   - Have fairly thick cables connecting the power to the board.
     Plan not to loose more than 50mV from the source to the LED matrix.
     So that would be 50mV / 3.5A = 14 mΩ. For both supply wires, so 7mΩ
     each trace.
     A 1mm² copper cable has about 17.5mΩ/meter, so you'd need a **2.5mm²
     copper cable per meter and panel**. Multiply by meter and number of
     panels to get the needed cross-section.
     (For Americans: that would be ~13 gauge wire for 3 ft and one panel)

   - While a star configuration for the cabeling would be optimal (each panel gets
     an individual wire from the power supply), it is typically sufficient
     using aluminum mounting brackets or bars as part of
     your power solution. With aluminum of 1mm² specific resistivity of
     about 28mΩ/meter, you'd need a cross sectional area of about 4mm² per panel
     and meter.

     In the following example you see the structural aluminum bars in the middle
     (covered in colored vinyl) dualing as power bars. The 60A/5V power supply is connected
     to the center bolts (display uses about 42A all LEDs on):
     ![Powerbar][powerbar]

   - Often these boards come with cables that have connectors crimped on.
     Some cheap cables are typically too thin; you might want to clip them close to
     the connector solder your proper, thick cable to it.

   - It is good to buffer the current spikes directly at the panel. The most
     spikes happen while PWM-ing a single line.
     So let's say we want to buffer the energy to power a single line without
     dropping more than 50mV. We use 3.5A which is 3.5Joule/second. We do
     about 140Hz refresh rate and divide that in 16 lines, so we need
     3.5 Joule/140/16 = ~1.6mJoule in the time period to display one line.
     We want to get the energy out of the voltage drop of 50mV; so with
     W = 1/2*C*U², we can calculate the capacitance needed:
       C = 2 * 1.6mJoule / ((5V)² - (5V - 50mV)²) = ~6400µF.
     So, 2 x 3300µF low-ESR capacitors in parallel directly
     at the board are a good choice (two, because lower parallel ESR; also
     fits easier under board).
     (In reality, we need of course less, as the highest ripple comes with
      50% duty cyle thus half the current; also the input is recharching all
      the time. But: as engineer plan for maximum and then some; in the picture
      above I am using 1x3300uF per panel and it works fine).

Now welcome your over-engineered power solution :)

[hub75]: ./img/hub75.jpg
[hub75-arrow]: ./img/hub75-other.jpg
[hub75-idc]: ./img/idc-hub75-connector.jpg
[coordinates]: ./img/coordinates.png
[powerbar]: ./img/powerbar.jpg
