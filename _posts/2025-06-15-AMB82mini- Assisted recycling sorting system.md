---
layout: post
title: 盲人導航系統
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

