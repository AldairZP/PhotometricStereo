# PhotometricStereo

OpenCV-based tools to generate normal maps using the Photometric Stereo technique.

This guide explains how to install dependencies on Linux, compile the C++ binaries, prepare images, and run the full pipeline (perspective transform, crop, albedo, and normals).

## Requirements

- Linux (tested on Ubuntu 24.04).
- C++ compiler (`g++`).
- `pkg-config` to resolve OpenCV flags.
- OpenCV C++ (`libopencv-dev`).
- Python 3 and `opencv-python` for interactive scripts.
- Active X11 graphical session for windowed scripts (`transform.py` and `crop.py`).

## Installation (Ubuntu/Debian)

Run in a terminal:

```zsh
sudo apt update
sudo apt install -y build-essential pkg-config libopencv-dev python3-pip
python3 -m pip install --user opencv-python
```

Verify `pkg-config` sees OpenCV 4:

```zsh
pkg-config --cflags --libs opencv4
```

If your distro exposes `opencv` instead of `opencv4`, replace `opencv4` with `opencv` in commands.

## Build

From the project root:

```zsh
cd /home/[user]/Projects/photometric/PhotometricStereo

# Build main binaries
g++ albedo.cpp -o albedo $(pkg-config --cflags --libs opencv4)
g++ normal.cpp -o normal $(pkg-config --cflags --libs opencv4)

# (Optional) Additional utilities if needed
g++ calibrate.cpp -o calibrate $(pkg-config --cflags --libs opencv4)
g++ getHighlights.cpp -o getHighlights $(pkg-config --cflags --libs opencv4)
```

This produces `albedo`, `normal` (and optional tools) in the project folder.

## Image Preparation

For each scene, place 3 photos of the same subject (fixed camera, varying light) in a folder using these names:

- `yourFolder/_1.jpg`
- `yourFolder/_2.jpg`
- `yourFolder/_3.jpg`

The repo includes sample folders under `Images/` (e.g., `Images/art/`).

## Workflow

Recommended order for best results:

1. Albedo (pixel-wise average of the 3 images).
2. Perspective Transform (straighten) — requires GUI.
3. Crop — requires GUI.
4. Normals.

### 1) Albedo

```zsh
./albedo Images/art/
```

Output: `Images/art/albedo.jpg` (average of `_1/_2/_3`).

### 2) Perspective Transform (GUI)

Requires X11 graphical session. On desktop Linux, run from a normal terminal. Over SSH, use X forwarding (`ssh -X`) with an X server on the client.

```zsh
python3 transform.py Images/art/
```

Instructions:
- A window shows `albedo.jpg` (or `_1.jpg` if albedo is missing).
- Click four corners in order: bottom-left, top-left, top-right, bottom-right.
- Press `Esc` to finish.

Outputs (per script):
- Copies `original_*.jpg`, transformed `affine_*.jpg`, and `affine_albedo.jpg` in the same folder.

Note: if you see `qt.qpa.xcb: could not connect to display`, run in a graphical session or with `ssh -X`. Offscreen mode will not allow interactive clicks.

### 3) Crop (GUI)

```zsh
python3 crop.py Images/art/
```

Instructions:
- Draw a rectangle over the subject (top-left → bottom-right).
- Press `Esc` to finish.

### 4) Normals

Build if you haven’t yet:

```zsh
g++ normal.cpp -o normal $(pkg-config --cflags --libs opencv4)
```

Run:

```zsh
./normal Images/art/ 20 500
```

Parameters:
- `threshold` (int): minimum grayscale intensity to ignore shadows (e.g., `20`).
- `iterations` (int): iterations for simulated annealing (commonly `200`–`7000`).

Output: normal map (e.g., `normal_20_500.jpg`) in the folder.

## Quick End-to-End Example

```zsh
cd /home/[user]/Projects/photometric/PhotometricStereo

# Build
g++ albedo.cpp -o albedo $(pkg-config --cflags --libs opencv4)
g++ normal.cpp -o normal $(pkg-config --cflags --libs opencv4)

# Albedo
./albedo Images/Aintwet/

# Perspective (graphical session)
python3 transform.py Images/Aintwet/

# Crop
python3 crop.py Images/Aintwet/

# Normals
./normal Images/Aintwet/ 20 500
```

## Troubleshooting

- `pkg-config: command not found` → install `pkg-config`:
	```zsh
	sudo apt install -y pkg-config
	```
- `fatal error: opencv2/...: No such file or directory` → install OpenCV dev:
	```zsh
	sudo apt install -y libopencv-dev
	```
- `qt.qpa.xcb: could not connect to display` when running GUI scripts → ensure `DISPLAY` is active (graphical session) or use `ssh -X`.
- `imread(...): can't open/read file` → check paths and names (`_1.jpg`, `_2.jpg`, `_3.jpg`, folder ends with `/`).

## Visualization & Examples

<img src="./Examples/_1.jpg" width="30%"> <img src="./Examples/_2.jpg" width="30%"> <img src="./Examples/_3.jpg" width="30%">
> Three source images; subject and camera are fixed; lighting varies.

<img src="./Examples/final_1.jpg" width="30%"> <img src="./Examples/final_2.jpg" width="30%"> <img src="./Examples/final_3.jpg" width="30%">
> Affine adjustments and cropping for straighter images.

<img src="./Examples/demonew.gif" width="50%">
> Visualization of estimated light positions.

<img src="./Examples/normal_20_500.jpg" width="40%"> <img src="./Examples/albedo.jpg" width="40%">
> Generated normal map alongside albedo.

<img src="./Examples/demo3.gif" width="40%"> <img src="./Examples/demo2.gif" width="40%">
> Map used via A-FRAME (three.js).

