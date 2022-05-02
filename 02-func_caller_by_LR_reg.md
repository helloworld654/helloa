# **通过LR寄存器查看函数调用者**

在函数入口处通过调用如下函数，可返回LR寄存器的值，即该函数的返回地址；通过该返回地址在反汇编文件中可查看到对应的位置，当前位置所在的函数，即为该函数的调用者函数。

    __attribute__( ( always_inline ) ) static inline uint32_t __get_LR(void)
    {
        register uint32_t result;
        __ASM volatile ("mov %0, lr\n" : "=r" (result) );
        return(result);
    }

生成反汇编指令：

    arm-none-eabi-objdump -d xxx.elf > xxx.asm

生成符号表指令(可能不需要)：

    arm-none-eabi-objdump -t xxx.elf > xxx.sym

以下为一示例，main.c:

    __attribute__( ( always_inline ) ) static inline uint32_t __get_LR(void)
    {
        register uint32_t result;
        __ASM volatile ("mov %0, lr\n" : "=r" (result) );
        return(result);
    }

    void func_B(void)
    {
        uint32_t lr_addr;
        lr_addr = __get_LR();
        printf("[%s] lr_addr:0x%x\r\n",__func__,lr_addr);
    }

    void func_A(void)
    {
        uint32_t lr_addr;
        lr_addr = __get_LR();
        printf("[%s] lr_addr:0x%x\r\n",__func__,lr_addr);
        func_B();
    }

    int main(void)
    {	 
        JTAG_Set(0x01);
        delay_init();
        NVIC_Configuration(); 	 //设置NVIC中断分组2:2位抢占优先级，2位响应优先级
        uart1_init(115200);
        while(1){
            // printf("hello world\r\n");
            func_A();
            delay_lms(2000);
        }
    }

串口输出结果(实际LR的值为返回地址加一)：

    [20:11:22.146] [func_A] lr_addr:0x800096b
    [20:11:24.155] [func_B] lr_addr:0x800093d

反汇编文件：

    080008f8 <func_B>:
    80008f8:	b590      	push	{r4, r7, lr}
    80008fa:	b083      	sub	sp, #12
    80008fc:	af00      	add	r7, sp, #0
    80008fe:	4673      	mov	r3, lr
    8000900:	461c      	mov	r4, r3
    8000902:	4623      	mov	r3, r4
    8000904:	607b      	str	r3, [r7, #4]
    8000906:	687a      	ldr	r2, [r7, #4]
    8000908:	4903      	ldr	r1, [pc, #12]	; (8000918 <func_B+0x20>)
    800090a:	4804      	ldr	r0, [pc, #16]	; (800091c <func_B+0x24>)
    800090c:	f000 fc14 	bl	8001138 <iprintf>
    8000910:	bf00      	nop
    8000912:	370c      	adds	r7, #12
    8000914:	46bd      	mov	sp, r7
    8000916:	bd90      	pop	{r4, r7, pc}
    8000918:	08001e5c 	.word	0x08001e5c
    800091c:	08001e48 	.word	0x08001e48

    08000920 <func_A>:
    8000920:	b590      	push	{r4, r7, lr}
    8000922:	b083      	sub	sp, #12
    8000924:	af00      	add	r7, sp, #0
    8000926:	4673      	mov	r3, lr
    8000928:	461c      	mov	r4, r3
    800092a:	4623      	mov	r3, r4
    800092c:	607b      	str	r3, [r7, #4]
    800092e:	687a      	ldr	r2, [r7, #4]
    8000930:	4904      	ldr	r1, [pc, #16]	; (8000944 <func_A+0x24>)
    8000932:	4805      	ldr	r0, [pc, #20]	; (8000948 <func_A+0x28>)
    8000934:	f000 fc00 	bl	8001138 <iprintf>
    8000938:	f7ff ffde 	bl	80008f8 <func_B>
    800093c:	bf00      	nop
    800093e:	370c      	adds	r7, #12
    8000940:	46bd      	mov	sp, r7
    8000942:	bd90      	pop	{r4, r7, pc}
    8000944:	08001e64 	.word	0x08001e64
    8000948:	08001e48 	.word	0x08001e48

    0800094c <main>:
    800094c:	b580      	push	{r7, lr}
    800094e:	af00      	add	r7, sp, #0
    8000950:	2001      	movs	r0, #1
    8000952:	f000 f80f 	bl	8000974 <JTAG_Set>
    8000956:	f000 f913 	bl	8000b80 <delay_init>
    800095a:	f000 f84f 	bl	80009fc <NVIC_Configuration>
    800095e:	f44f 30e1 	mov.w	r0, #115200	; 0x1c200
    8000962:	f000 fa87 	bl	8000e74 <uart1_init>
    8000966:	f7ff ffdb 	bl	8000920 <func_A>
    800096a:	f44f 60fa 	mov.w	r0, #2000	; 0x7d0
    800096e:	f000 f963 	bl	8000c38 <delay_lms>
    8000972:	e7f8      	b.n	8000966 <main+0x1a>
