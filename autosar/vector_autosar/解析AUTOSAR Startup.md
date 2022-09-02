# 解析AUTOSAR Startup

首先，从配置上找到工程的启动入口

 ![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkxibibnsYxpTdVJqpibg0L1YvT4jIhkZVDERvtMuw4rQuMGoAHZBBSVJ1tPDNvickbXBwSafSv347m38Ow/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

```c

   /* CODE_SETUP */

   _CODE_SETUP_START align(4) :> CODE_SETUP

   __CODE_SETUP_START = .;

      . = align(4);

      _Startup_Code_START = .;

      __Startup_Code_START = .;

      .brsStartup align(4) :>.

   _RESET = brsStartupEntry;

   _start = brsStartupEntry;

   _brsStartupEntry = brsStartupEntry;
```

从这个配置可以看到，brsStartupEntry是这个工程的入口。

然后，仿真设个断点看看：

 ![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkxibibnsYxpTdVJqpibg0L1YvT4A91geUTTfE04p1ric0qNVvEqkgjFVQclOOvbaibviafB1teibIbuWd3xKA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

对应的源码是

```c
/* =========================================================================== */

/*                                                                             */

/* Description: Entry point for all cores                                      */

/*                                                                             */

/* =========================================================================== */

BRS_SECTION_CODE(brsStartup)

 BRS_GLOBAL(brsStartupEntry)

BRS_LABEL(brsStartupEntry)

 

  // … …

 

 BRS_BRANCH(brsPreAsmStartupHook)
```

继续看仿真

 ![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkxibibnsYxpTdVJqpibg0L1YvT4ibvchoOehBn9ZjatfdATuqaWjCXdevQKUbnaSpiaib2wWTI4NA2wrdwKg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

后面它会跳到brsStartupZeroInitLoop，那么这个brsStartupZeroInitLoop做什么的呢？

```c
/* =========================================================================== */

/*                                                                             */

/* Description: Initialize memory blocks and areas with zero                   */

/*                                                                             */

/* =========================================================================== */

 BRS_GLOBAL(brsStartupZeroInitLoop)

BRS_LABEL(brsStartupZeroInitLoop)

/* read ID of actual running Core into Register 16 */

BRS_READ_COREID(r16)

/* Initialize memory sections with zeros */

#if defined (VLINKGEN_ZERO_INIT_BLOCK_COUNT_STARTUP)

# if (VLINKGEN_ZERO_INIT_BLOCK_COUNT_STARTUP>1uL)

  __as1(mov    _vLinkGen_ZeroInitBlocksArrayStartup, r11)

BRS_LABEL(_startup_block_zero_init_start)

  __as1(mov    r11, r12)

  __as2(addi   12, r11, r11)

  __as1(ld.w   0[r12], r13)               /* get start address */

  __as1(ld.w   4[r12], r14)               /* get end address */

  __as1(ld.w   8[r12], r15)               /* get core ID */

  __as1(cmp    r13, r14)                  /* check end of table */

  ___asm(be     _startup_block_zero_init_end)

  __as1(cmp    r15, r16)                  /* compare core ID */

  ___asm(bne    _startup_block_zero_init_start)

BRS_LABEL(_startup_block_zero_init_loop_start)

  __as1(st.w   r0, 0[r13])

  __as2(addi   4, r13, r13)

  __as1(cmp    r13, r14)                  /* compare to end address */

  ___asm(bh     _startup_block_zero_init_loop_start)

  ___asm(jr     _startup_block_zero_init_start)

BRS_LABEL(_startup_block_zero_init_end)

# endif /*VLINKGEN_ZERO_INIT_BLOCK_COUNT_STARTUP>1uL*/
```

从名字看就是用来初始化内存的，什么内存？就要看看这个

vLinkGen_ZeroInitBlocksArrayStartup了。

在vLinkGen_InitSections_Lcfg.c里面可以找到

```c
/*! Memory region blocks to be initialized with zeros directly after reset. */

const vLinkGen_MemArea vLinkGen_ZeroInitBlocksArrayStartup[VLINKGEN_ZERO_INIT_BLOCK_COUNT_STARTUP] =

{

  {

    /* .start = */ (uint32)0xFEBD0000uL, /* LOCAL_RAM_0 */

    /* .end   = */ (uint32)0xFEBF0000uL,

    /* .core  = */ (uint32)0uL

  },

  {

    /* .start = */ (uint32)0uL,

    /* .end   = */ (uint32)0uL,

    /* .core  = */ (uint32)0uL

  }

};
```

其实，这个内存就是RH850 MCU的LOCAL RAM来的。很巧妙，这段代码通过判断.start和.end的值来决定是否结束。如果这个结构体数组有好几个元素，它会一直遍历下去。

完了后，这会跳到_startup_block_zero_init_end，继续往下

```c
#else

  #error "Mandatory define VLINKGEN_ZERO_INIT_BLOCK_COUNT_STARTUP missing within vLinkGen configuration!"

#endif /*VLINKGEN_ZERO_INIT_BLOCK_COUNT_STARTUP*/

#if defined (VLINKGEN_ZERO_INIT_AREA_COUNT_STARTUP)

# if (VLINKGEN_ZERO_INIT_AREA_COUNT_STARTUP>1uL)

  __as1(mov    _vLinkGen_ZeroInitAreasArrayStartup, r11)

BRS_LABEL(_startup_area_zero_init_start)

  __as1(mov    r11, r12)

  __as2(addi   12, r11, r11)

  __as1(ld.w   0[r12], r13)               /* get start address */

  __as1(ld.w   4[r12], r14)               /* get end address */

  __as1(ld.w   8[r12], r15)               /* get core ID */

  __as1(cmp    r13, r14)                  /* check end of table */

  ___asm(be     _startup_area_zero_init_end)

  __as1(cmp    r15, r16)                  /* compare core ID */

  ___asm(bne    _startup_area_zero_init_start)

BRS_LABEL(_startup_area_zero_init_loop_start)

  __as1(st.w   r0, 0[r13])

  __as2(addi   4, r13, r13)

  __as1(cmp    r13, r14)                  /* compare to end address */

  ___asm(bh     _startup_area_zero_init_loop_start)

  ___asm(jr     _startup_area_zero_init_start)

BRS_LABEL(_startup_area_zero_init_end)

# endif /*VLINKGEN_ZERO_INIT_AREA_COUNT_STARTUP>1uL*/

#else

  #error "Mandatory define VLINKGEN_ZERO_INIT_AREA_COUNT_STARTUP missing within vLinkGen configuration!"

#endif /*VLINKGEN_ZERO_INIT_AREA_COUNT_STARTUP*/
```

这个vLinkGen_ZeroInitAreasArrayStartup又是干嘛的？

看这个结构体数组

```c
/*! Section groups to be intialized with zeros directly after reset. */

const vLinkGen_MemArea vLinkGen_ZeroInitAreasArrayStartup[VLINKGEN_ZERO_INIT_AREA_COUNT_STARTUP] =

{

  {

    /* .start = */ (uint32)_Startup_Stack_START,

    /* .end   = */ (uint32)_Startup_Stack_END,

    /* .core  = */ (uint32)0uL

  },

  {

    /* .start = */ (uint32)0uL,

    /* .end   = */ (uint32)0uL,

    /* .core  = */ (uint32)0uL

  }

};
```

很明显，这是startup stack来的，即系统的栈初始化。

好了，继续往下走

```c
/* =========================================================================== */

/*                                                                             */

/* Description: Jump to Brs_PreMainStartup() (BrsMainStartup.c)                */

/*                                                                             */

/* =========================================================================== */

 BRS_BRANCH(_Brs_PreMainStartup)
```

那么，这个Brs_PreMainStartup是什么东西？从上面的注释，可以在BrsMainStartup.c找到

```
/**********************************************************************************************************************

  FUNCTION DEFINITIONS

**********************************************************************************************************************/

/*****************************************************************************/

/**

 * @brief      Unified routine for Pre Main() Startup.

 * @pre        Stack pointer needs to be initilialized in StartUpCode before.

 * @param[in]  -

 * @param[out] -

 * @return     -

 * @context    Function is called from assembler startup code

 *             Called by all cores

 *             All APIs are called with current Core ID

 */

/*****************************************************************************/

void Brs_PreMainStartup(void)

{

  BrsHw_PreInitClock(BrsHw_GetCore()); /* optional callout to power up the PLL for faster Memory initialization */

  BrsHw_PreZeroRamHook(BrsHw_GetCore()); /* optional, empty by default */

 

// … …

 

  main();

}
```

这个是进入**main**函数之前的一些初始化，主要的也是一些RAM等初始化。本文就不细讲了。

到这里，我就找到了整个main函数之前的初始化了。再后面的就是ECUM、OS和BSWM等的初始化了。