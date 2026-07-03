# 企劃書：Codex QuickShot

## LightShot-style 截圖標註 + Codex 附件橋接工具

版本：Planning Draft v0.2
技術方向：**Rust + Tauri 2 + Native Platform Adapter**
產品型態：**常駐系統列 / Menu Bar 小工具，不佔據主視窗**
目標平台：Windows、macOS
核心目標：**截圖 → 標註 → 儲存 → 放入 Codex 聊天附件區 → 但不自動送出**

---

## 一、專案摘要

**Codex QuickShot** 是一個像 LightShot 一樣的快速截圖工具，但它的主要目的不是單純分享截圖，而是讓使用者可以更快把「有標註的截圖」交給 **Codex App** 理解。

它會常駐在：

```text
Windows：右下角系統列圖示
macOS：右上角 Menu Bar 圖示
```

平常不開主視窗、不佔桌面空間。使用者可以透過系統列圖示點擊使用，也可以設定快捷鍵啟用。這點會刻意模仿 LightShot 的使用體驗：快速選取任意螢幕區域，並能在截圖後立即編輯與標註。LightShot 官方也主打可選取桌面任意區域並快速截圖，且提供截圖後編輯能力。([Lightshot][1])

Codex QuickShot 的完整流程是：

```text
按快捷鍵或點擊 Tray 圖示
→ 框選截圖區域
→ 在截圖上畫紅線、畫框、畫箭頭、輸入文字
→ 儲存最終 PNG 到指定位置
→ 自動切回 Codex App
→ 將圖片放入目前聊天 composer 附件區
→ 不自動送出
→ 使用者確認後自行送出
```

---

## 二、產品定位

### 2.1 一句話定位

> **Codex QuickShot 是一個常駐系統列的跨平台截圖標註工具，可以像 LightShot 一樣快速截圖、畫重點，並把圖片放入 Codex 聊天附件區，但永遠不自動送出。**

### 2.2 產品不是什麼

Codex QuickShot 不是：

```text
不是 Codex App 內部插件
不是修改 Codex App 的工具
不是截圖雲端上傳服務
不是自動送出訊息工具
不是遠端控制 Codex 的私有 API 工具
不是常駐大型主視窗 App
```

### 2.3 產品是什麼

Codex QuickShot 是：

```text
桌面常駐小工具
LightShot-style 截圖標註工具
Codex App companion app
本機截圖管理工具
Codex 圖片上下文輔助工具
開源跨平台桌面工具
```

---

## 三、官方能力與可行性依據

Codex App 官方文件目前已說明，可以把圖片拖放到 prompt composer 作為上下文，因此本專案可以採用「圖片貼上 / 檔案拖放」這類不破壞 Codex App 的整合方式。([OpenAI 開發者][2])

但 Codex deep link 官方文件目前列出的 `codex://new` 參數主要是 `prompt`、`path`、`originUrl`，沒有看到可直接加入圖片附件草稿的 `image`、`attachment`、`file` 之類參數。這代表本專案不應該依賴未公開的附件 API，而應該用 OS 層級的剪貼簿、檔案拖放、聚焦視窗與貼上行為來達成「放入 composer 但不送出」。([OpenAI 開發者][3])

Tauri 2 適合這類常駐小工具，因為官方支援 system tray，能提供快速操作入口；Tauri 的 global shortcut plugin 也支援註冊全域快捷鍵。([Tauri][4])

Codex Plugins 可以打包 Skills、App integrations 與 MCP servers，所以本專案未來可以再提供一個 Codex Plugin / MCP server，讓 Codex 能查詢最近截圖、讀取截圖 metadata，但「截圖與貼入 composer」本身應該由 Desktop Companion App 負責。([OpenAI 開發者][5])

---

## 四、核心使用情境

### 情境一：工程師要讓 Codex 看錯誤畫面

```text
1. 使用者在瀏覽器或 IDE 看到錯誤。
2. 按下快捷鍵。
3. 框選錯誤訊息。
4. 用紅框圈出錯誤。
5. 用箭頭指向重點。
6. 輸入文字：「這裡按下後沒有反應」。
7. 按完成。
8. 圖片自動放入 Codex composer。
9. 使用者補一句：「請幫我分析紅框處的錯誤」。
10. 使用者自行送出。
```

### 情境二：UI 開發要標註畫面問題

```text
1. 使用者框選 Web UI。
2. 在錯位的元件上畫紅框。
3. 在錯誤間距處畫線。
4. 用文字標註「這裡應該對齊左側」。
5. 傳給 Codex 分析 CSS / layout 問題。
```

### 情境三：截圖中含敏感資訊

```text
1. 使用者框選畫面。
2. 發現畫面中有姓名、Email、token。
3. 使用馬賽克 / 模糊工具遮掉。
4. 再放入 Codex 附件區。
5. 使用者確認沒有敏感資料後才送出。
```

---

## 五、使用體驗設計

## 5.1 常駐模式：不佔據視窗

Codex QuickShot 預設啟動後不顯示主視窗。

### Windows

```text
啟動後只出現在右下角 System Tray。
不顯示主視窗。
不佔用工作列。
點擊圖示可開始截圖。
右鍵圖示可開啟選單。
設定頁只有需要時才打開。
```

### macOS

```text
啟動後只出現在右上角 Menu Bar。
可選擇是否隱藏 Dock icon。
點擊圖示可開始截圖。
右鍵 / 下拉選單可開啟功能選單。
設定頁只有需要時才打開。
```

---

## 5.2 Tray / Menu Bar 操作

建議選單：

```text
Codex QuickShot
├─ Capture Region and Attach
├─ Capture Region Only
├─ Capture Fullscreen and Attach
├─ Capture Active Window and Attach
├─ Open Recent Screenshots
├─ Open Screenshot Folder
├─ Settings
├─ Pause Hotkeys
└─ Quit
```

### 左鍵點擊圖示

預設行為：

```text
直接進入區域截圖模式
```

可在設定中改成：

```text
左鍵點擊開啟選單
雙擊進入截圖
```

### 右鍵點擊圖示

顯示完整選單。

---

## 5.3 快捷鍵設計

Windows 預設：

```text
Ctrl + Shift + 5：區域截圖、標註、附加到 Codex
Ctrl + Shift + 6：全螢幕截圖、標註、附加到 Codex
Ctrl + Shift + 7：目前視窗截圖、標註、附加到 Codex
Ctrl + Shift + 8：只截圖標註，不附加到 Codex
```

macOS 預設：

```text
Command + Shift + 5：區域截圖、標註、附加到 Codex
Command + Shift + 6：全螢幕截圖、標註、附加到 Codex
Command + Shift + 7：目前視窗截圖、標註、附加到 Codex
Command + Shift + 8：只截圖標註，不附加到 Codex
```

快捷鍵必須可以自訂，並且需要偵測是否與系統或其他工具衝突。

---

## 六、截圖與標註流程

## 6.1 完整流程

```text
使用者按快捷鍵 / 點擊 Tray
        ↓
擷取目前螢幕快照作為 frozen background
        ↓
畫面變暗，進入框選模式
        ↓
使用者拖曳選取區域
        ↓
選取完成後進入標註模式
        ↓
使用者畫紅線、紅框、箭頭、文字
        ↓
按下完成
        ↓
輸出合成後 PNG
        ↓
儲存到指定位置
        ↓
複製圖片到剪貼簿
        ↓
嘗試聚焦 Codex App
        ↓
嘗試貼入 / 拖入 Codex composer
        ↓
不送出
```

---

## 6.2 為什麼要先擷取 frozen background

標註工具不能直接在即時桌面上畫，否則會有幾個問題：

```text
工具列可能被截進圖片
半透明遮罩可能被截進圖片
螢幕內容可能在標註時變動
多螢幕與 DPI 座標會更難處理
```

正確做法：

```text
1. 快捷鍵觸發時先擷取完整螢幕快照。
2. 將快照顯示為 frozen background。
3. 使用者在 frozen background 上框選。
4. 標註以 vector object 記錄。
5. 完成後從原始快照裁切區域。
6. 將標註合成到裁切圖片。
7. 輸出 final PNG。
```

---

## 七、LightShot-style Annotation Editor

## 7.1 MVP 標註工具

第一版至少支援：

```text
紅色矩形框
紅色箭頭
直線
自由手繪筆
文字標註
Undo
Redo
刪除選取標註
完成
取消
```

這些功能足以支援大部分給 Codex 看圖的需求。

---

## 7.2 標註工具列

框選完成後，工具列浮在選取區域旁邊，不另外開主視窗。

工具列範例：

```text
[筆] [線] [箭頭] [框] [文字] [馬賽克] [復原] [重做] [完成] [取消]
```

工具列位置：

```text
優先顯示在選取框右側
若右側空間不足，顯示在下方
若下方空間不足，顯示在上方
永遠不遮住主要標註區域
```

---

## 7.3 標註模式快捷鍵

```text
R：矩形框
A：箭頭
L：直線
P：自由手繪筆
T：文字
M：馬賽克 / 模糊
Delete：刪除選取標註
Ctrl / Cmd + Z：Undo
Ctrl / Cmd + Shift + Z：Redo
Enter：完成並附加
Esc：取消
Ctrl / Cmd + S：只儲存
Ctrl / Cmd + C：複製最終圖片
```

---

## 7.4 第二階段標註功能

v0.2 之後加入：

```text
顏色選擇
線條粗細
文字大小
高亮筆
橢圓形
編號標記 1、2、3
可移動標註物件
可調整箭頭與框線大小
重新編輯最近截圖
```

---

## 7.5 隱私標註工具

v0.3 必須加入：

```text
馬賽克
Blur
黑色遮蔽條
局部裁切
一鍵清除 raw image
```

因為使用者很可能截到：

```text
姓名
學號
Email
電話
Token
API Key
內部網址
管理後台資料
學校系統資料
```

---

## 八、Codex 附件橋接設計

## 8.1 核心原則

Codex QuickShot 只做三件事：

```text
把圖片準備好
把圖片放到 Codex composer 附件區
停住，不送出
```

它絕對不做：

```text
不按 Enter
不按 Send
不呼叫未公開 API
不修改 Codex App
不讀 Codex token
不寫 Codex session database
```

---

## 8.2 附加方式順序

### 第一層：Clipboard Attach Mode

```text
1. 將 final PNG 寫入系統剪貼簿。
2. 同時寫入 file-drop 格式。
3. 找到 Codex App 視窗。
4. 聚焦 Codex App。
5. 嘗試聚焦 composer。
6. 送出 Ctrl+V / Cmd+V。
7. 不送出。
```

### 第二層：Drag-and-Drop Attach Mode

```text
1. 找到 Codex App 視窗。
2. 嘗試取得 composer 區域位置。
3. 模擬把 final PNG 拖放到 composer。
4. 不送出。
```

這是符合官方「圖片可拖放到 prompt composer」能力的做法。([OpenAI 開發者][2])

### 第三層：Manual Fallback Mode

如果自動附加失敗：

```text
顯示通知：
「截圖已儲存並複製到剪貼簿，請回到 Codex App 按 Ctrl+V / Cmd+V。」

提供按鈕：
- 開啟圖片
- 開啟資料夾
- 複製檔案路徑
- 重新嘗試附加
```

---

## 九、技術架構

## 9.1 技術選型

```text
App Shell：Tauri 2
核心語言：Rust
前端 UI：SvelteKit 或 React
標註層：Canvas + SVG
圖片處理：Rust image processing
Windows Adapter：Rust + Win32 / Windows API
macOS Adapter：Rust + Swift Helper
MCP Server：Rust
Plugin：Codex Plugin manifest + Skill + MCP config
```

Tauri 2 支援使用前端框架建立 UI，並以 Rust 為應用邏輯核心，也能在需要時整合 Swift、Kotlin 等原生能力，適合本專案的跨平台桌面工具架構。([Tauri][6])

---

## 9.2 整體架構圖

```text
Codex QuickShot
├─ Tray / Menu Bar App
│  ├─ Tray Icon
│  ├─ Tray Menu
│  ├─ Global Hotkeys
│  └─ Settings Window
│
├─ Capture Overlay
│  ├─ Frozen Background
│  ├─ Region Selection
│  ├─ Annotation Toolbar
│  └─ Annotation Canvas / SVG Layer
│
├─ Rust Core
│  ├─ Config Manager
│  ├─ Storage Manager
│  ├─ Metadata Manager
│  ├─ Annotation Model
│  ├─ Image Composer
│  ├─ History Manager
│  └─ Workflow Orchestrator
│
├─ Native Platform Adapter
│  ├─ Windows Adapter
│  │  ├─ Screen Capture
│  │  ├─ Clipboard
│  │  ├─ Window Finder
│  │  ├─ Input Simulation
│  │  └─ UI Automation
│  │
│  └─ macOS Adapter
│     ├─ ScreenCaptureKit / Screenshot Capture
│     ├─ NSPasteboard
│     ├─ Accessibility
│     ├─ App Activation
│     └─ Input Simulation
│
├─ Codex Attach Bridge
│  ├─ Clipboard Attach
│  ├─ Drag-and-Drop Attach
│  └─ Manual Fallback
│
└─ Optional Codex Plugin / MCP
   ├─ quickshot_get_latest
   ├─ quickshot_list_recent
   ├─ quickshot_get_image_info
   └─ quickshot_open_folder
```

---

## 9.3 為什麼用 Canvas + SVG

建議：

```text
底圖：Canvas / Image
標註物件：SVG Layer
輸出：Rust 或前端 Canvas 合成 final PNG
```

理由：

```text
Canvas 適合顯示截圖底圖。
SVG 適合管理矩形、箭頭、文字這類向量物件。
標註物件可移動、刪除、Undo / Redo。
完成後再合成成 PNG。
```

---

## 十、平台 Adapter 設計

## 10.1 共用介面

Rust core 不直接碰 Windows 或 macOS API，而是透過平台 adapter：

```rust
pub trait PlatformAdapter {
    fn capture_screen_snapshot(&self) -> Result<RawScreenshot>;
    fn write_image_to_clipboard(&self, path: &Path) -> Result<()>;
    fn write_file_drop_to_clipboard(&self, path: &Path) -> Result<()>;

    fn find_codex_app(&self) -> Result<Option<CodexWindow>>;
    fn focus_codex_app(&self, window: &CodexWindow) -> Result<()>;
    fn focus_codex_composer(&self, window: &CodexWindow) -> Result<()>;

    fn paste_into_codex(&self) -> Result<AttachResult>;
    fn drag_drop_file_to_codex(&self, path: &Path) -> Result<AttachResult>;
}
```

---

## 10.2 Windows Adapter

Windows 端建議使用：

```text
截圖：
Windows Graphics Capture
GDI fallback

剪貼簿：
Bitmap
PNG
CF_HDROP file-drop

找 Codex：
EnumWindows
Process name / window title matching

聚焦：
SetForegroundWindow
ShowWindow

輸入：
SendInput Ctrl+V

進階：
UI Automation 偵測 composer / 附件 chip
```

Windows Graphics Capture 官方文件說明可用於擷取應用程式視窗與顯示器；SendInput 是 Microsoft 提供的 Win32 API，可將鍵盤或滑鼠事件插入輸入串流，適合用於模擬 `Ctrl+V` 這類明確的使用者等價動作。([Microsoft Learn][7])

---

## 10.3 macOS Adapter

macOS 端建議使用：

```text
Rust 主流程
Swift Helper 處理 macOS 原生能力
```

macOS 端需要：

```text
截圖：
ScreenCaptureKit
CGWindowList fallback

剪貼簿：
NSPasteboard 寫入 PNG
NSPasteboard 寫入 file URL

找 Codex：
NSWorkspace runningApplications
Bundle Identifier / App Name matching

聚焦：
NSRunningApplication.activate

輸入：
CGEvent Cmd+V

輔助：
AXUIElement / Accessibility API
```

Apple 官方的 ScreenCaptureKit 用於高效能螢幕擷取；NSPasteboard 是 macOS App 與系統 pasteboard 互動的主要介面。([Apple Developer][8])

---

## 十一、檔案儲存設計

## 11.1 預設儲存位置

Windows：

```text
%USERPROFILE%\Pictures\CodexQuickShot
```

macOS：

```text
~/Pictures/CodexQuickShot
```

---

## 11.2 建議資料夾結構

```text
CodexQuickShot/
  2026/
    07/
      03/
        final/
          codex-quickshot_20260703_153000.png
        metadata/
          codex-quickshot_20260703_153000.json
        raw/
          codex-quickshot_20260703_153000_raw.png
```

### 預設策略

```text
final：預設保留
metadata：預設保留
raw：預設不永久保留，或只保留短時間
```

原因：raw image 可能含有未遮蔽的敏感資訊。

---

## 11.3 檔名格式

```text
codex-quickshot_yyyyMMdd_HHmmss.png
```

範例：

```text
codex-quickshot_20260703_153000.png
```

---

## 十二、Metadata 設計

每張圖片產生一份 metadata：

```json
{
  "id": "20260703_153000",
  "platform": "windows",
  "createdAt": "2026-07-03T15:30:00+08:00",
  "captureMode": "region",
  "selection": {
    "x": 320,
    "y": 180,
    "width": 1280,
    "height": 720
  },
  "output": {
    "finalImagePath": "C:\\Users\\User\\Pictures\\CodexQuickShot\\2026\\07\\03\\final\\codex-quickshot_20260703_153000.png",
    "rawImagePath": null,
    "metadataPath": "C:\\Users\\User\\Pictures\\CodexQuickShot\\2026\\07\\03\\metadata\\codex-quickshot_20260703_153000.json"
  },
  "annotations": [
    {
      "type": "rectangle",
      "x": 120,
      "y": 80,
      "width": 400,
      "height": 160,
      "strokeColor": "#ff0000",
      "strokeWidth": 4
    },
    {
      "type": "arrow",
      "from": { "x": 600, "y": 400 },
      "to": { "x": 430, "y": 220 },
      "strokeColor": "#ff0000",
      "strokeWidth": 4
    },
    {
      "type": "text",
      "x": 140,
      "y": 60,
      "text": "這裡是錯誤訊息",
      "fontSize": 24,
      "color": "#ff0000"
    }
  ],
  "codex": {
    "autoAttach": true,
    "attachedToCodex": true,
    "attachMethod": "clipboard",
    "sentAutomatically": false
  }
}
```

重點：

```text
sentAutomatically 永遠必須是 false。
```

---

## 十三、設定檔設計

建議使用 TOML：

```toml
[app]
start_at_login = true
show_tray_icon = true
show_main_window_on_launch = false
hide_taskbar_icon = true
default_tray_left_click = "capture_region_and_attach"

[capture]
default_mode = "region"
show_preview_before_attach = false
include_cursor = false
overlay_opacity = 0.35
freeze_screen_before_selection = true

[annotation]
enabled = true
default_color = "#ff0000"
default_stroke_width = 4
default_font_size = 24
tools = ["pen", "line", "arrow", "rectangle", "text", "mosaic"]
enable_undo_redo = true

[storage]
save_directory = ""
group_by_date = true
file_name_template = "codex-quickshot_{yyyyMMdd}_{HHmmss}.png"
write_metadata_json = true
keep_raw_image = false
auto_delete = false
keep_days = 30

[codex]
auto_attach = true
attach_mode = "clipboard_then_dragdrop"
focus_codex_after_capture = true
never_send_automatically = true
fallback_to_manual_paste = true
codex_app_name = "Codex"

[hotkeys.windows]
capture_region_and_attach = "Ctrl+Shift+5"
capture_region_only = "Ctrl+Shift+8"

[hotkeys.macos]
capture_region_and_attach = "Command+Shift+5"
capture_region_only = "Command+Shift+8"

[privacy]
show_sensitive_content_warning_on_first_run = true
confirm_before_attach = false
clear_clipboard_after_attach = false
keep_local_only = true

[advanced]
enable_ui_automation_detection = true
enable_drag_drop_fallback = true
debug_logging = false
```

---

## 十四、安全與隱私原則

Codex QuickShot 必須把安全設計當成產品特色。

### 14.1 永遠不做

```text
不自動送出 Codex 訊息
不讀 Codex token
不讀 Codex session database
不修改 Codex App
不注入 DLL
不 patch WebView / DOM
不呼叫未公開附件 API
不上傳截圖到第三方伺服器
不預設保存 raw image
```

### 14.2 必須提供

```text
第一次使用提醒敏感資料風險
完成後仍需使用者在 Codex 確認再送出
馬賽克 / Blur 工具
手動 fallback
清除最近截圖
清除剪貼簿
關閉 auto attach
設定儲存位置
```

### 14.3 第一次使用提醒文案

```text
Codex QuickShot 會截取你選取的畫面區域，並嘗試將圖片放入 Codex App 的附件區。
它不會自動送出訊息，也不會上傳截圖到第三方。
請在送出前確認圖片中沒有敏感資訊，例如姓名、Email、API Key、Token、內部系統資料。
```

---

## 十五、Codex Plugin / MCP 延伸設計

Desktop App 是主體；Plugin / MCP 是延伸。

### 15.1 Plugin 目的

```text
讓 Codex 知道 QuickShot 的截圖位置。
讓 Codex 可以查詢最近截圖。
讓 Codex 可以讀取 metadata。
讓 Codex 可以理解標註內容。
```

### 15.2 MCP Tools

第一版只做 read-only：

```text
quickshot_get_latest
quickshot_list_recent
quickshot_get_image_info
quickshot_open_folder
```

第二版再考慮：

```text
quickshot_capture_region
quickshot_capture_fullscreen
quickshot_capture_active_window
```

但即使 MCP 可以觸發截圖，也不應該自動送出 Codex 訊息。

---

## 十六、Repository 結構

```text
codex-quickshot/
  README.md
  LICENSE
  SECURITY.md
  CONTRIBUTING.md
  CODE_OF_CONDUCT.md

  docs/
    project-plan.md
    architecture.md
    tray-app-design.md
    annotation-editor.md
    attach-strategy.md
    security-model.md
    windows-adapter.md
    macos-adapter.md
    plugin-and-mcp.md
    roadmap.md
    troubleshooting.md

  apps/
    desktop/
      package.json
      src/
        components/
          TraySettings/
          CaptureOverlay/
          AnnotationToolbar/
          SettingsPage/
          RecentScreenshots/
        stores/
        styles/
      src-tauri/
        Cargo.toml
        tauri.conf.json
        src/
          main.rs
          tray.rs
          hotkeys.rs
          commands.rs
          setup.rs

  crates/
    quickshot-core/
      src/
        config.rs
        workflow.rs
        storage.rs
        metadata.rs
        annotation.rs
        image_composer.rs
        history.rs
        error.rs

    quickshot-platform/
      src/
        adapter.rs
        types.rs

    quickshot-windows/
      src/
        capture.rs
        clipboard.rs
        window_finder.rs
        input.rs
        ui_automation.rs
        attach_bridge.rs

    quickshot-macos/
      src/
        capture.rs
        pasteboard.rs
        app_activation.rs
        accessibility.rs
        attach_bridge.rs

    quickshot-mcp/
      src/
        main.rs
        tools/
          get_latest.rs
          list_recent.rs
          get_image_info.rs
          open_folder.rs

  native/
    macos-helper/
      Package.swift
      Sources/
        QuickShotHelper/
          ScreenCaptureService.swift
          PasteboardService.swift
          AccessibilityService.swift
          AppActivationService.swift

  plugins/
    codex-quickshot/
      .codex-plugin/
        plugin.json
      .mcp.json
      skills/
        quickshot/
          SKILL.md
      assets/
        icon.png

  tests/
    manual/
      windows.md
      macos.md
    fixtures/

  .github/
    workflows/
      ci.yml
      release.yml
    ISSUE_TEMPLATE/
    PULL_REQUEST_TEMPLATE.md
```

---

## 十七、開發里程碑

## Phase 0：技術可行性驗證

目標：證明核心流程可行。

```text
1. 建立 Tauri 2 tray app。
2. 啟動後不顯示主視窗。
3. 註冊全域快捷鍵。
4. 按快捷鍵後顯示 overlay。
5. 框選區域。
6. 儲存 PNG。
7. 複製到剪貼簿。
8. 聚焦 Codex App。
9. Ctrl+V / Cmd+V 貼入 composer。
10. 確認不會送出。
```

---

## Phase 1：Windows MVP

```text
1. Tray 常駐。
2. 快捷鍵。
3. Frozen screen overlay。
4. 區域框選。
5. 基本標註工具：
   - 紅框
   - 箭頭
   - 直線
   - 文字
   - Undo / Redo
6. 輸出 final PNG。
7. 儲存到指定資料夾。
8. Metadata JSON。
9. Clipboard attach。
10. Manual fallback。
11. 不自動送出。
```

版本：

```text
v0.1.0-alpha
```

---

## Phase 2：macOS MVP

```text
1. Menu Bar 常駐。
2. 快捷鍵。
3. Screen Recording 權限檢查。
4. Accessibility 權限檢查。
5. Frozen screen overlay。
6. 區域框選。
7. 基本標註工具。
8. NSPasteboard 寫入圖片。
9. 聚焦 Codex App。
10. Cmd+V attach。
11. Manual fallback。
12. 不自動送出。
```

版本：

```text
v0.2.0-alpha
```

---

## Phase 3：標註工具增強

```text
1. 自由手繪筆。
2. 高亮筆。
3. 顏色選擇。
4. 線條粗細。
5. 文字大小。
6. 馬賽克。
7. Blur。
8. 編號標記。
9. 可移動標註物件。
10. 可重新編輯最近截圖。
```

版本：

```text
v0.3.0-beta
```

---

## Phase 4：Codex Plugin + MCP

```text
1. quickshot_get_latest。
2. quickshot_list_recent。
3. quickshot_get_image_info。
4. quickshot_open_folder。
5. Codex Plugin manifest。
6. Skill 文件。
7. MCP 安裝說明。
```

版本：

```text
v0.4.0-beta
```

---

## Phase 5：正式開源版

```text
1. Windows installer。
2. macOS DMG。
3. GitHub Releases。
4. SECURITY.md 完整化。
5. CONTRIBUTING.md。
6. 自動測試。
7. 手動測試清單。
8. 簽章策略。
9. Roadmap 公開。
```

版本：

```text
v1.0.0
```

---

## 十八、驗收標準

### 18.1 產品驗收

```text
1. App 啟動後不顯示主視窗。
2. Windows 只出現在右下角 tray。
3. macOS 只出現在 menu bar。
4. 點擊圖示可開始截圖。
5. 快捷鍵可開始截圖。
6. 可框選螢幕區域。
7. 可在截圖上畫紅框、箭頭、線條、文字。
8. 可 Undo / Redo。
9. 完成後輸出 final PNG。
10. 圖片儲存到指定位置。
11. 可自動嘗試放入 Codex composer。
12. 絕對不自動送出。
13. 自動附加失敗時有 fallback。
```

### 18.2 安全驗收

```text
1. 不讀取 Codex token。
2. 不修改 Codex App。
3. 不寫入 Codex session。
4. 不注入任何 Codex process。
5. 不使用未公開 API。
6. 不自動按 Enter。
7. 不自動按 Send。
8. 不上傳圖片到第三方。
9. sentAutomatically 永遠是 false。
10. 使用者可以清除截圖歷史。
```

---

## 十九、開源策略

### License

建議：

```text
MIT License
```

原因：

```text
工具型專案友善
公司與個人都容易採用
開源貢獻門檻低
```

### README 第一段建議

```markdown
# Codex QuickShot

Codex QuickShot is an open-source desktop companion app for Codex App.

It works like a LightShot-style tray screenshot tool: press a hotkey, select a region, annotate it with arrows, boxes, lines, or text, save it locally, and attach it to the current Codex composer without sending automatically.
```

### GitHub Labels

```text
type: bug
type: feature
type: docs
type: security
platform: windows
platform: macos
area: tray
area: capture
area: annotation
area: codex-attach
area: clipboard
area: mcp
good first issue
help wanted
```

---

## 二十、給 Codex App 的實作 Prompt

這段可以直接交給 Codex App 建立專案：

```text
我要建立一個開源專案，名稱為 Codex QuickShot。

技術架構：
Rust + Tauri 2 + Native Platform Adapter。

產品型態：
這不是一般會佔據桌面的主視窗 App。
它要像 LightShot 一樣，常駐在系統列或 Menu Bar。

Windows：
啟動後只出現在右下角 System Tray，不顯示主視窗，不佔用工作列。
使用者可以點擊 Tray icon 開始截圖，也可以右鍵開啟選單。

macOS：
啟動後只出現在 Menu Bar。
使用者可以點擊 Menu Bar icon 開始截圖，也可以開啟選單與設定。

產品目標：
使用者按快捷鍵或點擊 Tray/Menu Bar icon 後，可以框選螢幕區域，並在截圖上進行 LightShot-style 標註，例如：
- 畫紅框
- 畫箭頭
- 畫直線
- 自由手繪
- 輸入文字
- Undo / Redo

完成標註後，App 要將標註合成到圖片，儲存為 final PNG 到使用者指定資料夾，並嘗試把該圖片放入目前 Codex App 的聊天 composer 附件區，但絕對不要自動送出訊息。

核心流程：
1. App 啟動後常駐 Tray / Menu Bar，不開主視窗。
2. 使用者按快捷鍵或點擊 Tray/Menu Bar icon。
3. App 擷取目前螢幕作為 frozen background。
4. 畫面變暗，進入區域框選模式。
5. 使用者框選截圖區域。
6. 框選完成後進入 annotation mode。
7. 使用者可使用 Rectangle、Arrow、Line、Pen、Text、Undo、Redo、Finish、Cancel。
8. 使用者按 Finish 後：
   - 從 frozen background crop 出選取區域。
   - 將 annotations 合成到 crop image。
   - 輸出 final PNG。
   - 儲存到指定資料夾。
   - 寫入 metadata JSON。
   - 複製 final PNG 到 clipboard / pasteboard。
   - 嘗試聚焦 Codex App。
   - 嘗試 Ctrl+V / Cmd+V 貼入 Codex composer。
   - 絕對不要自動按 Enter。
   - 絕對不要自動按 Send。
9. 若 Codex App 找不到或貼入失敗，進入 manual fallback，提示使用者手動貼上。

重要限制：
1. 不修改 Codex App。
2. 不讀寫 Codex App 私有資料。
3. 不讀取 Codex token。
4. 不注入 DLL。
5. 不 patch WebView / DOM。
6. 不使用未公開 API。
7. 不自動送出。
8. 不上傳截圖到第三方。
9. 所有截圖預設只保存在本機。
10. sentAutomatically metadata 永遠必須是 false。

請建立 monorepo 結構：
- apps/desktop：Tauri 2 desktop app
- crates/quickshot-core：核心設定、儲存、metadata、annotation model、image composer、workflow
- crates/quickshot-platform：平台 adapter trait 與共用型別
- crates/quickshot-windows：Windows adapter
- crates/quickshot-macos：macOS adapter
- crates/quickshot-mcp：MCP server
- plugins/codex-quickshot：Codex Plugin
- docs：文件

MVP 先完成：
1. Tauri 2 app 初始化。
2. 啟動後不顯示主視窗。
3. Windows tray / macOS menu bar。
4. Tray menu。
5. Global shortcut。
6. 設定檔讀寫。
7. Frozen screen overlay。
8. 區域框選。
9. Annotation mode。
10. Rectangle tool。
11. Arrow tool。
12. Line tool。
13. Text tool。
14. Undo / Redo。
15. Export final PNG。
16. 儲存到指定資料夾。
17. 寫入 metadata JSON。
18. Clipboard / Pasteboard 寫入 final PNG。
19. Codex App window finder。
20. Focus Codex App。
21. Ctrl+V / Cmd+V attach。
22. Manual fallback。
23. 不自動送出。
24. README.md。
25. SECURITY.md。
26. docs/architecture.md。
27. docs/tray-app-design.md。
28. docs/annotation-editor.md。
29. docs/attach-strategy.md。
30. docs/windows-adapter.md。
31. docs/macos-adapter.md。

請特別注意：
- App 不應該佔據主視窗。
- 只有設定頁需要時才打開。
- 截圖 overlay 是暫時視窗，完成或取消後立即關閉。
- 不要截到 QuickShot 自己的工具列。
- 多螢幕。
- DPI / Retina。
- 標註座標與輸出圖片座標一致。
- 不自動送出。
- 可關閉 auto attach。
- fallback 機制。
- raw image 敏感資料風險。
- 開源可維護性。
```

---

## 二十一、最終版本定位

這份企劃整合後，Codex QuickShot 的最終定位應該是：

```text
一個像 LightShot 一樣常駐系統列的跨平台截圖標註工具。
它不佔據桌面視窗，使用者可以用快捷鍵或 Tray/Menu Bar icon 啟用。
框選截圖後，可以直接畫紅線、紅框、箭頭、文字與遮蔽資訊。
完成後會儲存圖片，並嘗試把圖片放入 Codex App 目前聊天附件區。
但它永遠不自動送出，讓使用者保有最後確認權。
```

最重要的產品紅線：

```text
不佔主視窗。
不修改 Codex App。
不使用私有 API。
不自動送出。
不預設上傳圖片。
讓使用者先標註、先遮蔽、先確認，再送出。
```

[1]: https://app.prntscr.com/?utm_source=chatgpt.com "Lightshot — screenshot tool for Mac & Win"
[2]: https://developers.openai.com/codex/app/features?utm_source=chatgpt.com "Codex App Features"
[3]: https://developers.openai.com/codex/app/commands "Commands – Codex app | OpenAI Developers"
[4]: https://v2.tauri.app/learn/system-tray/?utm_source=chatgpt.com "System Tray"
[5]: https://developers.openai.com/codex/plugins?utm_source=chatgpt.com "Plugins – Codex"
[6]: https://v2.tauri.app/blog/tauri-20/?utm_source=chatgpt.com "Tauri 2.0 Stable Release"
[7]: https://learn.microsoft.com/en-us/uwp/api/windows.graphics.capture?view=winrt-28000&utm_source=chatgpt.com "Windows.Graphics.Capture Namespace - Windows apps"
[8]: https://developer.apple.com/documentation/screencapturekit/?utm_source=chatgpt.com "ScreenCaptureKit | Apple Developer Documentation"
