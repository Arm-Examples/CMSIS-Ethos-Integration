[![License](https://img.shields.io/github/license/Arm-Examples/CMSIS-Ethos-Integration?label=License)](./LICENSE)
[![Build](https://img.shields.io/github/actions/workflow/status/Arm-Examples/CMSIS-Ethos-Integration/Build.yml?logo=arm&logoColor=0091bd&label=Build%20example)](https://github.com/Arm-Examples/CMSIS-Ethos-Integration/tree/main/.github/workflows/Build.yml)

# CMSIS-Ethos-Integration

This repository contains Ethos-U integration guidance for a reproducible and verified developer journey.

## Project Configurations for AppKit-E8-AIML

Each column describes one combination of NPU and Vela memory mode.

| Setting                | U55: SRAM only      | U55: shared SRAM    | U85: SRAM only        | U85: shared SRAM      | U85: dedicated SRAM   |
| ---------------------- | ------------------- | ------------------- | --------------------- | --------------------- |---------------------- |
| Solution               | `Test-Ethos-U55`    | `Test-Ethos-U55`    | `Test-Ethos-U85`      | `Test-Ethos-U85`      | `Test-Ethos-U85`      |
| Vela `system`          | `RTSS_HP_SRAM_Only` | `RTSS_HP_SRAM_MRAM` | `Ethos_U85_SRAM_Only` | `Ethos_U85_SRAM_MRAM` | `Ethos_U85_SRAM_SRAM` |
| Vela `memory`          | `Sram_Only`         | `Shared_Sram`       | `Sram_Only`           | `Shared_Sram`         | `Dedicated_Sram`      |
| `NPU_QCONFIG`          | `1`                 | `2`                 | `1`                   | `2`                   | `1`                   |
| `NPU_REGIONCFG_0`      | `1`                 | `3`                 | `1`                   | `3`                   | `1`                   |
| `NPU_REGIONCFG_1`      | `0`                 | `0`                 | `0`                   | `0`                   | `0`                   |
| `NPU_REGIONCFG_2`      | Unused              | Unused              | Unused                | Unused                | `1`                   |
| `ETHOS_CACHE_SIZE`     | Not used            | Not used            | Not used              | Not used              | `393216` bytes        |
| `ethos_model`          | SRAM via AXI0       | MRAM via AXI1       | SRAM via `AXI_SRAM`   | MRAM via `AXI_EXT`    | SRAM1 via `AXI_SRAM`  |
| `ethos_arena`          | SRAM via AXI0       | SRAM via AXI0       | SRAM via `AXI_SRAM`   | SRAM via `AXI_SRAM`   | SRAM1 via `AXI_SRAM`  |
| `ethos_cache`          | Unused              | Unused              | Unused                | Unused                | SRAM0 via `AXI_SRAM`  |
| `SRAM0_SRAM1_COMBINED` | `1`                 | `1`                 | `1`                   | `1`                   | `0`                   |

Set `npu.type` to `Ethos-U55` or `Ethos-U85` as indicated by the column, and set `npu.macs` to `256`. Use the table's `system` and `memory` values for `vela` in the corresponding solution. After changing these settings, recompile both `hello_world` and `tiny_cnn`:

| NPU | Conversion command                                                 |
| --- | ------------------------------------------------------------------ |
| U55 | `python script/model-converter.py Test-Ethos-U55.cbuild-mlops.yml` |
| U85 | `python script/model-converter.py Test-Ethos-U85.cbuild-mlops.yml` |

> NOTES
>
> - `SRAM0_SRAM1_COMBINED` is set in `app_mem_regions.h`
> - Vela system names correspond to `System_Config.<system>` sections in `.cmsis/ensemble_vela.ini`.
> - **U85: dedicated SRAM** configuration requires `ETHOS_CACHE_SIZE` to match the Vela `arena_cache_size` of 393216 bytes.
