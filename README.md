# STM32CubeMX + STM32F103 + VS Code / Embedded IDE 开发模板

这个项目源于我对 STM32CubeIDE 编辑体验的不习惯：我希望继续用 STM32CubeMX 配置引脚、时钟和外设，同时在 VS Code 中编写代码。因此，我将 CubeMX 生成的工程接入 VS Code 的 **Embedded IDE（EIDE）** 插件，使用 Arm GCC 编译，通过 ST-Link 烧录。

本仓库既是一个 STM32F103C8T6 的基础模板，也是一次迁移过程的操作记录。下面以 Windows 为例，说明如何使用模板，以及如何将自己的 CubeMX 工程接入 EIDE。

## 1. 当前模板配置

| 项目 | 配置 |
| --- | --- |
| MCU | STM32F103C8T6，Cortex-M3，无硬件 FPU |
| 内存 | Flash 64 KB，RAM 20 KB |
| 时钟 | 外部高速晶振 HSE 8 MHz，经 PLL 配置为 72 MHz |
| CubeMX 工程 | [Model.ioc](Model.ioc)，由 STM32CubeMX 6.15.0 生成 |
| 固件库 | STM32Cube FW_F1 V1.8.7 |
| EIDE | 工程配置对应 Embedded IDE 3.27.2 |
| 编译器 | Arm GNU Toolchain，`arm-none-eabi-gcc` |
| 烧录配置 | ST-Link，SWD，起始地址 `0x08000000` |
| GCC 启动文件 | [Startup/startup_stm32f103c8tx.s](Startup/startup_stm32f103c8tx.s) |
| 链接脚本 | [STM32F103XX_FLASH.ld](STM32F103XX_FLASH.ld) |

模板用于搭建开发环境，尚未预置 LED 闪烁等应用功能。使用其他开发板时，需要根据实际晶振、电源和外设连接修改 `.ioc`。迁移到其他 MCU 时，还必须重新核对 CPU、器件宏、启动文件和内存布局。

## 2. 安装开发工具

| 工具 | 用途 |
| --- | --- |
| [VS Code](https://code.visualstudio.com/) | 编辑器 |
| [Embedded IDE / cl.eide](https://marketplace.visualstudio.com/items?itemName=CL.eide) | 工程管理、编译和烧录入口 |
| [Microsoft C/C++ / ms-vscode.cpptools](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cpptools) | 补全、跳转和编辑时错误诊断 |
| [STM32CubeMX](https://www.st.com/en/development-tools/stm32cubemx.html) | 配置硬件并生成初始化代码 |
| Arm GNU Toolchain | 编译 ARM 固件；可通过 EIDE 工具管理安装，或指定已有安装目录 |
| [STM32CubeProgrammer](https://www.st.com/en/development-tools/stm32cubeprog.html) | 使用 ST-Link 烧录所需的工具及驱动 |
| Git | 克隆和版本管理，也可以直接下载仓库 ZIP |

若还需要使用仓库保留的 CMake 构建方式，再安装 CMake（至少 3.22）和 Ninja，并将它们及 Arm GCC 的 `bin` 目录加入终端的 `PATH`。只通过 EIDE 构建时，不必先配置 CMake。

在 EIDE 的工具管理界面或 VS Code 设置中配置本机路径：

- `EIDE.ARM.GCC.InstallDirectory`：Arm GCC 安装根目录，目录下应存在 `bin/arm-none-eabi-gcc.exe`。
- `EIDE.STLink.ExePath`：本机 `STM32_Programmer_CLI.exe` 的完整路径。

这两个路径因电脑而异，建议放在用户设置中，不要把他人的安装路径直接复制进工程。

## 3. 直接使用本模板

1. 获取代码：

   ```powershell
   git clone https://github.com/fangzhang299/Initial-STM32CubeMX-STM32F103-Vscode-template.git
   cd Initial-STM32CubeMX-STM32F103-Vscode-template
   ```

2. 在 VS Code 中选择“文件 → 从文件打开工作区”，打开根目录的 `Model.code-workspace`。如果已经安装 `code` 命令，也可以执行：

   ```powershell
   code Model.code-workspace
   ```

3. 打开左侧 EIDE 面板，确认加载了 `Model : Debug`，配置本机的 GCC 和烧录工具路径。
4. 检查构建配置：工具链为 **GCC**，CPU 为 **Cortex-M3**，使用自定义链接脚本 `STM32F103XX_FLASH.ld`。
5. 点击 EIDE 的“构建”。本模板的 EIDE 配置会生成 `build/Debug/Model.elf` 和 `build/Debug/Model.hex`。
6. 按第 6 节连接硬件并执行烧录。

如果希望以此建立独立的新仓库，可以先下载 ZIP 并解压；克隆得到的目录会保留本仓库的 Git 历史和远程地址。

## 4. 将自己的 CubeMX 工程迁移到 EIDE

这里的“迁移”是为生成的源码配置一个 EIDE 工程。EIDE 的编译配置保存在 `.eide/eide.yml` 中，本教程不依赖 EIDE 自动解析 CMake 配置。

### 4.1 用 CubeMX 生成工程

1. 打开 STM32CubeMX，选择实际 MCU，配置引脚、时钟和所需外设。
2. STM32F103 使用 SWD 时，在 `System Core → SYS` 中选择 `Serial Wire`。
3. 在 `Project Manager` 中填写项目名和保存目录。
4. 本仓库采用 `Toolchain / IDE = CMake`、编译器 `GCC`。在 `Code Generator` 中选择将需要的库文件复制到项目，避免项目依赖另一台电脑上的固件库路径。
5. 开启保留用户代码的选项；可以按需要选择为每个外设生成独立的 `.c/.h` 文件。
6. 点击 `GENERATE CODE`，确认生成目录中包含 `Core`、`Drivers`、GCC 启动文件及 `.ld` 链接脚本。

ST 的 [CubeMX 手册](https://dev.st.com/stm32cube-docs/stm32cubemx/6.19.0/en/docs/markup/CubeMX_UserManual/chapters/04_4_stm32cubemx_user_interface.html)说明了 CMake 与 GCC 的生成选项。EIDE 官方也提供了使用 Makefile 生成格式的 [CubeMX 配置教程](https://em-ide.com/docs/getting-started/new_prj/)；本文使用仓库自身的 CMake 生成布局，随后手动配置 EIDE。

### 4.2 创建 EIDE 工程并放入生成文件

1. 在 EIDE 面板选择“新建项目 → 空项目”，类型选择 `Cortex-M`，填写工程名并选择保存位置。
2. 将 CubeMX 生成的工程文件放到这个 EIDE 工程根目录，保留 EIDE 创建的 `.eide` 和 `.code-workspace` 配置。也可以先创建 EIDE 工程，再让 CubeMX 生成到同一个根目录；注意不要无意中多嵌套一层同名文件夹。
3. 重新打开 EIDE 创建的 `.code-workspace`，在构建配置中选择 GCC 和实际 CPU。
4. 如需芯片支持包，在 EIDE 中为实际器件选择匹配的 DFP。本模板选择的是 `STM32F103C8`，支持包位于 `.pack/Keil/STM32F1xx_DFP.2.3.0`。

完成后应形成类似结构：

```text
工程根目录/
├── .eide/                       # EIDE 配置
├── .pack/                       # 本模板使用的芯片支持包
├── .vscode/                     # VS Code 任务
├── Core/Inc/                    # 应用头文件
├── Core/Src/                    # 应用和系统源文件
├── Drivers/CMSIS/               # CMSIS 头文件、芯片支持
├── Drivers/STM32F1xx_HAL_Driver/ # HAL 库
├── Startup/                     # 当前芯片的 GCC 启动文件
├── cmake/                       # 可选的 CMake 构建配置
├── Model.code-workspace
├── Model.ioc
├── CMakeLists.txt
├── CMakePresets.json
└── STM32F103XX_FLASH.ld
```

上面是本仓库整理后的布局。新工程中的项目名、芯片系列、启动文件和链接脚本名称可能不同，后续路径应以实际文件为准。

### 4.3 配置源文件范围

在 EIDE 的“项目资源”中添加普通源文件夹。STM32F1 本模板使用：

```yaml
srcDirs:
  - Core/Src
  - Drivers/STM32F1xx_HAL_Driver/Src
  - Startup
```

不要将整个 `Drivers` 添加为源文件目录，否则可能把 CMSIS 的示例、测试、其他内核代码和其他编译器的汇编一起编译。

在目标配置中排除下面三个 HAL 模板文件；目录前缀均为 `Drivers/STM32F1xx_HAL_Driver/Src/`：

```text
stm32f1xx_hal_msp_template.c
stm32f1xx_hal_timebase_rtc_alarm_template.c
stm32f1xx_hal_timebase_tim_template.c
```

应用已经有自己的 MSP 初始化与时基实现时，这些参考模板不应直接参与构建。若项目确实需要改用某个时基模板，应按相应 HAL 的方式有选择地配置。

本仓库已移除了当前模板不使用的 CMSIS 示例、测试、文档及 DSP/NN/RTOS 等组件。如果自己的应用使用这些组件，需要另行引入并配置。

### 4.4 配置头文件路径和宏定义

在 EIDE 的“C/C++ 属性”中添加包含目录。以本项目为例：

```text
Core/Inc
Drivers/STM32F1xx_HAL_Driver/Inc
Drivers/STM32F1xx_HAL_Driver/Inc/Legacy
Drivers/CMSIS/Device/ST/STM32F1xx/Include
Drivers/CMSIS/Include
```

这些是供 `#include` 搜索的路径，不需要把对应目录的全部源文件加入编译。

HAL 工程的关键预处理宏为：

```text
USE_HAL_DRIVER
STM32F103xB
```

在 EIDE 中填写宏时不加 `-D` 前缀。`STM32F103xB` 对应本例器件，换芯片时需按 CubeMX 生成配置调整。本模板还保留了 `STM32F10X_MD`，不要将这个旧式宏作为所有 STM32 HAL 工程的通用要求。

对于 CMake 格式的生成工程，可以对照 `cmake/stm32cubemx/CMakeLists.txt` 中的 `MX_Include_Dirs`、`MX_Defines_Syms` 和源文件列表逐项检查。新增中间件还可能需要库、额外编译选项和链接配置。

### 4.5 选择 GCC 启动文件和链接脚本

启动文件应同时匹配 MCU 和工具链。只编译一份当前器件的启动文件：

- 本仓库保留 `Startup/startup_stm32f103c8tx.s`，文件使用 `.syntax`、`.section` 等 GNU 汇编语法。
- 不要同时加入 Keil/ARMCC 版本的 `startup_stm32f10x_md.s`。其中的 `AREA`、`EQU`、`EXPORT` 等指令不能直接交给 GCC 汇编器。
- CubeMX 生成的 GCC 文件可能叫 `startup_stm32f103xb.s`。可以保留原名，也可以像本仓库一样整理进 `Startup`，但构建配置必须指向实际路径。

在 EIDE 的构建配置中启用自定义链接脚本，并设置：

```text
STM32F103XX_FLASH.ld
```

本例脚本中的 Flash 为 `0x08000000 / 64K`，RAM 为 `0x20000000 / 20K`。其他芯片或带 Bootloader 的应用要重新确定内存布局，不能照搬。

若同时保留 CMake 构建，移动启动文件后，还要同步修改 `cmake/stm32cubemx/CMakeLists.txt`。本仓库的路径是：

```cmake
${CMAKE_CURRENT_SOURCE_DIR}/../../Startup/startup_stm32f103c8tx.s
```

### 4.6 检查编译选项并构建

本模板采用 Cortex-M3、无硬件 FPU、C11、调试优化，并启用函数/数据独立段及链接时移除未使用段。链接选项包含数学库 `-lm`，使用 newlib-nano。

完成以上配置后，点击 EIDE 的“重新构建”。先解决第一条编译错误，再处理后续错误；成功时日志应显示 `build successfully` 并生成固件文件。

## 5. 开启代码补全和错误提示

编译报错和编辑时红色波浪线由不同组件提供。为本教程的 EIDE + Microsoft C/C++ 组合，在用户设置中配置：

```json
{
  "C_Cpp.errorSquiggles": "enabled",
  "C_Cpp.intelliSenseEngine": "default",
  "editor.renderValidationDecorations": "on"
}
```

把这些键合并进已有设置，不要直接覆盖整个设置文件。工作区中保留以下配置，让 EIDE 提供芯片宏、包含目录和编译器信息：

```json
"C_Cpp.default.configurationProvider": "cl.eide"
```

如果用户设置开启后仍无提示，检查 `.code-workspace` 和 `.vscode/settings.json`，删除其中的 `"C_Cpp.errorSquiggles": "disabled"`。工作区设置会覆盖用户设置。使用 VS Code 多个配置文件（Profile）时，也要确认改的是当前配置文件的设置。

保存后执行 `Developer: Reload Window`，重新打开 `.c` 文件，确认语言模式为 C。等待分析完成，通过 `Ctrl + Shift + M` 查看“问题”面板。若仍异常，可执行 `C/C++: Log Diagnostics` 检查实际编译器与包含路径。

## 6. 使用 ST-Link 烧录

先安装 STM32CubeProgrammer 和 ST-Link 驱动，再按开发板原理图连接：

| ST-Link 信号 | STM32F103 目标端 |
| --- | --- |
| SWDIO | PA13 |
| SWCLK | PA14 |
| GND | GND，共地 |
| NRST（按需） | NRST |

目标板需正确供电，探针电压参考和供电连接以实际 ST-Link 与开发板说明为准。

在 EIDE 中选择“烧录配置 → STLink”，使用本机已配置的 STM32CubeProgrammer：

- 接口：`SWD`。
- 本模板下载速度：`4000 kHz`；连接不稳定时可尝试降低。
- 起始地址：`0x08000000`，适用于当前不含 Bootloader 的模板。
- 程序文件：可使用 `${ExecutableName}.hex`；本模板生成文件为 `build/Debug/Model.hex`。
- 烧录后运行：按需要开启。

先构建成功，再点击“烧录”或“构建并烧录”。更多字段见 [EIDE 烧录器配置](https://em-ide.com/docs/modules/flasher/)。本仓库记录了本地编译验证；实际烧录与运行仍需在自己的硬件上验证。

## 7. 日常开发与再次生成代码

在 VS Code 中编写应用，在 CubeMX 中修改硬件配置。重新生成前先提交或备份修改，并把需要保留的应用代码放入 `USER CODE BEGIN` / `USER CODE END` 区域，或放入自己维护的源文件。

每次增加外设或中间件、重新生成代码后，检查：

1. EIDE 的源文件范围是否覆盖新文件，新增目录是否需要手动添加。
2. 包含目录、器件宏和所需库是否发生变化。
3. 启动文件、链接脚本是否仍正确，是否出现重复的启动文件或系统初始化文件。
4. 移动过启动文件时，CubeMX 是否重新生成了原路径，以及 CMake 文件是否重新引用了它。
5. 应用自定义的 CMake 配置优先放在根 `CMakeLists.txt` 的用户扩展位置；生成目录中的配置需要在再生成后复查。

## 8. 可选：使用 CMake 构建

EIDE 和 CMake 是两套构建入口，修改一套不会自动更新另一套。`.vscode/tasks.json` 中的 `build`、`flash`、`rebuild` 等任务调用的是 EIDE；以下命令调用 CMake：

```powershell
cmake --preset Debug
cmake --build --preset Debug
```

Release 对应：

```powershell
cmake --preset Release
cmake --build --preset Release
```

执行前应能在终端找到 `cmake`、`ninja`、`arm-none-eabi-gcc`。当前配置的 CMake Debug 构建在本地通过，输出 ELF；CMake 配置没有像 EIDE 那样自动附加 HEX 转换任务。Release 可按上述命令自行验证。

两种方式目前都使用 `build/Debug` 作为 Debug 输出位置，都会生成编译数据库，也可能产生同名输出。建议日常选择一种构建入口；切换后重新配置和构建，避免将另一种方式的旧产物当成最新结果。

## 9. 常见问题

### 复制或改名后，CMake 提示缓存目录不匹配

`build/Debug/CMakeCache.txt` 记录了旧工程的绝对路径。复制或移动工程时不要携带 `build`。如果已经复制，在确认进入新工程根目录后，删除该工程的 `build` 文件夹，再重新配置、构建。

仅修改文件夹名不要求修改所有内部项目名；如果希望输出文件名也变化，则需分别调整 EIDE 项目名、CMake 项目名，以及后续 CubeMX 生成时使用的项目名。

### 缺少 RTE_Components.h、jtest_fw.h、rt_misc.h，或汇编出现 AREA / EQU 报错

先检查是否把整个 `Drivers` 作为源目录，误编译了 CMSIS 测试、其他内核示例或 ARMCC 专用代码。按第 4 节缩小源目录，并确认仅使用当前 MCU 的 GCC 启动文件，不要为了这些错误盲目补齐全部测试头文件。

### RTE_Components.h 删除后又出现

EIDE 会自动生成这个组件配置头文件，当前模板中内容基本为空。保留即可，不需要手动维护；`.gitignore` 已忽略根目录下的该文件。它自动出现与误编译无关示例引发的缺头文件错误是两件事。

### EIDE 能编译，CMake 却找不到启动文件

分别检查 `.eide/eide.yml` 和 `cmake/stm32cubemx/CMakeLists.txt` 中的文件路径。启动文件移入 `Startup` 后，CMake 也需要更新。

### GitHub 上传出现 Connection was reset

检查 Git 的网络连接。使用本地代理时，可按自己的实际 HTTP/混合代理端口，仅为当前仓库设置代理；不要照搬其他电脑的端口，也不要把代理地址写入工程共享配置。

## 10. 复制模板与版本管理

提交或复制时保留源码、`.ioc`、`.eide`、`.vscode`、`.code-workspace`、链接脚本及所需库；本模板也保留了 EIDE 所需的 `.pack`。忽略规则已排除 `build`、编译数据库和自动生成的 `RTE_Components.h` 等文件。

**`.gitignore` 只控制 Git 跟踪规则，不会阻止资源管理器复制文件。** 手动复制模板时仍需排除 `build`。已被 Git 跟踪的文件也不会仅因新增忽略规则而自动取消跟踪。

后续移植其他芯片时，请重新核对器件配置，不要只修改项目名称。CMSIS、HAL 和芯片包中的第三方代码遵循各自附带的许可证，使用和分发时请保留相关声明。
