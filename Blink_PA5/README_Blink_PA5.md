# Project 01：點亮板子上的 LED（NUCLEO-F446RE / STM32F446RE）

## 目標

本專案是第一個 STM32 練習專案，目標是讓 **NUCLEO-F446RE 開發板上的內建 LED（LD2）閃爍**。

> 注意：不同 STM32 開發板的 LED 腳位可能不一樣。  
> 本 README 以 **NUCLEO-F446RE** 為例。  
> 如果你使用其他板子，請依照自己的板子文件查找對應的 LED 腳位。

---

## 本專案使用的板子與腳位

| 項目 | 內容 |
|---|---|
| 開發板 | NUCLEO-F446RE |
| MCU | STM32F446RE |
| 板上 LED | LD2 |
| LD2 對應腳位 | PA5 |
| CubeMX 設定 | PA5 → GPIO_Output |

在 NUCLEO-F446RE 上：

```text
LD2 = PA5
PA5 輸出 High → LED 亮
PA5 輸出 Low  → LED 滅
```

---

## 第 1 步：打開 STM32CubeMX

新版 STM32CubeIDE 2.x 已經不再內建 STM32CubeMX，因此專案建立流程改成：

```text
STM32CubeMX 建立專案
→ Generate Code
→ STM32CubeIDE 匯入專案
→ Build / Run / Debug
```

所以第一步請先打開：

```text
STM32CubeMX
```

不是先打開 STM32CubeIDE。

如果電腦沒有 STM32CubeMX，需要另外安裝 STM32CubeMX。

---

## 第 2 步：在 STM32CubeMX 建立新專案

在 STM32CubeMX 裡選擇：

```text
ACCESS TO BOARD SELECTOR
```

搜尋：

```text
NUCLEO-F446RE
```

選擇：

```text
NUCLEO-F446RE
```

然後按：

```text
Start Project
```

如果找不到 Board Selector，也可以使用 MCU Selector 搜尋：

```text
STM32F446RETx
```

但如果你使用的是 NUCLEO-F446RE 開發板，建議優先使用 **Board Selector**，因為它會比較符合開發板本身的設定。

---

## 第 3 步：設定 PA5 為 GPIO_Output

進入 Pinout 畫面後，找到：

```text
PA5
```

點選 PA5，設定成：

```text
GPIO_Output
```

建議將 User Label 設成：

```text
LD2
```

如果有設定 Label 為 `LD2`，之後程式可以使用：

```c
HAL_GPIO_TogglePin(LD2_GPIO_Port, LD2_Pin);
```

如果沒有設定 Label，也可以直接使用：

```c
HAL_GPIO_TogglePin(GPIOA, GPIO_PIN_5);
```

---

## 第 4 步：Project Manager 設定

到 STM32CubeMX 上方的：

```text
Project Manager
```

設定：

```text
Project Name: Blink_PA5
Project Location: 選擇你要放專案的資料夾
Toolchain / IDE: STM32CubeIDE
```

其中最重要的是：

```text
Toolchain / IDE = STM32CubeIDE
```

不要選成：

```text
Makefile
CMake
EWARM
MDK-ARM
```

---

## 第 5 步：Generate Code

按右上角：

```text
GENERATE CODE
```

成功後，STM32CubeMX 會產生一個可被 STM32CubeIDE 匯入的專案資料夾。

專案結構大致會像這樣：

```text
Blink_PA5/
├── Core/
│   ├── Inc/
│   └── Src/
│       └── main.c
├── Drivers/
├── Blink_PA5.ioc
└── STM32F446RETX_FLASH.ld
```

> 注意：`Debug/` 資料夾通常要在 STM32CubeIDE 第一次 Build 之後才會出現。

之後主要修改的檔案是：

```text
Core/Src/main.c
```

不要自己建立 Empty Project，否則可能出現 `Src/` 和 `Core/Src/` 混在一起，導致你改錯 main.c。

---

## 第 6 步：打開 STM32CubeIDE 並匯入專案

現在才打開：

```text
STM32CubeIDE
```

選擇：

```text
File → STM32 Project Create/Import
```

接著選：

```text
Import STM32 Project
→ STM32CubeMX/STM32CubeIDE Project
```

選擇剛剛由 STM32CubeMX 產生的 `Blink_PA5` 資料夾，然後按：

```text
Finish
```

如果你的版本沒有 `STM32 Project Create/Import`，也可以用：

```text
File → Open Projects from File System...
```

選擇剛剛產生的專案資料夾。

---

## 第 7 步：修改 Core/Src/main.c

打開：

```text
Core/Src/main.c
```

找到 `while (1)` 迴圈。

建議把 LED 閃爍程式寫在：

```c
/* USER CODE BEGIN 3 */
/* USER CODE END 3 */
```

之間，避免之後重新 Generate Code 時被覆蓋。

### 寫法一：使用 PA5 腳位名稱

```c
while (1)
{
  /* USER CODE END WHILE */

  /* USER CODE BEGIN 3 */
  HAL_GPIO_TogglePin(GPIOA, GPIO_PIN_5);
  HAL_Delay(500);
  /* USER CODE END 3 */
}
```

### 寫法二：使用 CubeMX Label：LD2

如果你在 CubeMX 裡把 PA5 的 User Label 設成 `LD2`，可以寫成：

```c
while (1)
{
  /* USER CODE END WHILE */

  /* USER CODE BEGIN 3 */
  HAL_GPIO_TogglePin(LD2_GPIO_Port, LD2_Pin);
  HAL_Delay(500);
  /* USER CODE END 3 */
}
```

比較建議使用第二種，因為它可以確認 CubeMX 的 Label 有正確產生。

---

## 第 8 步：Build + Run

在 STM32CubeIDE 裡選：

```text
Project → Build Project
```

確認 Console 顯示：

```text
0 errors
```

然後接上 NUCLEO-F446RE，按：

```text
Run
```

或：

```text
Run → Run As → STM32 C/C++ Application
```

注意：

```text
Build 只是編譯，不會燒進板子
Run / Debug 才會下載程式到板子
```

如果下載成功，Console 通常會看到類似：

```text
Download verified successfully
```

---

## 常用 GPIO HAL 函式

### 1. 延遲

```c
HAL_Delay(500);
```

代表延遲 500 ms，也就是 0.5 秒。

---

### 2. 讓 PA5 輸出 High

```c
HAL_GPIO_WritePin(GPIOA, GPIO_PIN_5, GPIO_PIN_SET);
```

對 NUCLEO-F446RE 的 LD2 來說：

```text
PA5 = High → LD2 亮
```

---

### 3. 讓 PA5 輸出 Low

```c
HAL_GPIO_WritePin(GPIOA, GPIO_PIN_5, GPIO_PIN_RESET);
```

對 NUCLEO-F446RE 的 LD2 來說：

```text
PA5 = Low → LD2 滅
```

---

### 4. 反轉 PA5 狀態

```c
HAL_GPIO_TogglePin(GPIOA, GPIO_PIN_5);
```

意思是：

```text
如果 PA5 原本是 High → 變 Low
如果 PA5 原本是 Low  → 變 High
```

所以這行常用來做 LED 閃爍。

---


## 完整 LED Blink 範例

```c
while (1)
{
  HAL_GPIO_TogglePin(GPIOA, GPIO_PIN_5);
  HAL_Delay(500);
}
```

執行結果：

```text
LD2 亮 0.5 秒
LD2 滅 0.5 秒
持續重複
```

---

## 常見問題

### 1. 插上 USB 後有燈亮，代表成功嗎？

不一定。

NUCLEO 板子上有些燈是電源燈或 ST-LINK 狀態燈，插上 USB 本來就會亮。

你要確認的是板子上標示為：

```text
LD2
```

的那顆 LED 是否按照你的程式閃爍。

---

### 2. LED 完全不亮怎麼辦？

請依序檢查：

```text
1. 你看的是否是 LD2，而不是電源燈或 ST-LINK 燈
2. CubeMX 裡 PA5 是否設定成 GPIO_Output
3. 程式是否寫在 Core/Src/main.c
4. 是否有成功 Build，且 0 errors
5. 是否有按 Run / Debug，而不是只按 Build
6. 如果按 Debug，程式是否停在 breakpoint，需要按 Resume 才會繼續執行
```

---

### 3. 為什麼建議用 TogglePin，不是只用 WritePin 點亮？

如果只寫：

```c
HAL_GPIO_WritePin(GPIOA, GPIO_PIN_5, GPIO_PIN_SET);
```

LED 只會一直亮，較難判斷程式是否真的有在執行。

使用：

```c
HAL_GPIO_TogglePin(GPIOA, GPIO_PIN_5);
HAL_Delay(500);
```

可以讓 LED 閃爍，比較容易確認程式有成功跑起來。

---

## 本專案重點整理

```text
板子：NUCLEO-F446RE
MCU：STM32F446RE
LED：LD2
腳位：PA5
CubeMX 設定：PA5 → GPIO_Output
主要程式：Core/Src/main.c
燒錄流程：Build → Run
```
