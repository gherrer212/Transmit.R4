/*
  CAN Bus Transmitter
  This code sends CAN messages from Arduino R4 Minima
*/

#include <Arduino_CAN.h>

const uint32_t CAN_ID = 0x123;  // Arbitrary CAN ID for messages
uint32_t msgCounter = 0;

void setup() {
  Serial.begin(115200);
  while (!Serial) {}

  // Initialize CAN bus at 250 kbps
  if (!CAN.begin(CanBitRate::BR_250k)) {
    Serial.println("CAN initialization failed.");
    while (true);
  }

  Serial.println("CAN Transmitter Ready");
}

void loop() {
  // Prepare CAN message with a simple counter
  uint8_t msg_data[8];
  msg_data[0] = 0xCA;
  msg_data[1] = 0xFE;
  msg_data[2] = 0xBE;
  msg_data[3] = 0xEF;
  memcpy(&msg_data[4], &msgCounter, sizeof(msgCounter)); // Last 4 bytes are the counter

  // Create CAN message
  CanMsg message(CanStandardId(CAN_ID), sizeof(msg_data), msg_data);

  // Send the CAN message
  if (CAN.write(message) == -1) {
    Serial.println("Error: CAN message not sent");
  } else {
    Serial.print("Message sent with counter: ");
    Serial.println(msgCounter);
  }

  msgCounter++; // Increment the message counter

  delay(1000); // Send one message per second
}
