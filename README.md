# Hardware LED Matrix Snake 🐍

**[Read the Full Technical Report (PDF)][Uploading 8x8_LED_Matrix_Arcade_Console.pdf…]()**

A classic Snake game built using an ESP32, discrete 74-series logic ICs, and an 8x8 LED matrix. The entire circuit is manually routed and soldered onto a custom Vero board layout, demonstrating low-level hardware multiplexing alongside embedded C++ game logic.

![Status](https://img.shields.io/badge/Status-Completed-success)
![Platform](https://img.shields.io/badge/Platform-ESP32-blue)
![Hardware](https://img.shields.io/badge/Hardware-Discrete%20Logic-orange)

## 💀 The Origin Story (A Lesson in Hubris)
I initially set out to build this game completely "bare-metal" — absolutely no microcontrollers, just pure 74-series logic chips, a 555 timer clock signal, and sheer willpower. 

The goal was to build a 2D coordinate state machine entirely out of flip-flops and logic gates. However, after a few days, my breadboard looked less like an organized circuit and more like a copper-wire spaghetti monster. I quickly realized that tracking a dynamically growing snake body array using only discrete logic meant I was accidentally trying to build a very slow, very bad computer from scratch. 

I surrendered my ego to the Silicon Gods, pivoted to an ESP32 to handle the game state logic, and kept the 74-series ICs strictly for the hardware multiplexing and display driving. 

## 🚀 Key Features
* **Bare-Metal Multiplexing:** Custom matrix scanning routine written in C++ to achieve a flicker-free display by illuminating each row for exactly 800 microseconds.
* **Discrete Logic Integration:** Utilizing two 74HC595 shift registers in a cascaded configuration for signal routing rather than pre-built I2C display drivers.
* **Custom Vero Board Layout:** Point-to-point soldered permanent prototype, prioritizing stable power delivery and clean signal paths.
* **Interrupt-Driven Controls:** Push-button inputs utilize native loop injection and software-level state buffering for zero-latency directional changes without halting the multiplexing engine.

## 🛠️ Hardware Architecture

### Components
* **Microcontroller:** ESP32 Development Board (30-pin)
* **Display:** 8x8 Common-Cathode LED Matrix
* **Logic ICs:** 2x 74HC595 SIPO Shift Registers
* **Inputs:** 4x Tactile Push Buttons

### 🔌 Circuit Routing 
To avoid "pin starvation" on the ESP32, the architecture relies on serial-to-parallel expansion. Two 74HC595 shift registers are cascaded: the serial output pin (Q7') of the first register is tied directly to the serial data input (DS) pin of the second. The first register sinks current from the rows (Active-Low), while the second sources current to the columns (Active-High). This design enables the ESP32 to push 16 bits of display data simultaneously using only 3 GPIO pins.

### 🛒 Bill of Materials (BOM)
The entire physical prototype was built on a highly cost-effective budget.

| Component Name | Quantity | Approx. Cost (INR) |
| :--- | :--- | :--- |
| ESP32 Development Board (30-pin) | 1 | ₹380 |
| 8x8 LED Matrix (Common Cathode) | 1 | ₹80 |
| 74HC595 SIPO Shift Register IC | 2 | ₹30 |
| Tactile Push Buttons | 4 | ₹20 |
| Solderless Breadboard (830 Tie-Points) | 1 | ₹90 |
| Jumper Wires (Assorted) | - | ₹60 |
| 5V/2A DC Power Adapter & Jack | 1 | ₹120 |
| **Total** | | **₹780** |

## 🧠 Technical Challenges (That I Actually Solved)

### 1. Matrix Ghosting & Flicker
**Problem:** Early iterations of the multiplexing loop caused "ghosting" (dim LEDs turning on when they shouldn't) due to a data-bleed issue during serial transmission.
**Solution:** The firmware was modified to strictly pull the `LATCH_PIN` LOW before initiating the `shiftOut()` command, and only pulling it HIGH after all 16 bits were fully loaded into the registers, which isolated the output buffer.

### 2. Switch Bouncing and Double-Triggering
**Problem:** The tactile push-buttons exhibited mechanical bouncing, occasionally registering multiple inputs in a single millisecond.
**Solution:** Using a standard `delay()` to debounce would have ruined the 800-microsecond display refresh rate. Instead, a software-level state buffer (`nextDx`, `nextDy`) was implemented to ensure only one valid, orthogonal directional change is executed per game tick.

## 📸 Media
> ![Circuit Schematic]<img width="1920" height="1080" alt="Untitled design" src="https://github.com/user-attachments/assets/09616321-27e4-4a15-b2b0-02cd6f5e8cb1" />
> *Complete Circuit Schematic detailing the ESP32 to Shift Register cascading and Matrix wiring.*

> ![Vero Board Top View]<img width="933" height="1152" alt="image" src="https://github.com/user-attachments/assets/1ee2dde7-931d-4bb1-8cb5-af4b3cc0f919" />
> *Top view showing the IC placement and matrix.*

> ![Bottom Solder Joints]<img width="1145" height="1293" alt="IMG_20260323_224703716 jpg" src="https://github.com/user-attachments/assets/32ee2858-59d5-40de-ac38-88f577448f26" />
> *Bottom view demonstrating the point-to-point solder routing.*

---
*Created by Satyam Krishnan Dutta - Electronics & Communication Engineering, NIT Jamshedpur*
