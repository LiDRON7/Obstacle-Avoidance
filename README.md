## OAK-D Pro Setup Guide (Fedora Linux)

# This guide explains how to properly install and configure OAK-D Pro cameras on Fedora Linux, including:

Topics: 
1) Installing required dependencies
2)Fixing Python compatibility issues if Any
3) Installing OAK Viewer

In case of Erros (Most commun) 
5) Fixing USB permission errors
6) Fixing X_LINK_DEVICE_NOT_FOUND
7) Creating a desktop shortcut

---------------------------------------

1) Installing required dependencies through the terminal:

```python
sudo dnf update -y
sudo dnf install -y \
    python3.11 python3.11-devel \
    python3-pip python3-virtualenv \
    git usbutils libusb1-devel \
    qt5-qtbase qt5-qtmultimedia qt5-qtdeclarative \
    mesa-libGLU
'''
