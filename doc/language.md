# Language Description of MVM
The MVM language itself consists of the following syntax groups:
* Expressions (operator, new, mux, ...)
* Statements (expr, var, if, if-else, while, ...)
* Primary Commands (fun, class, var, execute, ...) (Maybe "Primary Commands" should be called different...)
* Parser-Extension

The following sections describe the syntax for each group. The syntax is written down in the language of the parser extension itself (therefore, it is maybe buggy MVM code itself). And explanations behind.

*Important:* There is room for improvement!!!

## Expressions
```
class ExprNode;

parser new ParenthesesExprNode extends ExprNode = "(" ExprNode:expr ")";
parser new StringLiteralExprNode extends ExprNode = QuotedString:text;
parser new IntegerLiteralExprNode extends ExprNode = ;
parser new FloatLiteralExprNode extends ExprNode = ;
parser new IdentifierExprNode extends ExprNode = Identifier:name;
parser new LambdaExprNode extends ExprNode = FunctionDefinitionNode:definition;
parser new NewExprNode extends ExprNode = "new"_ ClassIdentifierNode:classId;

enum OperatorType {
    /* Unary Prefix: */     Not, Minus, PrePlusPlus, PreMinusMinus,
    /* Unary Postfix: */    PostPlusPlus, PostMinusMinus,
    /* Binary: */           And, Or, Add, Sub, Mul, Div, Mod, AndAssign, OrAssign, AddAssign, SubAssign, MulAssign, DivAssign, ModAssign, ...
    /* Name Operator: */    StructAccess /*.*/, PointerAccess /*->*/, ClassAccess /*::*/,
    /* Multi Operator: */   Parentheses /*()*/, Brackets /*[]*/, CurlyBrackets /*{}*/, AngleBrackets /*<>*/,
    /* Special Operator: */ CastOp, MuxOp
}

parser OperatorType.parseUnaryPrefix = [ "!" { result = Not; } | "-" { result = Minus; } | "++" { result = PrePlusPlus; } | "--" { result = PreMinusMinus; } ];
parser OperatorType.parseUnaryPostfix = [ "++" { result = PostPlusPlus; } | "--" { result = PostMinusMinus; } ]
parser OperatorType.parseBinary = [ "&&" { result = And; } | ... ];
parser OperatorType.parseNameOperator = [ "." { result = StructAccess; } | "->" { result = PointerAccess; } | "::" { result = ClassAccess; } ];
parser OperatorType.parseMultiBegin = [ "(" { result = Parentheses; } | "[" { result = Brackets; } | "\{" { result = CurlyBrackets; } | "<" { result = AngleBrackets; } ]
parser OperatorType.parseMultiEnd = [ ")" { result = Parentheses; } | "]" { result = Brackets; } | "\}" { result = CurlyBrackets; } | ">" { result = AngleBrackets; } ]
parser OperatorType = [ OperatorType.parseUnaryPrefix:result | OperatorType.parseUnaryPostfix:result | OperatorType.parseBinary:result | OperatorType.parseNameOperator:result | OperatorType.parseMultiBegin:result OperatorType.parseMultiEnd:typeEnd { if (result != typeEnd) fail; } ]

class OperatorNode extends ExprNode
{
    var optype : OperatorType;
    fun precedence() -> int
    {
        return ...;
    }
    fun checkPrecedence(expr : ExprNode, strict : bool) -> bool
    {
        var OperatorNode op = expr as OperatorNode;
        if (op == nil)
            return true;
        return strict ? (op.precedence() < precedence()) : (op.precedence() <= precedence());
    }
}

parser new UnaryExprNode extends OperatorNode =
        [ OperatorType.parseUnaryPrefix:optype ExprNode:expr | ExprNode:expr OperatorType.parseUnaryPostfix:optype ]
        { if (!checkPrecedence(expr, false)) fail; };

parser new BinaryExprNode extends OperatorNode =
        ExprNode:leftExpr OperatorType.parseBinary:optype { if (!checkPrecedence(leftExpr, !leftAssociative)) fail; }
        ExprNode:rightExpr { if (!checkPrecedence(leftExpr, leftAssociative)) fail; };

parser new NameExprNode extends OperatorNode =
        ExprNode:expr OperatorType.parseNameOperator:optype { if (!checkPrecedence(expr, false)) fail; } Identifier:name;

parser new MultiExprNode extends OperatorNode =
        ExprNode:expr OperatorType.parseMultiBegin:optype { if (!checkPrecedence(expr, false)) fail; }
        (ExprNode:args ("," ExprNode:args)*)?
        OperatorType.parseMultiEnd:typeEnd { if (optype != typeEnd) fail; };

parser new CastExprNode extends OperatorNode = ExprNode:expr "as"_ ClassIdentifier:classid { type = CastOp; if (!checkPrecedence(expr, false)) fail; };

parser new MuxExprNode extends OperatorNode =
        ExprNode:condExpr "?" { type = MuxOp; if (!checkPrecedence(condExpr, false)) fail; }
        ExprNode:trueExpr ":" ExprNode:falseExpr { if (!checkPrecedence(falseExpr, false)) fail; };
```

## Statements
```
parser new TypeExprNode = ExprNode:expr;
parser new LongIdentifierNode = (Identifier:ids ".")*;
parser new ClassIdentifierNode = LongIdentifierNode:name;
// ToDo: Template syntax for classes!

parser new VariableDefinitionNode = "var" (ClassIdentifierNode:classId ".")? Identifier:name (":" TypeExprNode:type)? ("=" ExprNode:initValue)? ";";

class StmntNode;

parser new BoolTypedParenthesesExprNode = ParenthesesExprNode:expr;

parser new BlockStmntNode extends StmntNode = "\{" (StmntNode:list)* "\}";
parser new IfStmntNode extends StmntNode = "if"_ BoolTypedParenthesesExprNode:condExpr StmntNode:thenStmnt ("else"_ StmntNode:elseStmnt)?;
parser new WhileStmntNode extends StmntNode = "while"_ BoolTypedParenthesesExprNode:condExpr StmntNode:stmnt;
parser new DoWhileStmntNode extends StmntNode = "do"_ StmntNode:stmnt "while"_ BoolTypedParenthesesExprNode:condExpr ";";
parser new BreakStmntNode extends StmntNode = "break" ";";
parser new ExprStmntNode extends StmntNode = ExprNode:expr ";";
parser new ReturnStmntNode extends StmntNode = "return"_ ExprNode:expr ";";
parser new VariableStmntNode extends StmntNode = VariableDefinitionNode:definition;
```

## Parser-Extension
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

## Primary Commands
```
// Function Grammar:
parser new ArgumentNode = Identifier:name ":" TypeExprNode:type;
parser new ArgumentList = ("(" (ArgumentNode:list ("," ArgumentNode:list)*)? ")")?;
parser new FunctionDefinitionNode = "fun"_ (ClassIdentifierNode:classId ".")? (Identifier:functionName)? ArgumentList:args ("->" TypeExprNode:returntype)? [ BlockStmntNode:stmnt | ";" ];

// Class Grammar:
class ClassItemNode;

parser new VariableClassItemNode extends ClassItemNode = VariableDefinitionNode:definition;
parser new FunctionClassItemNode extends ClassItemNode = FunctionDefinitionNode:definition;

parser new ClassDefinitionNode = "class"_ Identifier:className ("extends"_ ClassIdentifierNode:parentClasses ("," ClassIdentifierNode:parentClasses)*)? [ "\{" (ClassItemNode:items)* "\}" | ";" ];

parser new EnumDefinitionNode = "enum"_ Identifier:enumName "\{" (Identifier:identifiers ("," Identifier:identifier)*)? "\}";

// Primary Nodes:
class PrimaryNode;

parser new VariablePrimaryNode extends PrimaryNode = VariableDefinitionNode:definition;
parser new FunctionPrimaryNode extends PrimaryNode = FunctionDefinitionNode:definition;
parser new ClassPrimaryNode extends PrimaryNode = ClassDefinitionNode:definition;
parser new EnumPrimaryNode extends PrimaryNode = EnumDefinitionNode:definition;
parser new ParserPrimaryNode extends PrimaryNode = ParserDefinitionNode:definition;
parser new ExecutePrimaryNode extends PrimaryNode = "execute"_ BlockStmntNode:block;
parser new ReadPrimaryNode extends PrimaryNode = "read"_ FileName:filename ";";
parser new LoadPrimaryNode extends PrimaryNode = "load"_ Identifier:modulename ("as"_ Identifier:internalname)? ";";
```
Explanations:
* The `ExecutePrimaryNode` is used to specify code that should be executed immediately after parser the code (before parsing the next primary command).
* tbc
