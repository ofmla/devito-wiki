dates: proposal: 29-30 April
time: TBD (edited) 

Agreed sprint items:
* Revise install instructions 
  - Change recommended install route to docker (ubuntu, mac, windows, test on pipelines)
  - Add/update troubleshooting section to capture gcc mac issue, etc

potential sprint items:
- finish off numpydoc-ization
- website cleanup (move “citing” to separate tab, update papers/talks (note: Rice talk is now online: http://rice2019oghpc.rice.edu/program/, revise logos, change link to installation instructions, TYPO contributers/contributors, link to right twitter page (@DevitoCodes; kill obsolete @OpesciProject ?), replace HORRIBLE BANNER
- MMS for current examples (TTI, elastic, etc)
- more examples customization (domain shape, simulation time, etc)
- cleanup `conftest.py`, rewriting all the tests using the horrible pre-allocated `t0`, `t1`, `a`, ... etc
- cleanup tutorials — many of the seismic one aren’t even tested at this stage
- documentation should live at www.devitoproject.org/…, *not* at www.opesci.org/…
- a cheatsheet (jupyter-notebook based) showing how to express different linear operators (and their adjoints) in devito
- opesci should be purged entirely.
- https://github.com/opesci/devito/issues/639
- performance regression (this would be awesome; enabled via USP subscription?)
- mirror sympy's gitter in our of our slack channels
- brazil tutorials public and through CI (BinderHub, requires repo2docker support)
- Overthrust setup for 3D RTM (host data and find copyright)
- Revise 2019-roadmap.