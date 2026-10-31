# Gesture-Controlled Educational Games for Children

Touchless educational games for children, controlled by the player's **mouth and hand** through a webcam. Built in Python with OpenCV, MediaPipe and cvzone, with a Tkinter launcher.

This is the code for our final-year project at Ain Temouchent University (Computer Systems, class of 2025–2026): *Development of an Educational Game for Children Using Computer Vision in Python*.

## The two games

| | Game 1: Falling Object | Game 2: Pick the Right Object |
|---|---|---|
| **Goal** | Catch edible objects as they fall and avoid non-edible ones | Select the object whose name is shown on screen, out of three |
| **Controls** | **Open mouth** (face mode) or **open palm** (hand mode) | **Point** at an object with your hand, then **close your fist** to confirm |
| **Scoring** | Edible: +1. Non-edible: score resets to 0 and Game Over | Correct: +1. Wrong: score resets to 0. A new round starts after 2 seconds |
| **Tracking** | MediaPipe FaceMesh or Hands | MediaPipe Hands |

Objects: *edible* (pineapple, apple, carrot, mango) and *non-edible* (bomb, chair, computer, mouse).

## How it works

**Mouth opening (Game 1, face mode).** FaceMesh returns 468 facial landmarks. The y-coordinates of two groups of lip landmarks are averaged, and their gap is divided by the mouth width (landmarks 61 to 291). Dividing by the width makes the ratio independent of the distance from the camera. The mouth counts as open when the ratio is above **0.20**, a threshold chosen experimentally.

```python
mouth_ratio = mouth_gap / mouth_width   # distance-invariant
mouth_open  = mouth_ratio > 0.20
```

**Hand posture (both games).** MediaPipe Hands returns 21 landmarks per hand. The shared method `is_hand_closed()` compares the middle fingertip (landmark 12) with the base of the middle finger (landmark 9). Because the image y-axis points downward, a fingertip below its base means a closed fist.

- **Game 1, hand mode:** four palm points (landmarks 0, 5, 9, 13) are checked against the falling object, and only when the hand is open.
- **Game 2:** landmark 9 is the pointer, and a selection counts only when the fist is closed. This prevents accidental selections while the hand is just hovering.

**Collision detection.** The tracked point (mouth center, palm points, or the pointer) is tested against the object's bounding rectangle.

## Project structure

```
.
├── base/
│   ├── base_games.py      # Shared parameters: resolution, speed, score
│   └── base_tracker.py    # MediaPipe setup + is_hand_closed()
├── game1/
│   ├── game.py            # Game 1 loop: falling objects, face/hand modes
│   └── traker.py          # TrackFace() and TrackHands()
├── game2/
│   ├── game2.py           # Game 2 loop: object selection, 2 s wait, score
│   └── traker2.py         # TrackHands(): landmark 9 pointer, closed-fist trigger
├── images/
│   ├── eatable/           # annas, apple, carrot, mango
│   └── non_eatable/       # bomb, chair, computer, mouse
├── gui.py                 # Tkinter launcher (entry point)
└── requirements.txt
```

The design is object-oriented: `Base` and `Base_Tracker` are abstract classes inherited by both games, so the resolution, speed, MediaPipe setup and hand-posture logic live in one place.

## Installation

You need a computer with a **webcam** and **Python**. The pinned MediaPipe version (0.10.11) works with older Python releases, so use Python 3.9 to 3.11 if installation fails on a newer one.

```bash
git clone https://github.com/houari-igtm/Gesture-Controlled-Game.git
cd Gesture-Controlled-Game

python -m venv venv
# Windows:  venv\Scripts\activate
# Linux/macOS:  source venv/bin/activate

pip install -r requirements.txt
```

Main dependencies: OpenCV, MediaPipe, cvzone, NumPy (Tkinter ships with Python).

## Usage

```bash
python gui.py
```

Choose **Game 1** or **Game 2** in the menu. Play in a well-lit room, facing the camera.

| Key | Action |
|---|---|
| `C` | Switch between face mode and hand mode (Game 1) |
| `R` | Restart after Game Over (Game 1) |
| `K` | Close the game and return to the menu |

## Challenges

- **Calibrating the mouth threshold.** Faces differ in shape and sit at different distances from the camera. Dividing the lip gap by the mouth width made the ratio distance-independent, and the 0.20 threshold took several rounds of testing to work reliably on different facial shapes. Averaging groups of lip landmarks instead of using just two points also made the measurement steadier.
- **Accidental selections in Game 2.** Using the hand position alone made the game select objects whenever a hand passed over them. Requiring a closed fist as the confirmation gesture fixed this.
- **Face tracking in poor conditions.** Face detection suffers in bad lighting or awkward posture, so Game 1 includes a hand mode that the player can switch to with the `C` key.
- **Transparent PNG overlays.** Some PNG files without an alpha channel produced transparency artefacts when drawn with `cvzone.overlayPNG()`.
- **Unstable landmarks at the screen edge.** Landmark 9, used as the pointer in Game 2, becomes unreliable when the hand is partly outside the camera view.
- **Blocking the Tkinter menu.** A running game blocks the Tkinter main loop because both run in the same thread, so the menu is unresponsive until the game closes.
- **Placing objects on different screens.** Positioning objects dynamically so they look right on different devices and resolutions was difficult. Screen size is now set in one shared base class, but the layout still isn't fully adaptive.

## Limitations

- Detection gets worse in low light, and the face must be clearly visible in face mode.
- A mouth turned sideways can give a borderline ratio (around 0.18 to 0.22), so the result varies.
- Fall speed is fixed at 5 px per frame, with no difficulty levels.
- There is no audio and no multiplayer mode.
- The system uses pre-trained MediaPipe models and rule-based gesture logic. It does not train its own models.

## Future work

- Progressive difficulty (speed and number of objects increase with the score)
- Sound effects for success and failure
- Two-player mode (one player uses the mouth, the other the hand)
- Saved leaderboard (SQLite or JSON)
- New learning themes, such as wild vs. domestic animals or vehicles vs. buildings
- Running games in a separate thread to keep the menu responsive
- Learned gesture classifier to replace the fixed rules
- Browser or tablet version

## Authors

- Abdelaziz Boulefred
- Imad Eddine Lahouari Belhamri
- Fares Abdelrezek Benabdelmoumene

Supervised by Mr. Mohammed Hakim Bendiabdallah. Department of Mathematics and Computer Science, Ain Temouchent University – Belhadj Bouchaib, Algeria.

## References

1. C. Lugaresi et al., "MediaPipe: A Framework for Building Perception Pipelines," arXiv:1906.08172, 2019.
2. F. Zhang et al., "MediaPipe Hands: On-device Real-time Hand Tracking," ECCV Workshop on CV for AR/VR, 2020.
3. G. Bradski and A. Kaehler, *Learning OpenCV*, O'Reilly, 2008.
