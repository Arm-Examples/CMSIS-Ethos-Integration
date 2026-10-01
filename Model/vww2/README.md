# VWW-2

Arm ML Zoo's quantized MicroNet VWW-2 model detects whether a grayscale image
contains a person.

- **Input:** 50 x 50 grayscale image, 2500 `int8` values.
- **Output:** two `int8` scores: no person (index 0) and person (index 1).
- **Test vector:** expected output `[95, 40]`, classified as no person.
- **Validation:** at most two quantized steps of error per value and matching class.

`vww2_50_50_INT8.tflite` is the original model; its `_vela.tflite` and `_model.c`
files target Ethos-U85-256 with shared SRAM. See
[VELA_SUMMARY.md](VELA_SUMMARY.md) for compilation and memory details.
Enable `MODEL_VWW2` in the combined
[ML layer](../ML-MyModels.clayer.yml) to run its `TestModel_Int8` call in
[app_main.cpp](../../Source/app_main.cpp). See the
[integration guide](../../README.md#integrate-a-new-model) for conversion and
adding models. Reference vectors are provided as NumPy and C arrays. To regenerate the C
arrays, run from the repository root:

```sh
python Model/vww2/generate-vww2-reference.py
```

This requires NumPy and does not compile the model. U55-256 compilation was
verified with full NPU offload; inference accuracy still needs board validation.
Reconvert the model for the selected target before running it.

The model and NumPy vectors come from
[Arm-Examples/ML-zoo](https://github.com/Arm-Examples/ML-zoo/tree/fec0bb5bcd486eb2839cdb7d437d466067402a6a/models/visual_wake_words/micronet_vww2/tflite_int8)
at revision `fec0bb5bcd486eb2839cdb7d437d466067402a6a`, under Apache-2.0.
Original model SHA-1: `5d887ca438c0a7feeed3c8c22dce99b55565c8ea`.
