# stm32-toolkit

STM32 tools and guides I use: microROS, serial framing, .bin generation, and CubeMonitor port-sharing.

## Toolkits (submodules)

Each toolkit lives in its own repository and is included here as a git submodule.

| Toolkit | What it is |
|---------|------------|
| [Micro-ROS-with-Nucleo-board](Micro-ROS-with-Nucleo-board) | Ready-to-build micro-ROS firmware template for the STM32 Nucleo-G474RE (FreeRTOS + LPUART/DMA transport). |
| [SerialFrame](SerialFrame) | C library for bidirectional, fixed-layout binary serial framing between STM32 and MATLAB/Simulink (Waijung). |

## Guides

| Guide | Covers |
|-------|--------|
| [Generate a .bin file](docs/generate-bin-file.md) | Make STM32CubeIDE emit `.bin`/`.hex` so you can share compiled firmware without the source (plus on-chip read-out protection). |
| [CubeMonitor shared mode](docs/cubemonitor-shared-mode.md) | Run STM32CubeMonitor and an IDE debug session on the same ST-LINK at once (shared mode). |
| [CubeMonitor dashboard + CSV](docs/cubemonitor-dashboard-and-csv.md) | Customize the CubeMonitor dashboard (gauges, layout) and export acquired data to CSV. |
