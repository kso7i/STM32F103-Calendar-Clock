 # STM32F103 Calendar Clock / STM32F103 万年历时钟

This project is an OLED calendar clock based on the **STM32F103C8T6** microcontroller.
It uses **TIM2 timer interrupt** to update time, **USART2** to receive time-setting commands, and **PB5 external interrupt** to start or stop the clock.

本项目是一个基于 **STM32F103C8T6** 的 OLED 万年历时钟实验。
项目使用 **TIM2 定时器中断** 实现秒级计时，使用 **USART2 串口** 接收上位机校时命令，并通过 **PB5 外部中断按键** 控制时钟开始或暂停。

---

## Demo Display / 显示效果

Example OLED display:

OLED 显示示例：

```text
2026.06.08 Mon
08:09:15
2407224
2026是平年
```

If the current year is a leap year:

如果当前年份是闰年：

```text
2028.06.08 Thu
08:09:15
2407224
2028是闰年
```

Display description:

显示内容说明：

| Line   | English                         | 中文      |
| ------ | ------------------------------- | ------- |
| Line 1 | Date and weekday                | 日期和星期   |
| Line 2 | Current time                    | 当前时间    |
| Line 3 | Student ID                      | 学号      |
| Line 4 | Leap year or common year status | 闰年或平年判断 |

---

## Features / 功能特点

* Display date, weekday, time, student ID, and leap-year status on OLED
  在 OLED 上显示日期、星期、时间、学号和闰年/平年状态

* Use **TIM2 interrupt** to update the clock every second
  使用 **TIM2 定时器中断** 每秒更新时间

* Use **USART2** to receive commands from a serial assistant
  使用 **USART2 串口** 接收上位机命令

* Use **PB5 external interrupt** to start or stop the clock
  使用 **PB5 外部中断按键** 控制时钟开始或暂停

* Support UART time modification
  支持通过串口修改日期和时间

* Check whether the input date and weekday match
  支持校验输入的日期和星期是否匹配

* Support leap year and common year judgment
  支持闰年和平年判断

* Automatically handle date and time carry
  支持日期和时间自动进位

  * second to minute
    秒进位到分

  * minute to hour
    分进位到时

  * hour to day
    时进位到天

  * month-end carry
    月末进位

  * leap year February 29
    闰年 2 月 29 日判断

  * year carry
    年份进位

---

## Hardware / 硬件环境

| Module     | English                    | 中文                    |
|------------|----------------------------|-------------------------|
| MCU        | STM32F103C8T6              | STM32F103C8T6 单片机     |
| Display    | OLED module                | OLED 显示屏              |
| Downloader | USB-to-serial downloader   | USB 转串口下载器         |
| Wire       | Dupont wires               | 杜邦线                  |
| Debugger   | DAP-Link or ST-Link        | DAP-Link 或 ST-Link      |

## Software / 软件环境

| Software   | English                    | 中文           |
|------------|----------------------------|----------------|
| CubeMX     | STM32CubeMX                | STM32 配置工具 |
| Keil       | Keil MDK                   | Keil 编译软件  |
| Serial Tool| Serial debugging assistant | 串口调试助手   |

---

## Pin Configuration / 引脚配置

| Function                    | Pin                         | 中文说明          |
| --------------------------- | --------------------------- | ------------- |
| USART2_TX                   | PA2                         | 串口2发送         |
| USART2_RX                   | PA3                         | 串口2接收         |
| Button / External Interrupt | PB5                         | 外部中断按键        |
| OLED SCL                    | Depends on your OLED driver | 取决于你的 OLED 驱动 |
| OLED SDA                    | Depends on your OLED driver | 取决于你的 OLED 驱动 |

> The OLED pins depend on the OLED driver used in the project.
> OLED 的具体引脚取决于你工程中使用的 OLED 驱动代码。

---

## STM32CubeMX Configuration / STM32CubeMX 配置

### USART2 Configuration / USART2 配置

| Parameter             | Value        | 中文说明       |
| --------------------- | ------------ | ---------- |
| Mode                  | Asynchronous | 异步模式       |
| Baud Rate             | 115200       | 波特率 115200 |
| Word Length           | 8 Bits       | 8 位数据位     |
| Parity                | None         | 无校验        |
| Stop Bits             | 1            | 1 位停止位     |
| Hardware Flow Control | Disable      | 关闭硬件流控     |

---

### TIM2 Configuration / TIM2 配置

TIM2 is used to generate a 1-second interrupt.

TIM2 用于产生 1 秒一次的定时中断。

If the system clock is 72 MHz:

如果系统时钟为 72 MHz：

| Parameter      | Value                        | 中文说明         |
| -------------- | ---------------------------- | ------------ |
| Clock Source   | Internal Clock               | 内部时钟         |
| Prescaler      | 7199                         | 预分频值         |
| Counter Period | 9999                         | 自动重装载值       |
| NVIC           | Enable TIM2 global interrupt | 使能 TIM2 全局中断 |

Calculation:

计算公式：

```text
72 MHz / 7200 / 10000 = 1 Hz
```

So TIM2 enters the interrupt once per second.

所以 TIM2 每 1 秒进入一次中断。

---

### PB5 External Interrupt / PB5 外部中断配置

| Parameter         | Value                                                       | 中文说明       |
| ----------------- | ----------------------------------------------------------- | ---------- |
| GPIO Mode         | External Interrupt Mode with Falling edge trigger detection | 下降沿触发外部中断  |
| Pull-up/Pull-down | Pull-up                                                     | 上拉输入       |
| NVIC              | Enable EXTI line interrupt                                  | 使能 EXTI 中断 |

---

## UART Commands / 串口命令

The serial assistant should use the following settings:

串口助手建议使用以下配置：

```text
Baud rate: 115200
Data bits: 8
Stop bits: 1
Parity: None
Send new line: enabled
```

中文说明：

```text
波特率：115200
数据位：8
停止位：1
校验位：None
发送新行：开启
```

---

### Change Time / 修改时间

Command format:

命令格式：

```text
change time:YYYYMMDD-HHMMSS-W
```

Field description:

字段说明：

| Field | English        | 中文   |
| ----- | -------------- | ---- |
| YYYY  | Year           | 年    |
| MM    | Month          | 月    |
| DD    | Day            | 日    |
| HH    | Hour           | 时    |
| MM    | Minute         | 分    |
| SS    | Second         | 秒    |
| W     | Weekday number | 星期编号 |

Weekday definition:

星期编号定义：

| Number | Weekday | 中文  |
| ------ | ------- | --- |
| 1      | Mon     | 星期一 |
| 2      | Tue     | 星期二 |
| 3      | Wed     | 星期三 |
| 4      | Thu     | 星期四 |
| 5      | Fri     | 星期五 |
| 6      | Sat     | 星期六 |
| 7      | Sun     | 星期日 |

Example:

示例：

```text
change time:20260609-100105-2
```

If the date and weekday are correct, the board returns:

如果日期和星期匹配，单片机会返回：

```text
change time ok!
```

If the date or weekday is wrong, the board returns:

如果日期不存在，或者星期和日期不匹配，单片机会返回：

```text
change time fail!
```

---

### Stop Time / 暂停时间

Command:

命令：

```text
stop time
```

Return message:

返回信息：

```text
stop time ok!
```

---

### Start Time / 开始时间

Command:

命令：

```text
start time
```

Return message:

返回信息：

```text
start time ok!
```

---

## Test Cases / 测试用例

### 1. Valid Time Modification / 正确修改时间

Input:

输入：

```text
change time:20260609-100105-2
```

Expected return:

预期返回：

```text
change time ok!
```

Expected OLED display:

预期 OLED 显示：

```text
2026.06.09 Tue
10:01:05
2407224
2026是平年
```

---

### 2. Invalid Weekday Test / 星期错误测试

Input:

输入：

```text
change time:20260609-100105-3
```

Expected return:

预期返回：

```text
change time fail!
```

Reason:

原因：

```text
2026.06.09 is Tuesday, not Wednesday.
2026年6月9日是星期二，不是星期三。
```

---

### 3. Leap Year Carry Test / 闰年进位测试

Input:

输入：

```text
change time:20000228-235958-1
```

Expected return:

预期返回：

```text
change time ok!
```

After 2 seconds, OLED should display:

2 秒后，OLED 应显示：

```text
2000.02.29 Tue
00:00:00
2407224
2000是闰年
```

Reason:

原因：

```text
The year 2000 is a leap year.
2000年是闰年，所以2月28日之后会进入2月29日。
```

---

### 4. Common Year Carry Test / 平年进位测试

Input:

输入：

```text
change time:20010228-235958-3
```

Expected return:

预期返回：

```text
change time ok!
```

After 2 seconds, OLED should display:

2 秒后，OLED 应显示：

```text
2001.03.01 Thu
00:00:00
2407224
2001是平年
```

Reason:

原因：

```text
The year 2001 is not a leap year.
2001年不是闰年，所以2月28日之后会进入3月1日。
```

---

### 5. Invalid Date Test / 非法日期测试

Input:

输入：

```text
change time:20010229-235958-4
```

Expected return:

预期返回：

```text
change time fail!
```

Reason:

原因：

```text
2001 is not a leap year, so February 29 does not exist.
2001年不是闰年，所以2001年2月29日不存在。
```






---




Supported commands:

支持的命令：

```text
change time:YYYYMMDD-HHMMSS-W
stop time
start time
```


---

## Notes / 注意事项

* Do not delete the `USER CODE BEGIN` and `USER CODE END` areas generated by STM32CubeMX.
  不要删除 STM32CubeMX 生成的 `USER CODE BEGIN` 和 `USER CODE END` 区域。

* If STM32CubeMX regenerates the code, check whether your user code is still kept correctly.
  如果重新使用 STM32CubeMX 生成代码，需要检查用户代码是否被正确保留。

* TIM2 must be configured as **Internal Clock**.
  TIM2 必须配置为 **Internal Clock**。

* TIM2 global interrupt must be enabled.
  必须使能 TIM2 全局中断。

* USART2 interrupt must be enabled.
  必须使能 USART2 中断。

* The serial assistant should enable newline sending, such as `\r\n`.
  串口助手建议开启发送新行，例如发送 `\r\n`。

* If the OLED flickers, avoid clearing all four lines every second.
  如果 OLED 闪烁，不要每秒清空四行，应只刷新变化的行。



---

## License / 开源协议

This project is released for learning and educational use.

本项目主要用于 STM32 入门学习、课程实验和嵌入式基础训练。

You can use, modify, and share this project for educational purposes.

你可以将本项目用于学习、修改和二次开发。
