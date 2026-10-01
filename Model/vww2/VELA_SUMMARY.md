# Vela report: vww2_50_50_INT8

- accelerator: `ethos-u85-256`
- system config: `Ethos_U85_SRAM_MRAM`
- memory mode: `Shared_Sram`

```
Network summary for vww2_50_50_INT8
Accelerator configuration               Ethos_U85_256
System configuration              Ethos_U85_SRAM_MRAM
Memory mode                               Shared_Sram
Accelerator clock                                 400 MHz
Design peak SRAM bandwidth                      11.92 GB/s
Design peak Off-chip Flash bandwidth             0.72 GB/s

Total SRAM used                                 32.64 KiB
Total Off-chip Flash used                      149.41 KiB

CPU operators = 0 (0.0%)
NPU operators = 67 (100.0%)

Average SRAM bandwidth                           1.56 GB/s
Input   SRAM bandwidth                           0.27 MB/batch
Weight  SRAM bandwidth                           0.14 MB/batch
Output  SRAM bandwidth                           0.19 MB/batch
Total   SRAM bandwidth                           0.64 MB/batch
Total   SRAM bandwidth            per input      0.64 MB/inference (batch size 1)

Average Off-chip Flash bandwidth                 0.49 GB/s
Input   Off-chip Flash bandwidth                 0.05 MB/batch
Weight  Off-chip Flash bandwidth                 0.13 MB/batch
Output  Off-chip Flash bandwidth                 0.00 MB/batch
Total   Off-chip Flash bandwidth                 0.20 MB/batch
Total   Off-chip Flash bandwidth  per input      0.20 MB/inference (batch size 1)

Neural network macs                           3112594 MACs/batch
```
