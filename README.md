# heart-Attacks
/*
- A pulse sensor (SEN-11574) to measure heart rate using the photoplethysmogram technique[^10^] ¹¹¹².
- An ECG sensor (AD8232) to measure the electrical activity of the heart and calculate the cholesterol level using the oscillometric principle⁵⁶⁷.
- A pressure sensor (MPS20N0040D) to measure the blood pressure using the HX710B module¹⁴.

Here is an example of C code that reads the values from these sensors and displays them on a serial monitor:

c */
// Include the libraries for the sensors and modules
#include <Wire.h>
#include <PulseSensorPlayground.h>
#include <SparkFun_ADS1015_Arduino_Library.h>
#include <HX711.h>

// Define the pins for the sensors and modules
#define PULSE_PIN A0 // Pulse sensor analog input
#define ECG_PIN A1 // ECG sensor analog input
#define ADS_ADDR 0x48 // ECG sensor I2C address
#define DOUT_PIN 2 // Pressure sensor data pin
#define SCK_PIN 3 // Pressure sensor clock pin

// Create objects for the sensors and modules
PulseSensorPlayground pulseSensor; // Pulse sensor object
ADS1015 ads(ADS_ADDR); // ECG sensor object
HX711 scale(DOUT_PIN, SCK_PIN); // Pressure sensor object

// Define some variables for the sensor readings
int pulseRate; // Pulse rate in beats per minute (BPM)
int ecgValue; // ECG value in millivolts (mV)
float cholesterol; // Cholesterol level in milligrams per deciliter (mg/dL)
float pressure; // Blood pressure in millimeters of mercury (mmHg)

// Define some constants for the calculations
#define ECG_GAIN 1100 // ECG sensor gain factor
#define ECG_REF 3.3 // ECG sensor reference voltage
#define CHOL_A 0.002 // Cholesterol calculation coefficient A
#define CHOL_B 0.5 // Cholesterol calculation coefficient B
#define CHOL_C 100 // Cholesterol calculation coefficient C
#define PRES_A 0.004 // Pressure calculation coefficient A
#define PRES_B 0.5 // Pressure calculation coefficient B
#define PRES_C 50 // Pressure calculation coefficient C

void setup() {
  // Initialize serial communication
  Serial.begin(9600);

  // Initialize pulse sensor
  pulseSensor.analogInput(PULSE_PIN);
  pulseSensor.blinkOnPulse(LED_BUILTIN);
  pulseSensor.setThreshold(50);
  pulseSensor.begin();

  // Initialize ECG sensor
  ads.setGain(GAIN_TWOTHIRDS);
  ads.begin();

  // Initialize pressure sensor
  scale.set_scale();
  scale.tare();
}

void loop() {
  // Read pulse sensor
  int pulseValue = pulseSensor.getLatestSample();
  if (pulseSensor.sawStartOfBeat()) {
    pulseRate = pulseSensor.getBeatsPerMinute();
  }

  // Read ECG sensor
  int16_t ecgRaw = ads.readADC_SingleEnded(ECG_PIN);
  ecgValue = (ecgRaw * ECG_REF * 1000) / (ECG_GAIN * 4096);

  // Calculate cholesterol level from ECG value
  cholesterol = CHOL_A * ecgValue * ecgValue + CHOL_B * ecgValue + CHOL_C;

  // Read pressure sensor
  pressure = scale.get_units();

  // Calculate blood pressure from pressure value
  pressure = PRES_A * pressure * pressure + PRES_B * pressure + PRES_C;

  // Display the sensor readings on serial monitor
  Serial.print("Pulse Rate: ");
  Serial.print(pulseRate);
  Serial.println(" BPM");
  Serial.print("ECG Value: ");
  Serial.print(ecgValue);
  Serial.println(" mV");
  Serial.print("Cholesterol Level: ");
  Serial.print(cholesterol);
  Serial.println(" mg/dL");
  Serial.print("Blood Pressure: ");
  Serial.print(pressure);
  Serial.println(" mmHg");
  Serial.println();

  // Wait for a second
  delay(1000);
}
