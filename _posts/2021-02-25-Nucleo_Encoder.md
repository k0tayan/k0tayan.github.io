---
layout: post
title:  "Nucleoのカウンタでエンコーダーを読む"
date:   2021-02-25 22:09:30 +0900
categories: Nucleo
---

ロボコンでNucleoでエンコーダーを使う機会があったので、記録として残しておく。  
部内Wikiにも書いたが、特に機密情報を含んでいるわけでもないので、外部に公開することにした。  
なお、この記事を書いたのは、2019年3月6日である。  
<br>
この記事ではNucleo-F767ZIを例にして説明する。<br>
![Nucleo](https://os.mbed.com/media/cache/platforms/Nucleo144_perf_logo_1024_WcNkWBI.jpg.250x250_q85.jpg)

まず、MCUのデータシートを入手するために、MCUの型番を調べる。  
Nucleo-F767ZIならSTM32F767ZIが使われている。データシートはSTマイクロの公式にある。  
[データシート](https://www.st.com/ja/microcontrollers-microprocessors/stm32f767zi.html)  
ここの「製品スペック 」のDS11532がデータシートである。
次に、どのようなタイマーがあるか調べるために、データシートのタイマーのテーブルを見る。エンコーダのA/B相両方読みたいので、「Counter type」が「Up, Down, Up/down」となっているものを使う。(こういう解釈でいいのかわからない)
<img title='datasheet1.png' src='https://user-images.githubusercontent.com/16555696/109161731-c2c14300-77ba-11eb-9247-c7f3492ab0c5.png' width="1336">
今回はTIM3のタイマーを使うことにする

次に、alternate function mappingを見る。マイコンでは一つのペリフェラルに複数の機能が割り当てられていたりする。alternate function mappingを参照し、どの機能を使うかを指定する必要がある。
今回は、TIM3のCH1とCH2を使いたい。(エンコーダを読むときはCH1とCH2を使う)
<img title='datasheet2.png' src='https://user-images.githubusercontent.com/16555696/109161772-ce146e80-77ba-11eb-9ac8-16bf67c24035.png' width="1780">
AF2のとき、PC6がTIM3_CH1、PC7がTIM3_CH2に割り当てられる。
ちなみに、PC6はPortCの6番ということである。

さて、PC6とPC7をエンコーダの入力に使いたいわけだが、mbedがそのペリフェラルが既に使用していないか確認する必要がある。
![](https://os.mbed.com/media/uploads/jeromecoutant/nucleo_f767zi_zio_right_2019_1_14.png)
濃い青のピンは使えない。PC6とPC7は薄い青なので使える。


PC6とPC7にエンコーダーのAとBを接続し、以下のプログラムを実行すればエンコーダが読めるはずである。
```cpp
#include <mbed.h>
#include "stm32f7xx.h"
#include "stm32f767xx.h"
#include "stm32f7xx_hal_tim_ex.h"
#include "system_stm32f7xx.h"
#define MAX_TIMER_VALUE_HALF 32768 //16bitの最高値の半分だから32768

TIM_HandleTypeDef timer;
TIM_Encoder_InitTypeDef encoder;

void HAL_TIM_Encoder_MspInit(TIM_HandleTypeDef *htim)
{
   GPIO_InitTypeDef GPIO_InitStruct;

   __TIM3_CLK_ENABLE(); /* 使うタイマーのクロック入力を有効化 */
   __GPIOC_CLK_ENABLE(); /* どのポートのクロック入力を有効化 */
   GPIO_InitStruct.Pin = GPIO_PIN_6 | GPIO_PIN_7; /* 使うピン */ 
   GPIO_InitStruct.Mode = GPIO_MODE_AF_PP;
   GPIO_InitStruct.Pull = GPIO_PULLDOWN;
   GPIO_InitStruct.Speed = GPIO_SPEED_HIGH;
   GPIO_InitStruct.Alternate = GPIO_AF2_TIM3; /* データシートからどのAFか調べる */ 
   HAL_GPIO_Init(GPIOC, &GPIO_InitStruct);
}


int32_t getCount()
{
    int32_t count, hbits;
    static int encoder_high_bits = 0;
    core_util_critical_section_enter();
    count = TIM3->CNT;
    if ((TIM3->SR & (TIM_FLAG_UPDATE)) == (TIM_FLAG_UPDATE))
    {
        TIM3->SR = ~(TIM_IT_UPDATE);
        if (TIM3->CNT < MAX_TIMER_VALUE_HALF)
            encoder_high_bits += 1;
        else
            encoder_high_bits -= 1;
        count = TIM3->CNT;
    }
    hbits = encoder_high_bits;
    core_util_critical_section_exit();
    return  (hbits << 16) | count;
}

int main(){
   timer.Instance = TIM3; /* 使うタイマー */ 
   timer.Init.Period = 0xffff;
   timer.Init.Prescaler = 0;
   timer.Init.ClockDivision = TIM_CLOCKDIVISION_DIV1;
   timer.Init.CounterMode = TIM_COUNTERMODE_UP;

   /*4逓倍で読みたい*/
   encoder.EncoderMode = TIM_ENCODERMODE_TI12; /* CH1、CH2両方読む*/
   encoder.IC1Filter = 0x0f;
   encoder.IC1Polarity = TIM_INPUTCHANNELPOLARITY_RISING /*立ち上がりを読む*/; 
   encoder.IC1Prescaler = TIM_ICPSC_DIV4;
   encoder.IC1Selection = TIM_ICSELECTION_DIRECTTI;

   encoder.IC2Filter = 0x0f;
   encoder.IC2Polarity = TIM_INPUTCHANNELPOLARITY_FALLING /*立ち下がりを読む*/;    
   encoder.IC2Prescaler = TIM_ICPSC_DIV4;
   encoder.IC2Selection = TIM_ICSELECTION_DIRECTTI;

   /*エンコーダー開始*/
   HAL_TIM_Encoder_Init(&timer, &encoder);
   HAL_TIM_Encoder_Start(&timer,TIM_CHANNEL_ALL);

   /*エンコーダーリセット*/
   core_util_critical_section_enter();
   __HAL_TIM_CLEAR_FLAG(&timer, TIM_IT_UPDATE);
   core_util_critical_section_exit();

   int count = 0;
   while (1) {
      count = getCount();
      printf("%d\r\n", count);
   };
}
```