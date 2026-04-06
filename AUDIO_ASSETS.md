# Audio Assets - Moon Landing Game

All audio assets are royalty-free from Pixabay (CC0 / Free for commercial use).

## Background Music
- **Track:** Space Ambient Synth Loop
- **URL:** `https://cdn.pixabay.com/audio/2022/03/24/audio_07d931d38c.mp3`
- **Usage:** Plays continuously during gameplay
- **Volume:** 30%
- **Loop:** Yes

## Sound Effects

### Thrust (Engine Firing)
- **URL:** `https://cdn.pixabay.com/audio/2022/03/10/audio_32c7a5d76c.mp3`
- **Usage:** Plays while thrust key is held
- **Volume:** 40%
- **Loop:** Yes (while thrusting)

### Soft Landing (Success)
- **URL:** `https://cdn.pixabay.com/audio/2022/03/15/audio_c8c8a23f6d.mp3`
- **Usage:** Plays on successful landing
- **Volume:** 50%
- **Loop:** No

### Crash Landing (Failure)
- **URL:** `https://cdn.pixabay.com/audio/2022/03/09/audio_145f9e0e2a.mp3`
- **Usage:** Plays on crash/failed landing
- **Volume:** 60%
- **Loop:** No

## Implementation Notes

- Audio initialization happens on component mount but playback requires user interaction (browser autoplay policy)
- Background music starts when user clicks "START MISSION" button
- Thrust sound plays continuously while thrust key is held, stops on release
- Landing sounds trigger on game end state (won/lost)
- All audio is cleaned up on component unmount

## Attribution

All sounds sourced from Pixabay's free audio library:
- https://pixabay.com/audio/
