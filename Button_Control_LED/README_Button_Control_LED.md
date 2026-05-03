# Project 02：按下 B1 按鈕反轉 LED 狀態

## 目標

本專案延續第一個 LED Blink 練習，這次要練習：

```text
按下 B1 一次 → LD2 狀態反轉一次
```

也就是：

```text
原本 LED 滅 → 按一下 B1 → LED 亮
原本 LED 亮 → 按一下 B1 → LED 滅
```

開始前請先完成 [Project 00：環境建置](../Environment_Setup/README_Environment_Setup.md)。
本 README 以根目錄 [README.md](../README.md) 的共用開發板設定為基準。

---

## 本專案使用的腳位

| 元件 | 功能 | MCU 腳位 | CubeMX 設定 |
|---|---|---|---|
| LD2 | 板上使用者 LED | PA5 | GPIO_Output |
| B1 | 板上使用者按鈕 | PC13 | GPIO_Input |

---

## B1 按鈕的狀態

B1 是 **active-low**，意思是：

```text
放開 B1：PC13 = GPIO_PIN_SET
按下 B1：PC13 = GPIO_PIN_RESET
```

所以判斷 B1 是否被按下時，通常要寫：

```c
if (HAL_GPIO_ReadPin(GPIOC, GPIO_PIN_13) == GPIO_PIN_RESET)
{
    // B1 被按下
}
```

---

## LD2 LED 的控制

LD2 接在 PA5。

```text
PA5 = GPIO_PIN_SET   → LD2 亮
PA5 = GPIO_PIN_RESET → LD2 滅
```

常用控制方式：

```c
HAL_GPIO_WritePin(GPIOA, GPIO_PIN_5, GPIO_PIN_SET);    // LED 亮
HAL_GPIO_WritePin(GPIOA, GPIO_PIN_5, GPIO_PIN_RESET);  // LED 滅
HAL_GPIO_TogglePin(GPIOA, GPIO_PIN_5);                 // LED 狀態反轉
```

---

## CubeMX 設定

請確認 CubeMX 中：

```text
PA5  → GPIO_Output
PC13 → GPIO_Input
```

建議設定：

### PA5 / LD2

```text
GPIO mode: Output Push Pull
GPIO Pull-up/Pull-down: No Pull
Maximum output speed: Low
User Label: LD2
```

### PC13 / B1

```text
GPIO mode: Input mode
GPIO Pull-up/Pull-down: No Pull
User Label: B1
```

---

## 練習 1：按住 B1 控制 LD2

目標：

```text
按下 B1 → LD2 亮
放開 B1 → LD2 滅
```

因為 B1 是 active-low：

```text
按下 B1：PC13 = GPIO_PIN_RESET
放開 B1：PC13 = GPIO_PIN_SET
```

所以程式可以寫成：

```c
while (1)
{
    if (HAL_GPIO_ReadPin(GPIOC, GPIO_PIN_13) == GPIO_PIN_RESET)
    {
        HAL_GPIO_WritePin(GPIOA, GPIO_PIN_5, GPIO_PIN_SET);
    }
    else
    {
        HAL_GPIO_WritePin(GPIOA, GPIO_PIN_5, GPIO_PIN_RESET);
    }
}
```

這個版本的行為是：

```text
B1 被按住期間，LED 會保持亮
B1 放開後，LED 會保持滅
```

這一題主要練習：

```text
1. 使用 HAL_GPIO_ReadPin() 讀取按鈕
2. 理解 B1 是 active-low
3. 使用 HAL_GPIO_WritePin() 控制 LED 亮或滅
```

---

## 練習 2：簡單版按鈕 Toggle

這是最直覺的寫法：

```c
while (1)
{
    if (HAL_GPIO_ReadPin(GPIOC, GPIO_PIN_13) == GPIO_PIN_RESET)
    {
        HAL_GPIO_TogglePin(GPIOA, GPIO_PIN_5);
        HAL_Delay(200);
    }
}
```

這段程式的意思：

```text
如果偵測到 B1 被按下
→ 反轉 PA5
→ 等待 200 ms
```

簡單版可以執行，但它有一個問題：

```text
只要你按住 B1 不放，它會每 200 ms 反轉一次
```

所以如果你按住比較久，結果可能會變成：

```text
亮 → 滅 → 亮 → 滅 → 亮 → 滅 ...
```

因此簡單版比較像是：

```text
按鈕目前是按下狀態時，每 200 ms Toggle 一次
```

它不是嚴格的：

```text
按一下只 Toggle 一次
```

---

## 練習 3：偵測按下瞬間 Toggle

如果要做到真正的：

```text
按一下 B1 → LED 只反轉一次
```

就要判斷按鈕狀態的變化。

也就是偵測：

```text
上一輪：放開，GPIO_PIN_SET
這一輪：按下，GPIO_PIN_RESET
```

這個變化代表 B1 剛剛被按下。

請把變數宣告放在 `while (1)` 前面，例如放在 `/* USER CODE BEGIN 2 */` 區塊後面：

```c
GPIO_PinState last_button_state = GPIO_PIN_SET;
```

然後 `while (1)` 改成：

```c
while (1)
{
    GPIO_PinState current_button_state = HAL_GPIO_ReadPin(GPIOC, GPIO_PIN_13);

    if (last_button_state == GPIO_PIN_SET &&
        current_button_state == GPIO_PIN_RESET)
    {
        HAL_Delay(20);  // debounce，避免按鈕彈跳造成誤判

        if (HAL_GPIO_ReadPin(GPIOC, GPIO_PIN_13) == GPIO_PIN_RESET)
        {
            HAL_GPIO_TogglePin(GPIOA, GPIO_PIN_5);
        }
    }

    last_button_state = current_button_state;
}
```

正式版完整邏輯：

```text
1. 讀取目前 B1 狀態
2. 如果上一輪是放開，這一輪是按下
3. 代表發生一次「剛按下」
4. 延遲 20 ms 做 debounce
5. 再確認 B1 仍然是按下
6. 反轉 LED
7. 更新 last_button_state
```

---

## 為什麼需要 last_button_state？

因為 MCU 執行 while 迴圈的速度非常快。

如果只寫：

```c
if (button is pressed)
{
    toggle LED;
}
```

那麼只要按鈕還被按住，MCU 就會一直進入 if，一直反轉 LED。

所以我們要記住上一輪的按鈕狀態：

```c
GPIO_PinState last_button_state = GPIO_PIN_SET;
```

然後比較：

```c
last_button_state
current_button_state
```

這樣才能知道按鈕是：

```text
剛按下
一直按著
剛放開
一直放開
```

---

## 為什麼需要 HAL_Delay(20)？

機械式按鈕按下或放開時，接點不會立刻穩定，可能會在很短時間內快速震盪。

這個現象叫做：

```text
button bounce
按鈕彈跳
```

所以程式中加入：

```c
HAL_Delay(20);
```

代表等待 20 ms，讓按鈕訊號穩定一點，再重新確認一次是否真的按下。

這種做法叫做：

```text
debounce
消抖
```

---

## 補充：等待放開 vs 狀態變化偵測

在按鈕 Toggle LED 的練習中，常見有兩種寫法可以避免「按住不放一直 Toggle」：

```text
1. Toggle 後用 while 等待按鈕放開
2. 使用前一個狀態 + 當前狀態判斷按下瞬間
```

這兩種寫法都可以使用，但差別在於：**會不會卡住主程式**。

### 寫法一：Toggle 後用 while 等待放開

```c
if (HAL_GPIO_ReadPin(GPIOC, GPIO_PIN_13) == GPIO_PIN_RESET)
{
    HAL_Delay(20);  // debounce

    if (HAL_GPIO_ReadPin(GPIOC, GPIO_PIN_13) == GPIO_PIN_RESET)
    {
        HAL_GPIO_TogglePin(GPIOA, GPIO_PIN_5);

        while (HAL_GPIO_ReadPin(GPIOC, GPIO_PIN_13) == GPIO_PIN_RESET)
        {
            // 等待按鈕放開
        }
    }
}
```

這種寫法的效果是：

```text
按下 B1 → LED Toggle 一次
持續按住 B1 → 不會繼續 Toggle
放開 B1 → 程式才會繼續往下跑
```

它的優點是簡單、直覺，很適合單一按鈕 GPIO 練習。

但是它的缺點是：

```text
只要按鈕還沒放開，程式會卡在 while 裡面
```

### 多個按鈕時，while 等待放開的問題

假設有兩個按鈕：

```text
KEY1 控制 LED1
KEY2 控制 LED2
```

如果程式寫成這樣：

```c
while (1)
{
    if (HAL_GPIO_ReadPin(KEY1_GPIO_Port, KEY1_Pin) == GPIO_PIN_RESET)
    {
        HAL_GPIO_TogglePin(LED1_GPIO_Port, LED1_Pin);

        while (HAL_GPIO_ReadPin(KEY1_GPIO_Port, KEY1_Pin) == GPIO_PIN_RESET)
        {
            // 等待 KEY1 放開
        }
    }

    if (HAL_GPIO_ReadPin(KEY2_GPIO_Port, KEY2_Pin) == GPIO_PIN_RESET)
    {
        HAL_GPIO_TogglePin(LED2_GPIO_Port, LED2_Pin);

        while (HAL_GPIO_ReadPin(KEY2_GPIO_Port, KEY2_Pin) == GPIO_PIN_RESET)
        {
            // 等待 KEY2 放開
        }
    }
}
```

當你按住 KEY1 不放時，程式會卡在 KEY1 的 while 裡：

```text
KEY1 還沒放開 → 程式一直卡住
KEY2 的 if 不會被執行
所以 KEY2 暫時無法工作
```

因此，這種 blocking 寫法在單一按鈕練習可以用，但在多按鈕、多任務、UART、感測器、馬達控制等情境中就不太適合。

### 寫法二：前一狀態 + 當前狀態

比較推薦的寫法是記錄按鈕的上一輪狀態，並和目前狀態比較。

```c
GPIO_PinState last_button_state = GPIO_PIN_SET;

while (1)
{
    GPIO_PinState current_button_state = HAL_GPIO_ReadPin(GPIOC, GPIO_PIN_13);

    if (last_button_state == GPIO_PIN_SET &&
        current_button_state == GPIO_PIN_RESET)
    {
        HAL_Delay(20);  // debounce

        if (HAL_GPIO_ReadPin(GPIOC, GPIO_PIN_13) == GPIO_PIN_RESET)
        {
            HAL_GPIO_TogglePin(GPIOA, GPIO_PIN_5);
        }
    }

    last_button_state = current_button_state;
}
```

這種寫法偵測的是：

```text
上一輪：放開，GPIO_PIN_SET
這一輪：按下，GPIO_PIN_RESET
```

也就是偵測「剛按下」的瞬間。

所以按住不放時：

```text
第一輪：SET → RESET，Toggle 一次
之後：RESET → RESET，不會再 Toggle
```

### 多個按鈕時，前一狀態 + 當前狀態比較適合

多個按鈕可以各自記錄自己的 last state。

```c
GPIO_PinState last_key1_state = GPIO_PIN_SET;
GPIO_PinState last_key2_state = GPIO_PIN_SET;

while (1)
{
    GPIO_PinState current_key1_state = HAL_GPIO_ReadPin(KEY1_GPIO_Port, KEY1_Pin);
    GPIO_PinState current_key2_state = HAL_GPIO_ReadPin(KEY2_GPIO_Port, KEY2_Pin);

    if (last_key1_state == GPIO_PIN_SET &&
        current_key1_state == GPIO_PIN_RESET)
    {
        HAL_GPIO_TogglePin(LED1_GPIO_Port, LED1_Pin);
    }

    if (last_key2_state == GPIO_PIN_SET &&
        current_key2_state == GPIO_PIN_RESET)
    {
        HAL_GPIO_TogglePin(LED2_GPIO_Port, LED2_Pin);
    }

    last_key1_state = current_key1_state;
    last_key2_state = current_key2_state;

    HAL_Delay(10);  // 簡單降低掃描速度，也可稍微減少彈跳影響
}
```

這種寫法的特點是：

```text
KEY1 被按住時，不會一直 Toggle
KEY1 被按住時，程式仍然會繼續檢查 KEY2
不會因為等待 KEY1 放開而卡住整個主迴圈
```

所以多按鈕時，這種寫法比 `while 等待放開` 更推薦。

### 兩種寫法比較

| 寫法 | 按住會不會一直 Toggle | 會不會卡住主程式 | 適合情境 |
|---|---|---|---|
| 按下就 Toggle + Delay | 會 | 短暫卡住 | 用來觀察問題 |
| Toggle 後 while 等放開 | 不會 | 會卡住直到放開 | 單一 GPIO 按鈕練習 |
| 前一狀態 + 當前狀態 | 不會 | 不會長時間卡住 | 多按鈕、多任務，較推薦 |

### 對 NUCLEO-F446RE 的提醒

NUCLEO-F446RE 板上可以當一般 GPIO 輸入的使用者按鈕是：

```text
B1 = PC13
```

另一顆按鈕：

```text
B2 = NRST
```

是 Reset 按鈕，不適合當一般 GPIO 按鈕使用。

所以如果要練多按鈕，需要額外外接按鈕，例如：

```text
KEY1 = PC13，也就是板上 B1
KEY2 = PB3，外接按鈕
KEY3 = PB4，外接按鈕
```

### 這一段的結論

```text
Toggle 後 while 等放開：
適合單一按鈕練習，但按住時會卡住主程式。

前一狀態 + 當前狀態：
可以避免按住一直 Toggle，也不會卡住等待放開，較適合多按鈕與後續專案。
```

---

## 本專案重點整理

```text
LD2 = PA5
B1  = PC13

B1 放開 = GPIO_PIN_SET
B1 按下 = GPIO_PIN_RESET

LD2 亮 = GPIO_PIN_SET
LD2 滅 = GPIO_PIN_RESET

HAL_GPIO_ReadPin()   → 讀取按鈕狀態
HAL_GPIO_WritePin()  → 指定 LED 亮或滅
HAL_GPIO_TogglePin() → 反轉 LED 狀態
HAL_Delay()          → 延遲，也可用於簡單 debounce
```

---

## 最重要的觀念

簡單版：

```text
按鈕按住期間，每隔一段時間反轉一次
```

正式版：

```text
只在「放開 → 按下」的瞬間反轉一次
```

所以如果題目是「按下按鈕，LED 狀態反轉」，正式版會比簡單版更正確。
