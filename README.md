[![Codacy Badge](https://api.codacy.com/project/badge/Coverage/1f01028dd9db4231b24eab75934a2231)](https://www.codacy.com/app/krystophny/fffi?utm_source=github.com&utm_medium=referral&utm_content=krystophny/fffi&utm_campaign=Badge_Coverage)

`fffi` is a tool to transparently use shared libraries generated by Fortran in
Python with NumPy arrays. It is based on CFFI and currently assumes the use of
gfortran.

The focus of `fffi` is dynamical automatic generation of interfaces directly
within Python with a minimum of extra code generation and files to allow
for the simplest possible workflow for fast prototyping of Python/Fortran codes.
In this sense it complements [F2x](https://github.com/DLR-SC/F2x) which
instead supplies interfaces based on generated code and ISO C bindings.
Both tools aim to be more powerful and flexible alternatives to `F2Py`.

The following example can be found in `tests/01_arrays` directory and shows
basic usage using CFFI (fast) API mode with a shared library
`libtest_arrays.so` containing a module `mod_arrays`:
1. Import fffi and initialize `fortran_module` object `mod_arrays`
```python
from fffi import fortran_module
mod_arrays = fortran_module('test_arrays', 'mod_arrays')
```

2. Define and generate Python extension
  (only on first run or if library routines have been added/changed)
```python
mod_arrays.fdef("""
  subroutine test_vector(vec)
    double precision, dimension(:) :: vec
  end

  subroutine test_array_2d(arr)
    double precision, dimension(:,:) :: arr
  end
""")
mod_arrays.compile()
```
Internally, C function headers are generated for CFFI with the required
representation of array descriptors by the used Fortran compiler.

3. Load interface module to library
```python
  mod_arrays.load()
```
4. Calling of a subroutine `test_vector` in `mod_arrays` is as simple as
```python
  vec = np.ones(15)
  mod_arrays.test_vector(vec)
```

The general workflow to extend such a test is:

1. Possibly add new testing routines to `mod_arrays.f90` and `test_arrays.f90`
2. Run `make`. This generates a shared library `libtest_arrays.so` and
   a Fortran executable `test_arrays.x` for reference output
3. Edit and run `test1_arrays.py`

Current status:

* Code generation and invocation is transparent by encapsulation in Python class
* CFFI API mode is used (pre-generate C extension module as a wrapper)
* Subroutine signature definition in Fortran via `fdef` like CFFI `cdef`
* Array descriptor structs are working with gfortran 5, 6, 7, 8, 9

Next steps:

* Add flexibility with regard to types, floating-point precision, etc.
* Test numpy.array views on existing data and memory management behavior
* Allow for ABI mode in addition to API mode, and static library API mode
* Add support for Intel and PGI in addition to GNU Fortran compiler
* Test in real world applications
