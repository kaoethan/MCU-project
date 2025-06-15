---
layout: post
title: EdgeAI-盲人導航系統
author: [高屹]
category: [Lecture]
tags: [jekyll, ai]
---
# 邊緣計算微控制器原理與應用設計-盲人導航系統
This project uses QR code to play audio to assist blind people in navigation.

---
## AMB82-mini 硬體介紹
**系統架構圖**<br>
![](https://github.com/kaoethan/MCU-project/blob/main/images/AMB82-MINI.JPG?raw=true)<br>
**•	AMB82-mini**：具備雙核心 CPU 及 Wi-Fi 功能，支援 TensorFlow Lite Micro 與 OpenAI API。<br>
**•	ILI9341 TFT LCD**：負責顯示圖像、辨識結果或系統狀態。<br>
**•	PAM8403 + 4Ω 3W Speaker**：進行語音播放，傳遞 TTS 結果。<br>
**•	Camera (內建)**：每分鐘擷取影像，送至 Gemini Vision 模型。<br>
**•	MicroSD 卡**：儲存辨識結果與對應影像。<br>
**•	RTC 模組（若接入）**：確保時間戳記準確。<br>
<br>
**AMB82-Mini實體外觀圖**<br>
![](https://github.com/kaoethan/MCU-project/blob/main/images/AMB82-MINI1.jpg?raw=true)<br>
展示了 AMB82-Mini 的實體外觀，其中左側為搭載鏡頭的主開發板，右側為擴充模組，兩者皆具備豐富的 GPIO 排針與無線通訊模組。該圖有助於辨識實體接腳位置、連接模組佈局與實體安裝方向，是開發初期理解板子配置的第一步。鏡頭模組的存在也預示了該開發板具備即時影像擷取與傳送的能力，對於後續進行圖像辨識等 AI 應用至關重要。<br>
<br>
**AMB82-Mini主要介面說明圖**<br>
![](https://github.com/kaoethan/MCU-project/blob/main/images/AMB82-MINI2.jpg?raw=true)<br>
進一步聚焦於 AMB82-Mini 主板下方的三個 USB 介面與一個 RESET 按鈕，分別對應 UART_DOWNLOAD、USB OTG 與 Micro USB 功能，是韌體燒錄、USB 裝置掛載與序列埠通訊的關鍵接口。這張圖對於軟體開發者和系統整合者來說特別重要，因為錯誤的連接方式可能導致無法下載程式、裝置無法啟動或電力不穩。<br>
<br>
**AMB82-Mini 開發板前後視圖與 GPIO 腳位功能對應圖**<br>
![](https://github.com/kaoethan/MCU-project/blob/main/images/AMB82-MINI3.jpg?raw=true)<br>
提供最為關鍵的 GPIO 腳位分佈與功能對應說明，包含前後兩面的腳位排列，並以顏色分類標示各腳位用途，例如 SPI、I2C、UART、PWM、ADC、GPIO 等。開發者可依據此圖完成模組整合與接腳設計，例如將 ILI9341 TFT LCD 透過 SPI 腳位接入，或將麥克風與喇叭透過 I2S、PWM 控制端連接，此外也能依據腳位分配規劃按鈕輸入或感測器模組串接。<br>
<br>
**GPIO INT（General-Purpose Input/Output with Interrupts)** <br>
支援中斷觸發的 GPIO 腳位可用於偵測電壓變化（如上升緣/下降緣）並自動呼叫中斷函數，不需輪詢（polling），節省 CPU 運算資源，實現事件驅動控制。<br>
**應用實例:** <br>
•	按鈕按壓觸發畫面切換<br>
•	PIR 感測器啟動語音警示<br>
•	碰撞感測器觸發機械動作<br>
**對應腳位:** <br>
PF0（A0）、PF1（A1）、PF2（A2）、PF3（*A3）、PA0（A4）、PA1（A5）、PA2（A6）、PA3（A7）<br>
<br>
**PWM（Pulse Width Modulation）** <br>
以數位方式產生變化的電壓波形，用以模擬類比輸出，適合控制亮度、音訊輸出、馬達速度等。PWM 透過改變佔空比調整輸出功率。<br>
**應用實例:** <br>
•	LED 呼吸燈效果<br>
•	喇叭播放合成語音訊號<br>
•	•伺服馬達轉角控制<br>
**對應腳位:** <br>
PF6、PF7、PF8、PF9（LED_B）、PF12、PF13<br>
<br>
**UART（Universal Asynchronous Receiver/Transmitter）** <br>
UART 為非同步序列通訊，提供開發板與外部裝置（如藍牙模組、GPS、PC）之間的資料傳輸。AMB82-Mini 支援多組 UART（Serial1–3、LOG）。<br>
**應用實例:** <br>
•	與電腦透過 LOG_TX/RX 傳送 debug 資訊 <br>
•	與 HC-05 藍牙模組資料傳輸 <br>
•	接收 GPS 定位資訊 <br>
**對應腳位:** <br>
PA2（SERIAL1_TX）、PA3（SERIAL1_RX）、PD15（SERIAL2_TX）、PD16（SERIAL2_RX）、PE1（SERIAL3_TX）、PE2（SERIAL3_RX）、PF4（LOG_TX）、PF3（LOG_RX）<br>
<br>
**SPI（Serial Peripheral Interface）** <br>
SPI 為同步序列通訊，速度快，適用於 TFT 顯示器、SD 卡、Flash 等高速裝置。AMB82-Mini 提供兩組 SPI，包含 SPI 與 SPI1。 <br>
**應用實例:** <br>
•	控制 ILI9341 TFT LCD 顯示畫面<br>
•	儲存影像資料至 MicroSD 卡模組<br>
•	外接 Flash 記憶體儲存語音模型<br>
**對應腳位:** <br>
PF5（SPI1_MISO）、PF6（SPI1_SCLK）、PF7（SPI1_MOSI）、PF8（SPI1_SS）、PE3（SPI_MOSI）、PE2（SPI_MISO）、PE1（SPI_SCLK）、PE4（SPI_SS） <br>
 <br>
**I2C（Inter-Integrated Circuit)** <br>
I2C 為雙線式同步通訊協定，可連接多個從屬裝置如感測器、OLED 等。AMB82-Mini 提供兩組 I2C 介面。 <br>
**應用實例:** <br>
•	連接 MPU6050 姿態感測器<br>
•	讀取 SHT31 溫濕度感測值<br>
•	多感測模組串接（透過 TCA9548 擴展）<br>
**對應腳位:** <br>
PF2（I2C1_SDA）、PF1（I2C1_SCL）、PA1（I2C2_SDA）、PA0（I2C2_SCL）、PE4（I2C_SDA）、PE3（I2C_SCL）<br>
<br>
**SWD（Serial Wire Debug）** <br>
SWD 為 ARM Cortex-M 的單線除錯介面，用於實現斷點、變數觀察與即時燒錄。適合進行嵌入式除錯與韌體開發。<br>
**應用實例:** <br>
•	使用 J-Link 進行即時中斷除錯<br>
•	使用 OpenOCD 觀察變數、設定斷點<br>
•	韌體 OTA 更新前進行 low-level debug<br>
**對應腳位:** <br>
PA0（SWD_DATA）、PA1（SWD_CLK）<br>
注意：若啟用 SWD 模式，A4/A5 與 I2C2 將無法同時使用。<br>
<br>
**LED（On-Board LED Control）** <br>
板載 LED 可作為系統執行狀態、網路連線、錯誤警告等視覺提示，亦可當作普通 GPIO 控制。<br>
**應用實例:** <br>
•	網路連線成功後綠燈閃爍<br>
•	辨識結果語音播放時藍燈同步閃爍<br>
•	錯誤發生時 LED 閃爍提示<br>
**對應腳位:** <br>
PF9：LED_BUILTIN / LED_B（藍燈）、PE6：LED_G（綠燈）<br>
##  ILI9341 TFT LCD 硬體介紹<br>
ILI9341 是一款廣泛應用於嵌入式系統的 2.4 吋/2.8 吋彩色 TFT LCD 顯示模組，搭載 240×320 像素解析度 與 SPI 通訊介面。該模組內建 ILI9341 顯示驅動 IC，支援 262K 色彩顯示與圖形加速功能，能夠顯示影像、文字、圖形介面，常見於智慧手持裝置、嵌入式儀表與 IoT 應用。<br>
<br>
**ILI9341 TFT LCD實體外觀圖**<br>
![](https://github.com/kaoethan/MCU-project/blob/main/images/LCD1.jpg?raw=true)<br>
**AMB82 MINI and QVGA TFT LCD 接線圖**<br>
![](https://github.com/kaoethan/MCU-project/blob/main/images/LCD2.jpg?raw=true)<br>
**表一：ILI9341 TFT LCD 規格表**

| 項目       | 說明                                                              |
|------------|-------------------------------------------------------------------|
| 顯示尺寸   | 2.4 吋 / 2.8 吋 TFT LCD                                            |
| 解析度     | 240 × 320 pixels                                                  |
| 控制晶片   | ILI9341                                                           |
| 通訊介面   | SPI（支援 4 線序列通訊，亦可設定為 8-bit 並列）                  |
| 顯示色彩   | 262K（18-bit）真彩顯示                                            |
| 操作電壓   | 3.3V（邏輯電平，部分模組具備 5V 轉接）                           |
| 觸控功能   | 可選（部分模組具電阻式/電容式觸控，搭配 XPT2046）               |
| 背光模組   | LED 背光，PWM 可調整亮度                                          |

<br>**表二：ILI9341 TFT LCD 模組引腳定義與功能說明表**

| 序號 | 引腳標號   | 說明                                                           |
|------|------------|----------------------------------------------------------------|
| 1    | VCC        | 5V/3.3V電源輸入                                                 |
| 2    | GND        | 接地                                                             |
| 3    | CS         | 液晶屏片選信號，低電平使能                                       |
| 4    | RESET      | 液晶屏重定信號，低電平重定                                       |
| 5    | DC/RS      | 液晶屏寄存器/資料選擇信號，低電平：寄存器，高電平：數據         |
| 6    | SDI(MOSI)  | SPI匯流排寫資料信號                                              |
| 7    | SCK        | SPI匯流排時鐘信號                                                |
| 8    | LED        | 背光控制，高電平點亮，如無需控制則接3.3V常亮                     |
| 9    | SDO(MISO)  | SPI匯流排讀數據信號，如無需讀取功能則可不接                     |
|      |            | *(以下為觸摸屏信號線，如無需觸摸或模組本身不帶觸控，可不連接)*   |
| 10   | T_CLK      | 觸摸SPI匯流排時鐘信號                                            |
| 11   | T_CS       | 觸摸屏片選信號，低電平使能                                       |
| 12   | T_DIN      | 觸摸SPI匯流排輸入                                                |
| 13   | T_DO       | 觸摸SPI匯流排輸出                                                |
| 14   | T_IRQ      | 觸摸屏中斷信號，檢測到觸摸時為低電平                             | 
## PAM8403硬體介紹
PAM8403 是一款低功耗、高效率的 D 類音訊放大器晶片，可提供最高 3W + 3W 雙聲道立體聲輸出，適用於驅動如 4Ω/3W 或 8Ω/2W 的小型喇叭。它常被整合為 小型擴大器模組，可直接與微控制器（如 AMB82-Mini）搭配，用於播放語音合成（TTS）或音樂訊號。<br>

<br>**表三：PAM8403 音訊放大器模組規格表**

| 項目         | 規格說明                                                       |
|--------------|----------------------------------------------------------------|
| 工作電壓     | 2.5V ~ 5.5V                                                    |
| 最大輸出功率 | 3W × 2（於 5V、4Ω 負載）                                       |
| 音訊輸入     | 模擬音訊（L/R 左右聲道輸入）                                   |
| 控制方式     | 無需 MCU 控制，可直接使用 PWM 或 DAC 輸入                      |
| 音質表現     | 低 THD（總諧波失真）與雜訊，音質清晰                            |
| 功耗特性     | 高效率（>85%），待機電流極低                                    |
| 大小         | 模組極小（約 2×2 cm），易於整合至小型裝置                      |

<br>**表四：PAM8403 音訊放大器模組腳位定義**

| 腳位名稱           | 功能說明                                                         |
|--------------------|------------------------------------------------------------------|
| VCC                | 電源正極（建議供應 5V）                                         |
| GND                | 電源地                                                           |
| L_IN               | 左聲道音訊輸入（模擬/PWM）                                      |
| R_IN               | 右聲道音訊輸入（模擬/PWM）                                      |
| L_OUT+ / L_OUT−    | 左聲道喇叭輸出（差動）                                          |
| R_OUT+ / R_OUT−    | 右聲道喇叭輸出（差動）                                          |
| EN（可選）         | 啟用腳，低電平時模組進入待機模式（部分版本）                   |
<br>
## 4ohm 3W speaker硬體介紹 <br>
4Ω 3W 喇叭是一款常見於嵌入式系統、智慧裝置與音訊模組中的 小功率聲音輸出元件，具備結構緊湊、阻抗與功率規格標準化、易於搭配音訊擴大器模組（如 PAM8403）等優點。此類喇叭設計用於近距離音訊播放，適合語音提示、簡易音樂與 TTS（Text-to-Speech）語音輸出等應用。<br>
**表五：4Ω 3W 喇叭模組規格表**

| 項目               | 規格說明                                         |
|--------------------|--------------------------------------------------|
| 阻抗（Impedance）  | 4Ω（歐姆）                                       |
| 額定功率           | 3W                                               |
| 響應頻率範圍       | 約 200Hz ~ 10kHz（視型號而定）                  |
| 音壓靈敏度         | 約 85 ~ 90 dB（1W/1m）                           |
| 直徑尺寸           | 常見尺寸為 36mm / 40mm / 長條型                 |
| 結構類型           | 有紙盆、塑膠盆、防塵網、磁鐵等結構              |
| 音圈材質           | 銅線音圈 / 鋁音圈                                |
<br>
## 編碼設計流程圖
![](https://github.com/kaoethan/MCU-project/blob/main/images/789.jpg?raw=true)<br>
## 提示詞<br>
**給予範例並提示需求**<br>
範例<br>
1) examples> AmebaQR> QRCodeScanner(這是AMB82-MINI 上使用 QR Code 掃描的 Arduino 範例)<br>
2) examples> AmebaNN> MultimediaAI > TextToSpeech(這是AMB82-MINI 上使用 Google TTS 的 Arduino 範例)<br>
3) exmaples> AmebaMultimedia > SDCardPlayMP3(這是AMB82-MINI 使用 SD 卡播放 MP3 音訊的範例)<br>
Feature: scan QR code to speak location<br>
step 1. Scan QR code to get the text (name of the location)<br>
step 2. Text-to-Speech to get the mp3 of the text<br>
step 3. SDCardPlayMP3 to play the mp3 to speak the name of the location<br>
## 專案流程圖
![](https://github.com/kaoethan/MCU-project/blob/main/images/blind.jpg?raw=true)<br>
## arduino程式碼
```
/*
  這個範例整合了QR Code掃描、文字轉語音和MP3播放功能.
  當AMB82-mini掃描到QR Code後，會將QR Code中的文字內容轉換為語音並播放出來.

  參考來源:
  - 文字轉語音: https://ameba-arduino-doc.readthedocs.io/en/latest/amebapro2/Example_Guides/Neural%20Network/Text-to-Speech.html
  - QR Code掃描: https://www.amebaiot.com/en/amebapro2-arduino-video-qrcode/
  - SD卡MP3播放: https://ameba-doc-arduino-sdk.readthedocs-hosted.com/en/latest/amebapro2/Example_Guides/Multimedia/Play%20MP3%20with%20SD%20card.html

  作者: ChungYi Fu (Kaohsiung, Taiwan) - 文字轉語音部分
*/

#include "WiFi.h"
#include <WiFiUdp.h>
#include "GenAI.h"
#include "AmebaFatFS.h"
#include "VideoStream.h"
#include "QRCodeScanner.h"

// Wi-Fi 設定
char ssid[] = "Yikao";     // 您的Wi-Fi SSID (網路名稱)
char pass[] = "20030108";  // 您的Wi-Fi密碼

// 檔案系統和AI物件
AmebaFatFS fs;
GenAI tts;

// MP3 檔案名稱
String mp3Filename = "qrcode_speech.mp3";

// 影像設定
#define CHANNEL 0
VideoSetting config(CHANNEL);
QRCodeScanner Scanner;

// ---
// Wi-Fi 連線函式

// 此函式用於初始化並連接到您的Wi-Fi網路.
void initWiFi() {
  for (int i = 0; i < 2; i++) {
    WiFi.begin(ssid, pass);
    delay(1000);
    Serial.println("");
    Serial.print("Connecting to ");
    Serial.println(ssid);

    uint32_t StartTime = millis();
    while (WiFi.status() != WL_CONNECTED) {
      delay(500);
      if ((StartTime + 5000) < millis()) {
        break;
      }
    }

    if (WiFi.status() == WL_CONNECTED) {
      Serial.println("");
      Serial.println("STA IP address: ");
      Serial.println(WiFi.localIP());
      Serial.println("");
      break;
    }
  }
}

// ---
// SD 卡播放 MP3 函式

// 此函式用於從 SD 卡播放指定的 MP3 檔案.
void sdPlayMP3(String filename) {
  fs.begin();
  String filepath = String(fs.getRootPath()) + filename;
  File file = fs.open(filepath, MP3);
  if (!file) {
    Serial.println("Error opening MP3 file!");
    fs.end();
    return;
  }
  file.setMp3DigitalVol(120); // 設定音量，範圍通常為 0-255
  file.playMp3();
  file.close();
  fs.end();
}

// ---
// Setup 函式

// 這是程式的初始化部分，只會執行一次. 它會初始化 Wi-Fi、相機和 QR Code 掃描器.
void setup() {
  Serial.begin(115200);
  initWiFi(); // 初始化Wi-Fi連線

  // 設定相機影像通道
  Camera.configVideoChannel(CHANNEL, config);
  Camera.videoInit();

  Scanner.StartScanning(); // 開始QR Code掃描
  Serial.println("QR Code Scanner started.");
}

// ---
// Loop 函式

// 這是程式的主迴圈，會重複執行. 它會不斷檢查 QR Code 掃描結果，如果掃描到內容，則會進行文字轉語音並播放.
void loop() {
  delay(500); // 縮短延遲以提高掃描反應速度

  Scanner.GetResultString();
  Scanner.GetResultLength();

  if (Scanner.ResultString != nullptr && Scanner.ResultLength > 0) {
    Serial.print("Result String is: ");
    Serial.println(Scanner.ResultString);
    Serial.print("Result Length is: ");
    Serial.println(Scanner.ResultLength);

    String message = String(Scanner.ResultString);
    Serial.print("Converting text to speech: ");
    Serial.println(message);

    // 執行文字轉語音，語言設定為英文 (en-US)
    // 如果需要其他語言，請參考Google Translate API的語言代碼
    // 例如: "zh-TW" 代表繁體中文
    tts.googletts(mp3Filename, message, "en-US");
    delay(1000); // 等待MP3檔案生成完成

    Serial.println("Playing MP3...");
    sdPlayMP3(mp3Filename); // 播放生成的MP3檔案

    // 清除掃描結果，避免重複播放
    Scanner.ResultString = nullptr;
    Scanner.ResultLength = 0;
  }
}


```
## 實作成果展示<br>
[![盲人導航系統展示](https://img.youtube.com/vi/vs5l0WUSbJA/0.jpg)](https://www.youtube.com/watch?v=vs5l0WUSbJA)<br>
This site was last updated {{ site.time | date: "%B %d, %Y" }}.

