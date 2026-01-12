# DrivAer_dafoam

# DrivAer CFD Simulation - OpenFOAM v10

## Overview
This repository contains an OpenFOAM case for aerodynamic simulation of the DrivAer reference vehicle geometry. The DrivAer model is a realistic open-source vehicle geometry developed by TU Munich for CFD validation studies.

## Configuration
- **Vehicle Configuration**: DrivAer notchback/fastback/estateback
- **OpenFOAM Version**: 10
- **Solver**: simpleFoam (steady-state RANS)
- **Turbulence Model**: k-omega SST
- **Reynolds Number**: ~5-7 million (based on vehicle length)
- **Reference Velocity**: 30 m/s (108 km/h)

## Prerequisites
- OpenFOAM v10
- ParaView (for visualization)
- STL geometry file of DrivAer model
- Minimum 16 GB RAM recommended
- Parallel computing capability (4-32 cores)

## Directory Structure
```
DrivAer/
├── 0/              # Initial and boundary conditions
├── constant/       # Mesh and physical properties
│   ├── polyMesh/
│   └── triSurface/
├── system/         # Solver settings and discretization schemes
│   ├── controlDict
│   ├── fvSchemes
│   ├── fvSolution
│   ├── snappyHexMeshDict
│   └── decomposeParDict
└── README.md
```

## Geometry Setup

### DrivAer STL Files
Place the following STL files in `constant/triSurface/`:
- `DrivAer_body.stl` - Main vehicle body
- `DrivAer_wheels.stl` - Wheels (rotating)
- `DrivAer_ground.stl` - Ground plane (moving wall)

Ensure geometry is in meters and properly oriented (x: longitudinal, z: vertical).

## Mesh Generation

### Domain Size
- **Upstream**: 2-3 vehicle lengths
- **Downstream**: 8-10 vehicle lengths  
- **Lateral**: 3-4 vehicle lengths
- **Height**: 4-5 vehicle heights
- **Ground clearance**: ~0.15 m (typical)

### Meshing with snappyHexMesh

1. **Generate background mesh**:
```bash
blockMesh
```

2. **Run snappyHexMesh**:
```bash
snappyHexMesh -overwrite
```

### Target Mesh Quality
- **Cell count**: 10-30 million cells
- **Base cell size**: 0.1-0.2 m
- **Surface refinement**: 5-7 levels
- **Boundary layer**: 10-15 prism layers
- **y+ target**: 30-100 (wall functions) or y+ < 1 (resolved)
- **Expansion ratio**: 1.1-1.2

## Boundary Conditions

### Typical Setup (`0/` directory)

| Boundary | U | p | nut | k | omega |
|----------|---|---|-----|---|-------|
| inlet | fixedValue (30 m/s) | zeroGradient | calculated | fixedValue | fixedValue |
| outlet | zeroGradient | fixedValue (0) | calculated | zeroGradient | zeroGradient |
| walls | movingWallVelocity | zeroGradient | nutkWallFunction | kqRWallFunction | omegaWallFunction |
| vehicle | noSlip | zeroGradient | nutkWallFunction | kqRWallFunction | omegaWallFunction |
| wheels | rotatingWallVelocity | zeroGradient | nutkWallFunction | kqRWallFunction | omegaWallFunction |
| symmetry | symmetry | symmetry | symmetry | symmetry | symmetry |

## Running the Simulation

### Serial Execution
```bash
simpleFoam > log.simpleFoam &
```

### Parallel Execution
```bash
# Decompose the case
decomposePar

# Run in parallel (example: 16 cores)
mpirun -np 16 simpleFoam -parallel > log.simpleFoam &

# Reconstruct results
reconstructPar
```

### Monitoring Convergence
```bash
# Monitor residuals
tail -f log.simpleFoam

# Plot forces
foamMonitor postProcessing/forces/0/forces.dat
```

## Post-Processing

### Force Coefficients
Forces are calculated using the `forces` function object. Reference values:
- **Reference area**: Frontal area (~2.16 m² for DrivAer)
- **Reference length**: Vehicle length (4.613 m)
- **Reference velocity**: 30 m/s
- **Reference density**: 1.225 kg/m³

Expected **Cd** range: 0.23-0.30 (configuration dependent)

### Visualization with ParaView
```bash
paraFoam
```

**Recommended plots**:
- Pressure coefficient on vehicle surface
- Velocity streamlines and wake structure
- Q-criterion isosurfaces for vortex visualization
- Surface shear stress
- y+ distribution

### Extract Force Coefficients
```bash
foamLog log.simpleFoam
gnuplot -e "plot 'logs/Cd_0' with lines"
```

## Validation

Compare results against experimental data:
- Wind tunnel measurements from TU Munich
- Reference: Heft et al. (2012), "Experimental and Numerical Investigation of the DrivAer Model"

## Troubleshooting

### Common Issues

**High residuals / divergence**:
- Reduce relaxation factors in `system/fvSolution`
- Check mesh quality with `checkMesh`
- Verify boundary conditions

**Poor boundary layer resolution**:
- Increase number of prism layers
- Adjust first layer thickness
- Check y+ values: `yPlusRAS` or `yPlusLES`

**Memory issues**:
- Reduce cell count in snappyHexMeshDict
- Use more cores for parallel decomposition

## References

1. Heft, A.I., et al. (2012). "Introduction of a New Realistic Generic Car Model for Aerodynamic Investigations"
2. OpenFOAM User Guide v10
3. DrivAer geometry: https://www.epc.ed.tum.de/en/aer/research-groups/automotive/drivaer/

## Author
Dominik  
Technical University of Munich  
[Your contact information]

## License
Specify your license here (e.g., GPL-3.0 for OpenFOAM compatibility)
