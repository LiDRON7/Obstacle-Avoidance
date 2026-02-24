# OAK-D Pro Setup Guide (Windows 10 / 11)

Note: This guide explains how to properly install and configure OAK-D Pro cameras on Windows, including:

Topics:
1) Installing required dependencies
2) Fixing Python compatibility issues
3) Creating a virtual environment
4) Installing DepthAI
5) Installing OAK Viewer
6) Testing camera streams 

---------------------------------------

# 1) Installing required dependencies: 

Download and install: 

1) Python 3.11 (recommended version)
2) USB 3.0 port
3) USB-C data cable
4) Check Add Python to PATH
5) Ensure pip is selected

Download Python 3.11:

https://www.python.org/downloads/release/python-3119/

Verify installation in Command Prompt:

```bash
py --version
```

The following command line should appear: 

```bash
Python 3.11.x
```

# 2) Fixing Python Compatibility Issue 

DepthAI works best with Python 3.11. As of February 2026, newer versions of Python like Python 3.13.x, do not work since many of the libraries like OpenCV, DepthAI and NumPy haven't been published.

Remove old environments (if any):

```bash
rmdir /s /q oak_env
```
Create a new virtual environment:

```bash
py -3.11 -m venv oak_env
```
Activate the environment: 

```bash
oak_env\Scripts\activate
```

# 3) Upgrading Packaging Tools

Upgrade pip and build tools:

```bash
python -m pip install --upgrade pip setuptools wheel
```

# 4) Install DepthAI and OpenCV

Install required libraries:

```bash
pip install depthai opencv-python
```
Verify installation:

```bash
python -c "import depthai; print('DepthAI installed successfully')"
```

If no errors appear, installation is successful.

# 5) Install OAK Viewer

Install inside the virtual environment:

```bash
pip install depthai-viewer
```
Run:

```bash
depthai-viewer
```

If there are dependency conflicts, fix them using:

```bash
pip uninstall -y numpy opencv-python opencv-contrib-python
pip install numpy==1.26.4
pip install opencv-python==4.9.0.80
pip install depthai-viewer
```

# 6) Fix ModuleNotFoundError: No module named 'depthai' or 'cv2'

This usually means:

Virtual environment is not activated
Wrong Python version is being used

Fix:

```bash
oak_env\Scripts\activate
pip install depthai opencv-python
```

# 7) Testing Camera Streams 

If using the DepthAI repository:

Navigate to the examples folder:

```bash
cd depthai-core\examples\python
```

Run camera test:

```bash
python Camera\camera_all.py
```

You should see:

1) CAM_A (RGB camera)
2) CAM_B (Left mono)
3) CAM_C (Right mono)

Stop execution with (Ctrl + C).

# 8) Confirm Setup

Your setup is complete if: 

1) camera_all.py runs without errors
2) OAK Viewer connects successfully
3) Device appears in getAllAvailableDevices()
4) detection_network.py runs

To check if detection_network.py runs, use this command line: 

```bash
python DetectionNetwork\detection_network.py
```

---------------------------------------


# OAK-D Pro Setup Guide (Fedora Linux)

Note: This guide explains how to properly install and configure OAK-D Pro cameras on Fedora Linux, including:

Topics: 
1) Installing required dependencies
2) Fixing Python compatibility issues
3) Installing OAK Viewer
5) Fixing USB permission errors
6) Fixing X_LINK_DEVICE_NOT_FOUND
7) Creating a desktop shortcut

---------------------------------------

# 1) Installing required dependencies through the terminal:

```bash
sudo dnf update 
sudo dnf install
    python3.11 python3.11-devel 
    python3-pip python3-virtualenv 
    git usbutils libusb1-devel 
    qt5-qtbase qt5-qtmultimedia qt5-qtdeclarative 
    mesa-libGLU
```

# 2) fixing python compatibility (some feautres are only available in python 3.11)

- Removing possible enviorments:

```bash
rm -rf oakviewer_env
```

- Creating a new virtual enviorment (to test dependencies)

```bash
python3.11 -m venv oakviewer_env
```

- Run the enviorment

```bash
source oakviewer_env/bin/activate
```

- Verify version of python to see if you habe the 3.11

```bash
python --version
# you wiil see someyhing like
Python 3.11.x
```
# 3) Upgrading Packing Tools (So it installs correctly)

```bash
pip install --upgrade pip setuptools wheel
```
# 4) Install OAK viewer

```bash
pip install depthai-viewer
```
if there are dependencies conflics use this commad to fix them: 

```bash
pip uninstall -y numpy opencv-python opencv-contrib-python
pip install numpy==1.26.4
pip install opencv-python==4.9.0.80
pip install depthai-viewer
```

Now In case of Errors you can do the following depending on what appears. This is not a 200% fix all this is just the erros that the team encountred during setup phase. 

# 5) Fixing USB permission errors

if you see: Insufficient permissions to communicate with X_LINK_UNBOOTED

We fix it by: 

1) creating udev rules

```bash
sudo nano /etc/udev/rules.d/80-movidius.rules
```
Then add:

This basically overrides USB permission in order to be regonized in the OAK-Viewer 

```bash
SUBSYSTEM=="usb", ATTRS{idVendor}=="03e7", MODE="0666"
SUBSYSTEM=="usb", ATTRS{idVendor}=="03e7", ATTRS{idProduct}=="f63b", MODE="0666"
SUBSYSTEM=="usb", ATTRS{idVendor}=="03e7", ATTRS{idProduct}=="2485", MODE="0666"
```

Reload rules (just in case):

```bash
sudo udevadm control --reload-rules
sudo udevadm trigger
```

Add user to plugdev group:

```bash
sudo groupadd -f plugdev
sudo usermod -aG plugdev $USER
```

Now just: 

Log out and log back in.
Unplug and replug the camera.

# 6. Fix X_LINK_DEVICE_NOT_FOUND Error

If you see:

Failed to find device after booting
X_LINK_DEVICE_NOT_FOUND

This is usually caused by USB autosuspend or USB3 instability.

Disable USB Autosuspend (Temporary)

```bash
echo -1 | sudo tee /sys/module/usbcore/parameters/autosuspend
```

Disable USB Autosuspend (Permanent)

Edit grub:

```bash
sudo nano /etc/default/grub
```
Change this line from: 

GRUB_CMDLINE_LINUX=

To this: 
GRUB_CMDLINE_LINUX="usbcore.autosuspend=-1"

Update grub: 

```bash
sudo grub2-mkconfig -o /boot/grub2/grub.cfg
```

Test device (just in case)

```bash
lsusb | grep 03e7
```
Test with the framework directly: 

```bash
python -m depthai
```

6.2) Launch OAK-Viewer 

```bash
source ~/oakviewer_env/bin/activate
```

run: 

```bash
depthai-viewer
```

# 7) Creating Desktop Shortcut:

Create launcher:

```bash
nano ~/.local/share/applications/oak-viewer.desktop
```

In nano paste this:

[Desktop Entry]
Version=1.0
Type=Application
Name=OAK Viewer
Comment=Luxonis DepthAI Viewer
Exec=bash -c "source /home/$USER/oakviewer_env/bin/activate && depthai-viewer"
Icon=camera-video
Terminal=false
Categories=Development;Graphics;
StartupNotify=true

Then in bash make it an excecutable: 

```bash
chmod +x ~/.local/share/applications/oak-viewer.desktop
```

Refresh:

update-desktop-database ~/.local/share/applications


