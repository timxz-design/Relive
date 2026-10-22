# Relive — Project Brief & Roadmap

_Last updated: 2026-07-21_

## Vision
Make creating an immersive VR experience as simple as recording a video with a smartphone. Long term: photorealistic, editable, physically simulated environments — from single rooms up to whole buildings and outdoor areas — generated automatically from casual phone capture.

## MVP thesis
The MVP proves **one** thing: capture a room on a phone → explore it in VR minutes later, and have it feel real. If that lands, the core experience is validated and everything else is execution. If it doesn't, no amount of backend or object-generation work matters.

---

## Core decision: splats first, meshes later

The reconstruction step (DUSt3R / MASt3R / VGGT) is **not** the risky part — it works. The risky part is that a **meshed point cloud looks bad in VR** (blobby, holey, untextured; brutal at headset viewing distance).

**MVP renders Gaussian Splatting, not a mesh.** Path:

```
video → frames → DUSt3R/VGGT (poses + sparse cloud) → 3DGS training → .ply splat → VR splat viewer
```

This is a known recipe (see **InstantSplat**: DUSt3R poses + 3DGS for sparse views, trains in minutes). It's photorealistic straight from phone video, and skips meshing/texturing/hole-filling entirely.

- **Trade-off:** splats have no collision/physics and show artifacts far from the capture path. Acceptable — the MVP question is "explore," not "interact."
- **The mesh + object-generation + scene-graph pipeline (SAM2/Florence-2, Hunyuan3D, relations) is V2**, not MVP. That's where editability, physics, and clean assets — the real moat — live, but none of it is needed to validate the core experience.

---

## Roadmap

### Phase 0 — De-risk the magic (~1–2 weeks, no product code)
Just a GPU box and a notebook.
- Video → frames → DUSt3R/VGGT → InstantSplat-style 3DGS → `.ply`
- Load in desktop Unity splat viewer, view over Quest Link
- Sort out **metric scale** (see risks)
- **Gate:** does a real phone video of a real room feel good in the headset? Iterate here until yes. This is the whole ballgame.

### Phase 1 — Thin end-to-end (only after Phase 0 passes)
- Wrap pipeline as a **RunPod** endpoint (skip Colab-as-worker entirely)
- Minimal upload → cloud storage → job row → worker pulls, runs, uploads `.ply` → viewer downloads & renders
- Goal: "phone → headset" with no manual notebook steps. This is the demo.
- Keep every box thin.

### Phase 2 — Make it a product
- Real capture app with ARKit/ARCore (poses + metric scale for free)
- Accounts, job-status UI, scene library
- Standalone-Quest performance work (splat pruning / LOD)
- Formalize Postgres / queue / object storage here

### Phase 3+ — The vision (V2 pipeline)
- SAM2 / Florence-2 segmentation → scene graph → Hunyuan3D object replacement
- Mesh room + physics + editability + dynamic lighting
- Larger-scale environments (buildings, outdoors)
- The "generic scene representation → separate importer" split pays off most here — hold that line.

---

## Decisions to lock before starting

| Decision | Options | Current call |
|---|---|---|
| MVP renderer | Splat vs mesh | **Splat** (mesh only if collision/physics is a hard day-1 requirement) |
| Capture method | Raw video vs AR-app capture | **Raw video** for Phase 0; AR capture in Phase 2 |
| Reconstruction model | DUSt3R / MASt3R / VGGT | Bake-off in Phase 0; **try VGGT first** for pose quality on video-length sequences |
| GPU worker | Colab / RunPod / AWS | **RunPod** (serverless or persistent pod); skip Colab glue |
| VR target for first demo | Standalone Quest vs PCVR/Quest Link | **PCVR/Quest Link** first (no splat-count ceiling, no build pipeline) |

---

## Risk register

1. **Metric scale.** DUSt3R/VGGT output is scale-ambiguous. Wrong scale in VR = feel like a giant/ant = presence destroyed. Fix via ARKit/ARCore metric poses, a known-size reference object, or a manual scale slider (MVP). Decide early — touches capture.
2. **VR viewer is a whole workstream**, not one export line. Quest dev mode, APK signing/sideloading, and standalone Quest **cannot** render millions of splats at 72–90fps. Render on PC over Quest Link for the first demo; treat native-Quest performance as later optimization. Reference: Aras Pranckevičius's UnityGaussianSplatting.
3. **Colab unreliability.** Disconnects, session caps, no GPU guarantee. Don't build tooling around it — go RunPod.
4. **Premature backend.** FastAPI + Postgres + queue is product plumbing, not validation. A notebook + viewer answers the MVP question. Build the SaaS layer only after the pipeline produces something worth walking around.

---

## Architectural guardrail (non-negotiable)
AI pipeline outputs a **generic scene representation**: camera poses, room geometry, object transforms, meshes/splats, materials, scene graph. A **separate importer** builds the Unity (or Unreal / WebXR) scene from it. No renderer-specific logic in the pipeline.
