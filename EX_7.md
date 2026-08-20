# Experiment: Generation of Three-Address Code (TAC) using FLEX and BISON

## AIM
To write a program using FLEX and BISON to generate three-address code (TAC) for a simple arithmetic expression.

## ALGORITHM

### FLEX
1. Include required headers and define tokens for identifiers, numbers, and operators.
2. Use regular expressions to identify identifiers and numeric constants.
3. Return appropriate tokens to BISON for parsing.

### BISON
1. Declare tokens and define associativity for operators.
2. Use grammar rules to parse arithmetic expressions (e.g., `a = b + c * d`).
3. Generate three-address code during the parsing actions.
4. Maintain a temporary variable counter to represent intermediate results (e.g., `t1 = b * d`).

## PROCEDURE
1. Create the FLEX file `tac.l`:
   - Tokenize input using patterns for identifiers, numbers, and operators.
   - Pass tokens to BISON.
2. Create the BISON file `tac.y`:
   - Parse arithmetic expressions.
   - Generate three-address code using temporary variables (`t1`, `t2`, etc.) during parsing.
3. Compile and run:
   ```bash
   flex tac.l
   bison -d tac.y
   gcc tac.tab.c lex.yy.c -o tac -lfl
   ./tac
   ```
4. Input an arithmetic expression like:
   ```
   a = b + c * d
   ```
5. View generated three-address code.

## LOGIC / PSEUDOCODE

**Lexer (`tac.l`)** — tokenizes identifiers and numbers, storing the matched text as a string for BISON to use:

```
RULE: letter followed by letters/digits
    yylval.str = COPY of matched text
    RETURN token ID

RULE: one or more digits
    yylval.str = COPY of matched text
    RETURN token NUM

RULE: whitespace (space/tab/newline) -> IGNORE

RULE: any other single character -> RETURN that character itself
                                     ('=', '+', '-', '*', '/' passed through literally)
```

Note: `%option noyywrap` is set, so FLEX does not require a `yywrap()` definition of its own — but this program links with `-lfl` (the FLEX runtime library), which supplies the default `yywrap()`. Omitting `-lfl` at compile time causes an "undefined reference to `yywrap`" linker error (see Compilation Notes below).

**Parser (`tac.y`)** — parses the expression bottom-up and emits three-address code as each sub-expression is reduced:

```
UNION type: str (char *)
TOKENS (typed <str>): ID, NUM
NONTERMINAL (typed <str>): expr

PRECEDENCE (lowest to highest):
    left '+' '-'
    left '*' '/'

DECLARE tempCount = 1     // global counter for temporary variable names

GRAMMAR (with semantic actions):
    stmt -> ID '=' expr
            { CALL printAssign($1, $3) }              // e.g. "a = t2"

    expr -> expr '+' expr
            {
                temp = "t" + tempCount++
                CALL printTAC(temp, $1, "+", $3)        // e.g. "t2 = b + t1"
                $$ = temp
            }
         | expr '-' expr   { same pattern with "-" }
         | expr '*' expr   { same pattern with "*" }
         | expr '/' expr   { same pattern with "/" }
         | ID   { $$ = $1 }       // pass identifier through unchanged
         | NUM  { $$ = $1 }       // pass number through unchanged

FUNCTION printTAC(result, op1, operator, op2)
    PRINT result "=" op1 operator op2

FUNCTION printAssign(var, val)
    PRINT var "=" val

FUNCTION main()
    PRINT "Enter the expression:"
    CALL yyparse()
    RETURN 0

FUNCTION yyerror(message)
    PRINT "Error:" message
```

**Flow:** Because `*` and `/` have higher precedence than `+` and `-`, an expression like `b + c * d` is parsed with `c * d` reduced first. Each time a binary operator rule fires, a **new temporary variable** (`t1`, `t2`, ...) is generated and its computation is printed immediately — so the temporaries are emitted in the order the sub-expressions are actually evaluated (deepest/highest-precedence first), and the final `ID '=' expr` rule prints the assignment of the last temporary to the target variable.

## PROGRAM

### Lexer — `tac.l`

```lex
%{
#include "tac.tab.h"
#include <string.h>
#include <stdlib.h>
%}

%option noyywrap

%%

[a-zA-Z][a-zA-Z0-9]* {
    yylval.str = strdup(yytext);
    return ID;
}

[0-9]+ {
    yylval.str = strdup(yytext);
    return NUM;
}

[ \t\n]+ {
    /* Ignore whitespace */
}

. {
    return yytext[0];
}

%%
```

### Parser — `tac.y`

```yacc
%{
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

int tempCount = 1;

void printTAC(char *result, char *op1, char *operator, char *op2)
{
    printf("%s = %s %s %s\n", result, op1, operator, op2);
}

void printAssign(char *var, char *val)
{
    printf("%s = %s\n", var, val);
}

int yylex(void);
int yyerror(const char *s);
%}

%union {
    char *str;
}

%token <str> ID NUM
%type <str> expr

%left '+' '-'
%left '*' '/'

%%

stmt:
      ID '=' expr
      {
          printAssign($1, $3);
      }
      ;

expr:
      expr '+' expr
      {
          char temp[20];
          sprintf(temp, "t%d", tempCount++);
          printTAC(temp, $1, "+", $3);
          $$ = strdup(temp);
      }

    | expr '-' expr
      {
          char temp[20];
          sprintf(temp, "t%d", tempCount++);
          printTAC(temp, $1, "-", $3);
          $$ = strdup(temp);
      }

    | expr '*' expr
      {
          char temp[20];
          sprintf(temp, "t%d", tempCount++);
          printTAC(temp, $1, "*", $3);
          $$ = strdup(temp);
      }

    | expr '/' expr
      {
          char temp[20];
          sprintf(temp, "t%d", tempCount++);
          printTAC(temp, $1, "/", $3);
          $$ = strdup(temp);
      }

    | ID
      {
          $$ = $1;
      }

    | NUM
      {
          $$ = $1;
      }
      ;

%%

int main(void)
{
    printf("Enter the expression:\n");
    yyparse();
    return 0;
}

int yyerror(const char *s)
{
    printf("Error: %s\n", s);
    return 0;
}
```

## COMPILATION

```bash
bison -d tac.y
flex tac.l
gcc tac.tab.c lex.yy.c -o tac -lfl
```

### Compilation Notes (troubleshooting encountered)

The first compile attempt was run **without** the `-lfl` flag:

```bash
gcc tac.tab.c lex.yy.c -o tac
```

This failed with a linker error:

```
/usr/bin/ld: /tmp/ccpVibd2.o: in function `yylex':
lex.yy.c:(.text+0x4d1): undefined reference to `yywrap'
/usr/bin/ld: /tmp/ccpVibd2.o: in function `input':
lex.yy.c:(.text+0x10e7): undefined reference to `yywrap'
collect2: error: ld returned 1 exit status
```

As a result, no `tac` executable was produced, and the subsequent run attempts failed with `./tac: No such file or directory`. Re-running the exact same `bison -d tac.y` and `flex tac.l` steps (regenerating `tac.tab.c`/`tac.tab.h` and `lex.yy.c`) and then compiling **with** `-lfl` linked the FLEX runtime library (which supplies the default `yywrap()`), and the build succeeded, producing a working `tac` executable.

**Takeaway:** even with `%option noyywrap` in the `.l` file, `-lfl` should still be included at the link step per the procedure, since it provides other FLEX runtime symbols the generated scanner may reference.

## EXECUTION AND OUTPUT

**Input**

```
a=b+c*d
```

**Output**

```
Enter the expression:
t1 = c * d
t2 = b + t1
a = t2
```

This correctly reflects operator precedence: `c * d` is evaluated first into `t1`, then `b + t1` into `t2`, and finally `a` is assigned the value of `t2`.

## RESULT
The FLEX and BISON programs were successfully compiled and executed after resolving an initial linker error caused by a missing `-lfl` flag. The program correctly generated three-address code for the expression `a = b + c * d`, producing the intermediate temporary variables `t1` and `t2` in the correct order of operator precedence, verifying the working of the three-address code generator.
