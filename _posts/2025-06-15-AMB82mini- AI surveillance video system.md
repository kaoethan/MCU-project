---
layout: post
title: EdgeAI-AI監視錄影系統
author: [高屹]
category: [Lecture]
tags: [jekyll, ai]
---
# 邊緣計算微控制器原理與應用設計-AI監視錄影系統
This project uses the camera to take photos at regular intervals, sends them to AI to identify the scene, and stores the photos and scene descriptions.

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
| 背光模組   | LED 背光，PWM 可調整亮度                                          |<br>

**表二：ILI9341 TFT LCD 模組引腳定義與功能說明表**

| 序號 | 引腳標號   | 說明                                                             |
|------|------------|----------------------------------------------------------------|
| 1    | VCC        | 5V/3.3V電源輸入                                                 |
| 2    | GND        | 接地                                                            |
| 3    | CS         | 液晶屏片選信號，低電平使能                                       |
| 4    | RESET      | 液晶屏重定信號，低電平重定                                       |
| 5    | DC/RS      | 液晶屏寄存器/資料選擇信號，低電平：寄存器，高電平：數據             |
| 6    | SDI(MOSI)  | SPI匯流排寫資料信號                                             |
| 7    | SCK        | SPI匯流排時鐘信號                                               |
| 8    | LED        | 背光控制，高電平點亮，如無需控制則接3.3V常亮                      |
| 9    | SDO(MISO)  | SPI匯流排讀數據信號，如無需讀取功能則可不接                       |
|      |            | *(以下為觸摸屏信號線，如無需觸摸或模組本身不帶觸控，可不連接)*      |
| 10   | T_CLK      | 觸摸SPI匯流排時鐘信號                                           |
| 11   | T_CS       | 觸摸屏片選信號，低電平使能                                       |
| 12   | T_DIN      | 觸摸SPI匯流排輸入                                               |
| 13   | T_DO       | 觸摸SPI匯流排輸出                                               |
| 14   | T_IRQ      | 觸摸屏中斷信號，檢測到觸摸時為低電平                             | <br>

## PAM8403硬體介紹
PAM8403 是一款低功耗、高效率的 D 類音訊放大器晶片，可提供最高 3W + 3W 雙聲道立體聲輸出，適用於驅動如 4Ω/3W 或 8Ω/2W 的小型喇叭。它常被整合為 小型擴大器模組，可直接與微控制器（如 AMB82-Mini）搭配，用於播放語音合成（TTS）或音樂訊號。<br>

**表三：PAM8403 音訊放大器模組規格表**

| 項目         | 規格說明                                                       |
|--------------|----------------------------------------------------------------|
| 工作電壓     | 2.5V ~ 5.5V                                                    |
| 最大輸出功率 | 3W × 2（於 5V、4Ω 負載）                                       |
| 音訊輸入     | 模擬音訊（L/R 左右聲道輸入）                                   |
| 控制方式     | 無需 MCU 控制，可直接使用 PWM 或 DAC 輸入                      |
| 音質表現     | 低 THD（總諧波失真）與雜訊，音質清晰                            |
| 功耗特性     | 高效率（>85%），待機電流極低                                    |
| 大小         | 模組極小（約 2×2 cm），易於整合至小型裝置                      |<br>

**表四：PAM8403 音訊放大器模組腳位定義**

| 腳位名稱           | 功能說明                                                        |
|--------------------|----------------------------------------------------------------|
| VCC                | 電源正極（建議供應 5V）                                         |
| GND                | 電源地                                                         |
| L_IN               | 左聲道音訊輸入（模擬/PWM）                                      |
| R_IN               | 右聲道音訊輸入（模擬/PWM）                                      |
| L_OUT+ / L_OUT−    | 左聲道喇叭輸出（差動）                                          |
| R_OUT+ / R_OUT−    | 右聲道喇叭輸出（差動）                                          |
| EN（可選）         | 啟用腳，低電平時模組進入待機模式（部分版本）                       |<br>

## 4ohm 3W speaker硬體介紹
4Ω 3W 喇叭是一款常見於嵌入式系統、智慧裝置與音訊模組中的 小功率聲音輸出元件，具備結構緊湊、阻抗與功率規格標準化、易於搭配音訊擴大器模組（如 PAM8403）等優點。此類喇叭設計用於近距離音訊播放，適合語音提示、簡易音樂與 TTS（Text-to-Speech）語音輸出等應用。<br>

**表五：4Ω 3W 喇叭模組規格表**

| 項目               | 規格說明                                         |
|--------------------|-------------------------------------------------|
| 阻抗（Impedance）  | 4Ω（歐姆）                                       |
| 額定功率           | 3W                                               |
| 響應頻率範圍       | 約 200Hz ~ 10kHz（視型號而定）                    |
| 音壓靈敏度         | 約 85 ~ 90 dB（1W/1m）                           |
| 直徑尺寸           | 常見尺寸為 36mm / 40mm / 長條型                  |
| 結構類型           | 有紙盆、塑膠盆、防塵網、磁鐵等結構                 |
| 音圈材質           | 銅線音圈 / 鋁音圈                                |<br>

## GenAI 程式設計流程
在使用 GenAI 函式庫（特別針對 AMB82-MINI 開發板）進行程式設計時，整體流程可分為幾個明確的階段，根據功能需求（圖像辨識、語音辨識、文字生成、文字轉語音等）進行模組搭配。以下是 GenAI 程式設計的完整流程說明：<br>
**1.初始化階段** <br>
步驟:<br>
引入必要的頭檔（依功能需求）<br>
初始化 WiFi（連接網路）<br>
初始化 GenAI 函式庫與 API Key <br>
初始化設備（如 Camera、Microphone、RTC、LCD）<br>
**2.資料擷取階段(視應用)** <br>
根據應用目標，從不同來源擷取資料: <br>

圖像擷取(Camera):
```
GenAI_Camera camera;
camera.begin();
camera.capture();  // 擷取一張照片
```
語音錄音(Microphone）：
```
GenAI_Audio audio;
audio.begin();
audio.record(5);  // 錄 5 秒音
```
取得時間(RTC):
```
RTC.getTimeString();  // 取得目前時間字串
```
觸控輸入(ADC）：
```
int touchValue = analogRead(TOUCH_PIN);  // 讀取觸控
```
**3.呼叫AI模型推論** <br>
依需求選擇使用Gemini Vision、Text、Audio:<br>
圖像辨識(Vision):
```
String result = GenAI.visionDescription(camera.getImage());
```
語音辨識:(Whisper):
```
String transcript = GenAI.transcribe(audio.getWavData());
```
指令生成、故事生成(Text):
```
String prompt = "請用圖片寫一個童話故事";
String story = GenAI.generateText(prompt);
```
搭配RTC:
```
String prompt = "現在時間是 " + RTC.getTimeString() + "，請描述天氣狀況";
String result = GenAI.generateText(prompt);
```
**4.輸出結果(視需求)** <br>
可使用多種方式顯示/播放AI結果:<br>
顯示在LCD:
```
lcd.print(result);
```
播放語音(TTS):
```
TTS.speak(result);  // 使用 Google TTS 朗讀
```
儲存到SD卡(選用):
```
file = SD.open("/result.txt");
file.println(result);
file.close();
```
**5. 控制流程與互動** <br>
可以使用:<br>
按鈕 / 觸控切換模式<br>
定時器 / RTC 控制週期性觸發<br>
檢查 AI 回傳是否與前次不同，決定是否更新畫面/播放<br>
## 編碼設計流程圖
![](https://github.com/kaoethan/MCU-project/blob/main/images/789.jpg?raw=true)
## 程式生成提示語設計
程式生成提示語設計（Prompts for Code Generation）是一門設計如何清楚、有效地向 AI 模型（如 GPT、Gemini、Copilot 等）描述你想要產生的程式碼的技巧。良好的提示語可以幫助你獲得準確、可執行、易維護的程式碼。<br>

**為什麼提示語（Prompt）很重要？** <br>
AI 是根據你提供的文字提示來推論程式碼。提示設計得越清楚，輸出的程式碼越貼近你的需求。<br>

**設計程式生成提示語的 5 大原則:** <br>
1.明確說明目標功能 <br>
2.指定語言與平台 <br>
3.指出輸入 / 輸出規格 <br>
4.補充使用限制 / 框架 / API <br>
5.使用範例（很關鍵！）<br>
**小總結:** <br>
寫好提示語的重點是：「清楚、具體、有結構」。<br>
告訴 AI 你要什麼（功能、語言、框架）<br>
限制不想要的東西<br>
加入範例與邏輯限制<br>
越明確，越好用
## 專案提示詞
**給予範例並提示需求**<br>
範例<br>
1) examples> AmebaNN> MultimediaAI > GenAIVision(這是AMB82-MINI 上使用 GenAI 範例)<br>
2) exmaples> AmebaMultimedia > CaptureJPEG >SDCardSaveJPEG(這是AMB82-MINI 拍照後使用 SD 卡儲存照片的範例)<br>
3) examples > AmebaRTC > Simple_RTC.ino (這是AMB82-MINI 上使用 RTC 範例)
Function:
1) capture image per minute and send to Gemini Vision (1分鐘拍一張)<br>
2) if replied text has no change, then dont store the jpg and text<br>
 if replied text are different from the previous scene, then store the jpg and text (use date+time for the filename)
## 專案流程圖
![](https://github.com/kaoethan/MCU-project/blob/main/images/372.jpg?raw=true)
## AI監視錄影系統程式碼說明
**1.作業目標(Objective):** <br>
使用 AMB82-mini 開發板，每分鐘自動拍照一次，將照片送給 Gemini Vision 進行場景描述。如果與上一次的場景描述不同，則將該照片與描述儲存起來（使用日期與時間作為檔案名稱）。若與上次相同，則不儲存，節省空間。<br>

**2.開發板與功能（Board & Function）** <br>
Board: AMB82-mini（Realtek RTL8735B）<br>

👉 支援攝影機拍照、Wi-Fi 上傳、SD 卡儲存、RTC 實時時鐘功能。<br>

**3.功能流程說明（Function Flow）** <br>
(一)每分鐘自動拍照一次<br>
使用 RTC（實時時鐘） 或 millis() 計時器，每 60 秒觸發一次攝影機拍照。<br>

(二)照片送出給 Gemini Vision 做場景辨識<br>
拍下來的影像上傳給 Google Gemini Vision，得到一段文字描述（例如：”A park with people walking.”）<br>

(三)比對新回覆與上一次的文字是否相同<br>
如果相同 → 忽略，不存圖也不存文字 如果不同 → 儲存該張 JPG 圖片與文字檔，並使用 RTC 的日期與時間命名
## AI監視錄影系統arduino程式碼
```
/*
  This sketch captures an image every minute, sends it to Gemini Vision,
  and stores the image and description on an SD card only if the description changes.
  Filenames will include the date and time.
  Using 'gemini-1.5-flash' model as 'gemini-1.0-pro-vision' is deprecated.

  Credit : ChungYi Fu (Kaohsiung, Taiwan) - Original example codes
*/

#include <WiFi.h>
#include <NTPClient.h> // For getting time from NTP server
#include <WiFiUdp.h>   // Required for NTPClient

#include "GenAI.h"
#include "VideoStream.h"
#include "AmebaFatFS.h"

String Gemini_key = "AIzaSyAzAQRnNDlBXiac4E5SZcLSua-luXpbC3E"; // Paste your generated Gemini API key here
char wifi_ssid[] = "hahaha";       // Your network SSID (name)
char wifi_pass[] = "93034570";   // Your network password

WiFiSSLClient client;
GenAI llm;
VideoSetting config(768, 768, CAM_FPS, VIDEO_JPEG, 1); // 可根據需求調整解析度
#define CHANNEL 0

uint32_t img_addr = 0;
uint32_t img_len = 0;

String prompt_msg = "Please provide a brief summary of the image, including any text if visible."; // Simplified prompt
String previous_gemini_text = ""; // 儲存上一次 Gemini 的回應，用於比較

AmebaFatFS fs;
File logFile; // 用於儲存文字描述

// NTP Client setup
WiFiUDP ntpUDP;
// NTP 伺服器, GMT+8 時區偏移 (台灣時間)
// 請根據您的實際時區調整 8 * 3600
NTPClient timeClient(ntpUDP, "pool.ntp.org", 8 * 3600);

unsigned long lastCaptureTime = 0;
const unsigned long captureInterval = 60000; // 1 分鐘 (毫秒)

void initWiFi() {
  Serial.println("\n正在連接到 WiFi...");
  WiFi.begin(wifi_ssid, wifi_pass);

  uint32_t StartTime = millis();
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    if ((StartTime + 15000) < millis()) { // 增加 WiFi 連線超時時間
      Serial.println("連接 WiFi 失敗。正在重試...");
      StartTime = millis(); // 重設計時器以重試
    }
  }

  Serial.println("WiFi 連線成功。");
  Serial.print("IP 位址: ");
  Serial.println(WiFi.localIP());
}

String formatDateTime(unsigned long epochTime) {
  time_t rawtime = epochTime;
  struct tm * ti;
  ti = localtime(&rawtime);

  // 格式: YYYYMMDD_HHMMSS
  char buffer[20];
  sprintf(buffer, "%04d%02d%02d_%02d%02d%02d",
          ti->tm_year + 1900, ti->tm_mon + 1, ti->tm_mday,
          ti->tm_hour, ti->tm_min, ti->tm_sec);
  return String(buffer);
}

void setup() {
  Serial.begin(115200);

  initWiFi();

  // 初始化 NTP 客戶端
  timeClient.begin();
  Serial.println("正在從 NTP 伺服器更新時間...");
  while(!timeClient.update()) { // 等待時間更新完成
    Serial.print(".");
    delay(1000);
  }
  Serial.println("\n時間已更新。");
  Serial.print("當前時間: ");
  Serial.println(timeClient.getFormattedTime());

  config.setRotation(0); // 如果相機方向不同，請調整旋轉角度
  Camera.configVideoChannel(CHANNEL, config);
  Camera.videoInit();
  Camera.channelBegin(CHANNEL);
  Camera.printInfo();

  pinMode(LED_BUILTIN, OUTPUT);
  pinMode(LED_G, OUTPUT);

  // 初始化 SD 卡
  if (!fs.begin()) {
    Serial.println("錯誤: 無法初始化 SD 卡！");
    while (true); // 如果 SD 卡失敗則停止
  }
  Serial.println("SD 卡初始化成功。");

  lastCaptureTime = millis() - captureInterval; // 讓第一次捕捉立即發生
}

void loop() {
  timeClient.update(); // 定期更新時間

  // 檢查是否到了捕捉圖像的時間
  if (millis() - lastCaptureTime >= captureInterval) {
    lastCaptureTime = millis(); // 更新上次捕捉時間

    digitalWrite(LED_BUILTIN, HIGH); // 指示相機正在活動
    Serial.println("\n正在捕捉圖像...");
    Camera.getImage(0, &img_addr, &img_len);
    digitalWrite(LED_BUILTIN, LOW); // 關閉指示燈

    Serial.println("正在將圖像發送到 Gemini Vision (gemini-1.5-flash)...");
    // ****** 關鍵更改：將模型名稱從 "gemini-pro-vision" 更改為 "gemini-1.5-flash" ******
    String current_gemini_text = llm.geminivision(Gemini_key, "gemini-1.5-flash", prompt_msg, img_addr, img_len, client);

    // Simplify the Gemini response by trimming it to a certain length (e.g., 200 characters)
    if (current_gemini_text.length() > 200) {
      current_gemini_text = current_gemini_text.substring(0, 200) + "...";
    }

    Serial.println("\nGemini 回應:");
    Serial.println(current_gemini_text);

    // 與之前的文字進行比較
    // 檢查回應文字是否與上次不同，並且回應長度大於 0 (避免儲存空回應)
    if (current_gemini_text != previous_gemini_text && current_gemini_text.length() > 0) {
      Serial.println("偵測到場景變化或新的有效描述。正在儲存圖像和文字。");

      // 取得當前格式化的日期和時間作為檔名
      String dateTimeString = formatDateTime(timeClient.getEpochTime());

      // 儲存圖像
      String imageFileName = "/" + dateTimeString + ".jpg";
      File imageFile = fs.open(imageFileName);
      if (imageFile) {
        imageFile.write((uint8_t *)img_addr, img_len);
        imageFile.close();
        Serial.print("圖像已儲存為: ");
        Serial.println(imageFileName);
      } else {
        Serial.println("錯誤: 無法打開圖像文件進行寫入。");
      }

      // 儲存文字描述
      String logFileName = "/" + dateTimeString + ".txt";
      logFile = fs.open(logFileName);
      if (logFile) {
        logFile.print(current_gemini_text);
        logFile.close();
        Serial.print("描述已儲存為: ");
        Serial.println(logFileName);
      } else {
        Serial.println("錯誤: 無法打開描述文件進行寫入。");
      }

      previous_gemini_text = current_gemini_text; // 更新先前的文字
    } else {
      Serial.println("未偵測到明顯的場景變化或回應為空。不儲存。");
    }
  }

  // 為了避免迴圈執行過快，添加一個小延遲
  delay(100);
}


```

## 實作成果展示
![](https://github.com/kaoethan/MCU-project/blob/main/images/369.jpg?raw=true)<br>
**照片對應文字敘述** <br>
From a low-angle perspective, the image captures a person wearing glasses with a hand obscuring part of their face. The person has short, dark hair and fair skin. Their lips are slightly parted, and their tongue is visible. The hand appears to be resting gently on their face, with the fingers spread out. The glasses have a thick, dark frame and rectangular lenses, which reflect the light. The background is a muted, light gray color with subtle variations in tone. At the top, there's a bright, blurred light source, possibly a window or a lamp. The overall impression is an unconventional portrait with a strong focus on the hand and the person's facial features.<br>
![](https://github.com/kaoethan/MCU-project/blob/main/images/370.jpg?raw=true)<br>
**照片對應文字敘述** <br>
Here's a description of the image:<br>

The image is taken from a low angle, looking up at a person with short black hair and glasses. They are wearing a white shirt, a black vest, and a silver chain necklace. The person has a thoughtful expression, with a slight smile, and their hand is touching their chin. The background is a white ceiling with linear fluorescent lights. The perspective is skewed due to the low angle.<br>

This site was last updated {{ site.time | date: "%B %d, %Y" }}.


