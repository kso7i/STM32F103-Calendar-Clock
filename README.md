# STM32F103-Calendar-Clock
OLED calendar clock based on STM32F103C8T6, with timer interrupt, external interrupt, and UART time setting.
# STM32F103 Calendar Clock

This project is an OLED calendar clock based on the **STM32F103C8T6** microcontroller.
It uses a hardware timer interrupt to update time, an external interrupt button to start or stop the clock, and UART commands to modify the date and time.

本项目是一个基于 **STM32F103C8T6** 的 OLED 万年历时钟实验，支持定时器计时、外部中断控制、串口校时、星期校验和闰年判断显示。

---

## Features

* Display date, weekday, time, student ID, and leap-year status on OLED
* Use **TIM2 interrupt** to update the clock every second
* Use **USART2** to receive commands from a serial assistant
* Use **PB5 external interrupt** to start or stop the clock
* Support date and time modification through UART
* Check whether the input date and weekday match
* Support leap year and common year judgment
* Automatically handle date carry:

  * second to minute
  * minute to hour
  * hour to day
  * month-end carry
  * leap year February 29
  * year carry

---

## Hardware

| Module  | Description            |
| ------- | ---------------------- |
| MCU     | STM32F103C8T6          |
| Display | OLED module            |
| UART    | USART2                 |
| Timer   | TIM2                   |
| Button  | PB5 external interrupt |
| IDE     | STM32CubeMX + Keil MDK |

---

## Pin Configuration

| Function                    | Pin                           |
| --------------------------- | ----------------------------- |
| USART2_TX                   | PA2                           |
| USART2_RX                   | PA3                           |
| Button / External Interrupt | PB5                           |
| OLED SCL                    | According to your OLED driver |
| OLED SDA                    | According to your OLED driver |

> The OLED pins depend on the OLED driver used in the project.

---

## STM32CubeMX Configuration

### USART2

| Parameter             | Value        |
| --------------------- | ------------ |
| Mode                  | Asynchronous |
| Baud Rate             | 115200       |
| Word Length           | 8 Bits       |
| Parity                | None         |
| Stop Bits             | 1            |
| Hardware Flow Control | Disable      |

### TIM2

TIM2 is used to generate a 1-second interrupt.

If the system clock is 72 MHz:

| Parameter      | Value                        |
| -------------- | ---------------------------- |
| Clock Source   | Internal Clock               |
| Prescaler      | 7199                         |
| Counter Period | 9999                         |
| NVIC           | Enable TIM2 global interrupt |

Calculation:

```text
72 MHz / 7200 / 10000 = 1 Hz
```

### PB5 External Interrupt

| Parameter         | Value                                                       |
| ----------------- | ----------------------------------------------------------- |
| GPIO Mode         | External Interrupt Mode with Falling edge trigger detection |
| Pull-up/Pull-down | Pull-up                                                     |
| NVIC              | Enable EXTI line interrupt                                  |

---

## OLED Display Format

Example:

```text
2026.06.08 Mon
08:09:15
2407224
2026是平年
```

If the current year is a leap year:

```text
2028.06.08 Thu
08:09:15
2407224
2028是闰年
```

Display description:

| Line   | Content                         |
| ------ | ------------------------------- |
| Line 1 | Date and weekday                |
| Line 2 | Current time                    |
| Line 3 | Student ID                      |
| Line 4 | Leap year or common year status |

---

## UART Commands

The serial assistant should use:

```text
Baud rate: 115200
Data bits: 8
Stop bits: 1
Parity: None
Send new line: enabled
```

### Change Time

Command format:

```text
change time:YYYYMMDD-HHMMSS-W
```

Where:

| Field | Meaning        |
| ----- | -------------- |
| YYYY  | Year           |
| MM    | Month          |
| DD    | Day            |
| HH    | Hour           |
| MM    | Minute         |
| SS    | Second         |
| W     | Weekday number |

Weekday definition:

| Number | Weekday |
| ------ | ------- |
| 1      | Mon     |
| 2      | Tue     |
| 3      | Wed     |
| 4      | Thu     |
| 5      | Fri     |
| 6      | Sat     |
| 7      | Sun     |

Example:

```text
change time:20260609-100105-2
```

If the date and weekday are correct, the board returns:

```text
change time ok!
```

If the date or weekday is wrong, the board returns:

```text
change time fail!
```

---

### Stop Time

Command:

```text
stop time
```

Return message:

```text
stop time ok!
```

---

### Start Time

Command:

```text
start time
```

Return message:

```text
start time ok!
```

---

## Test Cases

### Valid Time Modification

Input:

```text
change time:20260609-100105-2
```

Expected result:

```text
change time ok!
```

OLED display:

```text
2026.06.09 Tue
10:01:05
2407224
2026是平年
```

---

### Invalid Weekday Test

Input:

```text
change time:20260609-100105-3
```

Expected result:

```text
change time fail!
```

Reason:

```text
2026.06.09 is Tuesday, not Wednesday.
```

---

### Leap Year Carry Test

Input:

```text
change time:20000228-235958-1
```

Expected result:

```text
change time ok!
```

After 2 seconds, OLED should display:

```text
2000.02.29 Tue
00:00:00
2407224
2000是闰年
```

Reason:

```text
The year 2000 is a leap year.
```

---

### Common Year Test

Input:

```text
change time:20010228-235958-3
```

After 2 seconds, OLED should display:

```text
2001.03.01 Thu
00:00:00
2407224
2001是平年
```

Reason:

```text
The year 2001 is not a leap year.
```

---

## Main Logic

The project mainly contains the following logic:

1. `Calendar_IsLeapYear()`

   Determines whether the current year is a leap year.

2. `Calendar_GetMonthDays()`

   Returns the number of days in the current month.

3. `Calendar_CalcWeek()`

   Calculates the weekday according to the date.

4. `Calendar_IsValid()`

   Checks whether the input date, time, and weekday are valid.

5. `Calendar_AddOneSecond()`

   Adds one second and handles all carry operations.

6. `Calendar_ShowOLED()`

   Refreshes OLED display.
   Only the changed lines are refreshed to reduce flicker.

7. `Calendar_ProcessCommand()`

   Parses UART commands and performs the corresponding operation.

---

## Project Structure

A typical STM32CubeMX project structure is shown below:

```text
STM32F103-Calendar-Clock/
├── Core/
│   ├── Inc/
│   └── Src/
├── Drivers/
├── MDK-ARM/
├── README.md
└── STM32F103-Calendar-Clock.ioc
```

---

## Notes

* Do not delete the `USER CODE BEGIN` and `USER CODE END` areas generated by STM32CubeMX.
* If STM32CubeMX regenerates the code, check whether user code is still kept correctly.
* TIM2 must be configured as **Internal Clock**.
* TIM2 global interrupt must be enabled.
* USART2 interrupt must be enabled.
* If the OLED flickers, avoid clearing all four lines every second. Only refresh the line that changes.

---

## License

This project is released for learning and educational use.
