# **Assembly langle study**

生成反汇编：
keil --> 魔术棒 --> User --> After Build/Rebuild --> 打勾 Run #2（填入 fromelf --text -a -c --output=led.dis Objects\led_c.axf)

CPU是如何运行程序的：

    int abcedfff = 8;
    int abcedeee = 10;
    int main(void)
    {
        volatile int tt = abcedfff;
        volatile int ll = abcedeee;
        tt++;
        ll--;
        return 0;
    }

led_c.axf:

    i.main
    main
        0x0800041c:    b50c        ..      PUSH     {r2,r3,lr}
        0x0800041e:    4807        .H      LDR      r0,[pc,#28] ; [0x800043c] = 0x20000000
        0x08000420:    6800        .h      LDR      r0,[r0,#0]
        0x08000422:    9001        ..      STR      r0,[sp,#4]
        0x08000424:    4806        .H      LDR      r0,[pc,#24] ; [0x8000440] = 0x20000004
        0x08000426:    6800        .h      LDR      r0,[r0,#0]
        0x08000428:    9000        ..      STR      r0,[sp,#0]
        0x0800042a:    9801        ..      LDR      r0,[sp,#4]
        0x0800042c:    1c40        @.      ADDS     r0,r0,#1
        0x0800042e:    9001        ..      STR      r0,[sp,#4]
        0x08000430:    9800        ..      LDR      r0,[sp,#0]
        0x08000432:    1e40        @.      SUBS     r0,r0,#1
        0x08000434:    9000        ..      STR      r0,[sp,#0]
        0x08000436:    2000        .       MOVS     r0,#0
        0x08000438:    bd0c        ..      POP      {r2,r3,pc}

全局变量 abcedfff 在flash中的地址为 0x20000000,由 led_c.map 文件可以搜索到
1) 先将abcedfff的地址0x20000000存入r0内
2)将地址0x20000000中的值存入r0内
3)将r0的值存入到栈 sp+4 内（局部变量tt）
4)重复1、2、3实现ll = abcedeee;
5)将tt的值（在栈sp+4的位置）存入到r0内
6)将r0中的值自加一
7)将r0中的值存入栈sp+4的位置（tt）
8)重复5、6、7实现ll--;
