# Experiment: Type Checking of Variables using FLEX and BISON

## AIM
To write a program using FLEX and BISON to implement type checking of variables in simple declarations and expressions, using a symbol table built during parsing.

## ALGORITHM
1. Start the program.
2. Use FLEX to tokenize keywords (`int`, `float`), identifiers, numbers and operators, passing them to BISON.
3. In BISON, define grammar rules for declaration statements and assignment statements.
4. On a declaration (e.g. `int a;`), insert the variable name and its type into the symbol table.
5. On an assignment (e.g. `a = b + c;`), look up the types of the result variable and operands in the symbol table.
6. If a variable used is not found in the symbol table, report it as undefined.
7. If all types match, print "No type mismatch"; otherwise print "Type mismatch".
8. End the program.

## PROCEDURE
1. Create FLEX File (`typecheck.l`)
   - Define patterns for keywords (`int`, `float`), identifiers, numbers, and operators.
   - Return these tokens to BISON for further processing.
2. Create BISON File (`typecheck.y`)
   - Define grammar rules for declarations and expressions.
   - Maintain a symbol table (array of name/type pairs) that is filled in on each declaration.
   - On each assignment, verify that the operand types and result type match and print the outcome.
3. Compile the FLEX and BISON Files
   ```bash
   flex typecheck.l
   bison -d typecheck.y
   gcc lex.yy.c typecheck.tab.c -o typecheck -lfl
   ```
4. Run the Program
   ```bash
   ./typecheck
   ```
   - Input variable declarations and expressions, ending each statement with `;`.
   - Observe output showing the type-checking result.
5. Test with Various Inputs
   - Try correctly typed expressions.
   - Try mismatched types (e.g., `int a; float b; a = b + 1;`).
   - Try using undeclared variables.
6. End the Program.

## LOGIC / PSEUDOCODE

**Lexer (`typecheck.l`)** — tokenizes type keywords, identifiers, numbers, and operators:

```
RULE: "int"     -> RETURN token INT
RULE: "float"   -> RETURN token FLOAT

RULE: letter/underscore followed by letters/digits/underscores
    yylval.str = COPY of matched text
    RETURN token ID

RULE: one or more digits
    yylval.str = COPY of matched text
    RETURN token NUM

RULE: "="  -> RETURN '='
RULE: "+"  -> RETURN '+'
RULE: "-"  -> RETURN '-'
RULE: "*"  -> RETURN '*'
RULE: "/"  -> RETURN '/'
RULE: ";"  -> RETURN ';'

RULE: whitespace (space/tab/newline) -> IGNORE
RULE: any other single character      -> RETURN that character itself

FUNCTION yywrap()
    RETURN 1
```

**Parser (`typecheck.y`)** — builds a symbol table on declarations and checks types on assignment:

```
DECLARE table[50] : array of { name, type }
DECLARE n = 0                       // number of symbols currently stored

FUNCTION insert(name, type)
    table[n].name = name
    table[n].type = type
    n = n + 1

FUNCTION typeOf(name)
    FOR i = 0 TO n-1
        IF table[i].name == name -> RETURN table[i].type
    RETURN "undefined"

GRAMMAR:
    program -> stmts

    stmts   -> stmts stmt
             | stmt

    stmt    -> decl
             | assign

    decl    -> INT ID ';'    { CALL insert($2, "int") }
             | FLOAT ID ';'  { CALL insert($2, "float") }

    assign  -> ID '=' expr ';'
        {
            lt = typeOf($1)                     // type of the LHS variable
            IF lt == "undefined"
                PRINT "Undefined variable:" $1
            ELSE IF lt == $3                     // $3 = type computed for the RHS expr
                PRINT "No type mismatch in expression:" $1 "= ..."
            ELSE
                PRINT "Type mismatch in assignment to" $1
        }

    expr    -> ID
        {
            t = typeOf($1)
            IF t == "undefined"
                PRINT "Undefined variable:" $1
            $$ = t                                // propagate type (or "undefined") upward
        }
             | NUM
        {
            $$ = "int"                            // numeric literals are always treated as int
        }
             | expr '+' expr
        {
            $$ = ($1 == $3) ? $1 : "mismatch"      // operand types must match
        }
             | expr '-' expr   { same pattern as '+' }
             | expr '*' expr   { same pattern as '+' }
             | expr '/' expr   { same pattern as '+' }

FUNCTION main()
    PRINT "Enter declarations and expressions:"
    CALL yyparse()
    RETURN 0

FUNCTION yyerror(message)
    PRINT "Syntax Error:" message
```

**Flow:** Declarations (`int a;`, `float b;`) populate the symbol table via `insert()`. When an assignment is parsed, each operand `ID` in the expression is looked up with `typeOf()` — if it isn't in the table, "Undefined variable" is printed immediately as a side effect of reducing that `ID`, and its propagated type is `"undefined"`. Binary operators (`+ - * /`) only succeed in propagating a matching type if both operands agree; otherwise the type becomes `"mismatch"`, which will then also fail to equal the LHS variable's declared type. Finally, the top-level `assign` rule compares the LHS variable's declared type against the type computed for the whole RHS expression, and reports "No type mismatch" or "Type mismatch" accordingly.

## PROGRAM

### Lexer — `typecheck.l`

```lex
%{
#include "typecheck.tab.h"
#include <string.h>
#include <stdlib.h>
%}

%%

"int"                       { return INT; }
"float"                     { return FLOAT; }

[a-zA-Z_][a-zA-Z0-9_]*      { yylval.str = strdup(yytext); return ID; }
[0-9]+                      { yylval.str = strdup(yytext); return NUM; }

"="                         { return '='; }
"+"                         { return '+'; }
"-"                         { return '-'; }
"*"                         { return '*'; }
"/"                         { return '/'; }
";"                         { return ';'; }

[ \t\n]+                    { /* skip whitespace */ }

.                           { return yytext[0]; }

%%

int yywrap() {
    return 1;
}
```

### Parser — `typecheck.y`

```yacc
%{
#include <stdio.h>
#include <string.h>
#include <stdlib.h>

struct sym {
    char name[20];
    char type[10];
} table[50];

int n = 0;

void insert(char *name, char *type) {
    strcpy(table[n].name, name);
    strcpy(table[n].type, type);
    n++;
}

char* typeOf(char *name) {
    int i;

    for (i = 0; i < n; i++) {
        if (strcmp(table[i].name, name) == 0)
            return table[i].type;
    }

    return "undefined";
}
%}

%union {
    char *str;
}

%token <str> ID NUM
%token INT FLOAT

%type <str> expr

%%

program:
    stmts
    ;

stmts:
    stmts stmt
    | stmt
    ;

stmt:
    decl
    | assign
    ;

decl:
    INT ID ';'
    {
        insert($2, "int");
    }
    |
    FLOAT ID ';'
    {
        insert($2, "float");
    }
    ;

assign:
    ID '=' expr ';'
    {
        char *lt = typeOf($1);

        if (strcmp(lt, "undefined") == 0)
            printf("Undefined variable: %s\n", $1);

        else if (strcmp(lt, $3) == 0)
            printf("No type mismatch in expression: %s = ...\n", $1);

        else
            printf("Type mismatch in assignment to %s\n", $1);
    }
    ;

expr:
    ID
    {
        char *t = typeOf($1);

        if (strcmp(t, "undefined") == 0)
            printf("Undefined variable: %s\n", $1);

        $$ = t;
    }

    |
    NUM
    {
        $$ = "int";
    }

    |
    expr '+' expr
    {
        $$ = (strcmp($1, $3) == 0) ? $1 : "mismatch";
    }

    |
    expr '-' expr
    {
        $$ = (strcmp($1, $3) == 0) ? $1 : "mismatch";
    }

    |
    expr '*' expr
    {
        $$ = (strcmp($1, $3) == 0) ? $1 : "mismatch";
    }

    |
    expr '/' expr
    {
        $$ = (strcmp($1, $3) == 0) ? $1 : "mismatch";
    }
    ;

%%

int main() {
    printf("Enter declarations and expressions:\n");
    yyparse();
    return 0;
}

int yyerror(char *s) {
    printf("Syntax Error: %s\n", s);
    return 0;
}
```

## COMPILATION

```bash
flex typecheck.l
bison -d typecheck.y
gcc lex.yy.c typecheck.tab.c -o typecheck -lfl
```

### Compiler Warnings Observed

The build succeeded but produced the following warnings, which are worth recording:

```
typecheck.y: warning: 16 shift/reduce conflicts [-Wconflicts-sr]
typecheck.y: note: rerun with option '-Wcounterexamples' to generate conflict counterexamples

typecheck.tab.c: In function 'yyparse':
typecheck.tab.c:1017:16: warning: implicit declaration of function 'yylex'
typecheck.tab.c:1238:7: warning: implicit declaration of function 'yyerror'; did you mean 'yyerrok'?
```

- **16 shift/reduce conflicts:** the `expr` grammar does not declare operator precedence/associativity (no `%left`/`%right` for `+ - * /`), so Bison's default conflict resolution (shift) is relied upon for every arithmetic operator combination. The program still runs correctly for the tested inputs because the type-checking logic only cares whether operand *types* match, not the exact parse shape/evaluation order — but for a grammar meant to also compute values or care about grouping, precedence declarations should be added.
- **Implicit declaration of `yylex`/`yyerror`:** the parser file doesn't forward-declare `int yylex(void);` and the `yyerror` signature (`int yyerror(char *s)`) doesn't match the standard `const char *` signature Bison expects. These are harmless warnings here since the functions are still defined correctly and linked successfully, but they can be silenced by adding a proper `int yylex(void);` prototype and matching `yyerror`'s parameter to `const char *s`.

## SAMPLE RUNS

**Test 1 — matching types (no mismatch)**

`input.txt`:
```
int a;
int b;
int c;
a = b * c;
```

Command: `./typecheck < input.txt`

Output:
```
Enter declarations and expressions:
No type mismatch in expression: a = ...
```

**Test 2 — mismatched types**

`input.txt`:
```
int a;
float b;
int c;
a = b + c;
```

Command: `./typecheck < input.txt`

Output:
```
Enter declarations and expressions:
Type mismatch in assignment to a
```

Here `b` (float) and `c` (int) don't match, so `expr` evaluates to `"mismatch"`, which then also doesn't match `a`'s declared type (`int`), so "Type mismatch" is printed. (Note: since `b + c` itself already mismatches, no separate "operand mismatch" message is printed — the mismatch surfaces only at the final assignment check.)

**Test 3 — undefined variable combined with type mismatch**

`input.txt`:
```
int a;
a = b + 10;
```

Command: `./typecheck < input.txt`

Output:
```
Enter declarations and expressions:
Undefined variable: b
Type mismatch in assignment to a
```

Here `b` was never declared, so `typeOf("b")` returns `"undefined"` and that message is printed as a side effect of reducing the `ID` inside `expr`. Since `"undefined" != "int"` (the type of `NUM`), the `+` expression's `$$` becomes `"mismatch"`, and the final check against `a`'s declared type (`int`) also fails — so both messages appear.

## RESULT
The FLEX and BISON programs were successfully compiled and executed. The type checker correctly built a symbol table from variable declarations, verified type consistency in assignment expressions, and correctly reported "No type mismatch" for consistently typed expressions, "Type mismatch" for inconsistently typed expressions, and "Undefined variable" for the use of undeclared identifiers — verifying the working of the type-checking mechanism.
