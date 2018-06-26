0.CPU type ( Arch/cpu.h )
------------------------------
	#define BYTE_ORDER BIG_ENDIAN

1.Linker Scripts
--------------------------
	MEMORY
	{
	    /* Put all sections expect data, rodata into ram2   */
	    /* On ORSoC dev board devices, for pages 132-134 ( 256Bytes pages)  */
	
		vectors : ORIGIN = 0x00000000, LENGTH = 0x00002000
		sdram (rwx)    : ORIGIN = 0x00002000, LENGTH = 0x13500
		sdram2 (rwx)	: ORIGIN = 0x13500, LENGTH = 0x02000000 
	}

	heap_size = 10*1024;
	SECTIONS
	{
		.vectors :
		{
			_vec_start = .;
			*(.vectors)
			_vec_end = .;
		} > vectors
	
	
		.text  :
		{
			. = ALIGN(4);
			*(.text)
			*(.text*)
			. = ALIGN(4);
		} > sdram2
	
		.data  :
		{
			. = ALIGN(4);
			_dst_beg = .;
			*(.data)
			*(.data*)
			_dst_end = .;
			. = ALIGN(4);
		} > sdram
	
		.rodata :
		{
			. = ALIGN(4);
			*(.rodata)
			*(.rodata.*)
			. = ALIGN(4);
		} > sdram
	
		.sbss  :
		{
			. = ALIGN(4);
			_sbss_beg = .;
			*(.bss)
			*(.bss*)
			_sbss_end = .;
			. = ALIGN(4);
		} > sdram2
	
	    .sdata : {
	        _gp = . + 0x800;
	        *(.srodata.cst16) *(.srodata.cst8) *(.srodata.cst4) *(.srodata.cst2) *(.srodata*)
	        *(.sdata .sdata.* .gnu.linkonce.s.*)
	     }	> sdram2
	
		.bss  :
		{
			. = ALIGN(4);
			_bss_beg = .;
			*(.bss)
			*(.bss*)
			*(COMMON)
			_bss_end = .;
			. = ALIGN(4);
		} > sdram2
	
		.heap :
		{
			. = ALIGN(4);
			__heap_start = .;
			. = . +heap_size;
			__heap_end = .;
		} > sdram2
	
	
		.stack :
		{
			. = ALIGN(4);
			*(.stack)
			_stack_top = .;
			. = ALIGN(4);
		} >sdram2
	
		PROVIDE(_stack_top = 0x02000000 );
	}

2.FreeRTOSConfig.h
----------------------------
	#include "board.h"
	
	#define configUSE_PREEMPTION            1
	#define configUSE_IDLE_HOOK             0
	#define configUSE_TICK_HOOK             0
	#define configCPU_CLOCK_HZ              ( ( unsigned long ) SYS_CLK )
	#define configTICK_RATE_HZ              ( ( TickType_t ) 1000 )
	#define configMINIMAL_STACK_SIZE        ( ( unsigned short ) 100)
	#define configTOTAL_HEAP_SIZE           ( ( size_t ) 49*1024 ) 

3.Port.c
---------------------

	/*
	 * Initialise the stack of a task to look exactly as if a call to
	 * portSAVE_CONTEXT had been called. Context layout is described in
	 * portmarco.h
	 *
	 * See header file for description.
	 */
	StackType_t *pxPortInitialiseStack( StackType_t *pxTopOfStack,
	                                       TaskFunction_t pxCode,
	                                       void *pvParameters )
	{
	    unsigned portLONG uTaskSR = mfspr(SPR_SR);
	
	    // Supervisor mode
	    uTaskSR |= SPR_SR_SM;
	    // Tick interrupt enable, All External interupt enable
	    uTaskSR |= (SPR_SR_TEE | SPR_SR_IEE);
	
	    // allocate redzone
	    pxTopOfStack -= REDZONE_SIZE/4;
	
	    /* Setup the initial stack of the task.  The stack is set exactly as
	    expected by the portRESTORE_CONTEXT() macro. */
	    *(--pxTopOfStack) = ( StackType_t )pxCode;         // SPR_EPCR_BASE(0)
	    *(--pxTopOfStack) = ( StackType_t )uTaskSR;        // SPR_ESR_BASE(0)
	
	    *(--pxTopOfStack) = ( StackType_t )0x00000031;     // r31
	    *(--pxTopOfStack) = ( StackType_t )0x00000030;     // r30
	    *(--pxTopOfStack) = ( StackType_t )0x00000029;     // r29
	    *(--pxTopOfStack) = ( StackType_t )0x00000028;     // r28
	    *(--pxTopOfStack) = ( StackType_t )0x00000027;     // r27
	    *(--pxTopOfStack) = ( StackType_t )0x00000026;     // r26
	    *(--pxTopOfStack) = ( StackType_t )0x00000025;     // r25
	    *(--pxTopOfStack) = ( StackType_t )0x00000024;     // r24
	    *(--pxTopOfStack) = ( StackType_t )0x00000023;     // r23
	    *(--pxTopOfStack) = ( StackType_t )0x00000022;     // r22
	    *(--pxTopOfStack) = ( StackType_t )0x00000021;     // r21
	    *(--pxTopOfStack) = ( StackType_t )0x00000020;     // r20
	    *(--pxTopOfStack) = ( StackType_t )0x00000019;     // r19
	    *(--pxTopOfStack) = ( StackType_t )0x00000018;     // r18
	    *(--pxTopOfStack) = ( StackType_t )0x00000017;     // r17
	    *(--pxTopOfStack) = ( StackType_t )0x00000016;     // r16
	    *(--pxTopOfStack) = ( StackType_t )0x00000015;     // r15
	    *(--pxTopOfStack) = ( StackType_t )0x00000014;     // r14
	    *(--pxTopOfStack) = ( StackType_t )0x00000013;     // r13
	    *(--pxTopOfStack) = ( StackType_t )0x00000012;     // r12
	    *(--pxTopOfStack) = ( StackType_t )0x00000011;     // r11
	    *(--pxTopOfStack) = ( StackType_t )0x00000010;     // r10
	    *(--pxTopOfStack) = ( StackType_t )0x00000008;     // r8
	    *(--pxTopOfStack) = ( StackType_t )0x00000007;     // r7
	    *(--pxTopOfStack) = ( StackType_t )0x00000006;     // r6
	    *(--pxTopOfStack) = ( StackType_t )0x00000005;     // r5
	    *(--pxTopOfStack) = ( StackType_t )0x00000004;     // r4
	    *(--pxTopOfStack) = ( StackType_t )pvParameters;   // task argument
	    *(--pxTopOfStack) = ( StackType_t )0x00000002;     // r2
	    *(--pxTopOfStack) = ( StackType_t )pxCode;         // PC
	
	    return pxTopOfStack;
	}

	BaseType_t xPortStartScheduler( void )
	{
	    if(setjmp((void *)jmpbuf) == 0) {
	        /* Start the timer that generates the tick ISR.
	     * Interrupts are disabled here already. */
	        prvSetupTimerInterrupt();
	
	        /* Start the first task. */
	        asm volatile (
	        " .global   pxCurrentTCB    \n\t"
	        /*   restore stack pointer          */
	        "   l.movhi r3, hi(pxCurrentTCB)        \n\t"
	        "   l.ori   r3, r3, lo(pxCurrentTCB)    \n\t"
	        "   l.lwz   r3, 0x0(r3)     \n\t"
	        "   l.lwz   r1, 0x0(r3)     \n\t"
	        /*   restore context                */
	        "   l.lwz   r9,  0x00(r1)   \n\t"
	        "   l.lwz   r2,  0x04(r1)   \n\t"
	        "   l.lwz   r6,  0x14(r1)   \n\t"
	        "   l.lwz   r7,  0x18(r1)   \n\t"
	        "   l.lwz   r8,  0x1C(r1)   \n\t"
	        "   l.lwz   r10, 0x20(r1)   \n\t"
	        "   l.lwz   r11, 0x24(r1)   \n\t"
	        "   l.lwz   r12, 0x28(r1)   \n\t"
	        "   l.lwz   r13, 0x2C(r1)   \n\t"
	        "   l.lwz   r14, 0x30(r1)   \n\t"
	        "   l.lwz   r15, 0x34(r1)   \n\t"
	        "   l.lwz   r16, 0x38(r1)   \n\t"
	        "   l.lwz   r17, 0x3C(r1)   \n\t"
	        "   l.lwz   r18, 0x40(r1)   \n\t"
	        "   l.lwz   r19, 0x44(r1)   \n\t"
	        "   l.lwz   r20, 0x48(r1)   \n\t"
	        "   l.lwz   r21, 0x4C(r1)   \n\t"
	        "   l.lwz   r22, 0x50(r1)   \n\t"
	        "   l.lwz   r23, 0x54(r1)   \n\t"
	        "   l.lwz   r24, 0x58(r1)   \n\t"
	        "   l.lwz   r25, 0x5C(r1)   \n\t"
	        "   l.lwz   r26, 0x60(r1)   \n\t"
	        "   l.lwz   r27, 0x64(r1)   \n\t"
	        "   l.lwz   r28, 0x68(r1)   \n\t"
	        "   l.lwz   r29, 0x6C(r1)   \n\t"
	        "   l.lwz   r30, 0x70(r1)   \n\t"
	        "   l.lwz   r31, 0x74(r1)   \n\t"
	        /*  restore SPR_ESR_BASE(0), SPR_EPCR_BASE(0) */
	        "   l.lwz   r3,  0x78(r1)   \n\t"
	        "   l.lwz   r4,  0x7C(r1)   \n\t"
	        "   l.mtspr r0,  r3, %1     \n\t"
	        "   l.mtspr r0,  r4, %2     \n\t"
	        /*   restore clobber register     */
	        "   l.lwz   r3,  0x08(r1)   \n\t"
	        "   l.lwz   r4,  0x0C(r1)   \n\t"
	        "   l.lwz   r5,  0x10(r1)   \n\t"
	        "   l.addi  r1,  r1, %0     \n\t"
	        "   l.rfe                   \n\t"
	        "   l.nop                   \n\t"
	        :
	        :   "n"(STACKFRAME_SIZE),
	            "n"(SPR_ESR_BASE),
	            "n"(SPR_EPCR_BASE)
	        );
	
	        /* Should not get here! */
	    } else {
	        /* Retrun by vPortEndScheduler */
	    }
	
	    return 0;
	}

4.Mbedtls Configuration
----------------------------------
	#define MBEDTLS_PLATFORM_CALLOC_MACRO mbedtls_calloc
	#define MBEDTLS_PLATFORM_FREE_MACRO	mbedtls_free
	/*
	 * Save RAM at the expense of interoperability: do this only if you control
	 * both ends of the connection!  (See comments in "mbedtls/ssl.h".)
	 * The optimal size here depends on the typical size of records.
	 */
	#define MBEDTLS_SSL_MAX_CONTENT_LEN             (5*1024)


