## Reset and clock control(RCC)——Clock

驱动system clock(SYSCLK)的时钟源有：

1. HSI，内部8MHz晶振
2. HSE，外部晶振4-32MHz，如f030一般也是使用8MHz的晶振
3. PLL clock——PLL是什么？

驱动特别功能的时钟源：

1. LSI，40kHz内部晶振用于驱动independent watchdog and optionally the RTC used for Auto-wakeup from Stop/Standby mode
2. 32.768 kHz low speed external crystal (LSE crystal) which optionally drives the real-time clock (RTCCLK)
3. HSI14, 14MHz的高速内部晶振，用于adc

以上时钟源：

1. 为了降低功耗，都可以关闭不需要的时钟源
2. 可以分频，来提供时钟给AHB和APB总线，AHB和APB最高不超过48MHz，对应AHB的时钟是HCLK，APB的时钟是PCLK

常用的几个功能的时钟来源：

##### Flash memory programming interface clock

1. only HSI clock

##### ADC

有两种时钟来源选择：（区别在于精度）

1. dedicated HSI14 clock, to run always at the maximum sampling rate
2. APB clock (PCLK) divided by 2 or 4

##### USART1

有四种时钟来源选择：

1. system clock
2. HSI clock
3. LSE clock
4. APB clock (PCLK)

##### Timer

1. APB clock * 1
2. APB clock * 2

##### Systick

1. AHB
2. AHB/8



### HSE Clock

___

使用外部晶振的好处是精度高，外围晶振电路要尽可能的靠近引脚

设置使用外围晶振：

1. SW将HSEON置1，然后等到HW将HSERDY置1时，clock release，此时可以产生中断（if enabled）

HSEON：Clock control register (RCC_CR)

HSERDY：Clock control register (RCC_CR)

#### 启动流程：

To switch ON the HSE oscillator, **512 HSE clock pulses need to be seen by an internal stabilization counter** after the HSEON bit is set. Even in the case that no crystal or resonator is connected to the device, <u>excessive external noise on the OSC_IN pin may still lead the oscillator to start</u>.

也就是说，启动的时候内部有一个计数器收到外部晶振的512个时钟脉冲时才会将HSERDY置1，但是噪声也可能会导致这种结果

#### 关闭使用：

Once the oscillator is started, it needs another 6 HSE clock pulses to complete a switching OFF sequence. If for any reason the oscillations are no more present on the OSC_IN pin, **the oscillator cannot be switched OFF, locking the OSC pins from any other use and introducing unwanted power consumption**. To avoid such situation, it is strongly recommended to always enable the Clock Security System (CSS) which is able to switch OFF the oscillator even in this case.

SW一旦把HSEON置1后，其实是不可以置0了（从HSEON的描述：This bit cannot be reset if the HSE oscillator is used directly or indirectly as the system ）clock.），那么如果外部晶振损坏了，只能使用上面推荐的CSS完成这种操作，具体要怎么做呢？

另外，HSEON在某种情况会自己置0：从HSEON的描述：Cleared by hardware to stop the HSE oscillator when entering Stop or Standby mode，当进入Stop或者Standby模式时自动置0，重新恢复run模式的时候应该会恢复HSEON=1，目前还没看到有关的描述，等研究模式的时候再看。



### HSI Clock

---

内部晶振是一个8MHz的晶振，默认是使用这个晶振的，因为HSEON默认是0，HSION默认是1

这个晶振主要是为了降低成本（无需外围电路），它的启动时间会比HSE快（可能是因为HSE启动需要计数512个外来脉冲），但缺点是不稳定，精度低

#### 校准

由于精度不高，ST会自动校准，校准值存放在HSICAL[7:0]中（从HSICAL[7:0]的描述：These bits are initialized automatically at startup），具体细节暂时不研究

HSION：Clock control register (RCC_CR)

HSIRDY：Clock control register (RCC_CR)

#### 启动和关闭流程

从HSION描述：Set by hardware to force the HSI oscillator ON when leaving Stop or Standby mode or in case of failure of the HSE crystal oscillator used directly or indirectly as system clock. This bit cannot be reset if the HSI is used directly or indirectly as system clock or is selected to become the system clock.

他的默认值是1，也就是芯片默认是使用HSI的，一旦HSION为1，且HSIRDY被硬件置1时，系统的时钟就使用了HSI，此时我们就不能通过SW将HSION置0了。而且特别注意的是，当系统进入了Stop和Standby模式的时候，HSION是会被硬件置1的，也就是在两种模式下必须使用HSI，即使我们在run模式下使用了外部晶振。

另外提到的是，如果我们使用了外部晶振，而外部晶振出现毁坏不工作的时候，我们可以切换使用HSI做为备份的时钟源，但具体的做法没有研究，文档参考

Refer to Section 7.2.7: Clock security system (CSS) on page 84.



### PLL Clock

---

PLL不是一个区别于HSI和HSE的时钟源，他是一个倍频器，用于将HSE或HSI倍频输出作为系统时钟，那么如果需要使用倍频，那么就需要提前设置几个条件：

1. 将哪个时钟源倍频（HSE还是HSI？）（）
2. 倍频倍数是多少（2, 3, …, 16倍）（如果使用的是HSE，那么在倍频前还可以先对HSE进行分频，而使用HSI的话，倍频前的时钟是HSI／2）（设置的寄存器在Clock configuration register (RCC_CFGR)）
3. 设置完后才允许PLL工作，以上设置的结果为PLL输出作为系统时钟的频率范围需要在16-48MHz

#### 运行中修改

在系统运行时修改PLL的配置（使用或不使用PLL，或者修改PLL参数：改倍数等等）需要一下步骤：

之后研究



### LSE Clock

---

32.768KHz的外部晶振，暂时不研究，和RTC有关



### LSI Clock

---

30-60KHZ的内部晶振，和IWDG和RTC有关，暂时不研究



### 系统时钟源的描述

---

如上面描述的三种可以作为系统时钟的时钟源

1. HSI
2. HSE
3. PLL

HSI在每次系统重启的时候总是会优先被选中的：**After a system reset, the HSI oscillator is selected as system clock.** When a clock source is used directly or through the PLL as a system clock, it is not possible to stop it.

而HSI切换到HSE或者PLL的过程需要等待标志为xxRDY置1：A switch from one clock source to another occurs **only if the target clock source is ready** **(clock stable after startup delay or PLL locked)**. If a clock source which is not yet ready is selected, the switch will occur when the clock source becomes ready

现在的问题是，我们在选择PLL或者HSE作为我们的系统时钟源时，是不是需要在代码中将HSION置0呢，而因为每次重启时寄存器清空，HSION默认=1，也就默认选择了HSI，那么内部是怎么进行判断而重新设置为HSE或者PLL作为系统时钟的呢？



### Clock security system (CSS)

---

功能描述是这样的：Clock Security System can be activated by software. In this case, the clock detector is enabled after the HSE oscillator startup delay, and disabled when this oscillator is stopped.可用来关闭外部晶振的使用，因为SW不能在HSE已经作为了系统时钟的时候将他关闭（即将HSEON=0），需要使用CSS来进行这种操作，但具体怎么使用后面再研究。



### MCO

---

功能描述：The microcontroller clock output (MCO) capability allows the clock to be output onto the external MCO pin，One of the following clock signals can be selected as the MCO clock:

1. HSI14
2. SYSCLK
3. HSI
4. HSE
5. PLL
6. LSE
7. LSI

也就是可以将以上的时钟脉冲从引脚输出