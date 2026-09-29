# Vela report: tiny_cnn

- accelerator: `ethos-u85-256`
- system config: `Ethos_U85_SRAM_MRAM`
- memory mode: `Shared_Sram`

```
Network summary for tiny_cnn_int8
Accelerator configuration               Ethos_U85_256
System configuration              Ethos_U85_SRAM_MRAM
Memory mode                               Shared_Sram
Accelerator clock                                 400 MHz
Design peak SRAM bandwidth                      11.92 GB/s
Design peak Off-chip Flash bandwidth             0.72 GB/s

Total SRAM used                                  5.28 KiB
Total Off-chip Flash used                        2.41 KiB

CPU operators = 0 (0.0%)
NPU operators = 6 (100.0%)

Average SRAM bandwidth                           1.87 GB/s
Input   SRAM bandwidth                           0.01 MB/batch
Weight  SRAM bandwidth                           0.00 MB/batch
Output  SRAM bandwidth                           0.01 MB/batch
Total   SRAM bandwidth                           0.02 MB/batch
Total   SRAM bandwidth            per input      0.02 MB/inference (batch size 1)

Average Off-chip Flash bandwidth                 0.26 GB/s
Input   Off-chip Flash bandwidth                 0.00 MB/batch
Weight  Off-chip Flash bandwidth                 0.00 MB/batch
Output  Off-chip Flash bandwidth                 0.00 MB/batch
Total   Off-chip Flash bandwidth                 0.00 MB/batch
Total   Off-chip Flash bandwidth  per input      0.00 MB/inference (batch size 1)

Neural network macs                             26112 MACs/batch
```
