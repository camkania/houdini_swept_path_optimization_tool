# houdini_swept_path_optimization_tool
 Tool to reduce the polycount of a series of combined static meshes.

## High Level Functions of the tool
- Convert a "blob" of static meshes into a voxelized singular mesh
- Convert that "blob" back into geometry with a significant reduction in the # of polygons

## Background
This tool was built to optimize the output of the [Snapshot tool](https://github.com/camkania/blender_snapshot_tool). Snapshot captures an animated ride or mechanism as a dense series of static meshes—one per sampled frame—which merge into a single "blob" describing the full swept-path envelope. That blob is accurate but very heavy (thousands of overlapping, self-intersecting surfaces). This tool rebuilds it as a clean, watertight, dramatically lighter mesh while preserving the true shape of the motion.

The original use case was representing the dynamic motion path of robotic ride systems in a way that is both accurate and efficient. The same pipeline has since been used to clean up and optimize engineering models for 3D printing.

## Requirements
- **Houdini** 20.5 or newer. The included `.hipnc` file opens in **Houdini Apprentice (Non-Commercial)**.

## Files
- `houdini_files/OptimizeSweptPath.hipnc` — the tool.
- `sample_animation/` — sample inputs, including `blender_sample_torus_anim_swept_path.obj` (a combined swept path exported from Blender) and the source `.blend` / `.fbx` files.

## How to operate

1. **Prepare your input.** Export the swept path—the combined static-mesh "blob"—from your DCC as a single geometry file (OBJ, FBX, or bgeo). If you use the Snapshot tool, this is the merged mesh it produces. A ready-made example lives in `sample_animation/blender_sample_torus_anim_swept_path.obj`.

2. **Open the tool.** Launch Houdini and open `houdini_files/OptimizeSweptPath.hipnc`. At the `/obj` level you'll find two geometry objects: `IMPORT_SWEPT_PATH` (the raw input) and `OPTIMIZE_SWEPT_PATH` (the pipeline that does the work).

3. **Point the tool at your mesh.** Dive inside `OPTIMIZE_SWEPT_PATH` and select the **`REFERENCE_IMPORTED_MESH`** (File SOP). Set its *Geometry File* to your exported swept path. (It ships pointed at the sample OBJ, so you can run it as-is to see the result first.)

4. **Voxelize.** The **`VOXELIZE_SNAKE`** node converts the overlapping meshes into a single VDB volume, fusing them into one watertight solid. Its **Voxel Size** is the main control:
   - *Closer to 0* → smaller voxels, more accurate to the original shape, higher polycount.
   - *Larger* → coarser voxels, lighter mesh, lower polycount.
   Adjust it until the shape is faithful enough for your purpose without carrying unnecessary detail.

5. **Convert back to polygons.** **`CONVERT_BACK_TO_POLYs`** rebuilds the VDB as clean polygon geometry with a significant reduction in polycount. The final result comes out of **`OPTIMIZE_SNAKE_OUT`**.

6. **Compare against the original.** Use the **`SET_COLOR`**, **`SET_ALPHA_VAL1`**, and **`ADJUST_VISIBILITY`** nodes to tint and fade the optimized mesh, then overlay it on the original swept path to confirm the envelope is preserved.

7. **Export the result.** Right-click the output node (`OPTIMIZE_SNAKE_OUT`) and choose **Save → Geometry**, then save as `name.obj` (or another geometry format). This reduced mesh is what you hand off for clearance studies, layout, review, or 3D printing.

### Quick reference

| Node | Role |
| --- | --- |
| `REFERENCE_IMPORTED_MESH` | Imports the combined swept-path blob |
| `VOXELIZE_SNAKE` | Converts the blob to a single VDB — **Voxel Size** controls accuracy vs. polycount |
| `CONVERT_BACK_TO_POLYs` | Rebuilds optimized polygon geometry from the VDB |
| `SET_COLOR` / `SET_ALPHA_VAL1` / `ADJUST_VISIBILITY` | Tint/fade for comparison against the original |
| `OPTIMIZE_SNAKE_OUT` | Final output — right-click → Save → Geometry to export |
