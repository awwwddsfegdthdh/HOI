# DexYCB Single-Trajectory Processing Plan

Last updated: 2026-05-02

## Summary

Process one DexYCB trajectory on `nju-vpn2` in the `dex` container and complete the C2HOI single-sample hand-object pipeline through `contact_points.npy`.

Default sample:

- sequence: `20200709_141754`
- camera: `836212060125`
- frames: 72
- hand: right
- object: `meta.yml` `ycb_grasp_ind=0`
- object pose: dual-track validation, first use DexYCB annotated pose to complete the pipeline, then compare with `objectpose`
- work dir: `/workspace/C2HOI/work_dirs/dexycb/20200709_141754__836212060125`

## Documentation

- This file is the source-of-truth plan for the current task.
- Any later plan change must be reflected here before execution continues.
- Execution state, command evidence, output paths, and failure points must be recorded in `C2HOI/docs/DEXYCB_PROCESSING_STATUS.md`.

## Control And Version Gate

- If execution reveals unplanned facts, stop and report before continuing.
- Do not silently bypass missing dependencies, unclear coordinate systems, unexpected data layouts, failed commands, or inconsistent output shapes.
- Do not write into the raw DexYCB data tree.
- Do not batch-process sequences during this task.
- Do not clean existing experiment outputs unless explicitly approved.
- Before restarting/stopping containers or killing processes, obtain explicit user approval for that specific action.
- Before modifying project contents, maintain a rollback path through Git.
- Use `git@github.com:awwwddsfegdthdh/HOI.git` as the intended version-management repository.
- The repository is currently empty and local SSH access previously failed; use HTTPS or the GitHub plugin when SSH is blocked.
- Keep tracked scope narrow: this plan, status docs, and task-specific scripts/wrappers only.
- Do not modify nested upstream repositories such as `Dyn-HaMR` or `FoundationPose` unless separately approved.
- Before and after each remote write, record the file list and diff or hash evidence.

## Implementation

1. Create a single-sample preparation script under `C2HOI/tools/`.
2. Read DexYCB RGB, depth, labels, `pose.npz`, and `meta.yml`.
3. Generate the normalized work directory:
   - `rgb/*.png`
   - `depth/*.png`
   - `depth/depth.npy`
   - `mask/object/*.png`
   - `meshes/object/object.obj`
   - `obj_poses/poses.npy`
   - `obj_poses/seq.npy`
   - `camera_data.json`
   - Dyn-HaMR intrinsics and camera files
4. Select the object from `grasp_ind=0`; if the object-to-mesh mapping is missing, stop for confirmation.
5. Run WiLoR/Dyn-HaMR and collect:
   - `dynhamr/smooth_fit/*world_results.npz`
   - `dynhamr/track_info.json`
6. Run object pose dual-track validation:
   - primary track: DexYCB `pose_y`
   - validation track: `objectpose/estimate_object_pose.py sequence`
   - save pose difference summary and overlay visualizations
7. Run `hand-object/optimize_hand.py` with explicit paths:
   - `--data_path`
   - `--out_path`
   - `--mano`
8. Confirm final outputs:
   - `sv_dict.npy`
   - `contact_points.npy`
   - `object.obj`
   - `object.urdf`

## Preconditions

- GPU/NVML: resolved. After the approved `dex` container restart on 2026-05-02, `docker exec dex nvidia-smi` succeeds.
- Conda activation: resolved. Use `/root/anaconda3/etc/profile.d/conda.sh`; available envs include `dynhamr`, `foundationpose`, and `wilor`.
- Sample integrity: resolved. For `20200709_141754/836212060125`, 72 RGB, depth, and label frames are present and aligned.
- Target object mapping: resolved. DexYCB class id `1` maps to `002_master_chef_can`; `grasp_ind=0` selects `ycb_ids[0]=1`; `/workspace/HOIDATA/DexYCB/models/002_master_chef_can/textured_simple.obj` exists.
- Camera calibration: unresolved. The local DexYCB root currently lacks the expected `calibration/` directory. Stop before generating `camera_data.json` until the source is approved.
- Objectpose validation dependencies: unresolved. The container currently has no `posetrack` Conda env, no `/workspace/C2HOI/third_party/co-tracker`, no `scaled_offline.pth`, and no `ckpts/moge2/model.pt`. Since DexYCB depth exists, MoGe can be avoided for the first run, but CoTracker/`posetrack` are still required if running `objectpose/estimate_object_pose.py sequence`.

## Verification

- Preparation:
  - 72 RGB, depth, and label frames are aligned.
  - Object masks are non-empty.
  - `obj_poses/poses.npy` has shape `[72, 4, 4]`.
- Hand side:
  - `*world_results.npz` exists.
  - `track_info.json` exists and covers the expected frame range.
- Object side:
  - Annotated pose overlay roughly matches the object mask.
  - If the validation track runs, record rotation and translation differences against annotation.
- Contact stage:
  - `contact_points.npy`, `sv_dict.npy`, `object.obj`, and `object.urdf` exist.
  - Output shapes and at least one visualization are manually inspected.

## Current Blockers And Notes

- GitHub SSH access from the local machine failed with a Windows OpenSSH/Git SSH issue.
- The GitHub repository was initialized through the GitHub plugin because local Git push authentication failed.
- The local `gh` CLI is not installed.
- Container GPU access was restored after restarting `dex` on 2026-05-02; future container restarts require explicit user approval first.
- The next approval decision is required before processing: either provide/copy the official DexYCB `calibration/` folder, approve using an external official calibration source for camera `836212060125`, or approve a fallback intrinsic source.
- The `objectpose` validation track requires approval before environment/dependency installation or downloading CoTracker assets.
