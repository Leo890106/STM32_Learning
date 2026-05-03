# STM32 Learning

這個 repository 用來整理 STM32 學習紀錄、範例專案和相關文件。

## 專案列表

- `Blink_PA5/`：NUCLEO-F446RE / STM32F446RE 的 PA5 LED 閃爍練習。
- `文件/`：STM32 相關參考文件。

## Git 管理原則

- 保留 STM32CubeMX / STM32CubeIDE 專案來源檔，例如 `.ioc`、`Core/`、`Drivers/`、linker script 和 README。
- 不提交 `.metadata/`、`Debug/`、`Release/` 等本機 workspace 或編譯輸出。
- 之後新增新的練習專案時，可以直接放在根目錄下，每個專案各自保留自己的 README。
