### Reset and clock control(RCC)——Reset

----

三种reset类型：

1. system reset
2. power reset
3. RTC domain reset

#### Power reset

触发reset方式：

1. Power-on/power-down reset (POR/PDR reset)
2. When exiting Standby mode

结果：

1. A power reset sets all registers to their reset values.

#### System reset

触发reset方式：

1. A low level on the NRST pin (external reset)——也就是将引脚NRST拉低，拉低多久呢？
2. Window watchdog event (WWDG reset)
3. Independent watchdog event (IWDG reset)
4. A software reset (SW reset) 
5. Low-power management reset——standby 和 stop mode会reset
6. Option byte loader reset
7. A power reset

结果：

1. A system reset sets all registers to their reset values **except the reset flags in the clockcontroller CSR register**.——只剩下CSR寄存器中的值还保留着

