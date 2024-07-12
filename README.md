#include <BLEDevice.h>
#include <BLEServer.h>
#include <BLEUtils.h>
#include <BLE2902.h>

// Define UUIDs for our custom service and characteristics
#define SERVICE_UUID "00000002-0000-0000-FDFD-FDFDFDFDFDFD"
#define TEMPERATURE_CHAR_UUID "2A6E"
#define HUMIDITY_CHAR_UUID "2A6F"

// BLE Server and characteristics
BLEServer* pServer;
BLEService* pService;
BLECharacteristic* pTemperatureChar;
BLECharacteristic* pHumidityChar;

// Function to simulate temperature reading
int readTemperature() {
    // Simulate a temperature reading (e.g., range between 20-30 degrees Celsius)
    return 20 + (rand() % 11);
}

// Function to simulate humidity reading
int readHumidity() {
    // Simulate a humidity reading (e.g., range between 40-60%)
    return 40 + (rand() % 21);
}

void setup() {
    // Initialize serial communication for debugging
    Serial.begin(115200);

    // Initialize BLE
    BLEDevice::init("ESP32_BLE_Device");

    // Create BLE Server
    pServer = BLEDevice::createServer();

    // Create BLE Service
    pService = pServer->createService(SERVICE_UUID);

    // Create BLE Characteristics for temperature and humidity
    pTemperatureChar = pService->createCharacteristic(
        TEMPERATURE_CHAR_UUID,
        BLECharacteristic::PROPERTY_READ | BLECharacteristic::PROPERTY_NOTIFY
    );
    pTemperatureChar->addDescriptor(new BLE2902());

    pHumidityChar = pService->createCharacteristic(
        HUMIDITY_CHAR_UUID,
        BLECharacteristic::PROPERTY_READ | BLECharacteristic::PROPERTY_NOTIFY
    );
    pHumidityChar->addDescriptor(new BLE2902());

    // Start the service
    pService->start();

    // Start advertising
    BLEAdvertising* pAdvertising = BLEDevice::getAdvertising();
    pAdvertising->addServiceUUID(SERVICE_UUID);
    pAdvertising->setScanResponse(true);
    pAdvertising->setMinPreferred(0x06);  // functions that help with iPhone connections issue
    pAdvertising->setMinPreferred(0x12);
    BLEDevice::startAdvertising();

    Serial.println("BLE service started, waiting for connections...");
}

void loop() {
    // Simulate sensor readings
    int temperature = readTemperature();
    int humidity = readHumidity();

    // Update the BLE characteristics with new sensor values
    pTemperatureChar->setValue(temperature);
    pTemperatureChar->notify();

    pHumidityChar->setValue(humidity);
    pHumidityChar->notify();

    // Log the values for debugging
    Serial.printf("Temperature: %d C, Humidity: %d %%\n", temperature, humidity);

    // Wait for 2 seconds before updating again
    delay(2000);
}
