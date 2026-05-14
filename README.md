# PORTABLE-COLORIMETRIC-SYSTEM-FOR-RAPID-MILK-SPOILAGE-DETECTION

## Overview
The **Smart Milk Quality Detection System** is an embedded Edge AI project that determines the freshness and quality of milk using color sensing. It analyzes the specific Red, Green, Blue, and Clear light profiles of a milk sample and outputs the classification (e.g., FRESH, SPOILED, MODERATE) in real time on an OLED screen.

## Project Structure
- **/code**: Contains the main Arduino sketch (`code.ino`) running the entire application (sensor reading, logic, and OLED display output).
- **/colour_sensor**: Contains a test sketch (`colour_sensor.ino`) to debug and calibrate the TCS34725 sensor independently.
- **/model creation**: Contains scripts and sample data used to train a neural network using TensorFlow. It also includes the exported `model.h` header file containing the TensorFlow Lite byte array.

## Technologies Used

### Hardware
- **Microcontroller**: Arduino compatible board (e.g., Arduino Uno, Nano, ESP32).
- **Color Sensor**: **Adafruit TCS34725** RGB Color Sensor (uses I2C communication).
- **Display**: **Adafruit SSD1306** OLED Display (128x64 resolution, uses I2C).

### Software & Libraries
- **Arduino IDE**: For writing and flashing firmware to the microcontroller.
- **Embedded Libraries**: `Wire.h`, `Adafruit_TCS34725.h`, `Adafruit_GFX.h`, `Adafruit_SSD1306.h`.
- **Machine Learning (Model Creation)**:
  - **Python / Google Colab**: Used as the primary environment for data processing and training.
  - **TensorFlow / Keras**: Used to construct and train a Multi-Layer Perceptron (Dense) neural network.
  - **Pandas & NumPy**: Used to structure the collected R, G, B samples into training datasets.
  - **TensorFlow Lite Converter**: Used to compress the model into a `.tflite` format so it can be converted to an embedded C-array (`model.h`).

## Current Implementation
1. The **TCS34725 sensor** gathers the Red, Green, Blue (RGB), and Clear (C) light vectors of the milk sample.
2. The current main logic algorithm uses a heuristic, rule-based approach taking the **Clear (C)** value to estimate quality:
   - `C > 2200` ➔ SPOILED
   - `C >= 1400` ➔ FRESH
   - `C >= 1000` ➔ MODERATE
3. Once processed, the exact parameters and the milk classification are driven over to the **SSD1306 OLED Screen**.

## Suggested Improvements & Future Scope

While the current foundation is excellent, there are a few important technical and hardware improvements that can upgrade this project into an enterprise-grade prototype:

### 1. Activating the TensorFlow Lite Edge AI
Currently, the `code.ino` file includes the Neural Network weights (`milk_model_tflite`), but the `loop()` uses basic `if/else` statements instead of running the AI model. 
* **Improvement**: We should integrate the `TensorFlowLite` or `EloquentTinyML` library. Instead of `if (c > 2200)`, we pass our `(R, G, B)` variables directly into the TFLite interpreter and let the Neural Network predict the precise status of the milk!

### 2. Hardware Enclosure & Ambient Light Control
The TCS34725 color sensor is heavily affected by ambient room light. A shadow passing over the sensor might falsely flag milk as "Spoiled."
* **Improvement**: Design a 3D-printed "dark box" or enclosure for the sensor. This guarantees that only the sensor's onboard white LED illuminates the milk sample, resulting in 100% consistent readings.

### 3. Model Training & Data Expansion
The dataset used in `model creation steps.txt` is quite small (only 6 samples). Neural networks require much more data to be highly accurate.
* **Improvement**: Collect at least 100-200 samples of milk in different stages of spoiling. This rich dataset will ensure the model captures the subtle color variations of degrading milk proteins.

### 4. Microcontroller Upgrade & IoT Dashboard
Standard Arduinos (like the Uno) have very limited RAM and might struggle to invoke TensorFlow Lite models. 
* **Improvement**: Swap the Arduino with an **ESP32** or **Arduino Nano 33 BLE Sense**. Using an ESP32 would also allow the system to upload the milk data to an IoT cloud dashboard (like Blynk or ThingSpeak), alerting a farmer or factory manager via Wi-Fi!
