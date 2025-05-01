# BootloaderStm32f411
## INTRO 

This is just a simple Bootloader project with stm32f411 some feature as below:
* Bootloader  will turn PD13 LED(orange led) on and off affter 3 seconds then send message via uart to pc
* Application   send message via uart to pc and T toggle PD15 LED(blue led) after every 1 second.

Sound simple right? The target of this Project is just make a basic one and be able to reuse it.

I won't go into detail about what a bootloader is. 
This project is based on the following tutorial from EmbeTronicX (with some minor modifications):
[here](https://embetronicx.com/tutorials/microcontrollers/stm32/bootloader/simple-stm32-bootloader-implementation-bootloader-tutorial/)
## REQUIREMENT
    You will need: 
        * Any STM32 board you have.The only thing that needs to be changed is the flash memory address. 
        * stm32cubeProgramer(or other tools that use may be st-link unity)
        * CP2102 module to send message to the PC.
        * Some knowledge revalant to Futnction pointer and interupt vector table.
    
## RESULT
DEMO:

![Bootloader Demo GIF](https://raw.githubusercontent.com/nghiahuynhtv01042002/BootloaderStm32f411/readme-update/DEMO_IMG/Bootloader_demo.gif)

There are meassges send to PC
![Bootloader Demo PNG](https://raw.githubusercontent.com/nghiahuynhtv01042002/BootloaderStm32f411/readme-update/DEMO_IMG/Bootloder_demo.png)