# Chip Support Library for NXP/FreeScale KinetisKEA MCUs
This is a Low-layer Support Library for KinetisKE MCUs of NXP/Freescale, I called it as CSL


### ����  
 CSL��һ����STM32-HALΪ������Ƶ����KinetisKEAϵ��MCU����������⣬�ҽ�����Ϊ������ṩ�꾡��ʹ��˵��

    1. �������д��KinetisKEA128������ƽ̨(S9KEAZ128AMLK)��������֧��KEAZ128��KEAZ64����΢������
    2. �������ڽ��͸�������ڲ�����϶ȣ�Ŀǰ���м������������Ǳ���ģ����Ǹ��������ô��������ںˣ�ʱ�ӣ��ڲ�flash���Լ�������һЩ����Ķ���
    3. �ڱ��ʱ�����޸���startup(asm)�ļ���system_keaz128xxx4.h/.c���������ǿ������͹ٷ��ṩ���ļ���Щ��ͬ�������ʹ��CSLʱ�����ʹ�����ṩ���ļ�

 ### �ļ�˵��  
 �ϴ���githubʱ����ֻ�ṩ�����ļ���������ģ�弰���̽���release�з���������IAR for ARM(v8.10.0)��MDK-ARM(v5.24a)��������Ŀǰ����ʹ�õ�������������

 1. �����ļ�  
    ǰ�����Ѿ�˵����Щ�ļ��Ǳز����ٵģ��������CSL����֯�ṹ  

    ```asm
    KinetisKEA_CSL  
        +-----inc\
        |-----------\ KinetisKE_csl.h  
        |-----------\ KinetisKE_csl_assert.h  
        |-----------\ KinetisKE_csl_clk.h  
        |-----------\ KinetisKE_csl_config.h    
        |-----------\ KinetisKE_csl_cortex.h    
        |-----------\ KinetisKE_csl_def.h  
        |-----------\ KinetisKE_csl_flash.h  
        |-----------\ KinetisKE_csl_modulexx.h   
        +-----src\  
        |-----------\ KinetisKE_csl.c  
        |-----------\ KinetisKE_csl_clk.c  
        |-----------\ KinetisKE_csl_cortex.c  
        |-----------\ KinetisKE_csl_flash.c  
        |-----------\ KinetisKE_csl_modulexx.c  
        +-----\ KinetisKE_csl_inc.h
    ```

      ����`KinetisKE_csl_inc.h`���������е�CSLͷ�ļ�������main����֮ǰ���ã����������������ļ������Ͼ���ɾ������Ӱ��ʹ��

### ʹ��˵��

1. ������������궨��ȷ��ʹ��оƬ�ͺ�

   ```c++
   #ifdef	CSL_KEAZ128xxx4
    #include "kinetis_keaz128xxx4.h"
    #define FLASH_SIZE				((uint32_t)(128U*1024U))		//Flash: 128KiB
   #elif defined(CSL_KEAZ64xxx4)
    #include "kinetis_keaz64xxx4.h"
    #define FLASH_SIZE				((uint32_t)(64U*1024U))		    //Flash: 64KiB
   #else
    #error "Please specify your MCU model!"
   #endif /* Model Selection */
   ```

2. ��ϵ��оƬ����ARM Cortex-M0+�ܹ�����Ҫʹ����ѧ������Ԥ�������ж���`ARM_MATH_CM0PLUS`��������оƬ��FPU��__�ʲ�Ҫ���両�����ܱ�̫��ϣ��__  

3. KinetisKEA���ϵ縴λ(POR)

   ����һ���Ƚϸ��ӵĻ��⣬�Ҿ����ü������Խ���һ���������ļ����׶�

   1) оƬ�ϵ������ִ��`Reset_Handler()` �������˺�����startup�ļ��ж���

   ```asm
   ; Reset Handler
   ; MDK-ARM
   Reset_Handler   PROC
                   EXPORT  Reset_Handler             [WEAK]
                   IMPORT  SystemInit
                   IMPORT  __main
                   LDR     R0, =SystemInit
                   BLX     R0
                   LDR     R0, =init_data_bss
                   BLX     R0
                   LDR     R0, =__main
                   BX      R0
                   ENDP
   ```

     �����Ͽ���ȷ��������ִ��`SystemInit()`������Ȼ��ִ������������`SystemInit()`�����У���Ҫ���л�����ʱ�����ã�ʹоƬ�������������Խ�����������ĳ�ʼ��������ʱ�����ú��ִ��`SystemClock_Update()`�������Ա㽫��ǰʱ�ӵ�Ƶ�ʱ���������Ϊ�ں���Flashʱ�ӳ�ʼ��֮��

   2) ����������������ִ��`SystemClock_Init()`�����������û����ø���ʱ�Ӳ�����ʱ��Ƶ�ʣ���Ϊ�µ��ں���Flash��ʼ�������� 

   3) ���ִ��`CSL_Init()`�������ú�����Ҫ������ô�����£�

   + ����Flashʱ�ӣ��������þ����Ƿ���Flash���ٻ��桢�ƶ��Լ�Flash�жϲ����ڲ�Flash���г�ʼ��
   + �������þ����Ƿ���ICS���ڲ�ʱ��Դ��ʧ���жϣ�����FLL��Ч��
   + ��ʼ��SysTick��ʱ��Ϊÿ1ms����һ���жϣ������Զ��壩

   4) ���϶��޴���ʱ�������û��������ʼ����ִ���û����룬ֱ���ϵ�����ٴθ�λ

4. CSL�е��û�����

   CSL�е��û����ü�`KinetisKE_csl_config.h`�����еĴ��������Ѿ�����������������ļ���Ҫ�涨��һЩ��Ҫ�жϵĿ�������ⲿʱ��Դ���ڲ��������ڲ��͹���������Ƶ�ʣ��Լ������û����ã����ڸ��ļ�ע�ͽ���ϸ���˴�����׸��������ʹ��ǰ�ǵð����޸�����

   ����֮�⣬���ļ����и�������ģ���ʱ��ѡͨ���ã���Ϊ�궨�塣�ڵ��������ʼ����ʹ�ܺ���ʱ����Щ�궨����Ҫ�û����Լ��������ʼ�������е��ã���������ĸ��ļ�

   __ע�⣺�û���ִ���Լ��ĳ�ʼ������ʱ���������ȳ�ʼ��GPIO/FGPIO����������������������ͻ��__

   __ע�⣺��ʼ���κ�����ʱ��������Ҫռ��PTA4/PTA5/PTB4/PTB6/PTB7/PTC4��__

5. ������ϸ�����������ģ��

6. ����ʹ���������



#### Documents

?	[ʹ�ð���](./Documents/Overview.md) ����������

#### ������־

 version 0.0.1 (2017.12.4)

1. ���ṩ����ģ��

version 0.2.5 (2017.12.22)

1. �޸��˴���bug
2. �ṩ��UART��PWT��PMC��FLASH��֧��
3. ����ģ������������

Copyright &copy; Yangtze University, Stark-Zhang, All rights reserved  
Copyright &copy; ������ѧ ������ϢѧԺ ��� ��������Ȩ��