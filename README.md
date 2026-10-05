# Person detection demo (nRF54LM20 DK, Axon)

This application is based on Person Detection application from [Edge AI Add-on for nRF Connect SDK](https://github.com/nrfconnect/sdk-edge-ai).
It runs the `person_det` Axon model on frames from an ArduCam Mega SPI Camera at 128×128 RGB565.
The 160×128 model input is padded horizontally with neutral gray (`src/main.c`).

This is a west workspace application. `west.yml` pins the compatible Edge AI Add-on release.

This application uses minimal and customized driver for the ArduCam Camera to achieve higher framerate.

## Requirements:

The application supports the following development kit:

* nRF54LM20 DK (target `nrf54lm20dk/nrf54lm20b/cpuapp`)

It also requires:

* ArduCam Mega 5MP SPI Camera (SKU B0401)
* Power Profiling Kit II - if you want to measure current and trace over GPIOs.

**Note:** Use Board Configurator to set VDD to 3.3 V.

### Pin mapping

Wire the ArduCam Mega SPI module to the DK (3.3 V logic).

| ArduCam Mega | nRF54LM20 DK |
| --- | --- |
| VCC | VDDIO |
| GND | GND |
| SCK | P1.04 |
| MISO | P1.05 |
| MOSI | P1.06 |
| CS | P1.07 |

Optionally wire the PPK to the DK.

| PPK | nRF54LM20 DK | trace function |
| --- | --- | --- |
| D1 | P1.10 | camera capture |
| D1 | P1.11 | image preprocessing |
| D1 | P1.12 | inference |
| D1 | P1.13 | postprocessing |

## SDK Setup

Either initialize a west workspace from this repository or use existing workspace with compatible Edge AI Add-on release.

## Configuration options

Postprocessing thresholds are Kconfig options (values in per mille, 0–1000):

- `CONFIG_SCORE_THRESHOLD` — detection score cutoff (default 250)
- `CONFIG_IOU_THRESHOLD` — non-maximum suppression IoU cutoff (default 450)

Tune them in `prj.conf` or via `menuconfig` under **Person Detection**.

## Build and flash

Debug build (logging enabled):

```bash
west build -b nrf54lm20dk/nrf54lm20b/cpuapp
west flash
```

Release build:

```bash
west build -b nrf54lm20dk/nrf54lm20b/cpuapp -- -DFILE_SUFFIX=release
west flash
```

## Runtime behavior

The application will either print on console the bounding box for each detection or inform about no detections:

```
Bounding box 0: head s32, [22.9, 17.0, 138.9, 128.0] score 0.849
No detections
```

### User interface

**LED0** (capture LED):
   Toggles (changes state between on and off) each time a frame is captured from the camera and processed.

**LED1** (detection LED):
   Turns on when persons are detected in the frame.
   Turns off when no detections are found.

## Image stream and result visualization

Captured image and model prediction are streamed over USB CDC ACM.
Connect the DK to the PC using SoC USB port.
The SoC USB port is located next to debugger USB port.
Run the host viewer with the USB CDC ACM serial port as the first argument.
The viewer will show the captured frame, framerate and bounding boxes with their prediction score.

```bash
python scripts/live_usb_person_detection.py --port /dev/ttyACM2
```

Dependencies: `pyserial`, `opencv-python`, `numpy`.

If the board is the only serial device connected with both USB ports to the host:

- `COM0`/`ttyACM0` and `COM1`/`ttyACM1` are for the debugger
- `COM2`/`ttyACM2` is USB CDC ACM

## License

5-Clause Nordic License, see [LICENSE](LICENSE).
Selected components have different licensing, see [NOTICE](NOTICE).
