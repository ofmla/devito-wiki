Commit message format: ``<tag: message>``

PR title format: ``<tag: title>``

``tag`` should:

* start with a lower case letter
* be one of the following words:

  * dsl: obv. Note: fd, differentiable, etc -- they all belong to dsl
  * types: anything concerning types that are not exposed to the user API
  * compiler: compilation (operator, ir, passes, symbolics, ...)
  * arch: jit and architecture (basically anything in devito/arch)
  * tests: obv
  * examples: obv
  * builtins: obv
  * bench: anything related to benchmarking inside devito/benchmarks
  * misc: tools, docstring/comment updates, pep8 fixups, etc
  * mpi: MPI related (this might overlap with `compiler`)
  * gpu: GPU related (this might overlap with `compiler`)
  * ckp: Checkpointing related
  * sympy: sympy-related patch
  * ci: continuous integration-related
  * reqs: package dependence updates
  * install: related to installation (docker, conda, python ...)

``message`` should:

* start with an upper case letter
* start with a verb in first person
* be as short as possible