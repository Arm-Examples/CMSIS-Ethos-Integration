[![License](https://img.shields.io/github/license/Arm-Examples/CMSIS-Ethos-Integration?label=License)](./LICENSE)
[![Build](https://img.shields.io/github/actions/workflow/status/Arm-Examples/CMSIS-Ethos-Integration/Build.yml?logo=arm&logoColor=0091bd&label=Build%20example)](https://github.com/Arm-Examples/CMSIS-Ethos-Integration/tree/main/.github/workflows/Build.yml)

# CMSIS Ethos-U integration example

This repository demonstrates a CMSIS-Toolbox application using TensorFlow Lite
Micro and the Arm Ethos-U NPU on the Alif AppKit-E8-AIML board. It provides
Ethos-U55 and Ethos-U85 configurations for the board's M55 high-performance
core. The test runs two Vela-compiled LiteRT models, `hello_world` and
`tiny_cnn`, and compares their outputs with embedded reference values. A
successful run prints `TEST RESULT: PASS` over the board's virtual COM port.

## Quick start in VS Code

1. Install [Keil Studio for VS Code](https://marketplace.visualstudio.com/items?itemName=Arm.keil-studio-pack).
   Clone this repository and open its root folder in VS Code. Use **Arm Tools:
   Activate Environment** if prompted; [Arm Tools Environment Manager](https://marketplace.visualstudio.com/items?itemName=Arm.environment-manager)
   installs the compiler, CMSIS-Toolbox, and build tools listed in
   [`vcpkg-configuration.json`](vcpkg-configuration.json). Activate an Arm Compiler
   license if you choose AC6; GCC is also supported.

2. Prepare the AppKit-E8 board as described in the
   [board layer README](Board/AppKit-E8_M55_HP/README.md). The device needs its
   ATOC and debug stubs programmed with Alif SETOOLS before this example can run.
   Use the `SE` position of SW4 for SETOOLS, then set SW4 to `U4` for UART4 test
   output through the PRG USB connector (J3).

3. Open the **CMSIS** view and select **Open Solution in Workspace**. Choose
   [`Test-Ethos-U85.csolution.yml`](Test-Ethos-U85.csolution.yml), then select
   the `AppKit-E8-U85` target, its `J-Link` target set, and the `Debug` build.
   Use the CMSIS **Build** action; the extension installs any missing public
   software packs. The supplied models target Ethos-U85 with 256 MACs and shared
   SRAM, so this is the starting configuration.

4. Use the CMSIS **Load and Run** or **Load and Debug** action with the board's
   J-Link connection. Open the board's UART4 virtual COM port to view the output.
   A successful test ends with `2 of 2 checks passed` and `TEST RESULT: PASS`.

For Ethos-U55, open [`Test-Ethos-U55.csolution.yml`](Test-Ethos-U55.csolution.yml)
and select `AppKit-E8-U55`. Recompile the models for U55 before running them;
see the [model converter guide](script/README.md) and the configuration table below.

The solutions also include `SSE-300-U55` and `SSE-320-U85` targets for Corstone
FVPs. Their board layers are in [`Board/Corstone-300/`](Board/Corstone-300/)
and [`Board/Corstone-320/`](Board/Corstone-320/).

## Command-line build

With the tools on `PATH` and the required packs available, build the U85
AppKit-E8 application with:

```sh
cbuild Test-Ethos-U85.csolution.yml --active AppKit-E8-U85@J-Link --toolchain GCC --packs
```

`--packs` downloads missing public packs. For a different target or compiler,
select it with `--active` or `--toolchain` respectively. The
[Build workflow](.github/workflows/Build.yml) checks the U55 and U85 solutions
with both AC6 and GCC.

## Project layout

| Path | Purpose |
| ---- | ------- |
| [`Test-Ethos-U55.csolution.yml`](Test-Ethos-U55.csolution.yml), [`Test-Ethos-U85.csolution.yml`](Test-Ethos-U85.csolution.yml) | Target, build, and MLOps settings for each NPU |
| [`Test-Ethos-U.cproject.yml`](Test-Ethos-U.cproject.yml) | Shared test application and layer selection |
| [`Board/AppKit-E8_M55_HP/`](Board/AppKit-E8_M55_HP/) | AppKit-E8 board layers, linker configuration, and board setup guide |
| [`Model/`](Model/) | Quantized and Vela-compiled models, model sources, and the ML layer |
| [`Source/test_main.cpp`](Source/test_main.cpp) | Runs both models and checks their outputs |
| [`script/`](script/) | Model converter and its [usage guide](script/README.md) |

The checked-in Vela outputs are ready to build. After changing an NPU or memory
configuration, generate the solution's `*.cbuild-mlops.yml` by opening or
building that solution, then recompile the models as described below and in the
[model converter guide](script/README.md).

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
