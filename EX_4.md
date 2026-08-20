# Experiment: Recognition of a Valid Variable using FLEX and BISON

## AIM
To write a program to recognize a valid variable which starts with a letter followed by any number of letters or digits using FLEX and BISON.

## ALGORITHM

### FLEX
1. Declare the required header file and variable declaration within `%{` and `%}`.
2. FLEX requires regular expressions or patterns to identify token of lexemes for recognizing a valid variable.
3. FLEX calls `yywrap()` function after input is over. It should return 1 when work is done or should return 0 when more processing is required.

### BISON
1. Declare the required header file and variable declaration within `%{` and `%}`.
2. Define tokens in the first section and also define the associativity of the operations.
3. Mention the grammar productions and the action for each production.
4. `$$` refers to the top of the stack position while `$1` refers to the first value, `$2` for the second value in the stack.
5. Call `yyparse()` to initiate the parsing process.
6. `yyerror()` function is called when none of the productions in the grammar match the input statement.

## PROCEDURE
1. Create the FLEX file `valvar.l` using a text editor.
2. Define token patterns for letters and digits, and return them as `LET` and `DIG` tokens.
3. Save and close the file.
4. Create the BISON file `valvar.y`.
5. Define grammar rules that recognize valid variable names (starting with a letter and followed by letters or digits).
6. Save and close the BISON file.
7. Open terminal/command prompt and compile using:
   ```bash
   flex valvar.l
   bison -d valvar.y
   gcc lex.yy.c valvar.tab.c -o valvar -lfl
   ```
8. Run the program: `./valvar`
9. Enter test inputs like `abc1`, `var123`, `1abc` to test validation.
10. Observe whether the variable is valid or invalid.

## LOGIC / PSEUDOCODE

**Lexer (`valvar.l`)** — scans the input one character at a time and classifies each as a letter or digit token:

```
RULE: single letter (a-z, A-Z)   -> RETURN token LET
RULE: single digit (0-9)          -> RETURN token DIG
RULE: newline                      -> RETURN 0 (end of input)
RULE: any other single character  -> RETURN that character itself

FUNCTION yywrap()
    RETURN 1   // no more input files to process
```

Note: unlike a typical identifier lexer, this `.l` file matches **one letter or digit at a time**, not a whole run of them — so the grammar (not the lexer) is responsible for stringing multiple `LET`/`DIG` tokens together into a valid variable name.

**Parser (`valvar.y`)** — defines the grammar for a valid variable name:

```
TOKENS: LET, DIG

GRAMMAR:
    variable -> var

    var  -> var DIG
          | var LET
          | LET

FUNCTION main()
    PRINT "Enter the variable:"
    CALL yyparse()             // reads tokens from yylex() and matches grammar
    PRINT "Valid variable"     // reached only if parsing completes without error
    RETURN 0

FUNCTION yyerror(message)
    PRINT "Invalid variable"
    EXIT
```

**Flow:** The grammar `var -> LET | var LET | var DIG` enforces that a valid variable must begin with a single letter (`LET`), and every subsequent character may be either a letter or a digit. If the very first character is a digit, no production matches and `yyerror()` fires immediately, printing "Invalid variable". If the whole input reduces cleanly to `variable`, "Valid variable" is printed.

## PROGRAM

### Lexer — `valvar.l`

```lex
%{
#include "y.tab.h"
%}

%%

[a-zA-Z]    { return LET; }
[0-9]       { return DIG; }

\n          { return 0; }

.           { return yytext[0]; }

%%

int yywrap()
{
    return 1;
}
```

### Parser — `valvar.y`

```yacc
%{
#include <stdio.h>
#include <stdlib.h>

int yylex();
int yyerror(const char *);
%}

%token LET DIG

%%

variable: var
         ;

var: var DIG
   | var LET
   | LET
   ;

%%

int main()
{
    printf("Enter the variable:\n");
    yyparse();
    printf("Valid variable\n");
    return 0;
}

int yyerror(const char *s)
{
    printf("Invalid variable\n");
    exit(0);
}
```

## COMPILATION

```bash
bison -y -d valvar.y
flex valvar.l
gcc y.tab.c lex.yy.c -o valvar -lfl
```

## EXECUTION AND OUTPUT

**Test 1 — valid variable (starts with a letter)**

Input:
```
abc123
```

Output:
```
Enter the variable:
Valid variable
```

**Test 2 — invalid variable (starts with a digit)**

Input:
```
123abc
```

Output:
```
Enter the variable:
Invalid variable
```

## RESULT
The FLEX and BISON programs were successfully compiled and executed. The parser correctly accepted variable names that start with a letter and are followed by any combination of letters or digits (e.g., `abc123`), and correctly rejected inputs starting with a digit (e.g., `123abc`), verifying the working of the valid variable recognizer.
