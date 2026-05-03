# Project 01：點亮板子上的 LED

## 目標

本專案是第一個 STM32 GPIO 輸出練習，目標是讓 **NUCLEO-F446RE 開發板上的內建 LED（LD2）閃爍**。

開始前請先完成 [Project 00：環境建置](../Environment_Setup/README_Environment_Setup.md)。

本 repository 的共用開發板資訊整理在根目錄 [README.md](../README.md)。本專案只需要用到：

| 元件 | 腳位 | CubeMX 設定 |
|---|---|---|
| LD2 | PA5 | GPIO_Output |

---

## CubeMX 設定

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

## 修改 Core/Src/main.c

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

## 常用 GPIO HAL 函式

### 1. 延遲

```c
HAL_Delay(500);
```

代表延遲 500 ms，也就是 0.5 秒。

### 2. 讓 PA5 輸出 High

```c
HAL_GPIO_WritePin(GPIOA, GPIO_PIN_5, GPIO_PIN_SET);
```

對 NUCLEO-F446RE 的 LD2 來說：

```text
PA5 = High → LD2 亮
```

### 3. 讓 PA5 輸出 Low

```c
HAL_GPIO_WritePin(GPIOA, GPIO_PIN_5, GPIO_PIN_RESET);
```

對 NUCLEO-F446RE 的 LD2 來說：

```text
PA5 = Low → LD2 滅
```

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
LED：LD2
腳位：PA5
CubeMX 設定：PA5 → GPIO_Output
主要程式：Core/Src/main.c
燒錄流程：Build → Run
```
