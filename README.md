# SECDED Hamming-Code Fault-Tolerant Memory

A digital logic project implemented in **Logisim Evolution** that demonstrates Single Error Correction and Double Error Detection (SECDED) using an extended Hamming code. The system combines memory storage, controlled fault injection, error classification, single-bit correction, and diagnostic counters in one circuit.

## Overview

Memory data can be corrupted by bit errors. Error-correcting codes add redundant bits that help detect and correct certain errors.

This project stores 4-bit data as an 8-bit extended Hamming codeword. During a read operation, errors can be injected into the read path to test the error detection and correction logic.

The system supports:

- Detection of error-free codewords.
- Correction of single-bit errors.
- Detection of double-bit errors.
- Controlled error injection at any of the eight codeword bit positions.
- Two independently controlled error-injection stages.
- Diagnostic counters for read operations, detected errors, corrected errors, and uncorrectable errors.

## Architecture

The system follows this data path:

```text
4-bit Input Data
       |
       v
Hamming Encoder (4-bit to 8-bit)
       |
       v
8-bit Memory Cell
       |
       v
Error Injection Stage A
       |
       v
Error Injection Stage B
       |
       v
Syndrome Generator
       |
       v
SECDED Error Classifier
       |
       v
Single-Bit Correction Logic
       |
       v
Data Extraction (4-bit)
       |
       v
4-bit Output Data
```

Diagnostic counters are integrated into the main circuit and record the read and error-classification results.

## SECDED Encoding

The 8-bit codeword contains four data bits and four redundant parity bits.

| Codeword Bits | Purpose |
|---|---|
| R1, R2, R4 | Hamming parity bits |
| R3, R5, R6, R7 | Four data bits |
| R8 | Overall parity bit |

The circuit uses even parity. The syndrome identifies the position of a potential error among R1–R7, while the overall parity check helps distinguish single-bit errors from double-bit errors.

### Error Classification

| Syndrome (S) | Overall Parity Flag (c) | Interpretation | Action |
|---|---|---|---|
| 0 | 0 | No error detected | No correction |
| Non-zero | 1 | Single-bit error in R1–R7 | Correct the indicated bit |
| 0 | 1 | Single-bit error in R8 | Correct R8 |
| Non-zero | 0 | Double-bit error detected | Disable correction |

These rules apply to zero, one, or two bit errors. Errors involving more than two bits may be misclassified.

## Main Modules

| Module | Responsibility |
|---|---|
| `Hamming_Encode_4to8` | Encodes four data bits into an eight-bit SECDED codeword. |
| `Mem_Cell_8` | Stores the encoded codeword using clock-controlled storage. |
| `Error_Inject_A` | Injects a selected bit error into the read path. |
| `Error_Inject_B` | Provides a second, independently controlled error-injection stage. |
| `Hamming_Syndrome_8` | Generates the syndrome and overall parity flag. |
| `SECDED_Classify` | Classifies the received codeword as error-free, single-bit error, or double-bit error. |
| `BitFlip_8` | Corrects a classified single-bit error and disables correction for detected double-bit errors. |
| `Data_Extract_4` | Extracts the four data bits from the corrected codeword. |
| `Main` | Integrates the memory data path, control signals, correction logic, outputs, and diagnostic counters. |

## Inputs and Outputs

### Inputs

| Signal | Description |
|---|---|
| `D1–D4` | Four-bit input data. |
| `Clk` | Clock signal for memory writes and diagnostic counters. |
| `WriteEn` | Enables storage of the encoded codeword on the active clock edge. |
| `Read` | Enables the data outputs and read-related counter operations. |
| `Reset` | Clears the diagnostic counters. |
| `InjectEn1` | Enables the first error-injection stage. |
| `InjectPos1` | Selects the bit to inject an error into in the first stage. |
| `InjectEn2` | Enables the second error-injection stage. |
| `InjectPos2` | Selects the bit to inject an error into in the second stage. |

### Outputs

| Signal | Description |
|---|---|
| `D1_out–D4_out` | Recovered four-bit data; forced to zero when `Read = 0`. |
| `noErr` | Indicates that no error was detected. |
| `singleErr` | Indicates a classified single-bit error. |
| `doubleErr` | Indicates a classified double-bit error. |
| `ReadCounter` | Counts active read operations. |
| `DetectCounter` | Counts reads classified as containing an error. |
| `FixCounter` | Counts reads classified as single-bit errors. |
| `UncorrectableCounter` | Counts reads classified as double-bit errors. |

Each diagnostic counter is four bits wide and wraps around after 15.

## Error Injection

Each injection stage uses an enable signal and a three-bit position selector.

| `InjectPos` | Bit Selected |
|---|---|
| `000` | R1 |
| `001` | R2 |
| `010` | R3 |
| `011` | R4 |
| `100` | R5 |
| `101` | R6 |
| `110` | R7 |
| `111` | R8 |

When an injection stage is disabled, it passes the codeword unchanged. When enabled, it inverts the selected bit.

Because the injection stages operate on the read path, injecting an error does not modify the stored codeword.

## How to Run the Project

1. Install and open Logisim Evolution.
2. Clone or download this GitHub repository.
3. Open the project `.circ` file in Logisim Evolution.
4. Select the `Main` circuit.
5. Set the input data bits `D1–D4`.
6. Assert `WriteEn` and apply the required clock edge to store the encoded codeword.
7. Deassert `WriteEn`, enable `Read`, and inspect the output data and error-classification signals.
8. To test a single-bit error, enable one injection stage and select a bit position.
9. To test a double-bit error, enable both injection stages and select two different bit positions.
10. Observe the classification outputs and diagnostic counters.
11. Assert `Reset` to clear the diagnostic counters before a new counter experiment.

**Note:** The exact `.circ` filename may differ. Open the circuit file included in the repository.

## Example Test

For input data `1010`, the encoded codeword is:

```text
Input data:       1010
Encoded codeword: 10110101
```

Injecting an error at R3 should produce a single-bit error classification. The correction logic should restore the original data output to `1010`.

Injecting errors at R3 and R5 simultaneously should produce a double-bit error classification, and single-bit correction should be disabled.

## Experimental Results

The following functional results were observed during simulation:

| Test | Observed Result |
|---|---|
| Single-bit error positions tested | 8 |
| Single-bit error positions corrected | 8/8 |
| Double-bit error patterns tested | 5 |
| Double-bit error patterns detected | 5/5 |
| Total reads in the diagnostic counter test | 10 |
| Final `ReadCounter` | 10 |
| Final `DetectCounter` | 7 |
| Final `FixCounter` | 5 |
| Final `UncorrectableCounter` | 2 |

These results describe the tested simulation cases; they do not establish physical hardware performance or guarantee correct classification for errors affecting more than two bits.

## Limitations

- The implementation handles a single four-bit data word rather than a large memory array.
- Single-bit errors are corrected, while detected double-bit errors are flagged but not corrected.
- Corrected data is not written back into memory.
- The injection mechanism models controlled bit flips in simulation.
- Errors affecting more than two bits may be misclassified.
- No physical FPGA implementation, power measurement, or timing benchmark was performed.

## Future Improvements

- Extend the data width and memory capacity.
- Add corrected-codeword write-back.
- Increase counter width and provide overflow handling.
- Automate exhaustive testing of all possible two-bit error combinations.
- Implement and evaluate the circuit on an FPGA.
- Add timing analysis and more comprehensive fault models.

## Technologies Used

- **Logisim Evolution** — digital circuit design and simulation.
- **Extended Hamming SECDED** — error detection and correction.
- **Digital logic components** — parity logic, XOR gates, multiplexers, registers, and counters.

## License

Add a license if you intend to permit others to reuse or modify this project. Without a license, standard copyright restrictions generally apply.


