# DrivAer DAFoam Case

CFD optimization case for the DrivAer reference vehicle geometry using DAFoam.

## Prerequisites

- DAFoam (OpenFOAM v1812)

## Geometry

Download the DrivAer geometry from:
https://syncandshare.lrz.de/getlink/fiWzvig1b2BhKK2YQQW3Cs/

Place the geometry files in the appropriate directory before meshing. (constant/triSurface)

## Usage

### Mesh Generation

Run the preprocessing script to generate the mesh:

```bash
./preProcessing
```

### Running the Case

After meshing is complete, run the optimization:

```bash
./runScript.sh
```

## Case Description

This case performs aerodynamic shape optimization on the DrivAer vehicle geometry using adjoint-based optimization methods provided by DAFoam.

## TODOS

- Check convergence with current mesh
- Check Y+ with current mesh
- Add inflation layers if needed
- Improve Mesh quality (maybe edge refinement...)
