# Feixin Gesture

A gesture-controlled music project built with **Max/MSP and MediaPipe**, using the **Rhythm and Time Toolkit (RTT) package** for Euclidean rhythm generation. It tracks both hands through a camera and translates gestures and hand movements into MIDI notes, rhythmic changes, and continuous control messages for live musical interaction and improvisation.

Max generates MIDI sequences for keys, drums, and bass. A separate DAW project receives this MIDI output and produces the music.

> **A separate DAW project with instruments is required to produce sound.** The original setup uses a **six-track Ableton Live Session**, with **two tracks for each instrument part—keys, drums, and bass—to switch between timbres**. The supplied Max project does not include this Live Session, instruments, or samples.

## Features

- **Two-hand tracking:** Up to two hands, with 21 landmarks per hand and gesture classification.
- **Custom OK detection:** Geometric calculations using hand landmarks inside Max.
- **Three MIDI sequence parts:** Independently toggle keys, drums, and bass.
- **Rhythm generation:** Randomized patterns and RTT's `rtt.euclidean~` generate 32-step drum sequences.
- **Continuous control:** Open-palm movement modulates MIDI CC output.
- **Visual feedback:** Camera image, hand skeletons, and control states. The recognition patch also accepts JPG and PNG images.

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
Six-track Ableton Live Session
(two tracks per instrument part for timbre switching)
      ↓
Audio output
```

## Requirements

| Component | Details |
| --- | --- |
| Max | Both patches were saved in **Max 9.0.5**. Use this version or later; other versions have not been verified. |
| Rhythm and Time Toolkit | Install separately through Max's Package Manager. Required for `rtt.euclidean~`. |
| Camera | Live control requires a video input and camera permission. |
| Internet connection | MediaPipe scripts and WASM load from jsDelivr; the recognition model loads from Google Storage. |
| DAW project with instruments | Receives Max's MIDI notes and generates audio. The original setup uses a six-track Ableton Live Session. |
| MIDI connection | A route from Max to the DAW. The patches currently target `from Max 1`. |

Install **Rhythm and Time Toolkit** through Max's **File → Show Package Manager**. See the [upstream installation instructions](https://github.com/pdmeyer/rhythm-and-time-toolkit#installation).

The JavaScript uses `@mediapipe/tasks-vision@0.10.0` with GPU processing. Offline copies of the model and all web dependencies are not bundled.

## Getting Started

1. Download and extract the project, preserving the relative locations of the Max patches, HTML file, `js/`, and `css/` folders.
2. Install RTT, then open both patches in the same Max application instance: `jweb-hands-gesture-recognizer.maxpat` for recognition, and `gesture system.maxpat` for sequencing and MIDI output.
3. Allow camera access and wait for the dependencies and model to load. Refresh the recognition patch's device list and select an input if needed.
4. Enable DSP in Max's audio settings. The palm-modulation modules use signal objects such as `cycle~` and `snapshot~`.
5. Open the separate DAW project. To reproduce the original setup, use the six-track Ableton Live Session with two alternative timbres for each instrument part.
6. Route Max's MIDI output to the DAW. The default target is **`from Max 1`**. If unavailable, update each `midiout` object to use an available output port.
7. Configure the receiving MIDI channels, load instruments, and enable input monitoring.
8. Open Presentation Mode in `gesture system.maxpat`, set the BPM, and test the keys / drum / bass switches before trying gesture control.

## Ableton Live Session and MIDI Routing

The original Live Session uses **six tracks in three pairs**. Each instrument part has two tracks with alternative timbres, allowing its sound to be switched during performance.

| Instrument part | Live tracks | MIDI note channel | Purpose |
| --- | --- | --- | --- |
| Keys | 2 | 1 | Switch between two keys timbres |
| Drums | 2 | 2 | Switch between two drum timbres |
| Bass | 2 | 3 | Switch between two bass timbres |

Max also sends **CC 1 and CC 10 on channel 1**. Map these messages to the desired parameters in the receiving project.

The six Live tracks correspond to three instrument parts, not six separate MIDI note channels. The instruments, effects, and detailed timbre-switching configuration belong to the separate Live Session. Use its routing if available, or create your own paired tracks and sound-switching setup in a DAW.

**Drums are sent on channel 2.** Set the drum instruments to receive that channel and adapt their note mappings as needed.

## Gesture Controls

These mappings reflect the connections in the supplied Max patches.

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

- Discrete gesture triggers are gated by hand count. Keep both hands visible for these controls.
- Holding a gesture does not continuously retrigger its action. To repeat it, make a fist first, then perform the target gesture again.
- Keys, drums, and bass use toggled playback states and can continue after a gesture. Use their switches to stop playback; removing your hands does not reliably stop it.
- Palm movement modulates time-varying CC output. The audible result depends on the DAW's mappings.
- Adjust tempo with the BPM field and `speed up` / `speed down` controls. Thumb gestures change rhythm event counts.
- `OK` uses geometric calculations in Max rather than a default MediaPipe category. Camera distance, hand angle, and thresholds affect recognition.

## Project Structure

```text
feixin_gesture/
├── README.md
├── LICENSE                              # GNU GPL v3.0 license text
├── gesture system.maxpat                 # Sequencing, gesture mapping, MIDI output
├── jweb-hands-gesture-recognizer.maxpat   # Tracking, landmark parsing, OK detection
├── jweb-hands-gesture-recognizer.html     # Web entry point for jweb
├── css/
│   └── mesh-style.css                    # Image and overlay styling
└── js/
    ├── hands-landmarks-index.js          # Hand landmark indices
    └── jweb-hands-gesture-recognizer.js   # MediaPipe and Max communication
```

The DAW project and RTT package are separate and are not included above.

The patches communicate through named Max message channels, including `left_gest`, `right_gest`, `left_xy`, `right_xy`, and `numhands`. Run both patches in the same Max instance. Multiple copies may interfere with one another.

## Troubleshooting

### Blank web view or missing camera image

Check camera permissions, the input device, and connectivity to `cdn.jsdelivr.net` and `storage.googleapis.com`. Use `reload` and inspect the Max Console for `jweb` errors. If the HTML file cannot be found, check its location and, if necessary, give `jweb` its absolute file URL.

### Missing `rtt.euclidean~`

Confirm RTT is installed and available to Max, then reopen the sequencing patch.

### Gestures work, but there is no sound

Open the separate DAW project and load instruments. Check the MIDI output port, receiving channels, input monitoring, and audio output. Test the interface switches first. Drum instruments must receive channel 2.

### Gestures do not trigger or cannot be repeated

Keep both hands visible, make a fist, then perform the target gesture. Check the left/right labels and adjust `flip_hands` / `flip_image` as needed.

### Palm movement does not change the sound

Confirm the gesture is `Open_Palm`, Max DSP is enabled, and the DAW has parameters mapped to CC 1 / CC 10 on channel 1.

## Code Attribution

The hand-tracking and gesture-recognition component is based on and modified from **[jweb-hands-gesture-recognizer](https://github.com/lysdexic-audio/jweb-hands-gesture-recognizer)** by **lysdexic-audio**, which provides MediaPipe hand tracking and gesture recognition through Max's `jweb`.

This project adapts that component for musical interaction, including custom OK detection, hand-position output, and integration with gesture-controlled keys, drum, and bass sequences and MIDI CC modulation.

The upstream project is licensed under **GNU GPL v3.0**. See its [original license](https://github.com/lysdexic-audio/jweb-hands-gesture-recognizer/blob/main/LICENSE).

## External Dependency and Acknowledgments

**[Rhythm and Time Toolkit](https://github.com/pdmeyer/rhythm-and-time-toolkit)** by **Philip Meyer** is used as a separately installed Max package, specifically for the `rtt.euclidean~` object.

No source code or example patches from the RTT repository have been copied or adapted into this project. RTT is an external dependency and is not bundled with this repository. Please install it through Max's Package Manager.

Thanks to lysdexic-audio for the gesture-recognition foundation and Philip Meyer for the RTT package.

## License

This project's code is distributed under the **GNU General Public License v3.0**. See [LICENSE](LICENSE) for the full license text.

The hand-tracking and gesture-recognition component includes code derived from `lysdexic-audio/jweb-hands-gesture-recognizer`. Copyright in the upstream code remains with its respective authors; this project's modifications do not replace their rights or notices.

Separately installed dependencies remain subject to their own licenses. The Ableton Live Session, instruments, and samples are not included in this repository.

## Verification Status

This documentation is based on the supplied patches, JavaScript, upstream documentation, and the project author's description of the six-track Ableton Live Session. End-to-end camera and DAW testing and cross-platform compatibility testing have not been performed as part of this documentation update. The Max version is the file's saved version, not a compatibility test result.
