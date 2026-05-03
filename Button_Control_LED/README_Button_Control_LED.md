# Project 02：按下 B1 按鈕反轉 LED 狀態

## 目標

本專案延續第一個 LED Blink 練習，這次要練習：

```text
按下 B1 一次 → LD2 狀態反轉一次
```

也就是：

```text
原本 LED 暗 → 按一下 B1 → LED 亮
原本 LED 亮 → 按一下 B1 → LED 暗
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

常用控制方式：

```c
HAL_GPIO_WritePin(GPIOA, GPIO_PIN_5, GPIO_PIN_SET);    // LED 亮
HAL_GPIO_WritePin(GPIOA, GPIO_PIN_5, GPIO_PIN_RESET);  // LED 暗
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
放開 B1 → LD2 暗
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
B1 放開後，LED 會保持暗
```

這一題主要練習：

```text
1. 使用 HAL_GPIO_ReadPin() 讀取按鈕
2. 理解 B1 是 active-low
3. 使用 HAL_GPIO_WritePin() 控制 LED 亮或暗
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
亮 → 暗 → 亮 → 暗 → 亮 → 暗 ...
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
        HAL_Delay(20);

        if (HAL_GPIO_ReadPin(GPIOC, GPIO_PIN_13) == GPIO_PIN_RESET)
        {
            HAL_GPIO_TogglePin(GPIOA, GPIO_PIN_5);
        }
    }

    last_button_state = current_button_state;
}
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

## 本專案重點整理

```text
LD2 = PA5
B1  = PC13

B1 放開 = GPIO_PIN_SET
B1 按下 = GPIO_PIN_RESET

LD2 亮 = GPIO_PIN_SET
LD2 暗 = GPIO_PIN_RESET

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
