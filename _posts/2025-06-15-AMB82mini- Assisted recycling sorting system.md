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
## AMB82-mini 硬體介紹<br>
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
## 編碼設計流程圖<br>
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
## 專案流程圖<br>
![](https://github.com/kaoethan/MCU-project/blob/main/images/blind.jpg?raw=true)<br>
## arduino程式碼<br>
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

