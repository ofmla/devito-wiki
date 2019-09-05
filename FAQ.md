### I create an Operator, look at the generated code, and the equations appear in a different order than I expected.

The Devito compiler computes a topological ordering of the input equations based on data dependency analysis. Heuristically, some equations might be moved around to improve performance (e.g., data locality). Therefore, the order of the equations in the generated code might be different than that used as input to the Operator.

### How can I see the compilation command with which Devito compiles the generated code ?

Set the environment variable `DEVITO_DEBUG_COMPILER=1`. When an Operator gets compiled, the used compilation command will be emitted to stdout.

### How can I change the compilation flags (for example, I want to change the optimization level from -O3 to -O0) ?

There is currently no API to achieve this straightforwardly. However, there are three work arounds:

* hacky way: change the flags explicitly in the Devito source code. In Devito v3.5, you can do that [here](https://github.com/opesci/devito/blob/v3.5/devito/compiler.py#L146)
* via env vars: use a [CustomCompiler](https://github.com/opesci/devito/blob/v3.5/devito/compiler.py#L444) -- just leave the `DEVITO_ARCH` environment variable unset or set it to `'custom'`. Then, `export CFLAGS="..."` to tell Devito to use the exported flags in place of the default ones.
* programmatically: subclass one of the compiler classes and set `self.cflags` to whatever you need. Do not forget to add the subclass to the [compiler registry](https://github.com/opesci/devito/blob/v3.5/devito/compiler.py#L475). For example, you could do

```
from devito import configuration, compiler_registry
from devito.compiler import GNUCompiler

class MyOwnCompiler(GNUCompiler):
    def __init__(self, *args, **kwargs):
        super(MyOwnCompiler, self).__init__(*args, **kwargs)
        # <manipulate self.cflags here >

# Make sure Devito is aware of this new Compiler class
compiler_registry['mycompiler'] = MyOwnCompiler
configuration.add("compiler", "custom", list(compiler_registry), callback=lambda i: compiler_registry[i]())

# Then, what remains to be done is asking Devito to use MyOwnCompiler

configuration['compiler'] = 'mycompiler'
```
