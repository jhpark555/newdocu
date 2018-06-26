1.FreeRTOS porting on OpenRISC
----------------------------------

[FreeRTOS_OpenRISC.patch](\\\..\files/openrisc.patch)
 patch against FreeRTOS V8.2.0 source directory

	unzip FreeRTOSV8.2.0.zip
    cd FreeRTOS8.2.0/FreeRTOS/Demo/Source
	patch -p0 < openrisc.patch
	cd Source
	