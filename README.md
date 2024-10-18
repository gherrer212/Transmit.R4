/*
  CANWrite with 120-ohm Resistor and D13 LED for Transmission Indication

  Write and send CAN Bus messages using a 120-ohm termination resistor for the Arduino R4 Minima.
  D13 pin will be toggled on/off with each successful message transmission.

  See the full documentation here:
  https://docs.arduino.cc/tutorials/uno-r4-wifi/can
*/

/**************************************************************************************
 * INCLUDE
 **************************************************************************************/

#include <Arduino_CAN.h>

/**************************************************************************************
 * CONSTANTS
 **************************************************************************************/

static uint32_t const CAN_ID = 0x20;  // CAN message ID
const int ledPin = 13;                // Pin D13 for onboard LED

/**************************************************************************************
 * SETUP/LOOP
 **************************************************************************************/

void setup()
{
  // Initialize Serial communication at 115200 baud rate
  Serial.begin(115200);
  while (!Serial) { }

  // Debug message: Confirm serial connection
  Serial.println("Serial communication started");

  // Initialize D13 as an output for the transmission indicator (LED)
  pinMode(ledPin, OUTPUT);
  digitalWrite(ledPin, LOW);  // Ensure LED is off initially

  // Begin CAN bus communication at 250k bit rate
  if (!CAN.begin(CanBitRate::BR_250k))
  {
    Serial.println("CAN.begin(...) failed. Please check connections.");
    for (;;) {}  // Infinite loop if CAN initialization fails
  }

  // Important: Ensure a 120-ohm termination resistor is present across CAN_H and CAN_L
  Serial.println("CAN communication started with 120-ohm termination.");
}

static uint32_t msg_cnt = 0;  // Message counter

void loop()
{
  // Debug message: Starting to send message
  Serial.print("Sending CAN message #");
  Serial.println(msg_cnt);

  // Assemble a CAN message with the format: 0xCA 0xFE 0x00 0x00 [4-byte message counter]
  uint8_t msg_data[] = {0xCA, 0xFE, 0, 0, 0, 0, 0, 0};  // Message data array
  memcpy((void *)(msg_data + 4), &msg_cnt, sizeof(msg_cnt));  // Copy the counter to the message

  CanMsg msg(CanStandardId(CAN_ID), sizeof(msg_data), msg_data);  // Create CAN message

  // Transmit the CAN message and capture/display any error codes if the transmission fails
  int rc = CAN.write(msg);
  if (rc < 0)
  {
    // Debug: Provide detailed error message
    Serial.print("CAN.write(...) failed with error code: ");
    Serial.println(rc);

    // Infinite loop to halt execution, but after showing error details
    for (;;) {}
  }
  else
  {
    // Debug: Successful transmission
    Serial.println("Message sent successfully");

    // Toggle the D13 LED each time a message is transmitted
    digitalWrite(ledPin, HIGH);  // Turn on LED
    delay(100);                  // Short blink to indicate transmission
    digitalWrite(ledPin, LOW);   // Turn off LED
  }

  // Increment the message counter
  msg_cnt++;

  // Send one message per second
  delay(1000);
}
