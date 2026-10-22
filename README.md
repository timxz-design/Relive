# Relive

Turn an ordinary phone photo/video into a fully explorable VR environment. Record a room, upload it, walk through it in a headset a few minutes later — no manual 3D modeling.

## Status
Pre-Phase-0. De-risking the core experience before any product code exists.

## The one thing we're validating first
> Can someone record a room with their phone and have a convincing "I'm actually there" moment in a VR headset a few minutes later?

Everything else (backend, accounts, object generation, physics) waits until this is proven.

## Repo layout
- `docs/` — project brief, roadmap, decisions, risk register
- `pipeline/` — AI reconstruction pipeline (frames → poses → 3D scene representation)
- `viewer/` — VR viewer (Unity / splat renderer)
- `backend/` — API + job orchestration (Phase 1+, deliberately empty for now)

## Key architectural principle
The AI pipeline produces a **generic scene representation** (poses, geometry, splats/meshes, materials, scene graph). Rendering lives in a **separate importer**. Never bake Unity-specific logic into the pipeline — that way Unreal / WebXR / native become importer swaps, not rewrites.

## Start here
Read [docs/PROJECT-BRIEF.md](docs/PROJECT-BRIEF.md).
