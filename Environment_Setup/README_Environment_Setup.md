# Project 00：環境建置

## 目標

本篇整理開始 STM32 練習前需要完成的環境建置與共用操作流程。

後續每一個任務都會沿用這個流程：

```text
STM32CubeMX 建立專案
→ Generate Code
→ STM32CubeIDE 匯入專案
→ Build / Run / Debug
```

共用開發板資訊請看根目錄 [README.md](../README.md)。

---

## 需要安裝的軟體

請先安裝：

```text
STM32CubeMX
STM32CubeIDE
```

新版 STM32CubeIDE 2.x 已經不再內建 STM32CubeMX，所以建議把兩個工具都獨立安裝好。

---

## 建立 STM32CubeMX 專案

打開：

```text
STM32CubeMX
```

選擇：

```text
ACCESS TO BOARD SELECTOR
```

搜尋並選擇：

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

使用 NUCLEO-F446RE 開發板時，建議優先使用 Board Selector，因為它會帶入比較符合開發板本身的預設設定。

---

## Project Manager 設定

到 STM32CubeMX 上方的：

```text
Project Manager
```

設定：

```text
Project Name: 依照任務名稱設定，例如 Blink_PA5
Project Location: 選擇你要放專案的資料夾
Toolchain / IDE: STM32CubeIDE
```

最重要的是：

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

## Generate Code

按右上角：

```text
GENERATE CODE
```

成功後，STM32CubeMX 會產生一個可被 STM32CubeIDE 匯入的專案資料夾。

專案結構大致會像這樣：

```text
Project_Name/
├── Core/
│   ├── Inc/
│   └── Src/
│       └── main.c
├── Drivers/
├── Project_Name.ioc
└── STM32F446RETX_FLASH.ld
```

> 注意：`Debug/` 資料夾通常要在 STM32CubeIDE 第一次 Build 之後才會出現。

之後主要修改的檔案是：

```text
Core/Src/main.c
```

不要自己建立 Empty Project，否則可能出現 `Src/` 和 `Core/Src/` 混在一起，導致你改錯 `main.c`。

---

## 匯入 STM32CubeIDE

打開：

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

選擇剛剛由 STM32CubeMX 產生的專案資料夾，然後按：

```text
Finish
```

如果你的版本沒有 `STM32 Project Create/Import`，也可以用：

```text
File → Open Projects from File System...
```

選擇剛剛產生的專案資料夾。

---

## Build / Run / Debug

在 STM32CubeIDE 裡選：

```text
Project → Build Project
```

確認 Console 顯示：

```text
0 errors
```

接上 NUCLEO-F446RE 後，可以使用：

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

## 常見問題

### 1. 找不到 Debug 資料夾

`Debug/` 通常要在第一次 Build 之後才會出現。

### 2. 程式沒有變化

請確認：

```text
1. 是否有成功 Build，且 0 errors
2. 是否有按 Run / Debug，而不是只按 Build
3. 是否修改的是 Core/Src/main.c
4. 如果按 Debug，程式是否停在 breakpoint，需要按 Resume 才會繼續執行
```

### 3. 不確定要改哪個 main.c

優先確認路徑是：

```text
Core/Src/main.c
```

不要改到其他測試資料夾或手動建立的 `Src/main.c`。

---

## 重點整理

```text
1. 先用 STM32CubeMX 建立專案
2. Toolchain / IDE 選 STM32CubeIDE
3. Generate Code
4. 用 STM32CubeIDE 匯入專案
5. 主要修改 Core/Src/main.c
6. Build 只編譯，Run / Debug 才會燒錄到板子
```
