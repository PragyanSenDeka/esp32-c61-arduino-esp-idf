# ESP32-C61 + Arduino code, built with ESP-IDF

**Program an ESP32-C61 board with normal Arduino code, even though the Arduino IDE doesn't support it yet.**

This repository has a free, beginner-friendly lab manual and a ready-to-build example project. They show how to compile and flash Arduino sketches (`setup()`, `loop()`, `Serial`, `delay()`...) on the ESP32-C61 by using **ESP-IDF with the Arduino core as a component**.

![ESP-IDF](https://img.shields.io/badge/ESP--IDF-v5.5.5-blue)
![Arduino core](https://img.shields.io/badge/arduino--esp32-3.3.12-teal)
![Chip](https://img.shields.io/badge/chip-ESP32--C61-red)
![License](https://img.shields.io/badge/manual-CC%20BY%204.0-lightgrey)

### 📄 [Read the lab manual (PDF)](./ESP32-C61_Lab1_Hello_World_ESP-IDF.pdf)

64 pages and 37 diagrams, written for first-year engineering students. Every step and every new term is explained, from installing ESP-IDF to seeing `Hello world!` on the serial monitor.

---

## The problem

The ESP32-C61 is missing from the Arduino IDE's board list. If you pick the closest board (ESP32-C6), uploading fails with:

```
A fatal error occurred: This chip is ESP32-C61, not ESP32-C6
```

If you unhide the ESP32-C61 entry in `boards.txt`, compiling fails instead:

```
Invalid value for 'FILENAME': Path
'{runtime.tools.esp32c61-libs.path}\bin\bootloader_qio_80m.elf' does not exist.
```

**Why:** the Arduino IDE's ESP32 package ships ESP-IDF *pre-compiled* for each chip, and the ESP32-C61 files are not shipped yet. The arduino-esp32 project itself says the ESP32-C61 (and the ESP32-C2) currently require **"Arduino as an ESP-IDF component"**.

## The solution in one minute

Use ESP-IDF as the build system and add the Arduino core as a component. ESP-IDF then compiles everything from source for the C61, and your Arduino code stays Arduino code.

Run these in the **ESP-IDF terminal** (ESP-IDF **v5.5.x**), in a folder with no spaces in its path:

```powershell
cd D:\esp\projects
idf.py create-project-from-example "espressif/arduino-esp32^3.3.12:hello_world"
cd hello_world
```

Pin the Arduino core version: replace the contents of `main\idf_component.yml` with

```yaml
dependencies:
  espressif/arduino-esp32:
    version: '^3.3.12'
```

Then set the target, build and flash (replace `COM19` with your board's port):

```powershell
idf.py set-target esp32c61
idf.py build
idf.py -p COM19 flash monitor
```

The first build takes 5–15 minutes. Press `Ctrl + ]` to leave the monitor. The [manual](./ESP32-C61_Lab1_Hello_World_ESP-IDF.pdf) explains what every one of these lines does.

## Tested with

| | Version |
|---|---|
| Board | ESP32-C61 development board (CP210x USB-to-UART bridge) |
| ESP-IDF | v5.5.5 |
| Arduino core (arduino-esp32) | 3.3.12, as an ESP-IDF component |
| esptool | v4.12.0 |
| Computer | Windows 10/11 |

> ⚠️ **Version matters.** Arduino core 3.3.x is designed for **ESP-IDF v5.5**. Espressif's newest installer can install ESP-IDF v6.x by default, so pick **v5.5.5** explicitly. Chapter 3 of the manual shows how.

## Works for other ESP chips too

Only one command changes: `idf.py set-target <chip>`.

| Chip | Arduino IDE today | This method |
|---|---|---|
| ESP32-C61 | ❌ not supported | ✅ `idf.py set-target esp32c61` |
| ESP32-C2 | ❌ not supported | ✅ `idf.py set-target esp32c2` (many boards need a 26 MHz crystal setting; see manual ch. 6.4) |
| ESP32, S2, S3, C3, C5, C6, H2, P4 | ✅ supported | ✅ also works |

Chapter 6 of the manual covers boards with native USB, bootloader addresses, and how to check versions for chips released in the future.

## What's in this repository

```
.
├── ESP32-C61_Lab1_Hello_World_ESP-IDF.pdf   the lab manual
├── examples/
│   └── hello_world/                         ready-to-build ESP-IDF + Arduino project
│       ├── CMakeLists.txt
│       ├── sdkconfig.defaults
│       └── main/
│           ├── CMakeLists.txt
│           ├── idf_component.yml            Arduino core pinned to ^3.3.12
│           └── main.cpp                     the sketch
├── LICENSE
└── README.md
```

`build/`, `managed_components/` and `sdkconfig` are not included on purpose: ESP-IDF creates them automatically the first time you run `idf.py set-target`.

## Common problems

| Symptom | Fix |
|---|---|
| `'idf.py' is not recognized` | You're in a normal terminal. Open the **ESP-IDF** terminal from the Start menu. |
| Stuck at `Connecting......` | Hold **BOOT**, tap **RST**, release **BOOT**, then retry. Try a different USB cable. |
| `Access is denied` / port busy | Close the Arduino IDE Serial Monitor or any other program using the port. |
| Garbage characters in the monitor | Baud mismatch: use `Serial.begin(115200)`. On ESP32-C2 boards, check the crystal setting. |
| Strange build errors after changes | `idf.py fullclean`, then `idf.py build`. |

The manual has a full troubleshooting chapter.

## What the manual covers

- How compiling and flashing work, and why "Upload" is really two jobs
- Software layers: your sketch, the Arduino core, ESP-IDF, FreeRTOS
- Why the Arduino IDE fails on the C61, and how ESP-IDF components fix it
- Installing ESP-IDF v5.5.5 on Windows; terminal basics, PATH and `idf.py`
- The 8-step lab: create, configure, build, find the port, flash, monitor
- Inside the build: object files, the linker, `.elf` vs `.bin`, hexadecimal sizes
- Flash partitions, the boot sequence, download mode, auto-reset, hash verification
- Using the same method for other ESP chips
- Troubleshooting, quick reference, glossary and self-test questions

## Author

Prepared by **Pragyan Sen Deka**, M.Tech student, Robotics & AI, IIT Guwahati.

Found a mistake, or got it working on another board? Please [open an issue](../../issues). Feedback and corrections are welcome.

> This is an independent guide. It is not an official publication of Espressif Systems, Arduino, IIT Guwahati or any other organisation.

## License

- **Lab manual (PDF):** [Creative Commons Attribution 4.0 (CC BY 4.0)](https://creativecommons.org/licenses/by/4.0/). You may share and adapt it, including for teaching, as long as you credit **Pragyan Sen Deka** and link back to this repository.
- **Example code:** MIT License (see `LICENSE`).

## References

- [Arduino as an ESP-IDF component](https://docs.espressif.com/projects/arduino-esp32/en/latest/esp-idf_component.html), Arduino-ESP32 documentation
- [arduino-esp32 on GitHub](https://github.com/espressif/arduino-esp32): supported chips and the C2/C61 note
- [espressif/arduino-esp32 in the ESP Component Registry](https://components.espressif.com/components/espressif/arduino-esp32)
- [ESP-IDF Windows installers](https://dl.espressif.com/dl/esp-idf/)
- [ESP32-C61 datasheet](https://documentation.espressif.com/esp32-c61_datasheet_en.html)

---

⭐ If this helped you get your ESP32-C61 running, consider starring the repository so others can find it.
