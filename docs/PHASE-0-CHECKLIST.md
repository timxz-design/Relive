# Phase 0 — De-risk the Magic (checklist)

Goal: prove a phone video of a real room feels good in a headset. No product code. One GPU + one notebook + one viewer.

## 0. Environment
- [ ] GPU access: RunPod persistent pod (or local NVIDIA GPU) with CUDA. Target a 24GB card (3090/4090/A5000) — 3DGS training wants VRAM.
- [ ] Python env with PyTorch matching the CUDA version.
- [ ] `ffmpeg` for frame extraction.

## 1. Capture test data
- [ ] Record 3–5 real rooms, ~30s each, slow steady sweep, good lighting, some overlap between viewpoints.
- [ ] Keep one "hard" case (low light / reflective surfaces / textureless wall) to see failure modes early.

## 2. Frame extraction
- [ ] `ffmpeg` → extract ~30–50 frames, favor sharp/low-blur frames and even coverage (don't dump all 900).
- [ ] Simple blur filter (variance of Laplacian) to drop motion-blurred frames.

## 3. Reconstruction (poses + sparse cloud) — bake-off
Try in this order, keep whichever gives the cleanest poses on video-length input:
- [ ] **VGGT** (newest, handles more views in one shot) — https://github.com/facebookresearch/vggt
- [ ] **DUSt3R** — https://github.com/naver/dust3r
- [ ] **MASt3R** (stronger matching) — https://github.com/naver/mast3r
- [ ] Output needed downstream: camera intrinsics/extrinsics + initial point cloud.

## 4. Gaussian Splatting
- [ ] **InstantSplat** (DUSt3R poses + 3DGS, sparse-view, fast) — https://github.com/NVlabs/InstantSplat — start here; it wires steps 3+4 together.
- [ ] Fallback / reference: original **3DGS** trainer (inria graphdeco `gaussian-splatting`) if you want to feed your own poses.
- [ ] Export scene as `.ply` splat file.

## 5. Metric scale (do NOT skip)
- [ ] Pick a scale strategy for MVP: manual slider in viewer is fine for Phase 0.
- [ ] Sanity check: stand in the headset — does a doorway feel ~2m tall? A giant/ant scene = fail.

## 6. VR viewer
- [ ] Unity + **UnityGaussianSplatting** (Aras Pranckevičius) — https://github.com/aras-p/UnityGaussianSplatting
- [ ] Import `.ply`, enable an XR rig, run **over Quest Link / PCVR** (no standalone build yet).
- [ ] Add a scale slider + basic locomotion (teleport or smooth).

## 7. The gate
- [ ] Put the headset on someone who wasn't involved. Watch their reaction.
- [ ] Decide: is the "I'm in the room" moment there? If yes → Phase 1. If no → iterate capture/reconstruction/splat quality before building anything else.

## Notes / gotchas to watch
- Standalone Quest can't render millions of splats at framerate — that's a Phase-2 problem, ignore for now.
- Textureless walls and reflections break correspondence-based reconstruction — expect artifacts there.
- Sparse-view splatting degrades as you walk away from the capture path — keep the demo near the swept viewpoints.
