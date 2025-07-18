---
layout: post
title: EdgeAI-AI輔助回收分類系統
author: [高屹]
category: [Lecture]
tags: [jekyll, ai]
---
# 邊緣計算微控制器原理與應用設計-AI輔助回收分類系統
This project uses a camera to take photos and sends them to AI to identify whether they can be recycled.

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
檢查 AI 回傳是否與前次不同，決定是否更新畫面/播放
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

1) GenAIVision_TTS(這是AMB82-MINI 使用 透過拍照→影像辨識→TTS→播放語音的範例)<br>
```
/*

This sketch shows the example of image prompts using APIs, plus text-to-speech using Google Translate API.

openAI platform - openAI vision
https://platform.openai.com/docs/guides/vision

Google AI Studio - Gemini vision
https://ai.google.dev/gemini-api/docs/vision

GroqCloud - Llama vision
https://console.groq.com/docs/overview

Example Guide: https://ameba-arduino-doc.readthedocs.io/en/latest/amebapro2/Example_Guides/Neural%20Network/Generative%20AI%20Vision.html
Example Guide: https://ameba-arduino-doc.readthedocs.io/en/latest/amebapro2/Example_Guides/Neural%20Network/Text-to-Speech.html

Credit : ChungYi Fu (Kaohsiung, Taiwan)

2025/04/25 example created by Richard Kuo, NTOU/EE
*/

String openAI_key = "";               // paste your generated openAI API key here
String Gemini_key = "";               // paste your generated Gemini API key here
String Llama_key = "";                // paste your generated Llama API key here
char wifi_ssid[] = "TCFSTWIFI.ALL";    // your network SSID (name)
char wifi_pass[] = "035623116";        // your network password

#include <WiFi.h>
#include <WiFiUdp.h>
#include "GenAI.h"
#include "VideoStream.h"
#include "AmebaFatFS.h"

WiFiSSLClient client;
GenAI llm;
GenAI tts;

AmebaFatFS fs;
String mp3Filename = "test_play_google_tts.mp3";

VideoSetting config(768, 768, CAM_FPS, VIDEO_JPEG, 1);
#define CHANNEL 0

uint32_t img_addr = 0;
uint32_t img_len = 0;

//String prompt_msg = "What type and name of the recyclables in the picture?";
String prompt_msg = "請問這個回收物是什麼?";

const int buttonPin = 1;          // the number of the pushbutton pin

void initWiFi()
{
    for (int i = 0; i < 2; i++) {
        WiFi.begin(wifi_ssid, wifi_pass);

        delay(1000);
        Serial.println("");
        Serial.print("Connecting to ");
        Serial.println(wifi_ssid);

        uint32_t StartTime = millis();
        while (WiFi.status() != WL_CONNECTED) {
            delay(500);
            if ((StartTime + 5000) < millis()) {
                break;
            }
        }

        if (WiFi.status() == WL_CONNECTED) {
            Serial.println("");
            Serial.println("STAIP address: ");
            Serial.println(WiFi.localIP());
            Serial.println("");
            break;
        }
    }
}

void setup()
{
    Serial.begin(115200);

    initWiFi();

    config.setRotation(0);
    Camera.configVideoChannel(CHANNEL, config);
    Camera.videoInit();
    Camera.channelBegin(CHANNEL);
    Camera.printInfo();
    
    pinMode(buttonPin, INPUT);
    pinMode(LED_B, OUTPUT);
    pinMode(LED_G, OUTPUT);    
}

void loop()
{
     if ((digitalRead(buttonPin)) == 1) {
        // Start MP4 recording after 3 seconds of blinking
        for (int count = 0; count < 3; count++) {
            digitalWrite(LED_B, HIGH);
            delay(500);
            digitalWrite(LED_B, LOW);
            delay(500);
        }

    // Vision prompt using same taken image
       Camera.getImage(0, &img_addr, &img_len);        
    // openAI vision prompt
    //  String text = llm.openaivision(openAI_key, "gpt-4o-mini", prompt_msg, img_addr, img_len, client);

    // Gemini vision prompt        
        String text = llm.geminivision(Gemini_key, "gemini-2.0-flash", prompt_msg, img_addr, img_len, client);

    // Llama vision prompt
    //  String text = llm.llamavision(Llama_key, "llama-3.2-90b-vision-preview", prompt_msg, img_addr, img_len, client); 
        Serial.println(text);
        tts.googletts(mp3Filename, text, "en-US");
        delay(500);
        sdPlayMP3(mp3Filename);

    }
}

void sdPlayMP3(String filename)
{
    fs.begin();
    String filepath = String(fs.getRootPath()) + filename;
    File file = fs.open(filepath, MP3);
    file.setMp3DigitalVol(120);
    file.playMp3();
    file.close();
    fs.end();
}
```
2) exmaples> AmebaSPI > LCD_Screen_ILI9341_TFT(這是AMB82-MINI 使用 顯示器的範例)<br>
1.Press button to capture an image<br>
2.Send Image to Google-Gemini and response message<br>
3.Send Message to Google-TTS and play mp3 file to speak 
## 專案流程圖
![](https://github.com/kaoethan/MCU-project/blob/main/images/recycle3.jpg?raw=true)<br>
## AI輔助回收分類系統程式碼說明
**1.作業目標(Objective):** <br>
AI-assisted Recycle System <br>
👉 使用人工智慧輔助的回收分類系統。主要功能是透過按鈕拍照，AI 辨識影像內容並語音播報，幫助使用者判斷垃圾屬於哪一類。<br>

**2.硬體設備(Hardware):** <br>
Development Board: AMB82-mini（MCU: Realtek RTL8735B）<br>
👉 使用 Realtek AMB82-mini 開發板，它是一款內建攝影機、支援 Wi-Fi、具備 AI 應用能力的微控制器。<br>

**3.功能說明(Features):** <br>
(一)按下按鈕拍照<br>
使用板上的按鈕觸發攝影機拍照。<br>

(二)送出照片到 Google Gemini（Vision 模型）分析內容<br>
利用 Google Gemini Vision AI 判斷照片裡的東西，例如「這是一個寶特瓶」或「這是一張紙」。<br>

(三)把 AI 分析出來的內容，透過 Google TTS 轉成語音並播放<br>
使用 Google Text-to-Speech (TTS) 將文字說出來，例如「這是一個可以回收的寶特瓶」。
## AI輔助回收分類系統arduino程式碼
```
/*
 功能說明：
 - 按下按鈕拍照
 - 送圖片至 Gemini Vision 分析並回傳文字
 - 將文字送 Google TTS 生成語音檔
 - 播放 MP3 語音
*/

#include <WiFi.h>
#include <WiFiUdp.h>
#include "GenAI.h"
#include "VideoStream.h"
#include "AmebaFatFS.h"

// === 請填寫以下資訊 ===
char ssid[] = "你的WiFi名稱";
const char password[] = "你的WiFi密碼";
String Gemini_key = "你的Gemini_API_KEY";  // https://makersuite.google.com/app/apikey

// ========== 全域變數 ==========
WiFiSSLClient client;
GenAI llm;
GenAI tts;
AmebaFatFS fs;
VideoSetting config(768, 768, CAM_FPS, VIDEO_JPEG, 1);
#define CHANNEL 0

const int buttonPin = 1;
String prompt_msg = "請問這個回收物是什麼 能不能回收?";
String mp3Filename = "voice.mp3";

uint32_t img_addr = 0;
uint32_t img_len = 0;

// ====== 初始化 WiFi ======
void initWiFi() {
  for (int i = 0; i < 2; i++) {
    WiFi.begin(ssid, password);
    delay(1000);
    Serial.println("");
    Serial.print("Connecting to ");
    Serial.println(ssid);

    uint32_t start = millis();
    while (WiFi.status() != WL_CONNECTED) {
      delay(500);
      if (millis() - start > 8000) break;
    }

    if (WiFi.status() == WL_CONNECTED) {
      Serial.println("\nConnected, IP address: ");
      Serial.println(WiFi.localIP());
      return;
    }
  }
  Serial.println("WiFi failed.");
}

// ====== MP3 播放函式 ======
void sdPlayMP3(String filename) {
  fs.begin();
  String filepath = String(fs.getRootPath()) + filename;
  File file = fs.open(filepath, MP3);
  file.setMp3DigitalVol(120);
  file.playMp3();
  file.close();
  fs.end();
}

void setup() {
  Serial.begin(115200);

  // 初始化 WiFi & 相機 & 按鈕
  initWiFi();
  pinMode(buttonPin, INPUT);
  pinMode(LED_B, OUTPUT);
  pinMode(LED_G, OUTPUT);

  config.setRotation(0);
  Camera.configVideoChannel(CHANNEL, config);
  Camera.videoInit();
  Camera.channelBegin(CHANNEL);
  Camera.printInfo();
}

void loop() {
  if (digitalRead(buttonPin) == 1) {
    // LED 藍色閃爍 3 次
    for (int i = 0; i < 3; i++) {
      digitalWrite(LED_B, HIGH); delay(300);
      digitalWrite(LED_B, LOW); delay(300);
    }

    // 拍照
    Camera.getImage(CHANNEL, &img_addr, &img_len);

    // 圖片送至 Gemini Vision
    String result = llm.geminivision(Gemini_key, "gemini-2.0-pro-vision", prompt_msg, img_addr, img_len, client);
    Serial.println("[Gemini 回應] " + result);

    // 將文字結果送至 Google TTS 並生成 MP3
    tts.googletts(mp3Filename, result, "zh-TW");
    delay(500);

    // 播放 MP3
    sdPlayMP3(mp3Filename);
    delay(2000);
  }
}


```

## 實作成果展示
![](https://github.com/kaoethan/MCU-project/blob/main/images/recycle2.jpeg?raw=true)<br>
測試使用圖片<br>
![](https://github.com/kaoethan/MCU-project/blob/main/images/recycle1.png?raw=true)<br>
執行結果<br>
This site was last updated {{ site.time | date: "%B %d, %Y" }}.

