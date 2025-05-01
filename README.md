# BootloaderStm32f411
## INTRO 

This is just a simple Bootloader project with stm32f411 some feature as below:
* Bootloader  will turn PD13 LED(orange led) on and off affter 3 seconds then send message via uart to pc
* Application   send message via uart to pc and toggle PD15 LED(blue led) after every 1 second.

Sound simple right? The target of this Project is just make a basic one and be able to reuse it.

I won't go into detail about what a bootloader is. 
This project is based on the following tutorial from EmbeTronicX (with some minor modifications):
[here](https://embetronicx.com/tutorials/microcontrollers/stm32/bootloader/simple-stm32-bootloader-implementation-bootloader-tutorial/)
## REQUIREMENT
You will need: 

    * Any STM32 board you have.The only thing that needs to be changed is the flash memory address. 
    * stm32cubeProgramer(or other tools that use may be st-link unity)
    * stm32cubeide(keilc, IAR)
    * Hercules to see message
    * CP2102 module to send message to the PC.
    * Some knowledge revalant to Futnction pointer and interupt vector table.
    
## RESULT
DEMO:

![Bootloader Demo GIF](https://raw.githubusercontent.com/nghiahuynhtv01042002/BootloaderStm32f411/readme-update/DEMO_IMG/Bootloader_demo.gif)

There are meassges send to PC
![Bootloader Demo PNG](https://raw.githubusercontent.com/nghiahuynhtv01042002/BootloaderStm32f411/readme-update/DEMO_IMG/Bootloder_demo.png)
## SOME EXPLAINATION
Since the bootloader and application are two independent programs, the flash memory will be divided into two regions with independently defined sizes. IN this project:

    Bootloader:  64K   (0x0800 0000 - 0x0800 FFFF)
    Application: 256K  (0x0801 0000 - 0x0804 FFFF)
    Free Space : 192K  (0x0805 0000 - 0x0807 FFFF)


    |----------------------------------------|
    |     FREE SPACE                         |   
    |----------------------------------------|
    |     Application's Main Function        |
    |----------------------------------------|
    |                                        |   
    |----------------------------------------|
    |     Application's Reset Handler        ||Interrupt
    |----------------------------------------||Vector
    |     Stack Memory's Start Address       ||Table's
    |----------------------------------------||app
    0x08010000 
    |----------------------------------------|
    |     Bootloader's Main Function         |  
    |----------------------------------------|
    |                                        |   
    |----------------------------------------|
    |     Bootloader's Reset Handler         ||Interrupt
    |----------------------------------------||Vector
    |     Stack Memory's Start Address       ||Table'
    |----------------------------------------||Bootloader
    0x08000000
               
### Interrupt vector table :

    The Interrupt Vector Table (IVT) is a data structure that maps each interrupt or exception to the address of its corresponding Interrupt Service Routine(isr).

Each firmware (bootloader and application) has its own IVT located at the start of its assigned flash :

    Bootloader IVT: starts at 0x08000000
    Application IVT: starts at 0x08010000


You can check it in **\Core\Startup\startup_stm32XXXX.s**

        .section  .isr_vector,"a",%progbits
        .type  g_pfnVectors, %object
            
        g_pfnVectors:
        .word  _estack
        .word  Reset_Handler
        .word  NMI_Handler
        .word  HardFault_Handler
        .word  MemManage_Handler
        .word  BusFault_Handler
        .word  UsageFault_Handler
        .word  0
        .word  0
        .word  0
        .word  0
        .word  SVC_Handler
        .word  DebugMon_Handler
 or using the command "path/arm-none-eabi-objdump.exe" -s -j file.elf > dump.lst. to see how it really store at assembly code 

    some example here
(type find /path -name "arm-none-eabi-objdump.exe" 2>/dev/null to find arm-none-eabi-objdump.exe if you are using shell bash).


### How the Bootloader Jumps to the Application
The bootloader must:

* Read the Application's Reset Handler Address from 0x08010004
* Set the Main Stack Pointer (MSP) to the value at 0x08010000 and Deinitialize Peripherals (optional but in my opinion we should do it adn re-init in application)
* Branch to the Application's Reset Handler which is stored at **(IVT address + 4)**

#### Note: 
in defaut the IVT is stored at 0x0800 0000.Therefore,In application the IVT is also stored at 0x0800 0000 is used by the bootloader so we need to change it by modify some line off code in **APPLICATION\Core\Src\system_stm32f4xx.c** 

    #define USER_VECT_TAB_ADDRESS
    #if defined(USER_VECT_TAB_ADDRESS)
    /*!< Uncomment the following line if you need to relocate your vector Table
        in Sram else user remap will be done in Flash. */
    /* #define VECT_TAB_SRAM */
    ======================/UNCOMEMNT HERE/=================
    #if defined(VECT_TAB_SRAM) 
    #define VECT_TAB_BASE_ADDRESS   SRAM_BASE       /*!< Vector Table base address field.
                                                        This value must be a multiple of 0x200. */
    #define VECT_TAB_OFFSET         0x00000000U     /*!< Vector Table base offset field.
                                                        This value must be a multiple of 0x200. */
    #else
    #define VECT_TAB_BASE_ADDRESS   FLASH_BASE      /*!< Vector Table base address field.
                                                        This value must be a multiple of 0x200. */
    ======================/EDIT HERE/=================

    #define VECT_TAB_OFFSET         0x00010000U     /*!< Vector Table base offset field.
                                                        This value must be a multiple of 0x200. */
    #endif /* VECT_TAB_SRAM */
    #endif /* USER_VECT_TAB_ADDRESS */
### Modify in linker scripts.
In Bootlaoder open **BOOTLOADER\STM32F411VETX_FLASH.ld** and modify flash memory block
    
    MEMORY
    {
    RAM    (xrw)    : ORIGIN = 0x20000000,   LENGTH = 128K
    FLASH    (rx)    : ORIGIN = 0x08000000,   LENGTH = 64K
    }
In Application open **APPLICATION\STM32F411VETX_FLASH.ld**and modify flash memory block

    MEMORY
    {
    RAM    (xrw)    : ORIGIN = 0x20000000,   LENGTH = 128K
    FLASH    (rx)    : ORIGIN = 0x08010000,   LENGTH = 256K
    }
## HOW TO FLASH CODE: 
Flash the Bootloader
Use STM32CubeIDE to flash the bootloader to the MCU.

Flash the Application
After the bootloader is flashed, use STM32CubeProgrammer to flash the application:

STM32CubeProgrammer is an easy-to-use tool; simply specify the address, and the tool will handle the rest.

# CONCLUSION
With the steps outlined above, you now have a basic bootloader implementation. This bootloader allows you to effectively transition from the bootloader to the application, manage the vector table correctly, and handle the memory regions for both the bootloader and the application.

Having this foundational bootloader gives you the flexibility to extend and reuse it for more advanced features, such as: Over-the-Air (OTA) Firmware Updates,etc

If you notice any mistakes or have suggestions, feel free to contact me through my GitHub