/*
  CANWrite with 120-ohm Resistor

  Write and send CAN Bus messages using a 120-ohm termination resistor for the Arduino R4 Minima.

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

/**************************************************************************************
 * SETUP/LOOP
 **************************************************************************************/

void setup()
{
  // Initialize Serial communication at 115200 baud rate
  Serial.begin(115200);
  while (!Serial) { }

  // Begin CAN bus communication at 250k bit rate
  if (!CAN.begin(CanBitRate::BR_250k))
  {
    Serial.println("CAN.begin(...) failed.");
    for (;;) {}  // Infinite loop if CAN initialization fails
  }

  // Important: Ensure a 120-ohm termination resistor is present across CAN_H and CAN_L
  Serial.println("CAN communication started with 120-ohm termination.");
}

static uint32_t msg_cnt = 0;  // Message counter

void loop()
{
  // Assemble a CAN message with the format: 0xCA 0xFE 0x00 0x00 [4-byte message counter]
  uint8_t const msg_data[] = {0xCA, 0xFE, 0, 0, 0, 0, 0, 0};  // Message data array
  memcpy((void *)(msg_data + 4), &msg_cnt, sizeof(msg_cnt));  // Copy the counter to the message
  CanMsg const msg(CanStandardId(CAN_ID), sizeof(msg_data), msg_data);  // Create CAN message

  // Transmit the CAN message and capture/display any error codes if the transmission fails
  if (int const rc = CAN.write(msg); rc < 0)
  {
    Serial.print  ("CAN.write(...) failed with error code ");
    Serial.println(rc);
    for (;;) { }  // Infinite loop if CAN transmission fails
  }

  // Increment the message counter
  msg_cnt++;

  // Send one message per second
  delay(1000);
}
