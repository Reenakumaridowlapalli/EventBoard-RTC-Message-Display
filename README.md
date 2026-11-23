<div align="center">

# 📟 EventBoard
### RTC-Driven Message Display System on LPC2148

*A smart notice board that shows the right message at the right time - no manual updates needed.*

</div>

---

## 📑 Table of Contents
1. [About the Project](#-about-the-project)
2. [At a Glance](#-at-a-glance)
3. [Features](#-features)
4. [System Architecture](#-system-architecture)
5. [Hardware & Software](#-hardware--software)
6. [How the System Behaves](#-how-the-system-behaves)
7. [Admin Mode](#-admin-mode)
8. [Message Database](#-message-database)
9. [Pin Mapping](#-pin-mapping)
10. [Code Organisation](#-code-organisation)
11. [Getting Started](#-getting-started)
12. [Troubleshooting](#-troubleshooting)
13. [Skills Demonstrated](#-skills-demonstrated)
14. [Future Scope](#-future-scope)
15. [Author](#-author)

---

## 📖 About the Project

**EventBoard** turns an LPC2148 into a self-running announcement board. Ten predefined messages are stored in the controller's memory, each tied to a time of day. The on-chip **RTC** keeps track of the clock, and whenever the time matches a message, it scrolls across a **16x2 LCD**.

When nothing is scheduled, the board does not sit idle - it shows the **live clock and room temperature** (from an **LM35** sensor read via the on-chip **ADC**).

Settings are protected. A user presses the **Admin switch**, enters a **keypad password**, and can then **edit the RTC time** and **choose which messages are active** - all without reflashing the firmware.

## ⚡ At a Glance

| | |
|---|---|
| 🧠 **Controller** | LPC2148 (ARM7TDMI-S) |
| 🕒 **Scheduler** | On-chip RTC |
| 📺 **Display** | 16x2 character LCD with scrolling |
| ⌨️ **Input** | 4x4 matrix keypad + admin switch |
| 🌡️ **Sensor** | LM35 via 10-bit ADC |
| 🚦 **Indicators** | 🟢 Green LED (message active) / 🔴 Red LED (idle) |
| 🔔 **Alert** | Buzzer |
| 💾 **Messages** | 10 stored, all enabled by default |

## ✨ Features

- 🕒 **Hands-free scheduling** - messages fire automatically when RTC time matches
- 📜 **Auto-scrolling text** - messages up to 80 characters fit on a 16-character line
- 🌡️ **Live temperature** shown alongside the clock in idle mode
- 🔐 **Password-protected configuration**
- 🛠️ **Interactive RTC editing** straight from the keypad
- ✅ **Per-message ON/OFF control** for the day's schedule
- 🚦 **Instant visual status** through green/red LEDs
- 🧩 **Modular firmware** - one driver per peripheral

## 🏗️ System Architecture

```
                         ┌───────────────────────────┐
  ┌──────────────┐       │                           │       ┌──────────────┐
  │ 4x4 KEYPAD   │──────▶│                          │──────▶│  16x2 LCD    │
  └──────────────┘       │                           │       └──────────────┘
  ┌──────────────┐       │         LPC2148           │       ┌──────────────┐
  │ ADMIN SWITCH │──────▶│                          │──────▶│  GREEN LED   │
  └──────────────┘       │   ┌─────┐       ┌─────┐   │       └──────────────┘
  ┌──────────────┐       │   │ RTC │       │ ADC │   │       ┌──────────────┐
  │    LM35      │──────▶│  └─────┘       └─────┘   │──────▶│   RED LED    │
  └──────────────┘       │                           │       └──────────────┘
                         └───────────────────────────┘
```

## 🧰 Hardware & Software

### Hardware
| Component | Role |
|-----------|------|
| LPC2148 development board | Brain of the system |
| 16x2 LCD | Shows messages, clock, temperature, menus |
| 4x4 keypad | Password and menu input |
| Push switch | Triggers Admin Mode |
| LM35 | Analog temperature sensing |
| Green + Red LEDs | Mode indication |
| Buzzer | Audible feedback |

### Software
| Tool | Use |
|------|-----|
| Embedded C | Firmware |
| Keil µVision | Compile & build |
| Flash Magic | Flash the `.hex` to the board |
| Proteus *(optional)* | Simulation |

## 🔄 How the System Behaves

```
        ┌──────────┐
        │ Power ON │
        └────┬─────┘
             ▼
     ┌─────────────────┐
     │ Initialise      │  LCD, RTC, ADC, keypad, LEDs, switch
     └────┬────────────┘
          ▼
   ┌─────────────────────────────┐
   │ Read RTC time + temperature │◀─────────────┐
   └────┬────────────────────────┘              │
        ▼                                       │
 ┌──────────────────────┐  YES   ┌─────────────────────────────┐
 │ Time matches an      │──────▶│ Scroll message  | 🟢 ON     │
 │ ENABLED message?     │        └──────────────┬──────────────┘
 └────┬─────────────────┘                       │
      │ NO                                      │
      ▼                                         │
 ┌─────────────────────────────┐                │
 │ Show clock + temp | 🔴 ON   │ ───────────────┘
 └─────────────────────────────┘
          ▲
          │ Admin switch pressed at any time → Admin Mode
```

**Screen examples**

```
 IDLE MODE  (🔴)              ADMIN LOGIN
┌────────────────┐           ┌────────────────┐
│Time : 09:20:45 │           │Enter Password  │
│Temp : 28 C     │           │****            │
└────────────────┘           └────────────────┘
```

## 🔐 Admin Mode

1. Press the **Admin switch**.
2. Enter the **password** on the keypad.
3. Pick an option from the menu:

| Option | What you can do |
|--------|-----------------|
| ⏱️ **Edit RTC** | Step through hour → minute → second (and date fields, if enabled) and type in new values |
| 🗂️ **Select Messages** | Choose a message number and toggle it ON / OFF |
| 🚪 **Exit** | Save and return to normal operation |

> 🔒 A wrong password denies access and the board continues in normal mode.

## 🗓️ Message Database

Each message is stored as a struct:

```c
#define TOTAL_MESSAGES 10

typedef struct {
    u8   hour;
    u8   minute;
    char text[80];
    u8   enabled;   // 1 = show, 0 = skip
} Message;
```

**Default schedule (all enabled at startup):**

| # | ⏰ Time | 💬 Message |
|---|--------|-----------|
| 1 | 07:45 | Good Morning! Classes Start Soon |
| 2 | 13:45 | C Programming Session in Classroom number 2 |
| 3 | 10:15 | C module theory exam in 4th floor lab1 |
| 4 | 10:15 | C module lab exam in 4th floor lab2 and middle lab at 10:30AM |
| 5 | 12:45 | Lunch Break from 1PM - 2PM |
| 6 | 09:45 | ARM Workshop on external interrupts in LAB1 at 10AM |
| 7 | 09:45 | ARM kit issue time from 10AM - 10:30AM in middle lab |
| 8 | 15:15 | Only 15 mins break time for next ARM session |
| 9 | 17:00 | Revise today's Class programs at home! |
| 10 | 17:45 | End of Day - See You Tomorrow! |

> ℹ️ Messages 3 & 4 share **10:15**, and 6 & 7 share **09:45**. These are shown one after the other.

## 📌 Pin Mapping

> ✏️ Replace with the pins used in your code / schematic.

| Peripheral | Signal | LPC2148 Pin |
|-----------|--------|-------------|
| LCD | Data lines | `P0.__ - P0.__` |
| LCD | RS / RW / EN | `P0.__` / `P0.__` / `P0.__` |
| Keypad | Rows | `P1.__ - P1.__` |
| Keypad | Columns | `P1.__ - P1.__` |
| Admin switch | Input | `P0.__` |
| LM35 | Analog out | `AD0.__` |
| Green LED | Output | `P0.__` |
| Red LED | Output | `P0.__` |
| Buzzer | Output | `P0.__` |

## 📂 Code Organisation

```
EventBoard/
├── main.c                  # Application flow
├── admin.c / admin.h       # Password check, RTC edit, message select
├── rtc.c / rtc.h           # RTC init, read, write
├── rtc_defines.h
├── lcd.c / lcd.h           # LCD driver + scrolling
├── lcd_defines.h
├── kpm.c / kpm.h           # Keypad scanning
├── kpm_defines.h
├── adc.c / adc.h           # ADC + LM35 conversion
├── adc_defines.h
├── delay.c / delay.h       # Delay routines
├── pin_connect_block.c/.h  # Pin function selection
├── defines.h               # Common macros
├── interrupts_defines.h    # Interrupt-related macros
├── types.h                 # u8 / u16 / u32 typedefs
├── Startup.s               # Startup code
└── README.md
```

## 🚀 Getting Started

```bash
# 1. Clone the repository
git clone https://github.com/<your-username>/<your-repo-name>.git
```

1. 📂 Open the project in **Keil µVision**.
2. 🔨 **Build** the target (`F7`) to generate the `.hex` file.
3. 🔌 Connect the board over USB-UART and enter ISP mode.
4. ⚡ In **Flash Magic**, select `LPC2148`, set the COM port & baud rate, load the `.hex`, and click **Start**.
5. 🔁 Reset the board - EventBoard is live!

> 🌡️ **LM35 formula:** `Temperature (°C) = (ADC value × Vref / 1023) × 100`

## 🩺 Troubleshooting

| Problem | Likely cause / fix |
|---------|-------------------|
| LCD shows blocks or nothing | Check contrast pot, RS/EN wiring, and init sequence |
| Wrong temperature | Verify Vref, ADC channel, and LM35 supply |
| Keypad gives wrong keys | Re-check row/column pin order and debouncing |
| RTC time resets on power-off | RTC battery / VBAT not connected |
| Flash Magic can't connect | Check COM port, baud rate and ISP/reset sequence |

## 🎓 Skills Demonstrated

- ARM7 (LPC2148) architecture & peripheral programming
- RTC, ADC, GPIO and interrupt/switch handling
- LCD and matrix keypad interfacing
- Analog sensor interfacing (LM35)
- Menu-driven, modular embedded C design

## 🔮 Future Scope

- 💾 Save message ON/OFF flags in non-volatile memory
- 🔑 Change password from Admin Mode
- 📅 Day/date-based scheduling
- 📡 UART / Bluetooth / Wi-Fi message updates

## 👩‍💻 Author

**Your Name**
🎓 Embedded Systems - Vector India
🔗 [GitHub](https://github.com/Reenakumaridowlapalli) · [LinkedIn](https://linkedin.com/in/<your-profile>)

---

<div align="center">⭐ If this project helped you, give it a star! ⭐</div>
