/*
  CANWrite

  Write and send CAN Bus messages from Arduino R4 Minima

  See the full documentation here:
  https://docs.arduino.cc/tutorials/uno-r4-wifi/can
*/

#include <CAN.h> // Include the CAN library for Arduino R4 Minima

void setup() {
  // Start Serial Monitor for debugging
  Serial.begin(115200);
  while (!Serial) {}

  // Initialize CAN bus at 500 kbps
  if (!CAN.begin(500E3)) {
    Serial.println("Starting CAN failed!");
    while (1);
  }

  Serial.println("CAN bus ready");
}

void loop() {
  // Start a new message
  CAN.beginPacket(0x123); // Send to ID 0x123
  CAN.write('H');
  CAN.write('e');
  CAN.write('l');
  CAN.write('l');
  CAN.write('o');
  CAN.endPacket(); // End the packet, and send the message
  
  Serial.println("Message sent: Hello");

  delay(1000); // Wait for a second before sending another message
}
