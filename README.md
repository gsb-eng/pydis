
[![Build Status](https://travis-ci.com/gsb-eng/bytecode_tools.svg?branch=master)](https://travis-ci.com/gsb-eng/bytecode_tools)
[![Coverage Status](https://img.shields.io/codecov/c/github/gsb-eng/bytecode_tools/master.svg)](https://codecov.io/github/gsb-eng/bytecode_tools?branch=master)
<br />

Bytecode Tools
===============

Bytecode tools are combination of multiple necessary modules to play with Python
bytecode. Most of the bytecode related modules are version specific and they won't support
other python versions.

Bytecode won't be same across versions, new opcodes will be added or opcodes will be removed or modified, so python standard library modules won't work with other version generated bytecodes.

Let's say `marshal` module, it's purpose is to `serialize` or `deserialize` code objects, they are version specific. Like wise `dis` module, there is a heavy difference in disassembling `bytecode` to `wordcode` AND opcode differences make this version incompatible.

Our goal is to make `bytecode_tools` work with any `Cpython` version, Aim is to build below tools.

|Tool|Purpose|Status|
|---|---|---|
|unmarshal|deserialize the code obejcts.|Completed|
|pydis|Disassembler for any Cpython version.|completed|
|pycdecode|Decoder for bytecode chache files (`pyc` files)|Completed|
|hackdis|Hack the disassembled bytecode|WIP|
|decompiler|A decompiler for Cpython x.x|Planned|


# How to deal with bytecode_tools?

Install
=======

1. Clone this repo and `python setup.py install` or use `pip install .`

2. Install directly from PyPi `pip install bytecode-tools`


Usage
======

pydis
=====

`pydis` is a python disassembler, it can be a drop in replacement for cpython's
`Lib/dis.py`.

Pydis supports all the cpython versions above 2.5, every verion above 2.5
supports other versions. This means, pydis decodes 2.6 bytes code in 3.6 and
vice versa.

Why pydis?
==========

Python's `dis` moduel is super helpful for looking inside code objects, but it
won't support other python versions. If the code object is created through
`python 3.5` and try to disassemble with `python3.6`, it won't work.

Each python version gets changes to opcodes, there will be new ones added and few
are deleted. Unless you recreate the code object with new python version, the
same code object can't be interpreted with old versions.


Disassemble a statement.

    >>> from bytecode_tools import pydis
    >>> pydis.dis("a=1")
    1           0 LOAD_CONST               0 (1)
                2 STORE_NAME               0 (a)
                4 LOAD_CONST               1 (None)
                6 RETURN_VALUE

Disassble a function object.

    >>> def foo():
          print(123)
          a = 1
          b = 2
          c = a + b
          return c

    >>> pydis.dis(foo)
      2           0 LOAD_GLOBAL              0 (print)
                  2 LOAD_CONST               1 (123)
                  4 CALL_FUNCTION            1
                  6 POP_TOP

      3           8 LOAD_CONST               2 (1)
                 10 STORE_FAST               0 (a)

      4          12 LOAD_CONST               3 (2)
                 14 STORE_FAST               1 (b)

      5          16 LOAD_FAST                0 (a)
                 18 LOAD_FAST                1 (b)
                 20 BINARY_ADD
                 22 STORE_FAST               2 (c)

      6          24 LOAD_FAST                2 (c)
                 26 RETURN_VALUE


To get the all the bytecode instructions.

    >>> pydis.instructions(foo.__code__)
    [LOAD_GLOBAL, LOAD_CONST, CALL_FUNCTION, POP_TOP, LOAD_CONST, STORE_FAST, LOAD_CONST, STORE_FAST, LOAD_FAST, LOAD_FAST, BINARY_ADD, STORE_FAST, LOAD_FAST, RETURN_VALUE]

Like wise other `dis` module options are available with `pydis`.

pyc decoder?
============

what are `pyc` files?
=====================

`pyc` files are python bytecode cache files, they will be used in `eval` loop at the `interpretation` stage. `Compilation` phase output gets serialized into `pyc` file along with some meta data to identify the python version and the time it was created. `pyc` files are heavily version specific, this means `pyc` file generated from one version of python won't be `intrepreted` with other version.

The meta data which has been serialized into to `pyc` files are identifiers for invalidating the `pyc` file incase if the source file is newer or the interpreter version has changed. This is mailny because of the incompatibility of the bytecode across versions. New `opcodes` will be added or deleted from version to version.

Bytecode tools `pycdecoder` helps with deserializing bytecode object and parse the metadata with any version of `cpython`.

Let's say you've a python file `test.py` with the below statements.

    a = 1
    print(a)

If you do `python -m test.py`, based on the versioon that you use. You'll get a `test.pyc` or `__pycahce__/test_cpython-37.pyc` (From pyhton3 pyc files are cached in `__pycahce__` dir).

I'm using python37 here, I've got the cache file under `__pycahce__`

    >>> from bytecode_tools import pycdecode
    >>> pycdecode.showpyc('<path_till_here>/__pycache__/test_pycdecoder.cpython-37.pyc')

    Magic     :  3394
    timestamp :  2019-05-23 02:28:58
    Size      :  15
    Bytecode  :
      1           0 LOAD_CONST               0 (1)
                  2 STORE_NAME               0 (a)

      2           4 LOAD_NAME                1 (print)
                  6 LOAD_NAME                0 (a)
                  8 CALL_FUNCTION            1
                 10 POP_TOP
                 12 LOAD_CONST               1 (None)
                 14 RETURN_VALUE

