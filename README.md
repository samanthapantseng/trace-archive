# Trace Archive - Hand Trail Gesture Tracker

A project that captures hand movements during conversations and exports them as plottable SVG files using the senseSpace framework.

## Project Location

**Important:** This project must remain in its current nested location:

```
senseSpace/
  └── students/
      └── trace_archive/
          └── gesture/
              ├── main.py
              ├── calibration.py
              ├── plot_latest.sh
              ├── detectors/
              │   └── trail.py
              ├── recordings/
              ├── calibrations/
              └── trails/
```

The code relies on relative imports from the parent `senseSpace` library. **Do not move this folder** outside the `senseSpace/students/` directory, or the imports will break.

## Project Evolution

This project has evolved significantly over time. While you'll see references to multiple gesture detectors and sound integration in the codebase, **the current implementation only uses the trail detector** ([`gesture/detectors/trail.py`](gesture/detectors/trail.py)).

The active workflow in [`gesture/main.py`](gesture/main.py) focuses exclusively on:

-   Hand trail tracking
-   SVG export
-   Plotter integration

Other features (sound, additional gesture detectors) are legacy code and not currently active.

## Concept: Conversations in Trace

The core idea is to capture the **physical traces of conversation** between people. As people talk, their hands move - gesturing, emphasizing, reaching out. This system:

1. **Tracks both hands** of all people in the camera view
2. **Records their movements** over a configurable duration (default: 30 seconds)
3. **Projects the 3D hand positions** onto a 2D vertical plane (wall view)
4. **Exports individual SVG files** for each person's left and right hand
5. **Plots the traces** on paper using an AxiDraw plotter

The result is a physical, plottable record of how people's hands moved during their conversation - a visual/tactile archive of gestural communication.

## Complete Workflow

### Step 1: Run the Tracker

```bash
cd senseSpace/students/trace_archive/gesture
python main.py --rec 'recordings/your_recording.ssrec' --calibration your_calibration
```

**What happens:**

-   Opens 3D visualization showing skeleton tracking
-   Records hand movements for **30 seconds** (configurable in [`detectors/trail.py`](gesture/detectors/trail.py) line 30)
-   Tracks both left and right hands of all people in view
-   Automatically exports SVG files when complete

**Live camera mode** (if you have ZED cameras):

```bash
python main.py --server server_ip --calibration your_calibration
```

### Step 2: Generated Files

After the recording duration completes, files are automatically saved to:

```
gesture/trails/trail_YYYYMMDD_HHMMSS/
├── all_trails_YYYYMMDD_HHMMSS.svg        # Preview of all hands (DO NOT PLOT)
├── person_0_left_YYYYMMDD_HHMMSS.svg     # Individual hand trail (PLOT THIS)
├── person_0_right_YYYYMMDD_HHMMSS.svg    # Individual hand trail (PLOT THIS)
├── person_1_left_YYYYMMDD_HHMMSS.svg     # If multiple people detected
└── person_1_right_YYYYMMDD_HHMMSS.svg
```

### Step 3: Plot the Trails

```bash
./plot_latest.sh
```

**What happens:**

1. Script finds the most recent `trails/trail_*` folder
2. Opens the first individual hand SVG in Inkscape
3. **You manually:** Go to Extensions > iDraw 2.0 Control > Apply
4. Wait for plotting to complete
5. Press **Enter** in the terminal
6. Next file opens automatically
7. Repeat for each hand

**Why manual plotting?**  
Inkscape's command-line interface crashes on macOS when trying to automate the iDraw extension. Manual triggering is the only reliable method.

## Key Files

-   **[`gesture/main.py`](gesture/main.py)** - Main application entry point (live camera or playback)
-   **[`gesture/detectors/trail.py`](gesture/detectors/trail.py)** - Core hand tracking and SVG export logic
-   **[`gesture/calibration.py`](gesture/calibration.py)** - Tool to define floor polygon for filtering
-   **[`gesture/plot_latest.sh`](gesture/plot_latest.sh)** - Sequentially opens SVG files for plotting
-   **`gesture/recordings/`** - Pre-recorded ZED camera files (`.ssrec`)
-   **`gesture/calibrations/`** - Saved calibration polygons (`.json`)
-   **`gesture/trails/`** - Exported SVG files organized by timestamp

## Configuration

### Recording Duration

Edit [`gesture/detectors/trail.py`](gesture/detectors/trail.py) line 30:

```python
self._duration_seconds = 30  # Change recording length
```

### Export Scale

Edit [`gesture/detectors/trail.py`](gesture/detectors/trail.py) lines 261-262:

```python
max_gesture_height_mm = 2000  # Adjust scale
max_gesture_width_mm = 2000
```

Fixed scale ensures all exports are comparable - the same real-world gesture size results in the same plotted size.

## Output Details

-   **Format:** A5 landscape (210mm × 148mm)
-   **Scale:** Fixed at 2000mm height × 2000mm width (typical arm movement range)
-   **Duration:** Configurable (default 30 seconds)
-   **Files:** Separate SVG for each person's left and right hand

## Tips

-   **Preview first:** Open `all_trails_*.svg` to see the combined visualization before plotting
-   **Multiple people:** Each person gets a unique color and separate SVG files
-   **Fixed scale:** All sessions use the same scale, making gestures comparable
-   **Individual plotting:** Plot each hand separately for cleaner results

## Dependencies

This project requires the senseSpace library and its dependencies:

-   PyQt5 (OpenGL visualization)
-   ZED SDK (for live cameras)
-   svgwrite (SVG generation)
-   See parent [`senseSpace/pyproject.toml`](../../pyproject.toml) for full list

## Troubleshooting

### Import Errors

If you see `ModuleNotFoundError: No module named 'senseSpaceLib'`:

-   Verify you're running from the correct nested location
-   Ensure `senseSpace/libs/` exists in the parent directory
-   Check that `senseSpaceLib` is installed: `cd ../../libs/senseSpaceLib && pip install -e .`

### No Hand Tracking

-   Verify skeleton data is present in your recording
-   Check calibration polygon includes the area where people are standing
-   Adjust confidence thresholds if needed

### SVG Not Exporting

-   Ensure recording duration has elapsed
-   Check write permissions in `trails/` directory
-   Look for error messages in terminal output

### AxiDraw Not Detected ("Failed to connect to iDraw2")

If you see errors like `error open com_port: None` or `Failed to connect to iDraw2`:

1. **Check connections:**

    - AxiDraw is plugged into Mac via USB
    - AxiDraw is powered on
    - Try different USB cable/port

2. **Check USB device:**

    ```bash
    ls -l /dev/cu.*
    ```

    Look for `/dev/cu.usbmodem*` or `/dev/cu.usbserial*`

3. **Restart sequence:**

    - Unplug USB from AxiDraw
    - Power off AxiDraw
    - Wait 10 seconds
    - Power on AxiDraw
    - Reconnect USB cable

4. **Check in Inkscape:**

    - Extensions > iDraw 2.0 Control
    - Look in Setup tab for detected port

5. **Install drivers (if needed):**
    - Download FTDI VCP drivers: https://ftdichip.com/drivers/vcp-drivers/
    - Restart Mac after installation

### SVG Files Empty or Incorrect

-   Verify hands were visible during recording in the visualization window
-   Check calibration polygon includes the interaction area
-   Ensure recording duration completed (30 seconds default)

## Related Documentation

-   [senseSpace README](../../README.md) - Main project documentation
-   [Client Examples](../../client/examples/) - Other senseSpace examples
-   [Recording Guide](../../RECORDING_IMPLEMENTATION.md) - How to record `.ssrec` files
