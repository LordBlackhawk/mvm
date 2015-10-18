# Module Concept of MVM
A MVM module is a collection of classes, methods, and variables with a unique name. The module is loaded within another module by `load [NAME];`

## Building a MVM Module
If compiling code which should be packed to a module instead of a executable, then put `module [NAME];` into the module code.
What happend within the compiler:
The compiler ...
* ... calls LLVM to optimize the user code.
* ... outputs the user code to a `module-[NAME]-user.bc`.
* ... builds are partial memory layout, see [optimizer concept](optimizer-concept.md).
* ... collects all public classes, methods, and variables and generates reflection informations for these public items.
* ... calls LLVM to optimize all code (user code, memory layout code, and reflection code).
* ... outputs the code to a `module-[NAME]-all.bc`.
* ... calls LLVM to compiles `module-[NAME]-all.bc` to `module-[NAME].so` (or `.dll` on windows).

## Loading a MVM Module
During the `load [NAME];` the compiler does the following steps.
The compiler ...
* ... looking for `module-[NAME].so` (or `.dll` on windows) in module search path (link include path of c/c++ compiler specified via command line options).
* ... loading `module-[NAME].so` in an OS-native way.
* ... calling some `registerModule` method, which is allowed to register additional parser, additional analysis, ... (see [extending MVM](parser-concept.md)).
* ... calling reflection code to add symbols definitions to search tree.
*Important:* All classes, methods, and variables of the module are usage at compile time!

## Linking a MVM Program against a MVM Module
During linking of a MVM program the following steps are done by the compiler (MVM does not distingish between linker and compiler):
The compiler ...
* ... collects all needed modules by the following steps:
    * Collect all externally called LLVM-functions.
    * Find modules which define this LLVM-functions.
    * Load `module-[NAME]-user.bc`.
    * Start again until no new modules have to be loaded.
* ... change all `external` LLVM-functions to `internal` (to enable further optimizations).
* ... calls LLVM to optimize all user code.
* ... builds a overall memory layout, see [optimizer concept](optimizer-concept.md).
* ... calls LLVM to optimize all code (user code and memory layout code).
* ... outputs the code to `program-with-mvm-modules.bc`.
* ... calls LLVM to compile `program-with-mvm-modules.bc` and link with additional libraries coming from outside of MVM.