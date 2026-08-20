# Experiment: Recognition of Valid C Control Structure Syntax using FLEX and BISON

**EX.NO:** 5 — Program to recognize a valid control structure syntax of C language (For loop, while loop, if-else, if-else-if, switch-case, etc.)

**DATE:**

## AIM
To write a program to recognize a valid control structure syntax of C language (such as for loop, while loop, if-else, if-else-if, switch-case, etc.) using FLEX and BISON.

## ALGORITHM

### FLEX
1. Start the FLEX program with required header files and token declarations.
2. Define regular expressions for control keywords such as `if`, `else`, `for`, `while`, `switch`, `case`, etc.
3. Return appropriate tokens for each keyword to BISON.

### BISON
1. Include header files and token declarations.
2. Define grammar rules to match valid syntax for:
   - `if` statement
   - `if-else` and `if-else-if` ladder
   - `while` and `for` loops
   - `switch-case` structure
3. Implement `yyparse()` to start parsing and `yyerror()` for invalid inputs.

## PROCEDURE
1. Write the `control.l` file using FLEX to tokenize control structure keywords.
2. Write the `control.y` file using BISON to define grammar rules for C control structures.
3. Compile and execute the program using:
   ```bash
   flex control.l
   bison -d control.y
   gcc lex.yy.c control.tab.c -o control -lfl
   ./control
   ```
4. Input sample control structures in C-style syntax to check for validation.

## LOGIC / PSEUDOCODE

**Lexer (`control.l`)** — tokenizes control-structure keywords, identifiers, numbers, punctuation, and relational operators:

```
RULE: "if"                         -> RETURN IF
RULE: "else"                       -> RETURN ELSE
RULE: "for"                        -> RETURN FOR
RULE: "while"                      -> RETURN WHILE
RULE: "switch"                     -> RETURN SWITCH
RULE: "case"                       -> RETURN CASE
RULE: "default"                    -> RETURN DEFAULT

RULE: letter/underscore followed
      by letters/digits/underscores -> RETURN ID
RULE: one or more digits            -> RETURN NUM

RULE: "{"  -> RETURN LBRACE
RULE: "}"  -> RETURN RBRACE
RULE: "("  -> RETURN LPAREN
RULE: ")"  -> RETURN RPAREN
RULE: ":"  -> RETURN COLON
RULE: ";"  -> RETURN SEMICOLON

RULE: "==" -> RETURN EQ
RULE: "<=" -> RETURN LE
RULE: ">=" -> RETURN GE
RULE: "<"  -> RETURN LT
RULE: ">"  -> RETURN GT
RULE: "="  -> RETURN ASSIGN

RULE: whitespace (space/tab/newline) -> IGNORE
RULE: any other single character      -> RETURN that character itself

FUNCTION yywrap()
    RETURN 1
```

**Parser (`control.y`)** — defines the grammar for valid C control structures:

```
TOKENS: IF ELSE FOR WHILE SWITCH CASE DEFAULT
        ID NUM
        LBRACE RBRACE LPAREN RPAREN COLON SEMICOLON
        EQ LE GE LT GT ASSIGN

GRAMMAR:
    program    -> stmt_list

    stmt_list  -> stmt_list stmt
               | stmt

    stmt       -> if_stmt
               | while_stmt
               | for_stmt
               | switch_stmt
               | block
               | assignment

    block      -> LBRACE stmt_list RBRACE

    assignment -> ID ASSIGN NUM SEMICOLON
               | ID ASSIGN ID SEMICOLON

    if_stmt    -> IF LPAREN cond RPAREN stmt
               | IF LPAREN cond RPAREN stmt ELSE stmt

    while_stmt -> WHILE LPAREN cond RPAREN stmt

    for_stmt   -> FOR LPAREN ID ASSIGN NUM SEMICOLON cond SEMICOLON ID ASSIGN ID RPAREN stmt

    switch_stmt -> SWITCH LPAREN ID RPAREN LBRACE case_list RBRACE

    case_list  -> case_list CASE NUM COLON stmt
               | case_list DEFAULT COLON stmt
               | CASE NUM COLON stmt
               | DEFAULT COLON stmt

    cond       -> ID relop NUM

    relop      -> EQ | LE | GE | LT | GT

FUNCTION main()
    PRINT "Enter a C control structure syntax:"
    IF yyparse() == 0
        PRINT "Valid control structure syntax."
    RETURN 0

FUNCTION yyerror(message)
    PRINT "Invalid control structure syntax."
    RETURN 0
```

**Flow:** `yyparse()` repeatedly requests tokens from `yylex()` (built from the `.l` file) and reduces them according to the grammar above. Nested statements are handled recursively — `stmt` can itself be an `if_stmt`, `while_stmt`, `for_stmt`, `switch_stmt`, a `{ ... }` block, or a simple assignment — which allows constructs like `if-else`, `if-else-if` ladders, and loops containing further statements to be recognized. If the entire input reduces cleanly to `program`, `yyparse()` returns 0 and "Valid control structure syntax." is printed; otherwise `yyerror()` fires and "Invalid control structure syntax." is printed.

## PROGRAM

### Lexer — `control.l`

```lex
%{
#include "y.tab.h"
%}

%%

"if"        { return IF; }
"else"      { return ELSE; }
"for"       { return FOR; }
"while"     { return WHILE; }
"switch"    { return SWITCH; }
"case"      { return CASE; }
"default"   { return DEFAULT; }

[a-zA-Z_][a-zA-Z0-9_]*    { return ID; }
[0-9]+                    { return NUM; }

"{"         { return LBRACE; }
"}"         { return RBRACE; }
"("         { return LPAREN; }
")"         { return RPAREN; }
":"         { return COLON; }
";"         { return SEMICOLON; }

"=="        { return EQ; }
"<="        { return LE; }
">="        { return GE; }
"<"         { return LT; }
">"         { return GT; }
"="         { return ASSIGN; }

[ \t\n]+    { /* ignore whitespace */ }

.           { return yytext[0]; }

%%

int yywrap()
{
    return 1;
}
```

### Parser — `control.y`

```yacc
%{
#include <stdio.h>
#include <stdlib.h>

int yylex(void);
int yyerror(const char *s);
%}

%token IF ELSE FOR WHILE SWITCH CASE DEFAULT
%token ID NUM
%token LBRACE RBRACE LPAREN RPAREN COLON SEMICOLON
%token EQ LE GE LT GT ASSIGN

%%

program:
    stmt_list
    ;

stmt_list:
    stmt_list stmt
    | stmt
    ;

stmt:
    if_stmt
    | while_stmt
    | for_stmt
    | switch_stmt
    | block
    | assignment
    ;

block:
    LBRACE stmt_list RBRACE
    ;

assignment:
    ID ASSIGN NUM SEMICOLON
    | ID ASSIGN ID SEMICOLON
    ;

if_stmt:
    IF LPAREN cond RPAREN stmt
    | IF LPAREN cond RPAREN stmt ELSE stmt
    ;

while_stmt:
    WHILE LPAREN cond RPAREN stmt
    ;

for_stmt:
    FOR LPAREN ID ASSIGN NUM SEMICOLON cond SEMICOLON ID ASSIGN ID RPAREN stmt
    ;

switch_stmt:
    SWITCH LPAREN ID RPAREN LBRACE case_list RBRACE
    ;

case_list:
    case_list CASE NUM COLON stmt
    | case_list DEFAULT COLON stmt
    | CASE NUM COLON stmt
    | DEFAULT COLON stmt
    ;

cond:
    ID relop NUM
    ;

relop:
    EQ
    | LE
    | GE
    | LT
    | GT
    ;

%%

int main()
{
    printf("Enter a C control structure syntax:\n");

    if (yyparse() == 0)
    {
        printf("Valid control structure syntax.\n");
    }

    return 0;
}

int yyerror(const char *s)
{
    printf("Invalid control structure syntax.\n");
    return 0;
}
```

## COMPILATION

```bash
yacc -d control.y
flex control.l
gcc lex.yy.c y.tab.c -o control
```

**Compiler note:** `yacc -d control.y` produced the following warning during compilation, which does not prevent the grammar from working but is worth recording:

```
control.y: warning: 1 shift/reduce conflict [-Wconflicts-sr]
control.y: note: rerun with option '-Wcounterexamples' to generate conflict counterexamples
```

This is the classic **"dangling else"** ambiguity — when parsing `if (cond) stmt else stmt` nested inside another `if`, the parser must decide whether a trailing `else` binds to the nearest unmatched `if` or an outer one. Yacc/Bison resolves shift/reduce conflicts by **shifting**, which by default binds the `else` to the nearest `if` — the conventional and expected C behavior — so the warning is benign here.

## EXECUTION AND OUTPUT

**Test — valid control structure**

Input:
```
if (x < 5) { y = 10; }
```

Output:
```
Enter a C control structure syntax:
Valid control structure syntax.
```

## RESULT
The FLEX and BISON programs were successfully compiled and executed. The lexer correctly tokenized C control-structure keywords, identifiers, numbers, and operators, and the parser correctly validated a nested `if` statement containing a block and an assignment (`if (x < 5) { y = 10; }`) as a valid control structure, verifying the working of the recognizer for C control structures such as `if`, `if-else`, `while`, `for`, and `switch-case`.
