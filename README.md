# ARES Rover

ARES Rover is an experimental Mars-rover platform combining terrain perception
research with embedded rover electronics. The repository currently contains:

- **Terrain AI notebooks** for inspecting the AI4Mars dataset and training a
  small PyTorch U-Net segmentation baseline.
- **Arduino Nano firmware** managed with PlatformIO.
- **A minimal LED blink sketch** for validating a Nano board and toolchain.

The project is under active development. The current firmware is a hardware
bring-up test, not a complete rover controller.

## Repository layout

```text
.
├── ARES_Terrain_AI.ipynb          # AI4Mars metadata and data-access study
├── ARES_Terrain_Training.ipynb    # Bounded terrain-segmentation baseline
├── ares-nano/
│   ├── platformio.ini              # PlatformIO environment for Arduino Nano
│   ├── src/main.cpp                # Main PlatformIO firmware
│   ├── include/                    # Project-local headers
│   ├── lib/                        # Project-local libraries
│   └── test/                       # PlatformIO tests
└── firmware/
    └── nano/blink_test/            # Standalone Arduino IDE blink test
```

## Terrain AI

### `ARES_Terrain_AI.ipynb`

This notebook documents the AI4Mars dataset, verifies the runtime, and
demonstrates a resource-conscious streaming and inspection workflow. It is
intended to run in Google Colab, with training and experiment artifacts stored
outside the repository.

### `ARES_Terrain_Training.ipynb`

This notebook trains a reproducible, lightweight PyTorch U-Net baseline using a
bounded sample from the `hassanjbara/AI4MARS` Hugging Face dataset mirror. The
current experiment configuration uses:

- 640 usable labeled samples
- 512 training samples and 128 validation samples
- 256 × 256 input images
- Four semantic classes: soil, bedrock, sand, and big rock
- Label value `255` for ignored or unlabeled pixels
- A CUDA GPU, with the notebook currently expecting a Tesla T4

This is a preliminary baseline rather than a full-dataset performance claim.
The notebook itself records important dataset and split limitations.

### Running the notebooks

1. Open the relevant notebook in Google Colab or Jupyter.
2. Provide the required runtime:
   - `ARES_Terrain_AI.ipynb`: a Colab runtime is recommended.
   - `ARES_Terrain_Training.ipynb`: a CUDA-enabled Kaggle runtime with a
     Tesla T4 is currently required by the notebook checks.
3. Install the notebook dependencies in the runtime if they are not already
   available:

   ```bash
   pip install torch numpy pillow matplotlib datasets
   ```

4. Run the cells from top to bottom.

Do not commit datasets, checkpoints, model weights, credentials, or generated
experiment artifacts. The repository `.gitignore` excludes common dataset and
checkpoint directories and extensions.

## Arduino Nano firmware

The PlatformIO project in [`ares-nano`](./ares-nano) targets an Arduino Nano
with the ATmega328P:

- Platform: `atmelavr`
- Board: `nanoatmega328`
- Framework: Arduino
- Serial monitor speed: 9600 baud
- Upload speed: 115200 baud

### Requirements

- [PlatformIO](https://platformio.org/) through the CLI or VS Code extension
- A compatible Arduino Nano
- USB data cable and the appropriate USB-serial driver, if required by the
  board

### Build, upload, and monitor

From the repository root:

```bash
cd ares-nano
pio run
pio run --target upload
pio device monitor
```

The current `src/main.cpp` blinks the built-in LED every 500 ms. Select the
correct upload port automatically or specify it explicitly when more than one
serial device is connected:

```bash
pio run --target upload --upload-port COM3
pio device monitor --port COM3 --baud 9600
```

### Standalone blink test

[`firmware/nano/blink_test`](./firmware/nano/blink_test) contains an equivalent
Arduino IDE sketch for checking the board independently of PlatformIO. Open
`blink_test.ino`, select the appropriate Arduino Nano board and processor, then
upload it using the Arduino IDE.

## Development notes

- Keep large datasets and generated training outputs outside Git.
- Record the dataset release and experiment configuration when comparing model
  results.
- Validate hardware changes on the target Nano before integrating sensors,
  motors, or communications.
- The AI notebooks and firmware are currently separate workstreams; no runtime
  path yet connects model inference to the Nano firmware.

## Status

| Area | Current state |
| --- | --- |
| Terrain dataset inspection | Notebook-based prototype |
| Terrain segmentation | Small, bounded U-Net baseline |
| Nano toolchain | Configured with PlatformIO |
| Nano application | LED blink bring-up test |
| Sensor and motor integration | Not implemented |
| On-device AI inference | Not implemented |

## Contributing

When contributing, keep changes focused and include the commands or notebook
runtime used to validate them. Avoid committing generated files, private
credentials, datasets, or model weights.
