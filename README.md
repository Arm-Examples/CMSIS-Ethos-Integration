[![License](https://img.shields.io/github/license/Arm-Examples/CMSIS-Ethos-Integration?label=License)](./LICENSE)
[![Build](https://img.shields.io/github/actions/workflow/status/Arm-Examples/CMSIS-Ethos-Integration/Build.yml?logo=arm&logoColor=0091bd&label=Build%20example)](https://github.com/Arm-Examples/CMSIS-Ethos-Integration/tree/main/.github/workflows/Build.yml)

# CMSIS Ethos-U integration example

This application tests Vela-compiled LiteRT models on the Alif AppKit-E8-AIML
board, using its M55 high-performance core and Arm Ethos-U NPU. Separate
solutions provide Ethos-U55 and Ethos-U85 configurations.

The application runs selected models through Ethos-U and compares its output
with embedded reference data. A successful run reports `TEST RESULT: PASS`.

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

## Project layout

| Path | Purpose |
| --- | --- |
| [`Test-Ethos-U55.csolution.yml`](Test-Ethos-U55.csolution.yml), [`Test-Ethos-U85.csolution.yml`](Test-Ethos-U85.csolution.yml) | Defines targets, build and MLOps settings |
| [`Test-Ethos-U.cproject.yml`](Test-Ethos-U.cproject.yml) | Project file that selects application, ML layer, and board layer |
| [`Source/app_main.cpp`](Source/app_main.cpp) | Start the application and runs test |
| [`Model/`](Model/) | Model layer containing multiple ML models |
| [`Board/AppKit-E8_M55_HP/`](Board/AppKit-E8_M55_HP/) | Contains AppKit-E8 board layer |
| [`script/`](script/) | Model converter and its [usage guide](script/README.md) |

## ML Layer

The project selects ML layer with multiple models:

```yaml
  layers:
    - layer: $SolutionDir()$/Model/ML-MyModels.clayer.yml
```

`Model/ML-MyModels.clayer.yml` lists LiteRT framework, dependencies and ML models.
Its `MODEL_*` defines enable the matching descriptors and test calls in `Source/app_main.cpp`.

Each model group lists the `README.md` file, `VELA_SUMMARY.md` with recorded compilation settings
and memory requirements, the source and compiled model and reference data for tests.


## Integration Test

The test runner is simple. It supports ML model with
**one int8 input, one int8 output, and full Ethos-U offload**.

- `TestModel_Int8()` runs the model, checks every output value against its
  tolerance, optionally checks the winning class, and records a PASS/FAIL result.

A model's `PASS` result confirms that inference executed successfully through
Ethos-U and matched its reference within the configured tolerance on the target
being tested.

Tests run sequentially and reuse a 64 KiB, 16-byte-aligned tensor arena in
`Source/model_test.cpp`. Board linker files place the model and arena in
NPU-accessible memory.

## Integrate a new model

Use an existing model as a template:

1. **Prepare the files.** Create `Model/<name>/` with the original quantized
   `.tflite`, reference input/output C arrays and header, and a brief README.

2. **Convert the model.** Add `<name>/<model>.tflite` to the target solution's
   `mlops.model.name` list (relative to `mlops.model.dir: ./ML`). Open or build
   the solution to regenerate its MLOps file, then run, for example:

   ```sh
   python script/model-converter.py Test-Ethos-U85.cbuild-mlops.yml
   ```

   The [converter](script/README.md) generates the compiled `.tflite`, model C
   array, and `VELA_SUMMARY.md`. Confirm zero CPU operators and review memory use.

3. **Register the files.** In `Model/ML-MyModels.clayer.yml`, add a model group
   listing those generated files, reference sources/header, original `.tflite`,
   and README. Add a unique define such as `MODEL_MY_MODEL`.

4. **Describe the test.** In `Source/app_main.cpp`, under that define, include
   the reference header, declare the generated model array, and initialize a
   `ModelData_Int8` with its name, model, input, expected output, buffer sizes,
   allowed error in quantized steps, and optional argmax check.

5. **Call the test.** Add `TestModel_Int8(Model_My_Model)` under the same
   `#if defined(MODEL_MY_MODEL)` guard, between the existing `TestInitialize()`
   and `TestSummary()` calls. Tests must run sequentially to share the arena.

6. **Build and validate.** Check that the model and arena fit NPU-accessible
   memory; increase `TENSOR_ARENA_SIZE` in `Source/model_test.cpp` if needed,
   allowing for TFLM metadata and scratch. Build for the matching AppKit-E8 NPU,
   load and run, and confirm the new model's `PASS` and final
   `TEST RESULT: PASS` on UART4.

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

Set `npu.type` to `Ethos-U55` or `Ethos-U85` as indicated by the column, and set `npu.macs` to `256`.
Use the table's `system` and `memory` values for `vela` in the corresponding solution.
After changing these settings, recompile both `hello_world` and `tiny_cnn`:

| NPU | Conversion command                                                 |
| --- | ------------------------------------------------------------------ |
| U55 | `python script/model-converter.py Test-Ethos-U55.cbuild-mlops.yml` |
| U85 | `python script/model-converter.py Test-Ethos-U85.cbuild-mlops.yml` |

> NOTES
>
> - `SRAM0_SRAM1_COMBINED` is set in `app_mem_regions.h`
> - Vela system names correspond to `System_Config.<system>` sections in `.cmsis/ensemble_vela.ini`.
> - **U85: dedicated SRAM** configuration requires `ETHOS_CACHE_SIZE` to match the Vela `arena_cache_size` of 393216 bytes.
