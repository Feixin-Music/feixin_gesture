# Feixin Gesture

A gesture-controlled music project built with **Max/MSP, MediaPipe, and Rhythm and Time Toolkit (RTT)**. It uses a camera to track both hands and translates gestures and hand movements into MIDI notes, rhythmic changes, and continuous control messages for live musical interaction and improvisation.

The Max patches combine hand tracking with generative sequencing across keys, drums, and bass. A separate DAW project receives the MIDI output and produces the music.

> **A DAW project with instruments is required to produce sound.** The original setup uses a **four-track Ableton Live Session** as its sound-generating project. The supplied Max project does not include this Live Session, instruments, or samples; you will need to provide the DAW project separately or build your own receiving setup.

## Features

- **Two-hand tracking:** Detects up to two hands, with 21 landmarks per hand and gesture classification.
- **Custom OK gesture:** Uses distances between hand landmarks to detect an OK gesture inside Max.
- **Three MIDI sequence parts:** Independently toggle keys, drums, and bass.
- **Rhythm generation:** Combines randomized patterns with RTT's `rtt.euclidean~` to generate 32-step drum sequences.
- **Continuous control:** Open-palm movement modulates MIDI CC output.
- **Visual feedback:** Displays the camera image, hand skeletons, and control states. The recognition patch also supports JPG and PNG images.

## Signal Flow

```text
Camera / Image
      ↓
jweb + MediaPipe
      ↓
Hand landmarks, gesture classification, and custom OK detection
      ↓
Max gesture mapping and sequence generation
      ↓
MIDI notes / CC
      ↓
DAW project with instruments → Audio output
(original setup: four-track Ableton Live Session)
```

## Requirements

| Component | Details |
| --- | --- |
| Max | Both patches were saved in **Max 9.0.5**. Use this version or later; other versions have not been verified. |
| Rhythm and Time Toolkit | Required for the `rtt.euclidean~` object used in the sequencing patch. |
| Camera | Live control requires an available video input and camera permission. |
| Internet connection | The web page loads MediaPipe scripts and WASM from jsDelivr, and the recognition model from Google Storage. |
| DAW project with instruments | Required to receive Max's MIDI notes and generate audio. The original project uses a four-track Ableton Live Session. |
| MIDI connection | An available MIDI route from Max to the DAW. The patches currently target `from Max 1`. |

Install **Rhythm and Time Toolkit** through Max's **File → Show Package Manager**. See the [upstream installation instructions](https://github.com/pdmeyer/rhythm-and-time-toolkit#installation) for details.

The JavaScript uses `@mediapipe/tasks-vision@0.10.0` with the recognizer configured for GPU processing. Offline copies of the model and all web dependencies are not bundled.

## Getting Started

1. Download and extract the project, preserving the relative locations of the Max patches, HTML file, `js/`, and `css/` folders.
2. Install RTT, then open both patches in the same Max application instance: `jweb-hands-gesture-recognizer.maxpat` for camera input and recognition, and `gesture system.maxpat` for gesture mapping, sequencing, and MIDI output.
3. Allow camera access and wait for the web dependencies and model to load. To change cameras, refresh the device list in the recognition patch and select an input.
4. Enable DSP in Max's audio settings. The palm-modulation modules use signal objects such as `cycle~` and `snapshot~`, which require DSP to run.
5. Open the separate DAW project containing your instruments. The original performance setup uses a four-track Ableton Live Session; this session must be supplied separately if you want to reproduce that setup.
6. Route Max's MIDI output to the DAW. The patches target **`from Max 1`** by default. If that port is unavailable, update each `midiout` object to use an available output port.
7. Configure the receiving instruments for the MIDI channels below, and enable input monitoring so incoming notes can be heard.
8. Open Presentation Mode in `gesture system.maxpat`, set the BPM, and test the keys / drum / bass switches before trying gesture control.

## DAW and MIDI Setup

Max generates the musical control data; the receiving DAW project supplies the instruments, effects, mixing, and audio output.

| MIDI channel | Output |
| --- | --- |
| 1 | Keys notes, plus CC 1 and CC 10 control messages |
| 2 | Drum notes |
| 3 | Bass notes |

The original Ableton Live Session contains **four tracks**, while the supplied Max patches expose the **three MIDI note channels** listed above. The fourth track's role, instrument assignments, effects, and complete track routing are not specified in the supplied files. Use the original Live Session's routing if available, or configure a receiving project around these MIDI outputs.

For a replacement DAW project, load instruments for the keys, drum, and bass parts, route the relevant MIDI channels to them, and map CC messages to the parameters you want to control. The resulting sound depends on your instruments and mappings.

**Drums are sent on channel 2.** Set the drum instrument to receive that channel and adapt its note mapping to the patch as needed.

## Gesture Controls

The following mappings reflect the connections in the current Max patches.

| Gesture | Label | Action |
| --- | --- | --- |
| Index finger pointing up | `Pointing_Up` | Toggles keys sequence playback. |
| V sign | `Victory` | Toggles drum sequence playback. |
| OK sign | `OK` | Toggles bass sequence playback. |
| Thumbs up | `Thumb_Up` | Randomizes the event count for the Kicks Euclidean pattern. |
| Thumbs down | `Thumb_Down` | Randomizes the event count for the Snare & clap Euclidean pattern. |
| Open palm | `Open_Palm` | Enables palm-position modulation: left hand controls CC 1; right hand controls CC 10. |
| Closed fist | `Closed_Fist` | Rearms gesture triggers so an action can be performed again. |

### Performance Notes

- Discrete gesture triggers are gated by the detected hand count. The current design requires both hands to be detected, so keep both hands in view.
- Holding a gesture does not continuously retrigger its action. To repeat an action, make a fist first, then perform the target gesture again.
- Keys, drums, and bass use toggled playback states and can continue playing after a gesture. Use their switches to stop playback rather than relying on removing your hands from view.
- Palm movement modulates time-varying CC output. Its audible effect depends on the receiving DAW's parameter mappings.
- Adjust tempo using the BPM field and the `speed up` / `speed down` interface controls. Thumb gestures change rhythm event counts.
- `OK` is detected through geometric calculations in Max rather than a default MediaPipe gesture category. Camera distance, hand angle, and detection thresholds can affect recognition.

## Project Structure

```text
feixin_gesture/
├── README.md
├── gesture system.maxpat                 # Sequencing, gesture mapping, MIDI output
├── jweb-hands-gesture-recognizer.maxpat   # Hand tracking, landmark parsing, OK detection
├── jweb-hands-gesture-recognizer.html     # Web entry point loaded by jweb
├── css/
│   └── mesh-style.css                    # Image and overlay styling
└── js/
    ├── hands-landmarks-index.js          # Hand landmark indices
    └── jweb-hands-gesture-recognizer.js   # MediaPipe setup, recognition, Max communication
```

The DAW project is separate and is not included in this file structure.

The patches communicate through named Max message channels, including `left_gest`, `right_gest`, `left_xy`, `right_xy`, and `numhands`. Both patches must run in the same Max instance. Opening multiple copies may cause messages to interfere with one another.

## Troubleshooting

### The web view is blank or the camera image is missing

Check camera permissions, the selected input device, and connectivity to `cdn.jsdelivr.net` and `storage.googleapis.com`. Use `reload` in the recognition patch and inspect the Max Console for `jweb` errors. If the local HTML file cannot be found, check its location and, if necessary, provide its absolute file URL to `jweb`.

### Max cannot find `rtt.euclidean~`

Confirm that RTT is installed and available to Max, then reopen the sequencing patch.

### Gestures are recognized, but there is no sound

Open the separate DAW project and confirm that instruments are loaded. Check the MIDI output port, receiving channels, input monitoring, and DAW audio output. Use the interface switches to verify that the sequences are running. The drum instrument must receive channel 2.

### Gestures do not trigger, or an action cannot be repeated

Keep both hands visible, make a fist, then perform the target gesture. Check the left/right labels in the recognition patch and adjust `flip_hands` / `flip_image` as needed.

### Moving an open palm does not change the sound

Confirm that the gesture is recognized as `Open_Palm`, Max DSP is enabled, and the DAW has parameters mapped to CC 1 / CC 10 on channel 1.

## Sources and Acknowledgments

Parts of this project's code and implementation are derived from the following repositories and have been integrated and extended for gesture-controlled music:

| Source project | Author / maintainer | Use in this project |
| --- | --- | --- |
| [rhythm-and-time-toolkit](https://github.com/pdmeyer/rhythm-and-time-toolkit) | Philip Meyer / `pdmeyer` | Rhythm and sequencing functionality, including the `rtt.euclidean~` Euclidean rhythm object. |
| [jweb-hands-gesture-recognizer](https://github.com/lysdexic-audio/jweb-hands-gesture-recognizer) | `lysdexic-audio` | The foundation for hand tracking, gesture recognition, and communication between MediaPipe and Max's `jweb`. |

The integrated project includes custom OK detection, two-hand control mappings, linked keys / drum / bass sequencing, and palm-position modulation of MIDI CC. Many thanks to the original authors for their work.

### License Notes

Both upstream repositories identify their licenses as **GPL-3.0**. See the [RTT license](https://github.com/pdmeyer/rhythm-and-time-toolkit/blob/main/LICENSE.md) and the [gesture recognizer license](https://github.com/lysdexic-audio/jweb-hands-gesture-recognizer/blob/main/LICENSE). Source code remains subject to the applicable copyright and license notices.

The supplied project archive does not include a separate `LICENSE` file. This README records the sources and upstream license information; it does not replace the full license texts.

## Verification Status

This documentation is based on the supplied Max patches, JavaScript implementation, upstream repository documentation, and the project author's description of the original four-track Ableton Live setup. End-to-end camera and DAW testing, and cross-platform compatibility testing, have not been performed as part of this documentation update. The Max version listed above is the file's saved version, not a compatibility test result.
