---
id: 45
title: Using Microblaze softcore processor
date: 2018-10-21
excerpt: |
  Xilinx provides its own softcore processor IP Microblaze to use in embedded systems application. In this post and a few that follow up, I will implement the Microblaze softcore processor on Spartan 3E FPGA. After that's done I will use Xilinx provided SDK(Software Development Kit) to write a C program to turn on LEDs using switches.
  
layout: post
permalink: /2018/10/21/using-microblaze-softcore-processor/
---

Xilinx provides its own softcore processor IP Microblaze to use in embedded systems application. In this post and a few that follow up, I will implement the Microblaze softcore processor on Spartan 3E FPGA. After that's done I will use Xilinx provided SDK(Software Development Kit) to write a C program to turn on LEDs using switches.

The whole step involved in setting up Microblaze system is somewhat lengthy but simple.

Step 1: Create a new project and select Embedded Processsor from the new source wizard. Enter file name and click next. 

![Step 1](/assets/images/Microblaze/Step1.png)

Step 2: On the next window it shows the summary of the file created and says XPS will be launched. Click finish

![Step 2](/assets/images/Microblaze/Step2.png)

Step 3: Xilinx Platform Studio will open up asking if you want to create a Base System using the BSB Wizard. Select yes.

![Step 3](/assets/images/Microblaze/Step3.png)

Step 4: On the Create New XPS Project window select the type of the Interconnect and click ok. In case of Spartan 3E FPGA, AXI interconnect system is not present so PLB system is selected as default.

![Step 4](/assets/images/Microblaze/Step4.png)

Step 5: On the Base System Builder window selct the option to create a new design and click next.

![Step 5](/assets/images/Microblaze/Step5.png)

Step 6: Select the Reset Polartiy as Active Low or Active High and click next. I selected Active High for this example.

![Step 6](/assets/images/Microblaze/Step6.png)

Step 7: On the System Configuration section, select the Single-Processor System and click next. Single-Processor System is easy to work with and is more than enough for this example.

![Step 7](/assets/images/Microblaze/Step7.png)

Step 8: On the next window enter the Reference Clock Frequency same as FPGA board clock frequency. Select Local Memory as required and click next. In case of Papilio One the frequency is 32 MHz.

![Step 8](/assets/images/Microblaze/Step8.png)

Step 9: On the Peripheral section add the two GPIO devices DIP_Switches and LEDS. Select GPIO Data Width as 4 for both the devices and click next.

![Step 9](/assets/images/Microblaze/Step9.png)

Step 10: On the Cache section it says no cacheable memory for the processor because I didn't add any cache memory. Click next.

![Step 10](/assets/images/Microblaze/Step10.png)

Step 11: Check the summary and see if the added devices are shown. Click finish

![Step 11](/assets/images/Microblaze/Step11.png)

Step 12: A Running Design window will pop up. Select ok

![Step 12](/assets/images/Microblaze/Step12.png)

Step 13: Now the Xilinx Platform Studio will open up with the configured processor system. Check for added devices in Bus Interfaces, Ports and Addresses window. Click on UCF File on the right side under Project Files.

![Step 13](/assets/images/Microblaze/Step13.png)

Step 14: Add the pin locations of the switches, leds, reset and clock.

![Step 14](/assets/images/Microblaze/Step14.png)

Step 15: After all the pin constraints are added click on the Project menu and select Design Rule Check.

![Step 15](/assets/images/Microblaze/Step15.png)

Step 16: After design rule check is completed it will show a warning saying floating connection. It doesn't cause any issue in the final implementation.

![Step 16](/assets/images/Microblaze/Step16.png)

Step 17: On the Hardware menu double click on Generate Netlist. This will take around 20-30 mins to complete.

![Step 17](/assets/images/Microblaze/Step17.png)

Step 18: After the process is complete on the console window it says "XST completed" Done!. Close the Platform Studio and go back to Project Navigator.

![Step 18](/assets/images/Microblaze/Step18.png)

Step 19: On the Processes window double click on Generate Top HDL Source.

![Step 19](/assets/images/Microblaze/Step19.png)

Step 20: Select the top module and double click on Generate Programming File. Ater that's done it will show a few warnings related to JTAG pins not connected.

![Step 20](/assets/images/Microblaze/Step20.png)

Step 21: Select the Microblaze System .xmp file and under processes tab double click on Export Hardware Design To SDK without Bitstream. This will open up the Software Design Kit where you can write a program in high level language for the processor.

![Step 21](/assets/images/Microblaze/Step21.png)

Step 22: Select a workplace destination and click ok.

![Step 22](/assets/images/Microblaze/Step22.png)

Step 23: On the File menu click on New -> Application Project.

![Step 23](/assets/images/Microblaze/Step23.png)

Step 24: Type the Project name and choose the required language. In this case I chose C and on the next window select example type as hello world. Click Finish

![Step 24](/assets/images/Microblaze/Step24.png)

Step 25: Navigate the project created earlier and under src select helloworld.c and rename it to Switch_led.c. 

![Step 25](/assets/images/Microblaze/Step25.png)

Double click on the renamed .c file and replace the default program with the below program.


```c
/***************************** Include Files *********************************/
#include <stdio.h>
#include "xparameters.h"
#include "xgpio.h"
#include "platform.h"

/************************** Variable Defintions ******************************/

XGpio Gpio_leds;  //Gpio_leds is user defined variable
XGpio Gpio_keys;  //Also same as above

int main()
{
	 int Status;
	 int i;
	 u32 Delay;
	 u32 DataRead;

	 init_platform();

	 Status = XGpio_Initialize(&Gpio_keys, XPAR_DIP_SWITCHES_DEVICE_ID);
	 if (Status != XST_SUCCESS) {
		  return XST_FAILURE;
	 }

	 XGpio_SetDataDirection(&Gpio_keys, 1, 0xFFFFFFFF);    //set Gpio_keys is input

	 Status = XGpio_Initialize(&Gpio_leds, XPAR_LEDS_DEVICE_ID);
	 if (Status != XST_SUCCESS) {
		  return XST_FAILURE;
	 }

	 XGpio_SetDataDirection(&Gpio_leds, 1, 0x0);         //set Gpio_leds is output

 while(1)
  {
	 DataRead = XGpio_DiscreteRead(&Gpio_keys, 1);

	 if((DataRead & 0x0f)!=0x0f){                      //if key is pushed
		for (Delay = 0; Delay < 2000; Delay++);
		DataRead = XGpio_DiscreteRead(&Gpio_keys, 1);  //read again
	 	if((DataRead & 0x0f)!=0x0f)
	 	{
 	 	      XGpio_DiscreteWrite(&Gpio_leds, 1, DataRead);
	 	}
	 }
	 else{                                             //if key is not pushed
			for (Delay = 0; Delay < 2000; Delay++);
			DataRead = XGpio_DiscreteRead(&Gpio_keys, 1);  //read again
		 	if((DataRead & 0x0f)==0x0f)
		 	{
	 	 	      XGpio_DiscreteWrite(&Gpio_leds, 1, DataRead);
		 	}
	 }
  }
    return 0;
}

```


Save the program file. This will automatically compile the program and after the compilation is done close SDK and go back to Project Navigator.

Step 26: Right click on the Hierarchy tab and select add source. Locate the .elf file from the SDK workplace -> debug folder and add it to the project. After the .elf file is added double click on Generate Programming File.

![Step 26](/assets/images/Microblaze/Step26.png)

Step 27: Once the process is completed view the Device Utilization Summary and note the resources used.

![Step 27](/assets/images/Microblaze/Step27.png)

Locate the bit file and upload it to the FPGA board.

![Switch_LED](/assets/images/Microblaze/Switch_LED.jpg)



 
