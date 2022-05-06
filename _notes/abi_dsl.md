# Spack Application Binary Interface Domain-Specific Language (ABI DSL)

## Goals

- Splicing in new spec should satisfy all symbols.

## Possible Functions

`abi_spec()` Called when a splice is requested. Only called on concrete spec. Recursive check on entire DAG.

`abi_match()` Could return a list/range all specs satisfied by given spec. Similar to provides.

Infers abstract constraints.  
Writes edge attributes.

## Package Requirements

- Which dependencies matter for ABI.
- Package needs to state which variants matter and in which direction.

## Use-cases

- Swapping MPIs
- Using MPITrampoline and wi4mpi
- transitive dependencies and ABI breaks
- transitive ABI exposure
  - nholmann json exposes its boost version to deps
- diamond ABI compatibility: if you replace C with C' and C' requires D', D'
must be compatible with D
```
     A
    / \
   B   C
    \ /
     D
```
- Splice MKL in for BLAS (i.e. libnames are totally different)
- [zlib](https://abi-laboratory.pro/?view=timeline&l=zlib)
- Less stable examples:
  - [Cap'nProto](https://abi-laboratory.pro/index.php?view=timeline&l=capnproto)
  - [hdf5](https://abi-laboratory.pro/index.php?view=timeline&l=hdf5)

## Key Caveats

- Mixed languages: i.e. a lua dependency doesn't break C ABI, but has an internal ABI break.
- You may not care about variants or they may be critical.
- Need to know when dependencies affect abi.
- Adding variants might affect ABI but subtracting doesn't, and vice-versa.

## ABI directives

```python
class MyPackage(Package):
    def canonicalize(self, spec):
      return add_compiler_and_runtime_info(spec)
		def get_abi_ver(self, spec):
      # Should return spec and whether or not variants, deps
      # matter.
    	return version[:1]
    # need a dependency listed?
		abi_propagates(build=True, run=True, link=True)
		abi_backward()
```

## Caveats with Variants and Dependencies

`boost+graph+json` satisfies `boost`, `boost+graph`, and `boost+json`  
Similar for "without".

If asked about an unknown variant, does not satisfy/trigger rebuild.

`graph=*` is "I don't care."

1. you have a `boost` that was built with `+graph` but the package doesn't know about it.
2. You want to splice in a `boost~graph`.
3. This will work, but it may cause issues.
    - because `boost~graph` satisfies `boost`
    - (query satisfies requirement)

1. You have a `boost~graph`
2. You want to splice in a `boost+graph`.
3. This will fail, but it shouldn't.

Requirements of the dependents are really what matters.

## Design features

- Should be directional
- Ideally use specs and don't create a new model.

## Related Functionality

- Abstract constraints on all edges. Can be inferred from `package.py` `depends_on(target)`.

## Related Work

Inferred types  
Parameterized types  
Type checkers
Covariants and contravariants of return types in Java. (e.g. Caller has to be compatible with return type.)

## Proposed Paper Titles

Expressing ABI in package managers.  
