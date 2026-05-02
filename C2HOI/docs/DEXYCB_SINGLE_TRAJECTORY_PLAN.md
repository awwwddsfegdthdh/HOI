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

- Resolve `docker exec dex nvidia-smi` failing with `Failed to initialize NVML`.
- Activate Conda in the container through `/root/anaconda3/etc/profile.d/conda.sh`.
- Confirm whether a `posetrack` environment must be created for the `objectpose` validation path.
- Confirm camera calibration source. If it cannot be found, stop and agree on an intrinsic fallback before processing.
- Confirm `grasp_ind=0` maps to an available YCB mesh.

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

## Current Blockers

- GitHub SSH access from the local machine failed with a Windows OpenSSH/Git SSH issue.
- The GitHub repository is reachable over HTTPS but appears empty.
- The local `gh` CLI is not installed.
- Container GPU access currently reports `Failed to initialize NVML`.
