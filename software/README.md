## Setting up the environments

### Prerequisites
- **Python**: Version 3.8 or 3.9 (recommended)
- **Operating System**: Linux (Ubuntu 18.04/20.04/22.04 recommended), macOS, or Windows
- **Hardware**: SiMoRa/Teensy-based controller (default) or legacy Arduino Due

### Step 1: Create a Virtual Environment (Recommended)

Using a virtual environment isolates dependencies and prevents conflicts:

```bash
# Create virtual environment
python3 -m venv octopi_env

# Activate virtual environment
# On Linux/macOS:
source octopi_env/bin/activate
# On Windows:
octopi_env\Scripts\activate
```

### Step 2: Install Python Dependencies

The `requirements.txt` file contains all Python dependencies with pinned versions for reproducibility:

```bash
# Upgrade pip first (recommended)
pip install --upgrade pip

# Install all dependencies
pip install -r requirements.txt
```

**What's included:**
- **GUI Framework**: PyQt5 (5.15.10), pyqtgraph (0.13.3), qtpy (2.4.1)
- **Numerical Computing**: numpy (1.21.6), scipy (1.7.3), pandas (1.3.5)
- **Computer Vision**: opencv-python (4.5.5.64), opencv-contrib-python (4.5.5.64)
- **XML Processing**: lxml (4.9.2)
- **Serial Communication**: pyserial (3.5) - for Teensy/Arduino communication
- **CRC Calculation**: crc (1.2.0) - for Teensy protocol CRC-8 checksums
- **Logging Utilities**: platformdirs (3.2.0) - for squid.logging module

**Alternative: Using system packages for Qt (Linux only)**

If you prefer to use system-installed Qt packages:

```bash
sudo apt-get install python3-pip python3-pyqt5 python3-pyqtgraph libcanberra-gtk-module libcanberra-gtk3-module
pip install -r requirements.txt
```

Note: This may cause version conflicts. Using the pip-installed versions from requirements.txt is recommended.

### Step 3: Install Camera Drivers

**The Imaging Source (TIS) Cameras:**
Follow the installation instructions at: https://github.com/TheImagingSource/tiscamera

**Daheng Cameras:**
1. Navigate to `drivers and libraries/daheng camera/`
2. Follow the README.md instructions in that folder
3. The Python `gxipy` module is included in `control/gxipy/` and doesn't require separate pip installation
4. You may need to install system-level drivers from Daheng's website

**Note**: Camera drivers are hardware-specific and may require system-level installation beyond Python packages.

### Step 4: Enable Serial Port Access (Linux/macOS)

To access the Teensy/Arduino controller without `sudo`:

```bash
# Add user to dialout group (Linux)
sudo usermod -aG dialout $USER

# Log out and back in, or run:
newgrp dialout
```

**macOS**: Usually no additional setup needed. If you encounter permission issues, check System Preferences > Security & Privacy.

**Windows**: Usually no additional setup needed. COM ports should be accessible automatically.
 
### (optional) install pytorch and torchvision on Jetson Nano
Follow instructions on https://forums.developer.nvidia.com/t/pytorch-for-jetson-nano-version-1-5-0-now-available/72048

```
sudo apt-get install libhdf5-serial-dev hdf5-tools libhdf5-dev zlib1g-dev zip libjpeg8-dev liblapack-dev libblas-dev gfortran
sudo apt-get install python3-pip libopenblas-base libopenmpi-dev 
pip3 install -U pip testresources setuptools
pip3 install Cython
wget https://nvidia.box.com/shared/static/3ibazbiwtkl181n95n9em3wtrca7tdzp.whl -o torch-1.5.0-cp36-cp36m-linux_aarch64.whl
pip3 install torch-1.5.0-cp36-cp36m-linux_aarch64.whl
```
```
sudo apt-get install libjpeg-dev zlib1g-dev
git clone --branch torchvision v0.6.0 https://github.com/pytorch/vision torchvision   # see below for version of torchvision to download
cd torchvision
sudo python3 setup.py install
```
## Configuring the software
Create a `configuration.txt` file in the software folder to set up variables for a specific machine. The file is loaded by [`control/_def.py`](https://github.com/hongquanli/octopi-research/blob/master/software/control/_def.py) There should be only one `configuration*.txt` file in the software folder. You may edit the [`configuration_example.txt` file](https://github.com/hongquanli/octopi-research/blob/master/software/configuration_example.txt) and rename it.

The following aspects are specified in the configuration file:
- stage movement signs (what is forward vs backward) (e.g. `STAGE_MOVEMENT_SIGN_X`)
- stage motor and lead screw specs (in particular screw pitch, e.g. `SCREW_PITCH_X_MM`)
- whether encoders are used and encoder-related settings (e.g. `USE_ENCODER_X`)
- whether homing is enabled for a particular axis (e.g. `HOMING_ENABLED_X`)
- whether tracking is enabled (`ENABLE_TRACKING`)
- plate reader related definations (`class PLATE_READER`)

## Using the software

### Launching the Spectrometer GUI

**For SiMoRa/Teensy backend (default):**
```bash
python3 main_spectrometer.py
```

**Other available launchers:**
```bash
python3 main.py                    # Main GUI
python3 main_camera_only.py       # Camera-only interface
python3 main_motion_only.py       # Motion control only
```

**Simulation mode (no hardware required):**
```bash
python3 main_spectrometer.py --simulation
# or
python3 main_simulation.py
```

### Teensy/Arduino Controller Compatibility

The software is configured to work with **SiMoRa/Teensy** controllers by default:
- Default controller type: `'Teensy'` (set in `microcontroller.py`)
- Increased timeout: `LAST_COMMAND_ACK_TIMEOUT = 5.0` seconds
- Increased retries: `MAX_RETRY_COUNT = 10`
- Stubbed unsupported commands: `set_axis_enable_disable()` is a no-op for Teensy firmware

**Legacy Arduino Due support:**
If you need to use an Arduino Due, modify the microcontroller initialization:
```python
microcontroller.Microcontroller(version='Arduino Due')
```

### Troubleshooting

**Import errors:**
- Ensure virtual environment is activated
- Verify all packages installed: `pip list`
- Check Python version: `python3 --version` (should be 3.8 or 3.9)

**Serial port errors:**
- Verify Teensy is connected: `ls /dev/tty*` (Linux/macOS) or Device Manager (Windows)
- Check user is in `dialout` group (Linux)
- Try different USB ports or cables

**Camera errors:**
- Verify camera drivers are installed
- Check camera is detected: `python3 tools/list_cameras.py`
- Ensure camera permissions (may need udev rules on Linux)

**GUI not launching:**
- Check Qt installation: `python3 -c "import PyQt5; print(PyQt5.__version__)"`
- Verify display is available (for headless systems, may need X11 forwarding)
