1.FreeRTOS porting on OpenRISC
----------------------------------

[FreeRTOS_OpenRISC.patch](https://github.com/jhpark555/newdocu/blob/master/docs/Files/openrisc.patch)
 patch against FreeRTOS V8.2.0 source directory

	unzip FreeRTOSV8.2.0.zip
    cd FreeRTOS8.2.0/FreeRTOS/Demo/Source
	patch -p0 < openrisc.patch
	cd Source
	
2.LWIP sys_arch port on OpenRISC
-----------------------------------
[LWIP OpenRISC.patch](https://github.com/jhpark555/newdocu/blob/master/docs/Files/lwip-sys.patch)
 patch against LWIP V1.4.1 ports directory

    copy sys_arch.c in lwip/ports into 
    lwip/contrib/port/FreeRTOS/OpenRISC in OpenRISC folder.