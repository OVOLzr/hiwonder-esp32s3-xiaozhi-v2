## 幻尔 ESP32-S3 小智 v2 (Hiwonder Exploit S3) 板级适配

本项目基于小智官方固件（[78/xiaozhi-esp32](https://github.com/78/xiaozhi-esp32)），为幻尔 ESP32-S3 开发板提供了完整的 v2 板级适配支持。本仓库包含了板级配置代码、驱动文件。

### 特性支持
- **屏幕显示**：ST7789 (320x240) 驱动适配，支持彩色动图界面。
- **音频回放**：ES8311 Codec + XL9555 功放延迟开启。
- **语音唤醒**：支持 AFE + 离线“你好小智”唤醒。
- **摄像头**：DVP 接口接入，复用音频 I2C 总线，支持 MCP 拍照。
- **个性定制**：支持替换自定义 `assets.bin`，完美实现彩色动态表情包和自定义主题。
---
###  如何使用

**1. 获取官方源码**
首先，确保你安装了 `git`，在终端里执行以下命令拉取小智官方的最新源码：
```bash
git clone https://github.com/78/xiaozhi-esp32.git
```

**2. 编译与烧录**
- 在终端执行 `idf.py menuconfig`。
- 在 `Xiaozhi Assistant -> Board Type` 中选择 `Hiwonder Exploit S3`。
- 开启唤醒词、配置分区表（推荐使用 `partitions/v2/8m.csv`，根据你的 Flash 大小选择）。
- 执行 `idf.py build flash monitor` 编译并烧录。

**3. 如何制作并替换自定义主题与表情包 (assets.bin)**
如果你想拥有彩色的动态表情和自定义字体：
- **生成资源包**：访问官方网页生成器 [xiaozhi.me/tools/assets-generator](https://xiaozhi.me/tools/assets-generator/)，选择 `ESP32-S3`、`320x240` 分辨率，挑选你喜欢的字体和 Emoji，生成并下载 `assets.bin` 文件。
- **放入源码**：将下载的 `assets.bin` 重命名为 `my_assets.bin`，放入源码的 `main/custom_assets/` 目录下。
- **开启配置**：在 `menuconfig` 中进入 `Xiaozhi Assistant -> Flash Assets`，选择 `Flash Custom Assets`，并在下方填入 `main/custom_assets/my_assets.bin` 的路径。
- **⚠️ 避坑指南（必看）**：官方 LVGL 解码器较老，如果你使用自定义 GIF，容易出现**画面重影**。请使用 [ezgif.com/optimize](https://ezgif.com/optimize) 将你的 GIF 处理一下（选择 `Coalesce (unoptimize)` 选项），将增量帧转为全帧，再拿去生成 `assets.bin`，即可完美解决！

### 源码放置位置示例 (Directory Structure)

请将本仓库的 `board` 目录复制到官方源码中，并参考以下结构：

```text
xiaozhi-esp32/ (官方源码根目录)
├── main/
│   ├── boards/
│   │   ├── hiwonder-exploit-s3/   <-- 将本仓库的 board 文件夹整个放在这里
│   │   │   ├── config.h
│   │   │   ├── config.json
│   │   │   ├── hiwonder_exploit_s3_board.cc
│   │   │   └── xl9555.c / xl9555.h
│   │   ├── Kconfig.projbuild      <-- 需修改，添加板级选项 (BOARD_TYPE_HIWONDER_EXPLOIT_S3)
│   │   └── CMakeLists.txt         <-- 需修改，添加编译路径 (BOARD_DIR "hiwonder-exploit-s3")
│   └── custom_assets/
│       └── my_assets.bin          <-- 将你生成的主题/表情包 assets.bin 放这里
```
