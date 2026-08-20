# Experiment: Simple Code Optimization (Constant Folding, Strength Reduction, Algebraic Simplification) using FLEX and BISON

## AIM
To write a program using FLEX and BISON to implement simple code optimization techniques such as constant folding, strength reduction and algebraic simplification, applied while parsing three-address code style assignment statements.

## ALGORITHM
1. Use FLEX to tokenize the input statements into identifiers, numbers and operators, passing them to BISON.
2. In BISON, define a grammar for assignment statements of the form `id = expr ;`
3. While reducing an `expr` production:
   - **Constant Folding:** if both operands are numeric constants, evaluate the operation immediately and replace it with the result.
   - **Algebraic Simplification:** apply rules such as `x + 0 -> x`, `x - 0 -> x`, `x * 1 -> x`, `x / 1 -> x`.
   - **Strength Reduction:** replace `x * 2` with `x + x`.
4. Print the optimized right-hand side once the statement is fully reduced.
5. Repeat for every statement until the input ends.

## PROCEDURE
1. Create the FLEX file `optimize.l` to tokenize identifiers, numbers, and the operators `+ - * / = ;`.
2. Create the BISON file `optimize.y` with a grammar for assignment statements and expressions.
3. In the semantic actions for each `expr` rule, check whether constant folding, strength reduction, or algebraic simplification applies, and print a comment indicating which optimization fired.
4. Compile using:
   ```bash
   flex optimize.l
   bison -d optimize.y
   gcc lex.yy.c optimize.tab.c -o optimize -lfl
   ```
5. Run `./optimize` and enter three-address-code-style statements ending with `;`, terminate input with Ctrl+D.
6. Verify that constants are folded, `x*1` / `x/1` / `x+0` / `x-0` are simplified, and `x*2` is converted to `x+x`.
7. Test with multiple statements to confirm all three optimization types are triggered correctly.

## LOGIC / PSEUDOCODE

**Lexer (`optimize.l`)** — tokenizes identifiers, numbers, and the assignment/arithmetic punctuation:

```
RULE: letter followed by letters/digits
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

**Parser (`optimize.y`)** — parses `id = expr ;` statements and, at every binary-operator reduction, checks for an applicable optimization before falling back to an unoptimized textual rebuild:

```
TOKENS (typed <str>): ID, NUM
NONTERMINAL (typed <str>): expr

PRECEDENCE:
    left '+' '-'
    left '*' '/'

GRAMMAR:
    stmt_list -> stmt_list stmt | stmt

    stmt -> ID '=' expr ';'
            { PRINT $1 "=" $3 }

    expr -> NUM   { $$ = $1 }
          | ID    { $$ = $1 }

          | expr '+' expr
            {
                IF $1 and $3 are both purely numeric
                    $$ = numeric-string of ($1 + $3)
                    PRINT "// Constant Folding: " $1 "+" $3 "->" $$
                ELSE IF $3 == "0"
                    $$ = $1
                    PRINT "// Algebraic Simplification: x + 0 -> x"
                ELSE IF $1 == "0"
                    $$ = $3
                    PRINT "// Algebraic Simplification: 0 + x -> x"
                ELSE
                    $$ = "$1 + $3"          // rebuild as text, no optimization applies
            }

          | expr '-' expr
            {
                IF $1 and $3 are both purely numeric
                    $$ = numeric-string of ($1 - $3)
                    PRINT "// Constant Folding: " $1 "-" $3 "->" $$
                ELSE IF $3 == "0"
                    $$ = $1
                    PRINT "// Algebraic Simplification: x - 0 -> x"
                ELSE
                    $$ = "$1 - $3"
            }

          | expr '*' expr
            {
                IF $1 and $3 are both purely numeric
                    $$ = numeric-string of ($1 * $3)
                    PRINT "// Constant Folding: " $1 "*" $3 "->" $$
                ELSE IF $3 == "1"
                    $$ = $1
                    PRINT "// Algebraic Simplification: x * 1 -> x"
                ELSE IF $3 == "2"
                    $$ = "$1 + $1"                       // strength reduction
                    PRINT "// Strength Reduction: x * 2 -> x + x"
                ELSE
                    $$ = "$1 * $3"
            }

          | expr '/' expr
            {
                IF $1 and $3 are both purely numeric
                    $$ = numeric-string of ($1 / $3)      // integer division
                    PRINT "// Constant Folding: " $1 "/" $3 "->" $$
                ELSE IF $3 == "1"
                    $$ = $1
                    PRINT "// Algebraic Simplification: x / 1 -> x"
                ELSE
                    $$ = "$1 / $3"
            }

FUNCTION main()
    PRINT "Enter Three Address Code statements (end with Ctrl+D):"
    CALL yyparse()
    RETURN 0

FUNCTION yyerror(message)
    PRINT "Syntax Error:" message
```

**Flow:** Each `expr` value is carried as a **string**, not a number — even after constant folding, the folded result is converted back to a string so it can be compared (`isdigit()`, `strcmp`) and re-embedded in later expressions. This lets the same grammar rule check "is this a constant?" (first character is a digit) at every level of nesting. As the parser reduces an expression bottom-up, each operator node independently checks, in order: (1) constant folding — are both sides numeric literals? (2) algebraic simplification — is one side an identity element (`0` for `+`/`-`, `1` for `*`/`/`)? (3) strength reduction — is the right side exactly `2` for a `*`? If none apply, the sub-expression is simply rebuilt as text (e.g. `"a * b"`) unoptimized, and passed up to the enclosing rule. The optimization comment is printed **as a side effect** the moment the relevant rule fires, so multiple optimization messages can appear before the final `id = result` line for statements with nested operators — though in the tested inputs each statement has only one top-level operator.

## PROGRAM

### Lexer — `optimize.l`

```lex
%{
#include "optimize.tab.h"
#include <string.h>
#include <stdlib.h>
%}

%%

[a-zA-Z][a-zA-Z0-9]*   { yylval.str = strdup(yytext); return ID; }
[0-9]+                 { yylval.str = strdup(yytext); return NUM; }

"="                    { return '='; }
"+"                    { return '+'; }
"-"                    { return '-'; }
"*"                    { return '*'; }
"/"                    { return '/'; }
";"                    { return ';'; }

[ \t\n]+               { /* skip whitespace */ }

.                      { return yytext[0]; }

%%

int yywrap() {
    return 1;
}
```

### Parser — `optimize.y`

```yacc
%{
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
#include <ctype.h>
%}

%union {
    char *str;
}

%token <str> ID NUM
%type <str> expr

%left '+' '-'
%left '*' '/'

%%

stmt_list:
    stmt_list stmt
    | stmt
    ;

stmt:
    ID '=' expr ';'
    {
        printf("%s = %s\n", $1, $3);
    }
    ;

expr:
    NUM
    {
        $$ = $1;
    }

    |
    ID
    {
        $$ = $1;
    }

    |
    expr '+' expr
    {
        if (isdigit($1[0]) && isdigit($3[0])) {
            char buf[20];

            sprintf(buf, "%d", atoi($1) + atoi($3));
            $$ = strdup(buf);

            printf("// Constant Folding: %s + %s -> %s\n",
                   $1, $3, $$);
        }

        else if (strcmp($3, "0") == 0) {
            $$ = $1;

            printf("// Algebraic Simplification: x + 0 -> x\n");
        }

        else if (strcmp($1, "0") == 0) {
            $$ = $3;

            printf("// Algebraic Simplification: 0 + x -> x\n");
        }

        else {
            char buf[40];

            sprintf(buf, "%s + %s", $1, $3);
            $$ = strdup(buf);
        }
    }

    |
    expr '-' expr
    {
        if (isdigit($1[0]) && isdigit($3[0])) {
            char buf[20];

            sprintf(buf, "%d", atoi($1) - atoi($3));
            $$ = strdup(buf);

            printf("// Constant Folding: %s - %s -> %s\n",
                   $1, $3, $$);
        }

        else if (strcmp($3, "0") == 0) {
            $$ = $1;

            printf("// Algebraic Simplification: x - 0 -> x\n");
        }

        else {
            char buf[40];

            sprintf(buf, "%s - %s", $1, $3);
            $$ = strdup(buf);
        }
    }

    |
    expr '*' expr
    {
        if (isdigit($1[0]) && isdigit($3[0])) {
            char buf[20];

            sprintf(buf, "%d", atoi($1) * atoi($3));
            $$ = strdup(buf);

            printf("// Constant Folding: %s * %s -> %s\n",
                   $1, $3, $$);
        }

        else if (strcmp($3, "1") == 0) {
            $$ = $1;

            printf("// Algebraic Simplification: x * 1 -> x\n");
        }

        else if (strcmp($3, "2") == 0) {
            char buf[40];

            sprintf(buf, "%s + %s", $1, $1);
            $$ = strdup(buf);

            printf("// Strength Reduction: x * 2 -> x + x\n");
        }

        else {
            char buf[40];

            sprintf(buf, "%s * %s", $1, $3);
            $$ = strdup(buf);
        }
    }

    |
    expr '/' expr
    {
        if (isdigit($1[0]) && isdigit($3[0])) {
            char buf[20];

            sprintf(buf, "%d", atoi($1) / atoi($3));
            $$ = strdup(buf);

            printf("// Constant Folding: %s / %s -> %s\n",
                   $1, $3, $$);
        }

        else if (strcmp($3, "1") == 0) {
            $$ = $1;

            printf("// Algebraic Simplification: x / 1 -> x\n");
        }

        else {
            char buf[40];

            sprintf(buf, "%s / %s", $1, $3);
            $$ = strdup(buf);
        }
    }
    ;

%%

int main() {
    printf("Enter Three Address Code statements (end with Ctrl+D):\n");
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
flex optimize.l
bison -d optimize.y
gcc lex.yy.c optimize.tab.c -o optimize -lfl
```

### Compiler Warnings Observed

The build succeeded but produced these warnings:

```
optimize.tab.c: In function 'yyparse':
optimize.tab.c:983:16: warning: implicit declaration of function 'yylex'
optimize.tab.c:1273:7: warning: implicit declaration of function 'yyerror'; did you mean 'yyerrok'?
```

These occur because `optimize.y` doesn't forward-declare `int yylex(void);`, and `yyerror`'s signature (`int yyerror(char *s)`) doesn't match the `const char *` signature Bison's generated caller expects. Both are harmless here — the program still links and runs correctly — but could be silenced by adding a proper `yylex` prototype and matching `yyerror`'s parameter type to `const char *s`.

## SAMPLE RUN

`input.txt`:
```
a = 2 + 4;
b = d * 1;
c = s * 2;
```

Command:
```bash
./optimize < input.txt
```

Output:
```
Enter Three Address Code statements (end with Ctrl+D):
// Constant Folding: 2 + 4 -> 6
a = 6
// Algebraic Simplification: x * 1 -> x
b = d
// Strength Reduction: x * 2 -> x + x
c = s + s
```

**Explanation of each result:**
| Statement | Optimization Applied | Result |
|---|---|---|
| `a = 2 + 4;` | Constant Folding (`2 + 4` → `6`) | `a = 6` |
| `b = d * 1;` | Algebraic Simplification (`x * 1 -> x`) | `b = d` |
| `c = s * 2;` | Strength Reduction (`x * 2 -> x + x`) | `c = s + s` |

## RESULT
The FLEX and BISON programs were successfully compiled and executed. The optimizer correctly performed constant folding on a purely numeric expression (`2 + 4 -> 6`), algebraic simplification on an identity multiplication (`d * 1 -> d`), and strength reduction on a multiplication by 2 (`s * 2 -> s + s`), verifying the working of all three code optimization techniques.

> **Note:** Only three of the several optimization rules coded into the grammar were exercised by this run — `x + 0`, `0 + x`, `x - 0`, and `x / 1` were not triggered by any test statement. If you'd like a fuller demonstration run covering those remaining rules, let me know and I can add the corresponding test inputs and expected output.
