# 幻尔 ESP32-S3 小智 v2 (Hiwonder Exploit S3) 板级适配

本项目基于小智官方固件（[78/xiaozhi-esp32](https://github.com/78/xiaozhi-esp32)），为幻尔 ESP32-S3 开发板提供了完整的 v2 板级适配支持。本仓库包含了板级配置代码、驱动文件及踩坑记录。

> ⚠️ **环境与硬件要求（非常重要）**
> - **ESP-IDF 版本**：必须使用 **v6.0.1** 或更新版本（本适配使用了新版 I2C 和 LCD 驱动）。
> - **硬件要求**：幻尔 ESP32-S3 开发板（带 ST7789 屏幕、ES8311 音频、DVP 摄像头）。
> - **Flash 容量**：必须为 **8MB** 或 **16MB**（因 v2 固件和唤醒模型占用空间较大，4MB 无法正常运行）。

### ✨ 特性支持
- **屏幕显示**：ST7789 (320x240) 驱动适配，支持彩色动图界面。
- **音频回放**：ES8311 Codec + XL9555 功放延迟开启（消除上电爆音）。
- **语音唤醒**：支持 AFE + 离线“你好小智”唤醒。
- **摄像头**：DVP 接口接入，复用音频 I2C 总线，支持 MCP 拍照。
- **个性定制**：支持替换自定义 `assets.bin`，完美实现彩色动态表情包和自定义主题。

---

### 📦 快速上手

**1. 获取官方源码**
```bash
git clone https://github.com/78/xiaozhi-esp32.git
```
2. 部署板级代码

将本仓库的 hiwonder-exploit-s3 目录整个复制到官方源码的 main/boards/ 目录下。

修改注册文件（具体参考下方代码）：

修改 main/Kconfig.projbuild，在 choice BOARD_TYPE 中添加选项。

修改 main/CMakeLists.txt，添加板级编译路径。

3. menuconfig 核心配置（避坑重点）
在终端执行 idf.py menuconfig，请务必核对以下配置：

板级选择：Xiaozhi Assistant -> Board Type 选择 Hiwonder Exploit S3。

Flash 与分区表（必看）：幻尔开发板通常为 8MB Flash，必须进入 Partition Table -> Custom partition CSV file，确保指向 partitions/v2/8m.csv（若为 16MB 则选 16m.csv）。(注：本仓库的 config.json 已默认追加了此配置，但建议在 menuconfig 中二次确认。)

唤醒词：

Wake Word Implementation Type 选择 Wakenet model with AFE。
进入 ESP Speech Recognition -> Load Multiple Wake Words (WakeNet9s)，勾选 wn9s_nihaoxiaozhi。
按 S 保存，Q 退出。

4. 编译与烧录

bash
idf.py build flash monitor
🎨 如何制作并替换自定义主题与表情包 (assets.bin)
如果你想拥有彩色的动态表情和自定义字体：

生成资源包：访问官方网页生成器 xiaozhi.me/tools/assets-generator，选择 ESP32-S3、320x240，挑选字体和 Twemoji，生成并下载 assets.bin。

放入源码：将下载的文件重命名为 my_assets.bin，放入源码的 main/custom_assets/ 目录下。

开启配置：在 menuconfig 中进入 Xiaozhi Assistant -> Flash Assets，选择 Flash Custom Assets，并在下方填入 main/custom_assets/my_assets.bin 的路径。

🚨 避坑指南：解决自定义 GIF 画面重影
官方底层 LVGL 解码器较老，不支持处理“增量帧”编码的 GIF（会出现重影）。请使用 ezgif.com/optimize 上传你的 GIF，在 Optimization method 中选择 Coalesce (unoptimize)，将增量帧转为全帧，再拿去生成 assets.bin 即可完美解决！

实现效果:
<img width="300" height="184" alt="实现效果" src="https://github.com/user-attachments/assets/5b37adca-0c33-4252-b933-8dda085a7446" />


📂 目录结构与代码修改参考
最终的官方源码目录结构应如下：

```text
xiaozhi-esp32/ (官方源码根目录)
├── main/
│   ├── boards/
│   │   ├── hiwonder-exploit-s3/   <-- 将本仓库的文件夹整个放在这里
│   │   │   ├── config.h
│   │   │   ├── config.json
│   │   │   ├── hiwonder_exploit_s3_board.cc
│   │   │   └── xl9555.c / xl9555.h
│   │   ├── Kconfig.projbuild      <-- 需修改
│   │   └── CMakeLists.txt         <-- 需修改
│   └── custom_assets/
│       └── my_assets.bin          <-- 将你生成的主题/表情包放这里
```
修改 1：main/Kconfig.projbuild
在 choice BOARD_TYPE 块中添加：

kconfig
config BOARD_TYPE_HIWONDER_EXPLOIT_S3
    bool "Hiwonder Exploit S3"
    depends on IDF_TARGET_ESP32S3
修改 2：main/CMakeLists.txt
在板级分支链中添加：

cmake
elseif(CONFIG_BOARD_TYPE_HIWONDER_EXPLOIT_S3)
    set(BOARD_DIR "hiwonder-exploit-s3")
