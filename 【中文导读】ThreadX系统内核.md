# ThreadX（Azure RTOS 实时操作系统内核）学习导读

> 目录：`01-RTOS操作系统/threadx-ThreadX系统内核`
> 来源：https://github.com/eclipse-threadx/threadx （压缩包：threadx-master.zip）

## 这是什么

ThreadX 是微软（原 Express Logic）的深度嵌入式实时操作系统，2019 年开源，在商业 RTOS 里以"极小、极快、确定性极强"著称：内核最小约 2KB Flash、调度确定性与任务数无关。全球部署量数十亿台（打印机、路由器、NASA 的部分航天设备也在用）。它是 **Eclipse 基金会项目 Azure RTOS** 的核心，配套还有 FileX（文件系统）、NetX Duo（网络）、GUIX（图形）。

## 和 FreeRTOS 的对比（学习视角）

| 维度 | ThreadX | FreeRTOS |
| --- | --- | --- |
| 调度 | 抢占式+优先级继承，轮转可关 | 抢占式+时间片轮转 |
| 对象创建 | 编译期静态创建为主（TX_XXX 控制块） | 运行时动态创建为主 |
| 中断处理 | 不用区分 FromISR 版本 API | 中断必须用 FromISR API |
| 许可 | MIT | MIT |
| 中文资料 | 较少 | 非常多 |

## 核心源码怎么读

- `common/src/`：全部内核实现。重点 `tx_thread_schedule.c`（调度主循环）、`tx_thread_context_save/load.S` 类汇编（上下文切换）、`tx_semaphore*.c`、`tx_mutex*.c`、`tx_queue*.c`、`tx_timer*.c`。
- `common/inc/`：`tx_api.h`、`tx_port.h` 等头文件。
- `ports/`：按芯片/编译器分目录，如 `ports/cortex_m4/gnu/`（STM32F4 可用）、`ports/cortex_m33/gnu/` 等。每个 port 里有 `tx_initialize_low_level.S`（启动时中断向量、SysTick 设置）和 `tx_thread_context*.S`（切换汇编）——**这是读透 RTOS 移植的最佳教材，汇编只有几百行**。
- `samples/`：演示工程（`sample_threadx.c` 一个文件就能看懂完整生命周期：`tx_kernel_enter` → 定义对象 → 创建线程 → 调度）。
- `utility/`：TrackX 事件追踪等分析工具。

## 学习要点

1. **`tx_kernel_enter()` 之前做了什么**：初始化数据段、构建优先级位图、调用应用 `tx_application_define()` 静态创建所有对象。
2. **看门狗式调度循环**：`_tx_thread_schedule` 是一个死循环，找到最高优先级就绪线程执行，其确定性分析比 FreeRTOS 更直接。
3. **无 FromISR 的设计**：ThreadX 通过"中断嵌套保护计数"统一处理，理解两种设计差异会让你对 RTOS 本质理解上一个台阶。
4. **时间片与抢占阈值（preemption threshold）**：ThreadX 独有的特性，允许任务临时提高抢占门槛来减少不必要切换——读 `tx_thread_preemption_change.c`。

## 建议购买/准备

- 不用买任何东西。想上板的话，STM32 Nucleo 系列（¥100 左右）+ STM32CubeIDE 自带 ThreadX（CubeMX 里一键勾选）。
- ESP32-S3 也可体验：ESP-IDF 同时支持 FreeRTOS 兼容层和原生 ThreadX demo（见 esp-idf `examples/system/threadx`）。

## 学习目标（自测）

- [ ] 能对比 FreeRTOS 和 ThreadX 在中断 API 上的设计差异并说明理由
- [ ] 能看懂一个 port 的上下文切换汇编
- [ ] 理解 preemption threshold 解决什么问题
