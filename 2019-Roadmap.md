#### Things that we would like to see _on top of_ Devito

* ? [easy/medium/hard/challenging/...]

#### Things that we would like to have _in_ Devito

* Code generation for GPUs (hard)
  * The long-term plan is to integrate [OPS](https://github.com/OP-DSL/OPS), just like we did for [YASK](https://github.com/intel/yask) (see `yask` backend in the Devito compiler)
  * However, we are keen to explore other (simpler?) alternatives. For example, OpenMP 4's offloading for GPUs? our own CUDA-based backend? other third-party tools?
  * We also have other issues to deal with. How do we efficiently implement sparse computations such as source injection? 


#### Experimentations that we would like to run -- "is it worth implementing it in the Devito compiler?"

* ? [easy/medium/hard/challenging/...]

#### Misc projects

* Implement _nummpi_, a lightweight package to use NumPy over MPI (hard)
  * Devito supports MPI. One of the key features of the implementation is the introduction of a new type, called `Data`, a subclass of `numpy.ndarray`, to manage data distributed over a set of MPI processes.
  * In the test suite, there are lots of self-containted examples showing how `Data` works, both sequentially (almost a classic `numpy.ndarray`) and with MPI. 
  * The `Data`-related stuff could (should?) be extracted from Devito into its own package (_nummpi_?), and clearly extended to implement as much `numpy.ndarray` API as possible.
  * Various things to consider. What happens when the user:
    * Assigns a `Data` to another `Data`?
    * Assigns a `Data` to a `numpy.ndarray`?
    * Assigns a (globally replicated) `numpy.ndarray` to a `Data`?
  * [DASK array](http://docs.dask.org/en/latest/array.html) does something conceptually similar, but here we actually would like to have:
    * A lightweight, self-contained package, that anyone can easily pip-install
    * An actual subclass of `numpy.ndarray`, not a wrapper
    * A very simple API, extremely close to that of a `numpy.ndarray`
  * Good news: some code and tests can be lifted from Devito


#### Issues on our GitHub issue tracker that we'd like to go ASAP

* #640 [challenging, requires deep knowledge of how the Devito compiler works]
* #639 [don't remember what's going on]
* #634 [simple, but boring]
* #608 [don't remember what's going on, but it should be a clearly localised issue]
* #606, #549, #520 [simple. These are a perfect execuse for someone that wants to learn about Devito and contribute immediately; we need some Jupyter-notebook-based tutorials describing how certain things work in Devito]
* #405 [simple. Like before, but this is more focused on "things on top of Devito".]


#### Wishful thinking, but we'd really love to...

* [NumPy-like docstrings](https://numpydoc.readthedocs.io/en/latest/format.html), with an example provided [here](https://sphinxcontrib-napoleon.readthedocs.io/en/latest/example_numpy.html#example-numpy)


#### We are open to ideas and keen to support you !

* Just share them with us :)