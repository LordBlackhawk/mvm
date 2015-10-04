# Extending MVM

It shell be possible to extend MVM with new syntax and new semantic analysis (new infos, new warnings, or errors).

An example for new semantic analysis is a warning for comparing boolean values with "==" or "!=" (although I dont know why this should be a good idea...):
```
load mvm;

fun mvm.BinaryNode.semAnaExpr(self, context : mvm.Context) -> mvm.AnaylsedExprNode
{
  var res = super(context);
  if ((self.operator == COMPARE_EQUAL) || (self.operator == COMPARE_NOT_EQUAL)) {
    if (self.leftExpr.semAnaExpr(context).type == bool.class) {
      self.addWarning("Do not use == or != with boolean typed expressions.");
    }
  }
  return res;
}
```
Explanation:
* The module "mvm" is the core compiler.
* All functions in MVM can be overwritten, the `super(...)` calls the overwritten version of this function defined in the mvm module.
* And yes, the call to `semAnaExpr(...)` is bad, since `super(...)` should have done this before, but this is demo code!
* Speaking of language syntax and interfaces, here we use the interface to the mvm compiler and require that the classes `mvm.BinaryNode`, `mvm.Context`, `mvm.AnaylsedExprNode` and the function `semAnaExpr(...)` are unchanged. This is the cost of extendability: Basic interfaces of the compiler have to be stable, otherwise it would work.
