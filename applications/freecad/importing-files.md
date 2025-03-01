## Importing Files into FreeCAD

### STLs and Most Supported 3D Mesh File Types
1. Open the "Mesh" workbench
1. "File" -> "Import" -> Choose the file to import
1. Click the "Meshes" menu -> "Analyze" -> "Evaluate and repair mesh..."
    a. Select the imported mesh from the drop down
    b. Run all of the mesh checks, attempt auto repair, or use the Mesh tools to manually repair
1. Switch to the "Part" workbench
1. Select the imported object, click the "Part" menu -> "Create shape from mesh..."
    a. Check the "Sew shape" box, try a tolerance of 0.01 to start
    b. Click the "OK" button
1. Select the imported object, click the "Part" menu -> "Check geometry" -> "Run check" button
    a. Address any reported errors, you can switch back to the Mesh toolbox and use those tools
    b. If no errors, click the "Close" button
1. Select the imported object, click the "Part" menu -> "Create a copy" -> "Refine shape"
1. Select the refined copy -> "Part" menu -> "Convert to solid"