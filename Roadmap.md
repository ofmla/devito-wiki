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

* **remainder spatial loops**
<br> generated code has 4 calls to spatial loop functions per time step, for handling:
x-interior, y-interior, x-interior, y-remainder, x-remainder, y-interior, x-remainder, y-remainder.
<br>These could be replaced with a single call to the function that used a min or ternary function for the loop traversal.
old space loop (with 1 interior and 3 remainder calls):
```
#pragma omp parallel num_threads(nthreads) private(r36,r37,r38)
  {
    #pragma omp for collapse(2) schedule(dynamic,1)
    for (int x0_blk0 = x_m; x0_blk0 <= x_M; x0_blk0 += x0_blk0_size) {
      for (int y0_blk0 = y_m; y0_blk0 <= y_M; y0_blk0 += y0_blk0_size) {
        for (int x = x0_blk0 - 4, xs = 0; x <= x0_blk0 + x0_blk0_size + 2; x += 1, xs += 1) {
         for (int y = y0_blk0 - 4, ys = 0; y <= y0_blk0 + y0_blk0_size + 2; y += 1, ys += 1) {
            #pragma omp simd aligned(p0:32)
            for (int z = z_m - 4; z <= z_M + 3; z += 1) {}
          }
        }
        for (int x = x0_blk0, xs = 0; x <= x0_blk0 + x0_blk0_size - 1; x += 1, xs += 1) {
         for (int y = y0_blk0, ys = 0; y <= y0_blk0 + y0_blk0_size - 1; y += 1, ys += 1) {
            #pragma omp simd aligned(b,p0,vel,wOverQ:32)
            for (int z = z_m; z <= z_M; z += 1) {}
          }
        }
      }
    }
  }
```
new space loop (single call):
```
#pragma omp parallel num_threads(nthreads) private(r36,r37,r38)
  {
    int xmax, ymax;
    #pragma omp for collapse(2) schedule(dynamic,1)
    for (int x0_blk0 = x_m; x0_blk0 <= x_M; x0_blk0 += x0_blk0_size) {
      for (int y0_blk0 = y_m; y0_blk0 <= y_M; y0_blk0 += y0_blk0_size) {
        xmax = min((x0_blk0 + x0_blk0_size + 2), x_M);
        ymax = min((y0_blk0 + y0_blk0_size + 2), y_M);
        for (int x = x0_blk0 - 4, xs = 0; x <= xmax; x += 1, xs += 1) {
          for (int y = y0_blk0 - 4, ys = 0; y <= ymax; y += 1, ys += 1) {
            #pragma omp simd aligned(p0:32)
            for (int z = z_m - 4; z <= z_M + 3; z += 1) {}
          }
        }
        xmax = min((x0_blk0 + x0_blk0_size - 1), x_M);
        ymax = min((y0_blk0 + y0_blk0_size - 1), y_M);
        for (int x = x0_blk0, xs = 0; x <= xmax; x += 1, xs += 1) {
          for (int y = y0_blk0, ys = 0; y <= ymax; y += 1, ys += 1) {
            #pragma omp simd aligned(b,p0,vel,wOverQ:32)
            for (int z = z_m; z <= z_M; z += 1) {}
          }
        }
      }
    }
  }
```