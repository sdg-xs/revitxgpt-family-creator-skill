# Implementation guidance

## Graph and portability

Modern Dynamo connectors use **output/input PORT IDs** in `Start` and `End`, not node IDs. Validate semantic wiring: directories to creator inputs 0/1, Boolean to input 2, creator output to result Watch and preview Python, preview output to Watch 3D. Use unique IDs and Manual mode.

The example uses `PythonNet3`, `CoreNodeModels.Input.Directory`, `CoreNodeModels.Input.BoolSelector` and `Watch3DNodeModels.Watch3D`. Check availability in the target installation. Embed runtime code/data; the delivered graph must not depend on the skill folder. Use a common geometry description for family and preview. Keep product-specific assumptions out of new products.

## Diagnose and correct modeling through code

Start from the observed defect and identify which stage produced it. Compare source measurements, geometry descriptions, generated native geometry and preview output. For wrong proportions or floating parts, revise dimensions and assembly placement. For deformed curves, failed solids or unexpected API results, isolate the failing construction and verify the actual API contract. For missing geometry or controls, inspect graph connections and output conversion. Make the smallest coherent correction that resolves the cause, regenerate and verify the affected behavior plus existing dimensional and family/type requirements.

Use user review to refine both fidelity and usability over successive runs. Preserve proven fixes in reusable code and checks when applicable, without expanding one local bug into a universal modeling restriction. The following is one example of an API mismatch, not the central purpose of this skill.

### Example: different arc conventions

```python
# Dynamo: points in sequence along the arc.
path = Arc.ByThreePoints(start, middle, end)
assert path.EndPoint.DistanceTo(end) < 0.01

# Revit: endpoints, then an interior point.
arc = DB.Arc.Create(start, end, middle)
```

Copying Revit's ordering into Dynamo once generated oversized loops. Endpoint and length checks in the native engine caught the defect; an independent renderer had masked it. Apply the same principle to other API differences, using checks appropriate to the intended geometry.

## Family creation

- Accept all Revit 2027 point releases; check installed API availability before claiming compatibility.
- Start with an empty non-hosted Generic Model template and set a compatible category. Specialty Equipment's enum is `OST_SpecialityEquipment`; use the category appropriate to the actual product.
- Run from a project, create a separate family document, use explicit transactions, roll back on failure and close safely. Include the failing component name in errors.
- Convert millimeters with `UnitUtils`; verify actual family bounds after regeneration. Nominal parameter values alone do not establish geometry dimensions.
- Create the desired type before deleting template types; Revit cannot delete the last type. Verify exactly one remains.
- Assign material parameters or IDs. For explicit shading RGB, set material color and disable render-appearance-driven shading. This does not verify photorealistic appearance assets.
- Native extrusions, `CreateSweptGeometry`, `CreateLoftGeometry` and `FreeFormElement.Create` are useful options. Check profile planarity, closure, intersection, consistent loft section ordering and `Application.ShortCurveTolerance`. Verify sweep profile orientation and attachment parameter.
- Save only after geometry, type and transaction checks. Preserve existing files. Avoid unsupported setters such as `GenericForm.Name`.

## Assembly and evidence

Derive checks from the product's use. AirBike examples: saddle/rail/post connections, pedal/crank/linkage endpoint agreement, opposed crank arms, equal rod lengths, fan clearances, grounded feet and an attached console. They are not requirements for unrelated products. Inferred assembly details support a plausible static visual model, not factory mechanism accuracy.

The example retains its agreed PDF envelope, 1300 × 610 × 1240 mm; manufacturer tables conflicted. These dimensions are product-specific. It contains a fixed pose and visible finish colors, without lettering or a removable bottle.

Development checks covered wiring, syntax, preview-only behavior, file preservation, geometry-definition bounds, profiles and selected assembly relationships. All 133 curved paths passed native Dynamo endpoint/short-arc checks. Full native solid tests crashed in the standalone host; final in-app Watch 3D and RFA creation remain unverified. User visual acceptance is not evidence of complete API execution. Preserve these limits when reusing the example.

When debugging, reproduce the reported error before changing geometry. If native testing is unavailable or crashes, state that limitation and preserve the deliverable; an offline illustration does not replace native verification.
