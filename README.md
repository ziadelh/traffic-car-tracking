# Traffic Car Detection, Tracking and Counting

Detecting, tracking and counting cars in traffic-camera footage from Laramie, Wyoming, with classical computer vision only (OpenCV, no machine learning).

<table>
  <tr>
    <td width="50%"><img src="docs/car-tracking.gif" alt="Cars tracked on the main street, each with a green box and an ID"></td>
    <td width="50%"><img src="docs/car-counting.gif" alt="A counter that goes up as cars cross the blue entry gate and then the yellow exit gate"></td>
  </tr>
  <tr>
    <td align="center"><sub>Tracking: a box and an ID on every car on the main street</sub></td>
    <td align="center"><sub>Counting: cars that cross the blue gate and then the yellow gate</sub></td>
  </tr>
</table>

| Notebook | What it does | Run it |
|---|---|---|
| [`car_detection_tracking.ipynb`](car_detection_tracking.ipynb) | Detects and tracks the cars on the main street | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/ziadelh/traffic-car-tracking/blob/main/car_detection_tracking.ipynb) |
| [`car_counting.ipynb`](car_counting.ipynb) | Counts the cars that go from the city centre to downtown | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/ziadelh/traffic-car-tracking/blob/main/car_counting.ipynb) |

## How it works

**Detection.** Frame differencing finds the pixels that changed since the previous frame, and OpenCV's MOG2 background subtraction finds the pixels that do not belong to the learned background. The two masks are combined and cleaned with morphology, and each blob is kept only if its size, aspect ratio, edge density and contrast look like a car. Only the main street is searched.

**Tracking.** Every car gets a Kalman filter that predicts where it will be in the next frame. Detections are matched to tracks by overlap and distance, a track survives a few missed frames, and a new track is only started near the edges of the scene, so each car keeps one ID.

**Counting.** Two virtual gates describe the route from the city centre to downtown: cars come from the left along the main street and cross the **blue gate**, then turn up the side street and cross the **yellow gate**. A car is counted once, when it has crossed the blue gate and then the yellow gate within a time window and has travelled far enough across the frame. A short memory of recently lost tracks lets a dropped track be picked up again.

## Results

The first recording is 2.97 minutes long and the tracker created 34 car tracks in it.

| Recording | Cars counted | Cars per minute |
|---|---|---|
| `Traffic_Laramie_1.mp4` | 6 | 2.02 |
| `Traffic_Laramie_2.mp4` | 0 | 0.00 |

The recordings are short, so I checked the counts by hand: I went through both at one frame per second and marked every car that comes from the left and turns up the side street. The first recording has 6 such cars. The second has none: its cars either carry on east or turn south, and the ones that do go up the side street come from the right.

<img src="docs/counted-cars.png" alt="The six cars counted in the first recording, circled where they cross the yellow gate" width="90%">

### What I corrected

My first version counted 5 and 0. Checking it against the footage showed that 4 of its 5 counts coincided with a real car crossing the yellow gate, 1 was people on the crosswalk, and 2 cars were missed:

| | By hand | Counted | Correct | False | Missed |
|---|---|---|---|---|---|
| First version, recording 1 | 6 | 5 | 4 | 1 | 2 |
| **Corrected, recording 1** | 6 | 6 | **6** | **0** | **0** |
| Both versions, recording 2 | 0 | 0 | 0 | 0 | 0 |

Three changes fixed it, and the detection and tracking did not change:

1. **The direction filter applies only at the blue gate.** It lets through tracks that move mostly sideways, which is right for the blue gate but wrong for the yellow one: a car crossing it is turning up the side street, so it moves steeply and was rejected exactly when it should count. The two missed cars (a pickup and a minivan) were turning at that moment.
2. **The yellow gate starts past the crosswalk** (0.70 of the frame width instead of 0.56), so people and cyclists on the crosswalk are no longer counted. The result is the same for any start between 0.66 and 0.78.
3. **One car, one count.** A car that the tracker split into two tracks used to be able to count twice. Two counts within one second and 100 pixels of each other are now treated as the same car.

## Limitations

- The corrections were made and checked on the same two recordings, so 6 out of 6 shows that the visible errors are gone, not how the counter would do on a different camera or at rush hour.
- The tracker follows most cars on the main street but misses some, especially cars that stop or that merge with another car in the mask (a few can be seen in the tracking clip). This is the usual limit of frame differencing and background subtraction.
- The gates and thresholds are set by hand for this camera angle.
- The natural next step is a trained detector such as YOLO with a tracker such as DeepSORT, which copes with stopped cars and occlusion without hand-tuned thresholds.

## Run it

Click a Colab badge, or run locally:

```bash
pip install -r requirements.txt
jupyter notebook
```

The two recordings are 111 MB and 66 MB, so they are not stored in the repository: the notebooks download them from the [`data-v1` release](https://github.com/ziadelh/traffic-car-tracking/releases/tag/data-v1) the first time they run. Tracking takes about a minute and counting about two minutes.

## Files

| Path | Purpose |
|---|---|
| `car_detection_tracking.ipynb` | Detection and tracking, with the results saved in the notebook |
| `car_counting.ipynb` | The counter, the check against the hand count and the counted cars |
| `results/` | The counting results as text |
| `docs/` | The clips and pictures used in this README |

## Credits

The two recordings come from the Historic Downtown Laramie web camera (visitlaramie.org) and were supplied with the course.

## Tech Stack

Python · OpenCV · NumPy · pandas · Matplotlib

## Author

Ziad Elhussein
