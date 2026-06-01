# PORTABLE COLORIMETRIC SYSTEM FOR RAPID MILK SPOILAGE DETECTION (Edge AI)

## 🥛 1. Comprehensive Project Overview
The **Smart Milk Quality Detection System** is an innovative embedded machine learning (Edge AI) project designed to instantly and non-destructively determine the freshness and safety of a milk sample.

In the dairy industry and home environments, milk spoilage is usually determined by smell, pH testing, or complex lab equipment. As milk spoils, lactic acid-producing bacteria break down the proteins, which subtly alters the optical density (opacity) and light reflectance of the liquid. This system uses a highly sensitive RGB color sensor to measure the precise light spectrum and opacity of the milk. The data is processed by an onboard microcontroller to classify the sample in real-time, instantly displaying the results via an OLED screen.

## ⚙️ 2. Architectural Structure & Files
- **/code**: Contains the production-ready `code.ino` firmware. This handles sensor initialization, reads light reflection variables, and drives the graphic OLED.
- **/colour_sensor**: Contains the `colour_sensor.ino` script. This is used for calibrating the ambient light environment and testing to ensure the I2C wires are connected properly.
- **/model creation**: Contains the AI workflow files:
  - `model creation steps.txt`: The step-by-step documentation detailing the Python/TensorFlow logic used in Google Colab to train the Multi-layer Perceptron.
  - `sample_collected.txt`: The raw dataset consisting of the mapped Red, Green, Blue, and classification vectors.
  - `model.h`: The compiled output of the TensorFlow model. This converts the complex floating-point model weights into an embedded hexadecimal `unsigned char` array that the microcontroller can store in its flash memory.

## 🔌 3. Hardware Requirements & Circuitry

### Core Components
1. **Microcontroller Board**: Arduino Uno, Arduino Nano, or an ESP32 for IoT capabilities.
2. **Adafruit TCS34725 RGB Color Sensor**: A sensitive optical sensor equipped with an integrated IR-blocking filter. This isolates the color spectrum to match human eye perception.
3. **Adafruit SSD1306 OLED Screen**: A 128x64 display that operates on I2C communication.

### I2C Wiring Guide (Standard Arduino Uno/Nano)
Both the Color Sensor and the OLED communicate using the **I2C protocol** (Inter-Integrated Circuit). Because I2C is a bus protocol, both devices share the exact same Arduino pins.

| Component Pin        | Arduino Pin      | Description                  |
| -------------------- | ---------------- | ---------------------------- |
| **VIN / VCC**        | 3.3V or 5V       | Power supplied to modules.   |
| **GND**              | GND              | Common ground.               |
| **SCL (Clock)**      | A5               | Synchronizes the I2C bus.    |
| **SDA (Data)**       | A4               | Carries the data payloads.   |

*Note: The TCS34725 sensor features a built-in bright white LED that emits light onto the milk sample. The reflected light bounces back into its photodiode array.*

## 🧠 4. Artificial Intelligence & Logic Walkthrough

### Part A: The Machine Learning Training Phase (Colab)
The Machine Learning aspects of this project were built using Python and Google's **TensorFlow / Keras** APIs. 
1. **Data Normalization**: Raw RGB values (ranging from 0-255) from the sensor were divided by 255.0 to map them between 0.0 and 1.0. This significantly improves neural network optimization.
2. **Neural Network Architecture**: A sophisticated 3-layer sequential network was built:
   - **Input Layer**: Analyzes the 3 normalized RGB values.
   - **Hidden Layers**: Two Dense layers (8 neurons and 6 neurons) running the `ReLU` (Rectified Linear Unit) activation function to identify non-linear lighting patterns.
   - **Output Layer**: A 3-neuron layer running `Softmax` outputting a probability percentage for three distinct classes (Fresh, Moderate, Unsafe).
3. **Compilation**: The model was converted to `TFLite` format resulting in `milk_model.tflite` to minimize memory. The Linux `xxd` command generated the `model.h` C-array.

### Part B: The Embedded C++ Logic Phase
While the `model.h` file contains advanced ML weights, **the current version of `code.ino` falls back to a rule-based heuristic**. It reads the `C` (Clear light) component from the sensor. Spoiled milk is visually thicker and reflects light differently, triggering the following thresholds:
   * **C > 2200**: Evaluated as `SPOILED`
   * **1400 <= C <= 2200**: Evaluated as `FRESH`
   * **1000 <= C < 1400**: Evaluated as `MODERATE`
   * **C < 1000**: Evaluated as `NORMAL/LIGHT`

## 🚀 5. Advanced Development Roadmap

To truly move this prototype into a production-level standard, the following phases are highly recommended:

### Phase 1: Activate the TensorFlow Lite Interpreter
The most critical missing layer in the current `code.ino` is that the neural network (stored dynamically in `milk_model_tflite`) is inactive. 
- **Action**: Install the `EloquentTinyML.h` or `TensorFlowLite_ESP32.h` libraries in the Arduino IDE.
- **Goal**: Rewrite the C++ logic so that the `loop()` function formats the latest sensor inputs into an `(R, G, B)` float array and feeds it strictly to the `.predict()` ML classifier instead of relying on hardcoded integer logic.

### Phase 2: Create a Controlled "Dark Chamber"
Light sensors are easily tricked by sun rays shining through a window or room lighting changing from fluorescent to warm incandescent. 
- **Action**: Design and 3D print a solid, opaque cup enclosure. The milk sample and TCS34725 sensor must sit inside this completely dark chamber.
- **Goal**: Ensure the only photons interacting with the milk and striking the sensor are produced by the TCS34725's onboard LED. This yields 99% accuracy consistency.

### Phase 3: Train at Scale
The existing neural network has memorized a minimal dataset of just 6 entries. Machine learning is heavily reliant on varied, dense patterns.
- **Action**: Diligently collect 150-300 samples mimicking completely spoiled, semi-spoiled, pure, mixed-water, and thick milk. Log the exact RGB readings against distinct timestamps.
- **Goal**: Re-train the Multi-layer Perceptron. A wider data baseline ensures the prototype performs brilliantly across any environment!

### Phase 4: Upgrade to Wi-Fi Dashboards
Arduino Unos lack internet capabilities. For agriculture management, analytics should be logged remotely.
- **Action**: Migrate from standard Arduino boards to the **ESP32** microcontroller. Use the ESP32 to ping a JSON payload to a web service like AWS IoT, ThingSpeak, or Blynk.
- **Goal**: Allow farm managers to see milk spoilage analytics on a mobile app in real-time without having to view the physical OLED display natively.
