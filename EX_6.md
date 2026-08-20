# Experiment: Implementation of a Calculator using FLEX and BISON

## AIM
To write a program to implement a Calculator using FLEX and BISON.

## ALGORITHM
1. Start the program.
2. In the definitions part of the FLEX file, include the regular definition for a digit.
3. In the rules part of the FLEX file, specify the pattern for a number and its action (return `NUM` with the numeric value in `yylval`) in `cal.l`.
4. In the BISON program, define grammar rules so that all the arithmetic operations `+`, `-`, `*`, `/` are evaluated using operator precedence.
5. Display error if the input does not match the grammar.
6. Provide the input.
7. Verify the output.
8. End.

## PROCEDURE
1. Create a file named `cal.l` and define patterns to identify numbers using regular expressions.
2. For matched digits, return the token `NUM` and store the number using `yylval`.
3. Create another file named `cal.y` to define grammar rules for arithmetic expressions using BISON.
4. Include operator precedence and associativity using `%left` and `%right`.
5. Add rules to evaluate expressions like `E + E`, `E - E`, `E * E`, and `E / E`.
6. Compile both files using the following commands:
   ```bash
   flex cal.l
   bison -d cal.y
   gcc lex.yy.c cal.tab.c -o calc -lfl
   ```
7. Run the compiled program: `./calc`
8. Enter arithmetic expressions and view the result.
9. Test with multiple expressions to verify both valid and invalid cases.

## LOGIC / PSEUDOCODE

**Lexer (`cal.l`)** — tokenizes numbers and passes operator/newline characters through as-is:

```
DEFINE DIGIT = one or more digits [0-9]+

RULE: DIGIT
    yylval = numeric value of the matched text     // convert string to double
    RETURN token NUM

RULE: space or tab   -> IGNORE

RULE: newline
    RETURN '\n'      // signals end of one expression/statement

RULE: any other single character
    RETURN that character itself   // '+', '-', '*', '/' passed through literally
```

Note: `%option noyywrap` is set, so no `yywrap()` function needs to be defined — FLEX assumes end of input after one file/stream.

**Parser (`cal.y`)** — defines grammar with operator precedence/associativity and evaluates the expression as it parses:

```
VALUE TYPE: double        // every token/nonterminal carries a double value

TOKEN: NUM

PRECEDENCE (lowest to highest):
    left  '+' '-'
    left  '*' '/'
    right UMINUS          // declared but unused in the grammar rules below

GRAMMAR (with semantic actions):
    statement -> E '\n'
                 { PRINT "Answer:" $1 }              // $1 = value of E

    E -> E '+' E   { $$ = $1 + $3 }
       | E '-' E   { $$ = $1 - $3 }
       | E '*' E   { $$ = $1 * $3 }
       | E '/' E   { $$ = $1 / $3 }
       | NUM       { $$ = $1 }

FUNCTION main()
    PRINT "Enter the expression: "
    CALL yyparse()
    RETURN 0

FUNCTION yyerror(message)
    PRINT "Invalid expression"
```

**Flow:** As `yyparse()` reduces the grammar bottom-up, each rule's semantic action (`$$ = $1 + $3`, etc.) computes the numeric result immediately — so by the time the top-level `statement: E '\n'` rule fires, `$1` already holds the fully evaluated result of the whole expression, which is printed as "Answer: <value>". If the input doesn't match any valid sequence of `NUM` and operators, `yyerror()` fires and "Invalid expression" is printed instead.

## PROGRAM

### Lexer — `cal.l`

```lex
%{
#include "cal.tab.h"
#include <stdlib.h>
%}

%option noyywrap

DIGIT [0-9]+

%%

{DIGIT} {
    yylval = atof(yytext);
    return NUM;
}

[ \t] {
}

\n {
    return '\n';
}

. {
    return yytext[0];
}

%%
```

### Parser — `cal.y`

```yacc
%{
#include <stdio.h>
#include <stdlib.h>

int yylex(void);
void yyerror(const char *s);
%}

%define api.value.type {double}

%token NUM

%left '+' '-'
%left '*' '/'
%right UMINUS

%%

statement:
    E '\n'
    {
        printf("Answer: %g\n", $1);
    }
    ;

E:
    E '+' E
    {
        $$ = $1 + $3;
    }
    | E '-' E
    {
        $$ = $1 - $3;
    }
    | E '*' E
    {
        $$ = $1 * $3;
    }
    | E '/' E
    {
        $$ = $1 / $3;
    }
    | NUM
    {
        $$ = $1;
    }
    ;

%%

int main(void)
{
    printf("Enter the expression: ");
    yyparse();
    return 0;
}

void yyerror(const char *s)
{
    printf("Invalid expression\n");
}
```

## COMPILATION

```bash
bison -d cal.y
flex cal.l
gcc lex.yy.c cal.tab.c -o calc
```

## EXECUTION AND OUTPUT

**Test 1**

Input:
```
2+2
```

Output:
```
Enter the expression: Answer: 4
```

**Test 2**

Input:
```
10+20
```

Output:
```
Enter the expression: Answer: 30
```

## RESULT
The FLEX and BISON programs were successfully compiled and executed. The calculator correctly evaluated arithmetic expressions using operator precedence and associativity, producing `Answer: 4` for the input `2+2` and `Answer: 30` for the input `10+20`, verifying the working of the calculator implementation.

> **Note:** Both test runs in this notebook use valid addition expressions only. No invalid-expression test (e.g. malformed input to trigger `yyerror()` / "Invalid expression") was run. If you'd like that added for completeness alongside the other experiments, let me know the input to use.
