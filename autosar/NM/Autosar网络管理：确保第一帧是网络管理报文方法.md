# Autosar网络管理：确保第一帧是网络管理报文方法

项目中，如果上Autosar网络管理，多数需求都会要求网络唤醒后的第一帧必须是网络管理报文。为什么要求是网络管理报文呢？为了使得同一网段的相关Node快速进入Normal模式，使车机功能正常。

那如何确保Node网络唤醒以后，第一帧报文是网络管理报文呢？本文讨论一下常规做法和“无奈”做法。

提示：本文基于CAN总线讨论

**常规做法**

其实这种常规做法在前面就讲过，主要是基于工具的配置，不管使用的是哪家工具，只要遵循Autosar规范开发，对应的工具都会有对应的配置，可以参考前文[Autosar网络管理PN：路由超时大Bug](http://mp.weixin.qq.com/s?__biz=MzUyNDU4NTc1NQ==&mid=2247484944&idx=1&sn=796eb70b20d845359f5e019656773d1a&chksm=fa2a5864cd5dd17285e66df13316d623d54c04647e9a19bd697311df110883dbf8cb7a91a0ea&scene=21#wechat_redirect)，在问题点1修复策略部分给出了配置注意事项，这里不再赘述。

**"无奈"做法**

之所以说是无奈做法，是因为要修改静态代码，如果不是项目要求时间紧或者实在排查不出来问题点，大家不要考虑修改静态代码的下下策。修改静态代码会带来如下几点风险：

1. 如果项目使用的代码包是购买的，修改静态代码，代码供应商不负责；
2. 修改静态代码，我们未必考虑的周全，Autosar模块交错联系，稍不慎可能引起其他问题。比如controller的状态切换，你可以手动调用xxIf模块去切换，但是这可能引起网络或者通信时序问题，因为controller本身也需要结合transceiver状态协调管理，由xxSM模块协调管控比你擅自修改要安全的多；
3. 修改静态代码需要修改者对自己修改部分充分验证和测试。

说了如上的警示，这里给出“无奈”策略。

上层模块数据的发送最终都会调用底层的驱动接口，不管是标定栈、通信栈、诊断栈、网络管理，发送的报文都会调用Can_Write()接口，在Can_Write()接口中需要传入const Can_PduType* PduInfo参数，PduInfo这个指针包含CANID，如果知道了CANID就可以区分出这个报文是不是当前Node的网络管理报文，一个Node对应一个网络管理报文，比如：0x509。此时我们可以怎么做呢？node网络一旦唤醒，上层所有模块均可发送报文，尤其周期性应用报文，一般应用报文的优先级要高于网络管理报文，如果不限制，网络管理报文不会被优先发送。我们可以通过参数PduInfo里的CANID判断当前发送的报文是不是网络管理报文，即判断PduInfo->Canid == 0x509与否，如果是则发送，之后发送不在拦截（因为第一帧网络管理报文已经发送），如果不是则发给上层一个假的消息，告知上层发送成功，实际没有发送出去。

这里又有另一个问题，何时开始拦截？在Repeat Message State判断合理。因为在Bus Sleep Mode或者Pre-Bus Sleep Mode下均不会发送报文。

伪代码示例如下所示：

```

retLocal_can = CanNm_GetState( CANNM_HANDLE_CAN, &nmStatePtr, &nmModePtr);

if(retLocal_can == E_OK)
{
  if(nmModePtr == NM_MODE_NETWORK)
  {
    if(nmStatePtr == NM_STATE_REPEAT_MESSAGE)
    {
      if (0x509 == CanPdu.id)
      {
        do{
          CanRet = CanWrite(); 
        }while(CanRet != CAN_OK);
        sendFlag = TRUE;
      }
      else if(sendFlag == TRUE)
      {
        CanRet = CanWrite(); 
      }
      else
      {
        CanRet = CAN_OK;
      }
    }
    else
    {
      CanWrite(); 
    }
  }
  elses
  {
    sendFlag = FALSE;
    CanRet = CAN_OK;
  }
}s
```

通过CanNM_GetState()接口获取当前CAN网络的状态。上述伪代码的if else太多了，大家可以在这个策略上优化。

**切记：**优先使用工具配置，不要轻易修改静态代码！