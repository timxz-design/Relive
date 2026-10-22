# Original brainstorm (preserved verbatim, 2026-07-21)

> This is the raw project concept as first written, kept for reference. The working plan lives in PROJECT-BRIEF.md, which revises this (notably: splat-first MVP, skip Colab-as-worker).

## Project Idea: Photo/Video to VR

Build an AI system that transforms ordinary photos or short smartphone videos into fully explorable VR environments. Users capture a room/place with their phone, upload, and receive a VR scene they can walk through in a headset — no manual 3D modeling.

Core concept: leverage modern AI for 3D reconstruction (DUSt3R/MASt3R/VGGT), scene understanding (SAM2 segmentation, Florence-2 labels), and generative 3D asset replacement (Hunyuan3D). Assemble into a Unity project, export as interactive VR.

All AI processing runs on remote cloud GPUs (Colab prototype → RunPod/AWS later). Local app only uploads media, tracks status, downloads the finished world.

MVP: record a 30s video of a room, upload it, explore it in VR a few minutes later.

Future: better reconstruction, texture generation, semantic understanding, physics, editable environments, dynamic lighting, large-scale environments (buildings, outdoors).

## High-level architecture (as originally drawn)
User → Photos/Video → Frontend (React/App) → FastAPI backend → upload + job creation → {Metadata DB (Postgres), Object Storage (Drive/S3)} → processing queue → Colab notebook (remote GPU worker) → AI pipeline → generated 3D world → Unity scene (.glb/.fbx/.unity) → VR app.

Colab is not the application — just a remote GPU worker that polls for waiting jobs, downloads input, runs the pipeline, uploads results. Later swap Colab for RunPod / Lambda / AWS / Azure / local GPU — nothing else changes.

## AI pipeline (original, full vision)
1. Frame extraction — FFmpeg, select ~40 useful frames (not all 900)
2. Reconstruction — DUSt3R/MASt3R → camera poses, depth, point cloud
3. Scene understanding — SAM2 segments → Florence-2 labels (bed, chair, desk, TV)
4. Scene graph — Room → Objects → Relations (desk next to window, chair facing desk); the "brain"
5. Object generation — per object: bounding box → Hunyuan3D → clean .glb
6. Room geometry — walls/floor/ceiling from DUSt3R or generated meshes
7. Scene assembly — snap objects into position with textures
8. Export — GLB/FBX/Unity package → Quest / SteamVR

Split the pipeline into independent stages (preprocessing, reconstruction, segmentation, recognition, scene graph, mesh gen, texture gen, assembly, export) — each can become its own model over time.

## Original MVP (as written)
Video → extract 30 frames → DUSt3R → point cloud → simple mesh → Unity → walk around. No generated furniture, no semantics, no physics, no object replacement. Just: "Can someone record a room and explore it in VR ~10 minutes later?"

## Architectural recommendation (kept)
Don't put Unity-specific logic in the AI pipeline. The pipeline produces a generic scene representation (camera poses, room geometry, object transforms, meshes, materials, scene graph). A separate importer builds the Unity scene. Then Unreal/WebXR/other renderers are importer swaps, not retraining/restructuring.
