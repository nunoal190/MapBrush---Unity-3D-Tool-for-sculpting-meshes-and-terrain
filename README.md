# MapBrush — Modular Terrain Sculpting

MapBrush is a free, open-source Unity Editor tool for sculpting compatible modular terrain meshes directly in the Scene view.

Use it to raise hills, lower ground, smooth uneven surfaces and flatten areas while preserving configured borders and mesh thickness.

Originally developed for a terrain map built from wedge-shaped sections, MapBrush works with meshes that follow its supported vertex layout.

> **Compatibility:** MapBrush requires a structured, two-layer grid mesh with matching top and bottom vertices. It is not a general-purpose sculpting tool for arbitrary meshes and does not edit Unity Terrain objects.

## Features

### Raise and Lower

Raise terrain using the brush, or hold **Shift** to lower it.

- Adjustable brush radius.
- Adjustable height change per second.
- Continuous sculpting while holding or dragging the mouse.
- Soft falloff towards the edge of the brush.

### Smooth

Reduce sharp height differences by blending vertices towards the heights of their grid neighbours.

- Adjustable smoothing strength.
- Brush falloff for gradual transitions.
- Protected vertices remain fixed.

Smoothing adjusts vertex heights within the existing grid. It does not add geometry or change mesh topology.

### Flatten

Bring terrain towards a target height.

- Sample the target height from the surface at the start of a stroke.
- Alternatively, enter a target height manually.
- Adjust strength to control how quickly the terrain approaches the target.

The flatten target uses the surface object's **local Y coordinate**.

### Protected Borders

Keep selected grid boundaries unchanged while editing.

Available settings:

- **Inner Border Rows:** protect rows at the beginning of the grid.
- **Side Border Columns:** protect columns on both sides.
- **Protect Outer Border:** optionally protect the final row.

These controls are useful for modular sections whose existing boundaries must remain unchanged.

> Border protection preserves existing positions. It does not automatically align, weld or repair seams between neighbouring meshes.

### Vertex Preview

Display the configured top-surface vertices in the Scene view:

- **Green:** editable vertices.
- **Orange:** protected vertices.

Use the preview to check the grid configuration before sculpting.

### Thickness Preservation

Each edited top vertex moves together with its corresponding bottom vertex.

This preserves the pair's existing separation along the mesh's local vertical axis.

It does not calculate a constant thickness measured perpendicular to a sloped surface.

### Undo and Redo

Sculpting supports Unity's Undo and Redo workflow, with changes grouped by brush stroke.

### Collider Updates

The selected surface's MeshCollider is refreshed as the mesh changes, so its collision geometry follows the edited terrain.

### Optional Editable Copies

Choose between:

- **Edit Assigned Mesh:** edit the mesh currently assigned to the object.
- **Create Editable Copy:** create a separate mesh asset before editing.

Creating a copy is recommended when the original mesh should remain unchanged or when several objects share it.

### Demo Scene

The included demo contains three terrain samples:

- A flat surface.
- A raised or mountainous surface.
- A lowered or valley-like surface.

Use these to try the brushes and inspect a supported mesh setup.

## Tested Environment

- **Unity Editor:** 6000.6.0f1
- **Operating system:** Windows
- **Demo render pipeline:** Universal Render Pipeline (URP)

Other Unity versions, operating systems and render pipelines have not been verified.

The sample materials use URP. Other render pipelines may require replacement materials.

## Installation

1. Download the `.unitypackage` from this repository's **Releases** section.
2. Open your Unity project.
3. Select **Assets > Import Package > Custom Package**.
4. Choose the downloaded package and import its contents.
5. Wait for Unity to finish compiling.
6. Open **Tools > MapBrush > Open MapBrush**.

Alternatively, copy the `Assets/MapBrush` folder and its accompanying `.meta` files into your project's `Assets` folder.

## Quick Start

1. Open the included scene:

   `Assets/MapBrush/Samples/Scenes/MapBrushDemo.unity`

2. Select one of the sample terrain objects in the Hierarchy.
3. Open **Tools > MapBrush > Open MapBrush**.
4. Check the selected mesh information.
5. Choose **Create Editable Copy** to preserve the sample mesh, or **Edit Assigned Mesh** to modify it directly.
6. Choose a brush mode.
7. Adjust the radius and strength.
8. Enable the brush using the window's brush control.
9. Move the pointer over the selected surface in the Scene view.
10. Hold the left mouse button to sculpt.
11. Save the mesh using **Save Mesh**, then save the scene.

For Raise/Lower mode, hold **Shift** while sculpting to lower the surface.

## Supported Terrain and Meshes

MapBrush is designed for modular terrain sections with a structured grid and two corresponding vertex layers.

Suitable uses include:

- Wedge-shaped terrain sections using the supported layout.
- Rectangular terrain sections using the same layout.
- Procedurally generated terrain pieces that preserve the required vertex ordering.
- Existing hills and valleys created by changing the heights of a compatible grid.

The outline alone does not determine compatibility. A mesh can look suitable while having an unsupported internal vertex layout.

### Required Vertex Layout

A compatible mesh must contain:

1. A complete bottom grid.
2. A complete top grid.
3. Matching top and bottom vertices at each grid position.
4. Row-major vertex ordering within both layers.

All bottom vertices must appear first in the vertex array, followed by all top vertices.

For a grid with:

- `R` row segments.
- `C` column segments.

Each layer contains:

```text
(R + 1) × (C + 1) vertices
```

The total mesh vertex count must be:

```text
2 × (R + 1) × (C + 1)
```

For example:

```text
Row Segments:       6
Column Segments:   12

Vertices per layer:
7 × 13 = 91

Total mesh vertices:
91 × 2 = 182
```

The index layout is:

```text
bottomIndex = row × (C + 1) + column

topIndex = ((R + 1) × (C + 1)) + bottomIndex
```

Matching top and bottom vertices must have the same local X and Z coordinates, with the top vertex above the bottom vertex in local Y.

> Having the correct vertex count is not sufficient. The vertex ordering, pairing and surface connectivity must also match the expected grid structure.

Additional duplicated vertices for side walls, UV seams or hard edges can make a mesh incompatible.

### Required Object Setup

The selected scene object needs:

- A **MeshFilter** containing the compatible mesh.
- A **MeshRenderer** to display it.
- An enabled, non-convex **MeshCollider**.
- A **MapBrush Surface** component.
- A readable mesh.
- Positive, non-zero transform scale.

Editing is intended for Edit Mode.

### Unsupported Inputs

MapBrush does not directly support:

- Unity Terrain objects or TerrainData.
- Arbitrary imported 3D models.
- Single-layer planes without a matching bottom layer.
- Meshes with unrelated or reordered vertices.
- Skinned meshes.
- Meshes with an incompatible number of vertices.
- Arbitrary ProBuilder meshes.
- Overhangs or caves requiring general 3D sculpting.
- Meshes with negative or zero transform scale.

The package does not include an automatic mesh converter or terrain generator.

## Configuring Your Own Surface

1. Select a compatible mesh object in the Hierarchy.
2. Add the **MapBrush Surface** component.
3. Set **Row Segments** and **Column Segments** to match the mesh's original grid.
4. Configure the protected borders.
5. Enable **Show Vertex Preview**.
6. Inspect the preview in the Scene view.
7. Open the MapBrush window.
8. Prepare the mesh for editing.
9. Start sculpting.

Changing Row Segments or Column Segments does not rebuild or subdivide the mesh. These settings describe its existing layout.

If your terrain already contains height variation, it can still be compatible provided its vertex ordering and top/bottom pairing have been preserved.

## Brush Controls

| Action | Control |
|---|---|
| Sculpt | Hold or drag the left mouse button over the selected surface |
| Lower terrain | Hold Shift while using Raise/Lower |
| Stop and disable the brush | Escape |
| Undo | Unity's Undo command |
| Redo | Unity's Redo command |
| Navigate the Scene view | Standard Scene view navigation controls |

The brush operates on one selected surface at a time.

Raise/Lower moves vertices along the surface object's local up direction. The brush radius is measured in world-space units across the plane perpendicular to that direction.

## Saving Your Work

### Save Mesh

Use **Save Mesh** to persist changes to an existing editable mesh asset.

Also save the scene to preserve object assignments and scene changes.

### In-Memory Meshes

A mesh can look correct and provide working collisions in the current session without being saved as a persistent asset.

For an in-memory mesh, use **Create Editable Copy** to create a mesh asset, then save the scene.

### Shared Meshes

Multiple objects can reference the same mesh asset.

Editing that asset changes the geometry used by all objects that reference it.

Duplicating a GameObject does not automatically create an independent mesh. Use **Create Editable Copy** when the duplicate needs its own terrain shape.

MapBrush explicitly refreshes the selected object's MeshCollider. Do not assume that other objects sharing the mesh have also had their colliders refreshed.

### Prefabs

Assigning an editable copy to a prefab instance can create component overrides.

Save or apply those overrides according to your intended prefab workflow.

Editing a shared mesh asset can affect other instances that use the same asset.

## Limitations

- Editor-only sculpting workflow.
- One selected surface per editing operation.
- Requires the documented two-layer grid layout.
- No automatic topology detection or conversion.
- No mesh generation or subdivision.
- Brush detail is limited by the existing vertex density.
- No automatic seam stitching between separate surfaces.
- No texture painting, vegetation placement or material blending.
- No runtime sculpting or multiplayer synchronization.
- No Unity Terrain editing.

Protected borders and thickness preservation help maintain the intended structure, but they do not guarantee that extreme edits will avoid every possible mesh intersection.

## Troubleshooting

### The window reports an incompatible mesh

Check:

- Row Segments and Column Segments.
- Total vertex count.
- Bottom-first, top-second vertex ordering.
- Matching top/bottom positions.
- Mesh readability.

A matching vertex count alone does not prove compatibility.

### The vertex preview does not appear

- Select the object containing MapBrush Surface.
- Enable Show Vertex Preview.
- Confirm that the mesh passes the component's checks.
- Make sure you are viewing the Scene window.

### The brush does not affect the terrain

- Prepare the selected mesh for editing.
- Enable the brush.
- Confirm that you are outside Play Mode.
- Check that the MeshCollider is enabled and non-convex.
- Move the pointer over the top surface.
- Check whether the vertices under the brush are protected.
- Increase the brush radius if the mesh has widely spaced vertices.

### Sculpting looks angular

The mesh may have too few vertices for the desired detail.

MapBrush edits existing vertices; it does not add new ones. Use a denser compatible mesh if you need finer terrain features.

### Another object changes when I sculpt

Both objects probably reference the same mesh asset.

Create an editable copy for the object that needs an independent shape.

### The demo material appears pink

The sample material uses URP.

Check that your project is configured for URP or replace the sample material with one supported by your render pipeline.

### My changes disappeared after reopening

Check that the edited mesh was saved as a persistent asset and that the scene was saved with the correct mesh assignment.

Working geometry during a session does not, by itself, confirm that the mesh has been saved.

## Feedback and Bug Reports

Please use this repository's **Issues** section to report bugs or suggest improvements.

For bug reports, include:

- Unity version.
- Operating system.
- Render pipeline.
- Steps to reproduce the issue.
- Relevant Console messages.
- Mesh grid settings and vertex count.
- Screenshots where useful.

## Development

MapBrush was developed from a practical need to edit modular terrain sections in a Unity project.

Development included AI-assisted C# code generation with OpenAI ChatGPT/Codex, followed by manual integration and iterative testing by the author.

## License

See the repository's `LICENSE` file for the terms governing use, modification and redistribution.
