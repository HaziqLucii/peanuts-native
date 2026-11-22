# inKONBINI: One Store, Many Stories (`0100DFD026430000`)

<!-- PEANUTS: local Android bring-up profile. -->

Switch update 1.0.203.0 (`v196608`), Nintendo SDK 20, ARM64, NVN, Unity 6000.0.65f1. Modules: `main`, `subsdk0`, `sdk`. The build IDs are in [game.tsv](game.tsv).

**Android status: bring-up.** The profile was written by `scripts/build_ipa_from_nsp.py` from the survey with `dm/games/profile_guess.py` and was not checked by hand. The game boots, draws and plays on an AYN Thor (Adreno 740). On Turnip the contributor sees a few small graphical glitches and plays through them. On the system driver the picture flickers heavily and shows many glitches. None of this has been traced.

## What was run

All runs: AYN Thor, Snapdragon QCS8550, Android 13, test app `peanuts-native-0.1.6-dev` built from commit `606320f` (the current tree), upscaler sgsr1, frame generation off.

| Driver | Frame rate | Length | Result |
|---|---|---|---|
| Turnip_Gen8_V39-patched (Mesa 27.0.0-devel, driver ID 18) | 60 fps, `env.txt` override | 131 s | median 58.5 fps (55.9 to 62.6), GPU busy 10.3 ms (p50) at 1,862 draws; minor glitches, no flicker |
| System (Qualcomm Adreno Vulkan Driver) | 60 fps, `env.txt` override | 49 s | median 58.9 fps (45.9 to 62.5), GPU busy 8.7 ms (p50) at 1,674 draws; heavy flicker and glitches |
| Turnip_Gen8_V39-patched | game default (30 fps) | 62 s | 29.9 to 30.0 fps; minor glitches |

- **The profile keeps the game's own rate.** The game sets present interval 2 on the 60 Hz display, so the default is 30 fps. The 60 fps runs set `DM_PRESENT_INTERVAL=display` and `DM_FPS_TARGET=60` in the app's `env.txt`, and the host then presented with interval 1. In the `0.1.3-dev` logs `present cap 60 fps not applied` shows that `DM_FPS_TARGET` does nothing for this game (it has no frame step); the rate came from the display interval. The profile does not set `present_interval`.
- Earlier runs of 5 to 109 seconds on a build from commit `fb218c7` (`peanuts-native-0.1.3-dev`) with Turnip V39 gave the same two rates, with and without the override.
- No new pipelines were built late in a run (`new pipelines 0`).

## Not checked

- Whether the game's speed is right at 60 fps (it was not compared with the 30 fps run).
- A Turnip driver other than the V39 build above.
- Gameplay beyond the first minutes of each run, DLC, and any phone but the Thor.
- The glitches and the system driver's flicker; no frame was captured and compared.
- `[RED]` lines: each run prints 13, all `gfx: mtl_* not yet on Android (NVN → Vulkan replay)` statistics hooks. They were not compared with the same lines of other games.

## Rewrite and shaders

- The shader scan finds 24,755 shaders: 142 compute, 1 geometry, 12,308 pixel and 12,304 vertex. 24,744 come from the game's data files and 11 from `main`.
- `shader_map.tsv`, `shader_pairs.tsv` and `shader_facts.tsv` are the unedited output of the split.
- The device has no mesh shaders; the host skips those draws (`DM_VK_MESH_EMU=0`). The log line is the same on the system driver and on Turnip.

## Profile and memory

- The NPDM declares 16 MB of system resource and `main` never sets a heap, so the game maps its memory itself: a 5 GB block with a 4.375 GB alias region and the Switch application pool of `0xCD500000` bytes.
