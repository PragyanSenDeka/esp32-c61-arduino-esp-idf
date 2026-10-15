# ESP32-C61 + Arduino code, built with ESP-IDF

**Program an ESP32-C61 board with normal Arduino code and Arduino libraries, even though the Arduino IDE doesn't support it yet.**

This repository has free, beginner-friendly lab manuals. They show how to compile and flash Arduino sketches (`setup()`, `loop()`, `Serial`, `delay()`...) on the ESP32-C61 by using **ESP-IDF with the Arduino core as a component**, and how to bring your existing Arduino projects and their libraries along.

![ESP-IDF](https://img.shields.io/badge/ESP--IDF-v5.5.5-blue)
![Arduino core](https://img.shields.io/badge/arduino--esp32-3.3.12-teal)
![Chip](https://img.shields.io/badge/chip-ESP32--C61-red)
![License](https://img.shields.io/badge/manuals-CC%20BY%204.0-lightgrey)

## 📚 The labs

| Lab | What you'll do | Manual |
|---|---|---|
| **Lab 1: Hello, ESP32-C61!** | Install ESP-IDF, create an Arduino-ready project, build and flash `Hello world!`. Covers how compiling, flashing and booting really work. | 📄 [Read Lab 1 (PDF)](./ESP32-C61_Lab1_Hello_World_ESP-IDF2.pdf) |
| **Lab 2: Your Arduino Projects on ESP-IDF** | Port a real Arduino sketch and its Library Manager libraries (MD_Parola + MD_MAX72XX) to scroll text on a MAX7219 LED matrix. Includes a recipe for any library. | 📄 [Read Lab 2 (PDF)](./lab2-arduino-projects/ESP32-C61_Lab2_Arduino_Projects_on_ESP-IDF.pdf) |

Both are written for first-year engineering students: every step and every new term is explained, with plenty of diagrams. Start with Lab 1.

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

## The solution in one minute (Lab 1)

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

The first build takes 5–15 minutes. Press `Ctrl + ]` to leave the monitor. The [Lab 1 manual](./ESP32-C61_Lab1_Hello_World_ESP-IDF2.pdf) explains what every one of these lines does.

## Bringing your own Arduino projects (Lab 2 in one minute)

ESP-IDF doesn't search your Arduino libraries folder. Every library must become a **component**: a folder with a `CMakeLists.txt` inside.

1. Copy a working project's five recipe files (`CMakeLists.txt`, `sdkconfig.defaults` and the three files in `main\`) into a new folder.
2. Copy each Library Manager library into a `components\` folder inside it. Don't copy built-in libraries such as `Wire`, `SPI` or `WiFi`: they are already part of the Arduino core.
3. Give each library a `CMakeLists.txt`:

   ```cmake
   idf_component_register(SRC_DIRS "src"
                          INCLUDE_DIRS "src"
                          REQUIRES arduino-esp32 <other libraries it needs>)
   ```

4. Put your sketch in `main\main.cpp`, add `#include "Arduino.h"` at the top, and move any pins that aren't safe on your board. On the C61, GPIO 14–17 and 19–21 belong to the flash/PSRAM chip.
5. List the libraries in `main\CMakeLists.txt`. This is the step everyone forgets:

   ```cmake
   idf_component_register(SRCS "main.cpp"
                          INCLUDE_DIRS "."
                          REQUIRES arduino-esp32 <libraries your sketch includes>)
   ```

6. `idf.py set-target esp32c61`, `idf.py build`, `idf.py -p COMx flash monitor`.

The [Lab 2 manual](./lab2-arduino-projects/ESP32-C61_Lab2_Arduino_Projects_on_ESP-IDF.pdf) walks through this with a real project and covers other library layouts, dependencies and troubleshooting.

## Tested with

| | Version |
|---|---|
| Board | ESP32-C61-DevKitC-1 (CP210x USB-to-UART bridge) |
| ESP-IDF | v5.5.5 |
| Arduino core (arduino-esp32) | 3.3.12, as an ESP-IDF component |
| esptool | v4.12.0 |
| Lab 2 hardware | 4-in-1 MAX7219 LED matrix (FC16 type) on GPIO 22, 23, 24 |
| Computer | Windows 10/11 |

> ⚠️ **Version matters.** Arduino core 3.3.x is designed for **ESP-IDF v5.5**. Espressif's newest installer can install ESP-IDF v6.x by default, so pick **v5.5.5** explicitly. Chapter 3 of the Lab 1 manual shows how.

## Works for other ESP chips too

Only one command changes: `idf.py set-target <chip>`.

| Chip | Arduino IDE today | This method |
|---|---|---|
| ESP32-C61 | ❌ not supported | ✅ `idf.py set-target esp32c61` |
| ESP32-C2 | ❌ not supported | ✅ `idf.py set-target esp32c2` (many boards need a 26 MHz crystal setting; see Lab 1, ch. 6.4) |
| ESP32, S2, S3, C3, C5, C6, H2, P4 | ✅ supported | ✅ also works |

Chapter 6 of the Lab 1 manual covers boards with native USB, bootloader addresses, and how to check versions for chips released in the future.

## What's in this repository

```
.
├── ESP32-C61_Lab1_Hello_World_ESP-IDF2.pdf        Lab 1 manual
├── lab2-arduino-projects/
│   └── ESP32-C61_Lab2_Arduino_Projects_on_ESP-IDF.pdf   Lab 2 manual
└── README.md
```

The full Lab 2 example (`main.cpp` and every `CMakeLists.txt`) is printed in the Lab 2 manual, so you can recreate the project from it.

## Common problems

| Symptom | Fix |
|---|---|
| `'idf.py' is not recognized` | You're in a normal terminal. Open the **ESP-IDF** terminal from the Start menu. |
| `fatal error: X.h: No such file or directory` | Add the library's folder name to `REQUIRES` in `main\CMakeLists.txt`, and check the library has its own `CMakeLists.txt`. |
| `undefined reference to ...` | The library's `.cpp` files weren't compiled: point `SRC_DIRS` at the folder that holds them. |
| Board keeps restarting after flashing | Your sketch uses flash pins (GPIO 14–17, 19–21 on the C61). Move to free pins. |
| Stuck at `Connecting......` | Hold **BOOT**, tap **RST**, release **BOOT**, then retry. Try a different USB cable. |
| `Access is denied` / port busy | Close the Arduino IDE Serial Monitor or any other program using the port. |
| Garbage characters in the monitor | Baud mismatch: use `Serial.begin(115200)`. On ESP32-C2 boards, check the crystal setting. |
| Strange build errors after changes | `idf.py fullclean`, then `idf.py build`. |

Each manual has a full troubleshooting chapter.

## What the manuals cover

**Lab 1**
- How compiling and flashing work, and why "Upload" is really two jobs
- Software layers: your sketch, the Arduino core, ESP-IDF, FreeRTOS
- Why the Arduino IDE fails on the C61, and how ESP-IDF components fix it
- Installing ESP-IDF v5.5.5 on Windows; terminal basics, PATH and `idf.py`
- The 8-step lab: create, configure, build, find the port, flash, monitor
- Inside the build: object files, the linker, `.elf` vs `.bin`, hexadecimal sizes
- Flash partitions, the boot sequence, download mode, auto-reset, hash verification
- Using the same method for other ESP chips

**Lab 2**
- Why ESP-IDF needs a `CMakeLists.txt` for every library, and what header files are
- What `REQUIRES` controls, and the error you get when it's missing
- Choosing safe pins, and why GPIO 19–21 are off-limits on the C61
- The 9-step lab: porting the MD_Parola LED matrix project
- A recipe for any library: folder layouts, `depends=`, built-in libraries, AVR-only libraries

Both include troubleshooting, a quick reference, a glossary and self-test questions.

## Author

Prepared by **Pragyan Sen Deka**, Associate , IIM Calcutta Innovation Park Technology and Innovation Council ,M.Tech student, Robotics & AI, IIT Guwahati.

Found a mistake, or got it working on another board? Please [open an issue](../../issues). Feedback and corrections are welcome.

> This is an independent guide. It is not an official publication of Espressif Systems, Arduino, IIT Guwahati or any other organisation.

## License

- **Lab manuals (PDFs):** [Creative Commons Attribution 4.0 (CC BY 4.0)](https://creativecommons.org/licenses/by/4.0/). You may share and adapt them, including for teaching, as long as you credit **Pragyan Sen Deka** and link back to this repository.

## References

- [Arduino as an ESP-IDF component](https://docs.espressif.com/projects/arduino-esp32/en/latest/esp-idf_component.html), Arduino-ESP32 documentation
- [arduino-esp32 on GitHub](https://github.com/espressif/arduino-esp32): supported chips and the C2/C61 note
- [espressif/arduino-esp32 in the ESP Component Registry](https://components.espressif.com/components/espressif/arduino-esp32)
- [ESP-IDF Windows installers](https://dl.espressif.com/dl/esp-idf/)
- [ESP32-C61 datasheet](https://documentation.espressif.com/esp32-c61_datasheet_en.html)
- [ESP32-C61-DevKitC-1 user guide](https://docs.espressif.com/projects/esp-dev-kits/en/latest/esp32c61/esp32-c61-devkitc-1/user_guide.html): header pin map
- [MD_Parola](https://github.com/MajicDesigns/MD_Parola) and [MD_MAX72XX](https://github.com/MajicDesigns/MD_MAX72XX) by majicDesigns

---

⭐ If this helped you get your ESP32-C61 running, consider starring the repository so others can find it.
