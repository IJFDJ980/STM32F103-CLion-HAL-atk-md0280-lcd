# STM32F103-CLion-HAL-atk-md0280-lcd
适配CLion + STM32CubeMX HAL库 + ARM-GCC工具链的正点原子新版MD0280 TFTLCD驱动移植工程

## 一、项目介绍
本工程移植了正点原子新版MD0280 LCD驱动，原生官方驱动仅支持Keil MDK，本仓库完成跨工具链适配：
- 开发IDE：CLion 202x
- 芯片型号：STM32F103ZET6（精英板）
- 固件库：STM32 HAL库（CubeMX自动生成）
- 编译工具：ARM-GCC + CMake
- 屏幕模块：正点原子MD0280 2.8寸TFTLCD（FSMC接口驱动，新版驱动）

## 二、踩坑记录（全网稀缺，重点）
网上无同款CLion+GCC移植新版原子LCD完整教程，3天调试解决全部兼容问题：
1. GCC编译器隐式函数声明报错：补充头文件声明，替换标准库延时函数
2. `__nop()` 编译器宏不兼容：替换为HAL标准 `__NOP()`
3. CMake编译链接失败：手动在CMakeLists添加全部驱动源文件
4. FSMC硬件配置关键：16位数据宽度、NE4片选、地址偏移正确映射到0x6C000000
5. 驱动分层架构适配：新版驱动分层解耦，单独重写底层FSMC读写接口适配HAL库

## 三、快速使用步骤
1. Clone本仓库到本地CLion
2. 用STM32CubeMX重新生成HAL工程（FSMC参数严格和本工程一致）
3. 将仓库内`atk_md0280`系列驱动文件复制到工程Core目录
4. 在`CMakeLists.txt`内添加`.c`驱动源文件路径
5. 编译下载，上电即可点亮屏幕
6. 内置LCD屏幕调试打印函数，可直接替代串口在线观测变量

## 四、调试工具封装
工程内置屏幕格式化打印函数`lcd_debug_printf()`，用法等同于`printf`，无需串口，直接在屏幕定点打印变量做板载调试：
```c
// 示例：第0行白色文字打印系统运行毫秒数
lcd_debug_printf(0, 0xFFFF, "系统运行时间：%d ms", HAL_GetTick());
五、硬件接线说明
FSMC 接口硬件默认精英板标准接线，无需改动排线：
FSMC_D[0:15] → LCD D0~D15
FSMC_NE4 → LCD CS 片选
FSMC_A10 → LCD RS 寄存器选择引脚
FSMC_NOE → LCD RD 读信号
FSMC_NWE → LCD WR 写信号
六、版权说明
驱动源码版权归属正点原子，本项目仅做学习用途的工具链适配修改，禁止商用。
