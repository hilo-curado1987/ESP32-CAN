# ESP32 CAN

## Introduction

- CAN stands for Controller Area Network (CAN bus). This protocol is very popular in automotive domain. In order to understand more about history, benefits, characteristics, message format of CAN, you can refer:
http://www.ni.com/white-paper/2732/en/
http://www.ti.com/lit/an/sloa101b/sloa101b.pdf
- As you knew in Introduction to ESP32, ESP32 also supports CAN interface. So I am going to make a demo for this with Arduino.
- In this demo, 2 ESP32 modules will be used: first module will send the string "hellocan" to second module. The second module will convert the string to upper case and respond it back to first module and first module will show the result in theTerminal.

## Hardware

 ESP32 only supplies CAN controller. So you need CAN transceiver for this demo. I bought 2 CAN trasceivers here.
 
 ![Connection](https://1.bp.blogspot.com/-nuM3-7ENLbE/WaqHAVw-vjI/AAAAAAAAEPk/HsVOT7jnwdkSv9iFcQOgCIfjSmYJ3xuQQCEwYBhgL/s320/esp32_CAN_1.jpg)]
 
 
 - ESP32 GPIO5 will act as CAN_Tx.
- ESP32 GPIO4 will act as CAN_Rx.
- So you can connect pins belows (ESP32_X: module ESP32 X, CAN_X: module CAN X, where X is 1 or 2 since we have 2 modules):
| ESP32 Pin Configuration | CAN Configuration |
| :----------------------| :----------------|
| ESP32_1 IO5 | CAN_1 CTX | 
| ESP32_1 IO4 | CAN_1 CRX |
| CAN_1 CANH | CAN_2 CANH |
| CAN_1 CANL | CAN_2 CANL |
| ESP32_2 IO5 | CAN_2 CTX |
| ESP32_2 IO4 | CAN_2 CRX |


## Software

- In order to make this demo, I used CAN driver which is made by Thomas Barth (Thanks Thomas :))
- Download the CAN library for ESP32 here. Unzip it and copy to "Arduino/libraries" folder.
- Source code for first ESP32 module: receiving string, converting to upper case and respond back:

#include <ESP32CAN.h>
#include <CAN_config.h>

/* the variable name CAN_cfg is fixed, do not change */
CAN_device_t CAN_cfg;

void setup() {
    Serial.begin(115200);
    Serial.println("iotsharing.com CAN demo");
    /* set CAN pins and baudrate */
    CAN_cfg.speed=CAN_SPEED_1000KBPS;
    CAN_cfg.tx_pin_id = GPIO_NUM_5;
    CAN_cfg.rx_pin_id = GPIO_NUM_4;
    /* create a queue for CAN receiving */
    CAN_cfg.rx_queue = xQueueCreate(10,sizeof(CAN_frame_t));
    //initialize CAN Module
    ESP32Can.CANInit();
}

void loop() {
    CAN_frame_t rx_frame;
    //receive next CAN frame from queue
    if(xQueueReceive(CAN_cfg.rx_queue,&rx_frame, 3*portTICK_PERIOD_MS)==pdTRUE){

      //do stuff!
      if(rx_frame.FIR.B.FF==CAN_frame_std)
        printf("New standard frame");
      else
        printf("New extended frame");

      if(rx_frame.FIR.B.RTR==CAN_RTR)
        printf(" RTR from 0x%08x, DLC %d\r\n",rx_frame.MsgID,  rx_frame.FIR.B.DLC);
      else{
        printf(" from 0x%08x, DLC %d\n",rx_frame.MsgID,  rx_frame.FIR.B.DLC);
        /* convert to upper case and respond to sender */
        for(int i = 0; i < 8; i++){
          if(rx_frame.data.u8[i] >= 'a' && rx_frame.data.u8[i] <= 'z'){
            rx_frame.data.u8[i] = rx_frame.data.u8[i] - 32;
          }
        }
      }
      //respond to sender
      ESP32Can.CANWriteFrame(&rx_frame);
    }
}



- Source code for second ESP32 module: sending the string that need to be converted to upper case, receiving response and show it to Terminal

## Result

![Results](https://1.bp.blogspot.com/-aoKC-qLvmxA/WaqHBfTtfCI/AAAAAAAAEPo/Rj9ecF_g1msfqKJeYWiQnZs-um-p3ReuQCLcBGAs/s640/esp32_CAN.png)]
