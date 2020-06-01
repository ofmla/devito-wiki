## Prioritized tasks 

1. **Performance benchmarks**
<br>Add performance benchmarks to continuous integration. This can include adding an image capturing performance on benchmark problems to the top level.

1. **Free surface boundary condition**
<br>Handle free surface boundary condition by *smartening* stencil behavior at boundary. Currently the free surface BC is handled by copying slices of wavefields per timestep into a mirror array with appropriate polarity reversal. This will is non optimal for performance and free surface modeling is critical to industry scale uptake. 

1. **GPU something something**
<br>Make it go fast

1. **Eikonal solver hackathon**
<br>Use Devito to implement a fast sweeping method TTI anisotropic eikonal solver. This has many valuable use cases including improving illumination compensation (potential improvements for FWI workflows), enabling "expanding box" (potential improvements in throughput especially for large problems), and the ability to generate traveltime tables.

## Un-prioritized tasks

* **mpi ```collect``` function**
<br>A ```Data``` method to collect and return MPI distributed wavefields not requiring MPI calls directly. e.g. ```u.collect()``` 

* **Test sincos** for trig in TTI system
<br>TTI systems often has ```sin(a) cos(a)``` nearby in mathematical expressions. The ```sincos``` function can be vectorized and is worth investigating for performance relative individual ```sin``` and ```cos``` calls.

