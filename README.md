# STM32 Learning

這個 repository 用來整理 STM32 學習紀錄、範例專案和相關文件，每個資料夾內都有README說明。

## 共用開發板資訊

目前練習專案預設使用：

| 項目 | 內容 |
|---|---|
| 開發板 | NUCLEO-F446RE |
| MCU | STM32F446RE |
| 板上使用者 LED | LD2 |
| LD2 對應腳位 | PA5 |
| LD2 CubeMX 設定 | PA5 → GPIO_Output |

在 NUCLEO-F446RE 上：

```text
LD2 = PA5
PA5 輸出 High → LD2 亮
PA5 輸出 Low  → LD2 滅
```

如果使用其他 STM32 開發板，LED、按鈕和外設腳位可能不同，請以該開發板的 user manual / schematic 為準。

## 專案列表

- `Environment_Setup/`：Project 00，STM32CubeMX / STM32CubeIDE 環境建置與基本流程。
- `Blink_PA5/`：Project 01，LD2 / PA5 LED 閃爍練習。
- `Button_Control_LED/`：Project 02，使用板上 B1 按鈕控制 LD2 狀態。
- `文件/`：STM32 相關參考文件。
