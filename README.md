# Real-Time Object Tracking using Classical Computer Vision (OpenCV)

**Author:** Aditya Chahar  |  **Reg. No.:** 24BAI10371  |  **Course:** Computer Vision

A collection of six command-line Python programs that detect and track objects in a live camera feed using classical (non-deep-learning) computer vision techniques implemented with OpenCV: colour thresholding, contour tracking, MeanShift, CamShift, Lucas-Kanade sparse optical flow and Farneback dense optical flow.

Every program reads frames from a webcam, processes them frame by frame, draws the tracking result on the frame, and shows it in a window until the user presses **q**.

## Programs in this repository

| File | Technique | What it does |
| --- | --- | --- |
| `trackColor.py` | HSV colour thresholding | Shows the original frame, the binary mask and the pixels that fall inside an HSV range. |
| `trackObject.py` | Colour mask + contours + image moments | Finds the largest blob in the HSV range, draws its enclosing circle and centroid, and draws a movement trail. |
| `meanshiftOT.py` | Histogram back-projection + MeanShift | Tracks a region using its hue histogram and draws an axis-aligned rectangle. |
| `camshiftOT.py` | Histogram back-projection + CamShift | Same idea as MeanShift, but the window adapts its size and rotation (rotated box). |
| `kanadeOpticalFlowOT.py` | Shi-Tomasi corners + Lucas-Kanade optical flow | Tracks up to 100 feature points and draws their trails. |
| `denseOpticalFlow.py` | Farneback dense optical flow | Colour-codes the direction (hue) and speed (brightness) of motion for every pixel. |

## Tech stack

- Python 3.8 or newer
- OpenCV (`opencv-python`)
- NumPy (1.x recommended, see "Known issues")

## Prerequisites

- A working webcam (the programs open device `0`).
- A desktop session that can display OpenCV windows (`cv2.imshow`). See "Running without a webcam or display" if you do not have these.
- Python and `pip` available in your terminal. Check with:

  ```bash
  python --version
  pip --version
  ```

## Setup (step by step)

1. **Clone the repository**

   ```bash
   git clone https://github.com/<your-github-username>/<repo-name>.git
   cd <repo-name>
   ```

2. **Create and activate a virtual environment**

   Linux / macOS:

   ```bash
   python -m venv venv
   source venv/bin/activate
   ```

   Windows (Command Prompt):

   ```bash
   python -m venv venv
   venv\Scripts\activate
   ```

3. **Install the dependencies**

   ```bash
   pip install -r requirements.txt
   ```

4. **Check the installation**

   ```bash
   python -c "import cv2, numpy; print(cv2.__version__, numpy.__version__)"
   ```

   This should print the two version numbers without an error.

## Running the programs

Run any program from the repository root:

```bash
python trackColor.py
python trackObject.py
python meanshiftOT.py
python camshiftOT.py
python kanadeOpticalFlowOT.py
python denseOpticalFlow.py
```

- Click on the video window and press **q** to quit. The camera is released and the windows are closed.
- `trackColor.py`, `trackObject.py`, `meanshiftOT.py` and `camshiftOT.py` depend on an HSV colour range that you must set for your own object (next section).
- `kanadeOpticalFlowOT.py` and `denseOpticalFlow.py` need no configuration. Move an object (or your hand) in front of the camera and watch the trails / colours.

### Configuring the colour range

The colour-based programs contain a variable named `color_range` in the form `[[H_min, S_min, V_min], [H_max, S_max, V_max]]` (OpenCV ranges: H 0-179, S 0-255, V 0-255).

| File | Value in the code | Meaning |
| --- | --- | --- |
| `trackColor.py` | `[[0,0,0],[0,0,255]]` | Placeholder range. Edit it to the HSV range of your object. |
| `trackObject.py` | `[[0,0,0],[179,50,100]]` | Dark, low-saturation objects (black / dark grey). |
| `meanshiftOT.py` | `[[0,0,128],[0,0,255]]` | Bright, unsaturated (white-ish) pixels inside the start window. |
| `camshiftOT.py` | `[[0,0,0],[0,0,0]]` | Placeholder that selects no pixels. **You must edit it** or the tracker has nothing to lock on to. |

To find the range for your object, take a frame, convert it to HSV, and read the H, S and V values of the object's pixels. Then widen the range slightly on each side.

### Initial tracking window (MeanShift and CamShift)

`track = (240, 100, 400, 160)` is the starting window. Hold the object you want to follow inside that region when the program starts. The histogram of that region is what gets tracked afterwards.

### Running without a webcam or display

- **Video file instead of a webcam:** change `cv2.VideoCapture(0)` to `cv2.VideoCapture("path/to/video.mp4")` in the program you want to run.
- **Server / headless machine:** run under a virtual display, for example `xvfb-run -a python trackObject.py` (Linux, requires `xvfb`), or use a machine with a desktop session.

## Repository structure

```
.
├── trackColor.py
├── trackObject.py
├── meanshiftOT.py
├── camshiftOT.py
├── kanadeOpticalFlowOT.py
├── denseOpticalFlow.py
├── requirements.txt
├── README.md
└── Project_Report.docx
```

## How each technique works

- **Colour thresholding** converts each frame from BGR to HSV and keeps only the pixels inside `color_range` with `cv2.inRange`. A bitwise AND with the frame shows just those pixels.
- **Contour tracking** turns the mask into contours, picks the largest by area, and computes its enclosing circle and its centroid from image moments. Blobs with a radius of 25 pixels or less are ignored. The centroids are stored in a list and drawn as a trail; the trail is cleared after 10 frames without a detected object.
- **MeanShift** builds a hue histogram of the starting window, back-projects it onto each new frame to get a probability image, and moves the window towards the densest region (`cv2.meanShift`).
- **CamShift** does the same but with `cv2.CamShift`, so the window also changes size and orientation. The result is drawn with `cv2.boxPoints` and `cv2.polylines`.
- **Lucas-Kanade** finds strong corners in the first frame (`cv2.goodFeaturesToTrack`) and follows them from frame to frame (`cv2.calcOpticalFlowPyrLK`), drawing a coloured line between the old and new position of each point.
- **Farneback** computes a motion vector for every pixel (`cv2.calcOpticalFlowFarneback`). The vectors are converted to polar form; the angle becomes hue and the magnitude becomes brightness.

## Known issues and compatibility notes

These are limitations of the current code. Depending on your OpenCV and NumPy versions you may need the small changes listed here.

| File | Issue | Fix |
| --- | --- | --- |
| `trackObject.py` | `_, contours, _ = cv2.findContours(...)` is the OpenCV 3 return format. On OpenCV 4 it raises a `ValueError`. | Use `contours, _ = cv2.findContours(...)` on OpenCV 4, or install OpenCV 3.x. |
| `camshiftOT.py` | `np.int0` was removed in NumPy 2.0. | Use NumPy 1.x (`numpy<2`, as in `requirements.txt`) or replace `np.int0(points)` with `points.astype(int)`. |
| `kanadeOpticalFlowOT.py` | Newer OpenCV versions reject float coordinates in `cv2.line` / `cv2.circle`. If every tracked point is lost, the next call also fails. | Convert the coordinates with `int(...)`; re-detect corners when no points remain. |
| `denseOpticalFlow.py` | The hue scaling `theta * (180 / (np.pi/2))` goes beyond OpenCV's 0-179 hue range, so colours wrap around. | Use `theta * 180 / np.pi / 2`. |
| `meanshiftOT.py`, `camshiftOT.py` | The start window is used as `(x, y, w, h)` by the tracker, but the crop indexes it as `[row, col]`, so the histogram region and the tracking window are not the same area. | Use one consistent convention for both. |
| `trackColor.py`, `trackObject.py`, `denseOpticalFlow.py`, `kanadeOpticalFlowOT.py` | The read result is not checked, so the program stops with an error if the camera is missing or a video file ends. | Check `if not response: break` after `capture.read()`. |
| Colour trackers | Tracking depends only on colour, so similarly coloured background objects and strong lighting changes can confuse the result. | Use a distinctive object colour and steady lighting. |

## License

Submitted as coursework for the Computer Vision course. All rights reserved by the author.
