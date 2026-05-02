# DexYCB Processing Status

Last verified: 2026-05-02

## Server Paths

- DexYCB root: `/home/renjie/project_dex/HOIDATA/DexYCB`
- Object models: `/home/renjie/project_dex/HOIDATA/DexYCB/models`
- C2HOI root: `/home/renjie/project_dex/C2HOI`
- Container C2HOI root: `/workspace/C2HOI`

## Raw DexYCB Data

Verified contents:

- 1 subject package extracted: `20200709-subject-01`
- 100 interaction sequences under `20200709-subject-01/20200709-subject-01`
- 800 camera-view directories total
- 100 `pose.npz` files
- 100 `meta.yml` files
- object model directory exists with YCB objects such as mustard bottle, cracker box, mug, power drill, etc.

Interpretation:

- `subject` means the human participant/operator.
- `sequence` means one recorded interaction/trial by that subject.
- `camera view` means one synchronized camera recording of that sequence.

## Important Distinction

DexYCB `pose.npz` and `meta.yml` are part of the DexYCB data package. They are not evidence that C2HOI has processed the sequence.

For C2HOI processing, look for outputs such as:

- hand side: `dynhamr/smooth_fit/*world_results.npz`, `dynhamr/track_info.json`
- object side: `track/track.npz`, `pose/pose.npz`, `pose_track/pose_track.npz`
- hand-object side: `sv_dict.npy`, `contact_points.npy`, `object.urdf`

## C2HOI Processing State

Current verified status:

- DexYCB raw data: present.
- DexYCB dataset-provided pose/meta: present for all 100 sequences.
- DexYCB converted into C2HOI expected hand/object layout: not found.
- DexYCB Dyn-HaMR/WiLoR hand processing: not found.
- DexYCB objectpose processing: not found.
- DexYCB hand-object optimization: not found.

No files named `track.npz`, `pose_track.npz`, `sv_dict.npy`, `contact_points.npy`, or `object.urdf` were found under `/home/renjie/project_dex` during the 2026-05-02 status check.

## Existing Non-DexYCB Or Test Processing

`pure_right` under `C2HOI/test_data/dynhamr_input` has been partially prepared:

- 380 RGB frames
- 380 depth frames
- intrinsics file
- `dynhamr/cameras/pure_right/shot-0/cameras.npz`
- `dynhamr/hamer_out/pure_right/pure_right.pkl`
- `dynhamr/track_preds/pure_right/...`

But `C2HOI/outputs/dynhamr_rgbd/video-custom/wilor-rgbd-mesh/pure_right-all-shot-0-0-380` only contained Hydra config and `run_opt.log`; no `*world_results.npz` was found there. Treat `pure_right` as partial, not fully successful.

Old Dyn-HaMR demo/test outputs exist for:

- `demo1`
- `pure_002`
- `pure_002` variants: `depth-enabled`, `wilor-compare`, `wilor-rgbd-compare`

These are useful evidence that parts of Dyn-HaMR have run before, but they are not DexYCB processing outputs.

## Next Status Update Should Record

When a new run is attempted, append:

- selected subject/sequence/camera;
- exact source path;
- generated working data path;
- hand-side output paths;
- object-side output paths;
- hand-object output paths;
- failure point if any;
- command evidence or log path.
