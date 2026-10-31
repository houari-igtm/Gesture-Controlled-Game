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

## Limitations

- Detection gets worse in low light, and the face must be clearly visible in face mode.
- A mouth turned sideways can give a borderline ratio (around 0.18 to 0.22), so the result varies.
- The game may not run well at every resolution. Object positions and sizes are calculated for a fixed frame size, so other resolutions can misplace objects or cut off the interface. Recommended: 1280×720 (720p), which is the default in base/base_games.py. Use a 720p-capable webcam and a screen of at least that size.
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
