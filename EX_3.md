# Experiment: Recognition of Valid Arithmetic Expressions using FLEX and BISON

## AIM
To write a program to recognize a valid arithmetic expression that uses operators `+`, `-`, `*` and `/` using FLEX and BISON.

## ALGORITHM

### FLEX
1. Declare the required header file and variable declaration within `%{` and `%}`.
2. FLEX requires regular expressions to identify valid arithmetic expression tokens/lexemes.
3. FLEX calls `yywrap()` function after input is over. It should return 1 when work is done or should return 0 when more processing is required.

### BISON
1. Declare the required header file and variable declaration within `%{` and `%}`.
2. Define tokens in the first section and also define the associativity of the operations.
3. Mention the grammar productions and the action for each production.
4. `$$` refers to the top of the stack position while `$1`, `$2` refer to respective values in the stack.
5. Call `yyparse()` to initiate the parsing process.
6. `yyerror()` function is called when no productions in the grammar match the input statement.

## PROCEDURE
1. Open a text editor and write the FLEX source file `art_expr.l`.
2. Define regular expressions for identifiers, digits, operators, and ignore whitespaces.
3. Save and close the FLEX file.
4. Open another text file and write the BISON source file `art_expr.y`.
5. Define tokens and grammar rules to parse arithmetic expressions using `+`, `-`, `*`, and `/`.
6. Save and close the BISON file.
7. Use the following commands to compile and run:
   ```bash
   flex art_expr.l
   bison -d art_expr.y
   gcc lex.yy.c art_expr.tab.c -o art_expr -lfl
   ./art_expr
   ```
8. Enter expressions as input. If the expression is valid, it displays "Valid Expression", otherwise "Invalid Expression".

## LOGIC / PSEUDOCODE

**Lexer (`art_expr.l`)** — scans the input character stream and returns a token for each lexeme:

```
RULE: letter followed by letters/digits  -> RETURN token ID
RULE: one or more digits                  -> RETURN token DIG
RULE: spaces/tabs                          -> IGNORE
RULE: newline                              -> RETURN 0 (end of input)
RULE: any other single character          -> RETURN that character itself
                                              (so '+', '-', '*', '/', '(', ')' etc.
                                               are passed through as literal tokens)

FUNCTION yywrap()
    RETURN 1   // no more input files to process
```

**Parser (`art_expr.y`)** — defines the grammar for a valid arithmetic expression and drives parsing:

```
TOKENS: ID, DIG
PRECEDENCE (lowest to highest):
    left  '+' '-'
    left  '*' '/'
    right UMINUS        // unary minus, highest precedence

GRAMMAR:
    stmt  -> expn

    expn  -> expn '+' expn
           | expn '-' expn
           | expn '*' expn
           | expn '/' expn
           | '-' expn        %prec UMINUS      // unary minus
           | '(' expn ')'
           | DIG
           | ID

FUNCTION main()
    PRINT "Enter the Expression:"
    CALL yyparse()              // reads tokens from yylex() and matches grammar
    PRINT "Valid Expression"    // reached only if parsing completes without error
    RETURN 0

FUNCTION yyerror(message)
    PRINT "Invalid Expression"
    EXIT
```

**Flow:** `yyparse()` repeatedly calls `yylex()` (generated from the `.l` file) to fetch tokens, and matches them against the grammar rules from the `.y` file. If the input reduces cleanly to `stmt`, control returns to `main()` and "Valid Expression" is printed. If at any point no grammar rule matches the token sequence, `yyerror()` fires, printing "Invalid Expression" and terminating.

## PROGRAM

### Lexer — `art_expr.l`

```lex
%{
#include <stdio.h>
#include "y.tab.h"
%}

%%

[a-zA-Z][0-9a-zA-Z]*    { return ID; }

[0-9]+                  { return DIG; }

[ \t]+                  { ; }

\n                      { return 0; }

.                       { return yytext[0]; }

%%

int yywrap()
{
    return 1;
}
```

### Parser — `art_expr.y`

```yacc
%{
#include <stdio.h>
#include <stdlib.h>

int yylex();
int yyerror(const char *s);
%}

%token ID DIG

%left '+' '-'
%left '*' '/'
%right UMINUS

%%

stmt: expn
    ;

expn: expn '+' expn
    | expn '-' expn
    | expn '*' expn
    | expn '/' expn
    | '-' expn %prec UMINUS
    | '(' expn ')'
    | DIG
    | ID
    ;

%%

int main()
{
    printf("Enter the Expression:\n");
    yyparse();
    printf("Valid Expression\n");
    return 0;
}

int yyerror(const char *s)
{
    printf("Invalid Expression\n");
    exit(0);
}
```

## COMPILATION

```bash
bison -y -d art_expr.y
flex art_expr.l
gcc y.tab.c lex.yy.c -o art_expr -lfl
```

## EXECUTION AND OUTPUT

**Test 1 — valid expression**

Input:
```
a+b*c
```

Output:
```
Enter the Expression:
Valid Expression
```

**Test 2 — invalid expression**

Input:
```
a+*b
```

Output:
```
Enter the Expression:
Invalid Expression
```

## RESULT
The FLEX and BISON programs were successfully compiled and executed. The lexer correctly tokenized identifiers, digits, and operators, and the parser correctly applied the grammar rules with defined operator precedence and associativity (`+`/`-` lower than `*`/`/`, unary minus highest). Valid arithmetic expressions such as `a+b*c` were accepted, while syntactically invalid expressions such as `a+*b` were correctly rejected, verifying the working of the arithmetic expression recognizer.
