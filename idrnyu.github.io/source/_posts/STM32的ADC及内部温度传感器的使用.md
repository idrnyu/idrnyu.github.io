---
title: STM32的ADC及内部温度传感器的使用
date: 2018-03-13 10:49:23
tags: 
- c语言
- STM32, 单片机
categories:
- STM32
---

## STM32的ADC及内部温度传感器的使用
>ADC的用途范围可以说是非常的广泛~甚至是可以说差不多必不可少了~大部分单片机嵌入式系统ADC都基本要用到~包括牛人CZZ也一样！
>STM32 自带1-3个ADC模块，采样精度达到了12位，比起当年使用的AVR单片机的10位来说，上了个小档次了~本测试程序采用了ADCDMA的中断方式，这 样CPU就可以把ADC的任务交给DMA这个勤劳肯干的部下了，当DMA完成了一次任务之后会产生中断，告诉CPU可以来查收结果了！DMA也是在嵌入式 系统中非常常用的，例如在LCD中，数据拷贝中等。。。在STM32F103RBT6中，ADC1和ADC2共用一组管脚

<!-- more -->
## 总体编程思路和顺序如下：
	1. 初始化RCC相关，使得系统有时钟，功能模块如ADC、DMA有时钟。
	2. GPIO相关初始化，比如常用的指示灯，ADC的管家要设置为输入等。
	3. NVIC向量中断的配置，因为这里使用了DMA中断和中断服务程序编写（下例中暂不使用）
	4. DMA配置（下例中暂不使用）
	5. ADC初始化

## 以下是参考代码，使用ADC1的IN0脚
```c
void ADC_GPIO_Init()
{
  GPIO_InitTypeDef GPIO_InitStructure;
  RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA|RCC_APB2Periph_ADC1,ENABLE);
  GPIO_DeInit(GPIOA);
  GPIO_InitStructure.GPIO_Pin=GPIO_Pin_0;
  GPIO_InitStructure.GPIO_Mode=GPIO_Mode_AIN;//设为模拟输入
  GPIO_Init(GPIOA,&GPIO_InitStructure); 
}

void ADC_configuration()
{
  ADC_InitTypeDef ADC_InitStructure;
  ADC_InitStructure.ADC_Mode=ADC_Mode_Independent;//独立模式
  ADC_InitStructure.ADC_ScanConvMode=DISABLE;//连续多通道模式
  ADC_InitStructure.ADC_ContinuousConvMode=DISABLE;//单次转换
  ADC_InitStructure.ADC_ExternalTrigConv=ADC_ExternalTrigConv_None;//转换由软件而不是外部触发启动
  ADC_InitStructure.ADC_DataAlign=ADC_DataAlign_Right;//右对齐
  ADC_InitStructure.ADC_NbrOfChannel=1;//扫描通道数
  ADC_Init(ADC1,&ADC_InitStructure);
  //ADC_RegularChannelConfig(ADC1,ADC_Channel_0,1,ADC_SampleTime_7Cycles5);
  ADC_Cmd(ADC1,ENABLE);//使能或者失能指定的ADC
  ADC_ResetCalibration(ADC1);//重置指定的ADC的校准寄存器
  while(ADC_GetResetCalibrationStatus(ADC1));//等待校准寄存器初始化
  ADC_StartCalibration(ADC1);//开始校准
  while(ADC_GetCalibrationStatus(ADC1));//等待校准完成 
  //ADC_SoftwareStartConvCmd(ADC1,ENABLE);//使能指定的ADC的软件转换启动功能
}
 
u16 GetADCValue(u8 ADC_Channel)//ADC_Channel_x 0~17
{
  u16 adc_value;
  ADC_RegularChannelConfig(ADC1,ADC_Channel,1,ADC_SampleTime_7Cycles5);
  ADC_SoftwareStartConvCmd(ADC1,ENABLE);//使能指定的ADC的软件转换启动功能
  while(ADC_GetFlagStatus(ADC1,ADC_FLAG_EOC)==RESET);//检查制定ADC标志位置1与否 ADC_FLAG_EOC 转换结束标志位
  adc_value=ADC_GetConversionValue(ADC1);
  return adc_value;//返回最近一次ADCx规则组的转换结果
}
```
>当使用内部温度传感器时，需要使能温度传感器通  `ADC_TempSensorVrefintCmd(ENABLE);`
	温度传感器通道号是ADC_Channel_16，此通道的采样时间调到最大，来保证精度；

## 温度的计算公式如下：
<img src="/Dom/imgs/2018_03_13/313stmADC.png" width="100%"/>
	V25、Avg_Slope的典型值分别为1.43、4.3mV/C
	TEMP=(1.43-Vsense)/0.0043+25;
