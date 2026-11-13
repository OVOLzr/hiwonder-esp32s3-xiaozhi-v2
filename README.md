# 幻尔 ESP32-S3 小智 v2 (Hiwonder Exploit S3) 板级适配

本项目基于小智官方固件（[78/xiaozhi-esp32](https://github.com/78/xiaozhi-esp32)），为幻尔 ESP32-S3 开发板提供了完整的 v2 板级适配支持。

### 特性支持
- **屏幕显示**：ST7789 (320x240) 驱动适配，支持彩色动图。
- **音频回放**：ES8311 Codec + XL9555 功放延迟开启。
- **语音唤醒**：支持 AFE + 离线“你好小智”唤醒。
- **摄像头**：DVP 接口接入，复用音频 I2C 总线，支持 MCP 拍照。
- **个性定制**：支持替换自定义 `assets.bin`，实现动态表情和自定义主题。

---

### 运行效果
<img width="300" height="184" alt="ezgif com-resize" src="https://github.com/user-attachments/assets/3208c747-a5a3-4f41-b922-3346c01414b0" />

---

### 快速上手

1. **部署代码**：将本仓库的 `hiwonder-exploit-s3` 目录复制到官方源码的 `main/boards/` 下，并参考文末修改 `Kconfig.projbuild` 和 `CMakeLists.txt`。
2. **选择板型**：使用 `idf.py menuconfig`，在 `Xiaozhi Assistant -> Board Type` 中选择 `Hiwonder Exploit S3`，然后正常编译烧录即可。

> ⚠️ **核心避坑配置（必须核对）**
> - **环境版本**：必须使用 **ESP-IDF v6.0.1** 或更新版本（本适配使用了新版 I2C 和 LCD 驱动）。
> - **分区表**：幻尔板子通常为 8MB Flash，必须在 `Partition Table -> Custom partition CSV file` 指向 **`partitions/v2/8m.csv`**（若为 16MB 则选 `16m.csv`）。*(本仓库的 `config.json` 已默认追加，建议二次确认)*。
> - **唤醒词**：`Wake Word Implementation Type` 选择 `Wakenet model with AFE`，并在 `ESP Speech Recognition` 中勾选 `wn9s_nihaoxiaozhi`。

---

### 制作自定义主题与表情包 (assets.bin)

1. **生成资源包**：访问官方生成器 [xiaozhi.me/tools/assets-generator](https://xiaozhi.me/tools/assets-generator/)，选择 `ESP32-S3`、`320x240`，生成并下载 `assets.bin`。
2. **放入源码**：重命名为 `my_assets.bin`，放入源码的 `main/custom_assets/` 目录下。
3. **开启配置**：在 `menuconfig` 中进入 `Xiaozhi Assistant -> Flash Assets`，选择 `Flash Custom Assets`，填入路径 `main/custom_assets/my_assets.bin`。

> **避坑：解决自定义 GIF 画面重影**
> 官方底层 LVGL 解码器较老，不支持处理“增量帧”编码的 GIF。请使用 [ezgif.com/optimize](https://ezgif.com/optimize) 上传你的 GIF，在 `Optimization method` 中选择 **`Coalesce (unoptimize)`** 转为全帧，再去生成 `assets.bin` 即可。

---

### 目录结构与代码修改参考

最终源码目录结构应如下：

```text
xiaozhi-esp32/
├── main/
│   ├── boards/
│   │   ├── hiwonder-exploit-s3/   <-- 将本仓库文件夹放在这里
│   │   │   ├── config.h
│   │   │   ├── config.json
│   │   │   ├── hiwonder_exploit_s3_board.cc
│   │   │   └── xl9555.c / xl9555.h
│   │   ├── Kconfig.projbuild
│   │   └── CMakeLists.txt
│   └── custom_assets/
│       └── my_assets.bin          <-- 将你的表情包放这里
```
修改 1：main/Kconfig.projbuild
在 choice BOARD_TYPE 块中添加：

```kconfig
config BOARD_TYPE_HIWONDER_EXPLOIT_S3
    bool "Hiwonder Exploit S3"
    depends on IDF_TARGET_ESP32S3
```
修改 2：main/CMakeLists.txt
在板级分支链中添加：

```cmake
elseif(CONFIG_BOARD_TYPE_HIWONDER_EXPLOIT_S3)
    set(BOARD_DIR "hiwonder-exploit-s3")
```
