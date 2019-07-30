The `make-pbs.py` program can be used to quickly generate PBS file to run (OpenMP, MPI, MPI+OpenMP) benchmarks. The program is located under `benchmarks/user`. For example, executing it as

`python make-pbs.py generate --load anaconda3/personal --load intel-suite --load mpi -nn 1 -nn 2 -nn 4 -nn 8 -ncpus 24 -mem 120 -np 2 -nt 12 -P acoustic --shape 600 600 600 -so 12 --tn 100 --arch haswell --mpi basic -r ~/opesci/results`

One obtains 4 files (one for each `nn` argument), which look like as below

```
...
```