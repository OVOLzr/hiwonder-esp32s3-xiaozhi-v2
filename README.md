# 幻尔 ESP32-S3 小智 v2 (Hiwonder Exploit S3) 板级适配

本项目基于小智官方固件（[78/xiaozhi-esp32](https://github.com/78/xiaozhi-esp32)），为幻尔 ESP32-S3 开发板提供了完整的 v2 板级适配支持。本仓库包含了板级配置代码、驱动文件。

### 特性支持
- **屏幕显示**：ST7789 (320x240) 驱动适配，支持彩色界面。
- **音频回放**：ES8311 Codec + XL9555 功放延迟开启，完美消除上电爆音。
- **语音唤醒**：支持 AFE + 离线“你好小智”唤醒。
- **摄像头**：DVP 接口接入，复用音频 I2C 总线，支持 MCP 拍照。
- **个性定制**：支持替换自定义 `assets.bin`，完美实现彩色动态表情包和自定义主题。

### 📦 如何使用
1. 将本仓库的 `board/hiwonder-exploit-s3` 目录复制到官方源码的 `main/boards/` 下。
2. 根据 `docs/` 目录中的说明，修改 `main/Kconfig.projbuild` 和 `main/CMakeLists.txt`。
3. 将你的 `my_assets.bin` 放到 `main/custom_assets/` 目录下。
4. 使用 `idf.py menuconfig` 选择 `Hiwonder Exploit S3` 并开启自定义 Assets，编译烧录。

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
