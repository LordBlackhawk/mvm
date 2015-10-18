# Optimizer Concept
The optimizer concept for MVM consists of two parts: An abstraction layer to be able to do memory layout optimizations and user-define optimization rules.

## Memory Layout and Memory Layout Optimizations
The MVM language does not allow to specify the memory layout of a class, but the only allows to tell the compiler: There should be a variable for each class instance. Do what you want.
Example:
```
class Node;
class Edge;

var Node.connections : Collection<Edge>;
var Edge.left : Node;
var Edge.right : Node;

fun calculateReachableNodes(start : Node) -> Collection<Node>
{
    var Node.visited = false;
    var result : Collection<Node>;

    fun visit(node : Node) {
        if (node.visited)
            return;
        node.visited = true;
        result.add(node);
        for (var edge : node.connections) {
            visit(edge.left);
            visit(edge.right);
        }
    }

    visit(start);
    return result;
}
```
In the example code there is a method local variable for the class `Node`. In C++ a possible implementation would be `std::map<Node*, bool>` (or better `std::set<Node*>`), which is used to save the value of the visited variable. But there are also other possiblities to implement this, e.g., the struct `Node` (C++ view) could have a `visitied` field of type bool. At the beginning of the method `calculateReachableNodes` this variable could be set to false for all `Node` instances (This optimization is possible, since `calculateReachableNodes` is not reentrant).
The frame work of MVM should be able to chose both of the suggested C++ implementations for the code above (without the user changing any line of code!). This is possible since the memory layout of classes is not until all user code is known and the optimizer can analyse the complete code (there is a variant where only the complete module is known and not that aggressive optimizations can be applied).
From optimizing C++ code it is known that optimizing the performance, often requires to change the data structures the code is working on. In C++ this means that the user has to change code. In MVM we try to enable the compiler to change/do the memory layout by its own and therefore be able to do optimizations unthinkable for C++ compilers.

(TODO: REWRITE COMPLETE SECTION!)

## Helping MVM Compiler to chose "Good" Memory Layout
C++ programmer known that chosing a good memory layout depends havily on the use cases. E.g. think of two different use cases; for each of the use cases there is an optimal memory layout, but for the other use case this layout is the worst case. In this case it is important to know which use case is used more often.
Therefore, it should be possible to tell the MVM compiler what is important.
To get this kind of informations, the MVM compiler should be able to generate analysis code additional to the memory layout. If running the prepare code on an test suite, we should get informations, which use case of a class is important for the overall performance and therefore do an optimized memory layout with the help of this analysis informations.

(TODO: REWRITE COMPLETE SECTION!)

## User-Defined Optimization Rules
The MVM languate should provide a way to define user defined optimization rules.
Example:
```
class StackItem;
class Stack;
fun Stack.push(item : StackItem);
fun Stack.pop() -> StackItem;
optimization(s : Stack, x : StackItem, y : StackItem) { s.push(x); y = s.pop(); } { y = x; }
```
The syntax is none-sense, but hopefully the idea is clear. It shell be possible to tell the compiler that a call to `push` and then to `pop` can be replace by the value pushed.

The optimizations are done on the public interface of a module, before including the module code itself.

The idea behind user-define optimizations are the same as for memory layout. This shell be possible to do performance optimizations without changing a lot of code. In the first case the user does not have to change any code, but only supply test case the be able to value the code concerning importance for performance. In this case the user has to supply abstract rules that the implementation of e.g. a class should statisfy.

Example:
You are analysis performance of your code and find that `A(...); B(...);` is used often, but for performance it would be better to use `C(...); D(...);`, then you have different possiblities:
* Try to find all occures of `A(...); B(...);` and replace by hand by `C(...); D(...);`.
* Add a new optimization telling the compiler to do this optimization for you.

The second way has different advantages:
* It is a local change in code.
* If the code evoles and e.g. `A(...);` get additional duties, the optimization may be not correct anymore. Removing the optimization is just a local change.
* If the optimziation is wrong, automatic error deduction can help you finding the error: If you have an test case showing the problem, you can perform the test case with different combinations of optimizations to find the optimization break the test case (This is not a new idea, I think llvm is already using something like this).

Hopefully, MVM can help to write readable code, which only changes if feature change. Additionally there is code, which helps the compiler to generate optimized code. This is more natural development process: First implement the feature itself, and afterwards if the performance is bad, improve it. In the next cycle you may implement a new feature, which makes some optimizations wrong; no problem just remove the optimizations and add new ones afterwards, if performance is too bad.
