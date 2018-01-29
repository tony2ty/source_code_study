# freemodbus 软件理解
注意：本文档采用ASCII模式

## 版本
> freemodbus-v1.5.0-master

## 功能

1. 发送流程

2. 接收流程
* 由事件驱动

* 函数 xMBASCIIReceiveFSM（接受状态机函数）在收完一帧字符串时，调用函数 xMBPortEventPost 发送事件 EV_FRAME_RECEIVED；
* 函数 main 中的函数 eMBPoll 调用函数 xMBPortEventGet 查询  EV_FRAME_RECEIVED 事件，在收到该事件后，调用函数 peMBFrameReceiveCur，该函数指向函数 eMBASCIIReceive
* 函数 eMBASCIIReceive 进行 LCR检查，正常后返回函数 eMBPoll
* 函数 eMBPoll 调用函数 xMBPortEventPost 发送事件 EV_EXECUTE；
* 函数 eMBPoll 调用函数 xMBPortEventGet 查询事件 EV_EXECUTE 后，查询 xFuncHandlers 结构体，寻找对应的命令并执行命令对应的函数
* 在地址不为 MB_ADDRESS_BROADCAST 时，需要响应master，调用函数 peMBFrameSendCur ， 该函数指向函数 eMBASCIISend

1. 主要函数

```
eMBErrorCode eMBInit( eMBMode eMode, UCHAR ucSlaveAddress, UCHAR ucPort, ULONG ulBaudRate, eMBParity eParity )
{
  ...
  switch ( eMode )
  {
#if MB_RTU_ENABLED > 0
  case MB_RTU:
      pvMBFrameStartCur = eMBRTUStart;
      pvMBFrameStopCur = eMBRTUStop;
      peMBFrameSendCur = eMBRTUSend;
      peMBFrameReceiveCur = eMBRTUReceive;
      pvMBFrameCloseCur = MB_PORT_HAS_CLOSE ? vMBPortClose : NULL;
      pxMBFrameCBByteReceived = xMBRTUReceiveFSM;
      pxMBFrameCBTransmitterEmpty = xMBRTUTransmitFSM;
      pxMBPortCBTimerExpired = xMBRTUTimerT35Expired;

      eStatus = eMBRTUInit( ucMBAddress, ucPort, ulBaudRate, eParity );
      break;
#endif
#if MB_ASCII_ENABLED > 0
  case MB_ASCII:
      pvMBFrameStartCur = eMBASCIIStart;
      pvMBFrameStopCur = eMBASCIIStop;
      peMBFrameSendCur = eMBASCIISend;
      peMBFrameReceiveCur = eMBASCIIReceive;
      pvMBFrameCloseCur = MB_PORT_HAS_CLOSE ? vMBPortClose : NULL;
      pxMBFrameCBByteReceived = xMBASCIIReceiveFSM;
      pxMBFrameCBTransmitterEmpty = xMBASCIITransmitFSM;
      pxMBPortCBTimerExpired = xMBASCIITimerT1SExpired;

      eStatus = eMBASCIIInit( ucMBAddress, ucPort, ulBaudRate, eParity );
      break;
#endif
  default:
      eStatus = MB_EINVAL;
  }
  ...
}


```

函数实现 eMBInit实现 RTU与ASCII 模式的相关函数指针映射


如果采用
