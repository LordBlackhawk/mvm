# Extending MVM

It shell be possible to extend MVM with new syntax and new semantic analysis (new infos, new warnings, or errors).

## Semantic Analysis

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

## Syntax Extension

The parser of the MVM compiler is extendable and there is an extension specially designed for parser definition. The following example defines parsers for the parser extension in itself:
```
class ParserExprNode;
parser new ParserExprListNode = (ParserExprNode:list)*;

enum PaserExprBracketType { ZeroOrOne, ZeroOrMore, OneOrMore }
parser ParserExprBracketType = [ "?" { result = ZeroOrOne; } | "*" { result = ZeroOrMore; } | "+" { result = OneOrMore; } ];

parser new ParserExprTerminalNode extends ParserExprNode = QuotedString:text ("_"_)?:expectWhitespace;
parser new ParserExprNonterminalNode extends ParserExprNode = ClassIdentifierNode:classIdentifier ("." Identifier:functionName)? (":" Identifier:resultIdentifier)?;
parser new ParserExprBracketNode extends ParserExprNode = "(" ParserExprListNode:exprs ")" ParserExprBracketType:type (":" Identifier:countIdentifier)?;
parser new ParserExprOrNode extends ParserExprNode = "[" ParserExprListNode:conditions ("|" ParserExprListNode:conditions)+ "]";
parser new ParserExprLambdaNode extends ParserExprNode = BlockStmntNode:block;

parser new ParserDefinitionNode = "parser"_ ("new"_)?:newClass ClassIdentifierNode:thisClass ("." Identifier:functionName)? ("extends"_ ClassIdentifierNode:baseClass)? "=" ParserExprListNode:exprs ";";
```
Explanation:
* `BlockStmntNode`, `ClassIdentifierNode`, and so one are nodes defined by the parser for the basic MVM language.
* `parser new (...);` is an abbreviation, which defines the resulting class. See the following example:
```
// Short form:
parser new ParserExprTerminalNode extends ParserExprNode = QuotedString:text ("_"_)?:expectWhitespace;
// Long form:
class ParserExprTerminalNode extends ParserExprNode {
  var text : string;
  var expectWhitespace : bool;
}
parser ParserExprTerminalNode = QuotedString:text ("_"_)?:expectWhitespace;
```
