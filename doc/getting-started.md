# Getting started

## First MVM Compiler
The first MVM compiler should be written in C++. This C++ getting started compiler is allowed to not support the following MVM features.
The C++-variant of the MVM compiler ...
* ... is not [extendable](parser-concept.md) itself, but generates extendable code.
* ... does not support [loading modules](module-concept.md), but generates reflections for the generated code.
* ... does not support execute command.
* ... has only one purpoose: Compiling the second MVM compiler.
* ... does not support operator and method overloading.
* ... does not support other integer types than int.
* ... does not support any floating point types.

## Second MVM Compiler
The second MVM compiler should be written in MVM itself. It consists of the following components:
* MVM compiler core (written in MVM, but not using full MVM features)
* MVM runtime (written in C++, reusing some parts of the MVM compiler written in C++, hand written MVM module interface)
* LLVM interface (written in C++, reusing some parts of the MVM compiler written in C++, hand written MVM module interface)

