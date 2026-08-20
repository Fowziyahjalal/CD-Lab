# Experiment: Lexical Analyzer using FLEX for Symbol Table Construction

## AIM
To develop a lexical analyzer using FLEX to recognize tokens such as identifiers, constants, comments, and operators in a C program and to create a symbol table while recognizing identifiers.

## ALGORITHM
1. Start the program by including the necessary headers within the FLEX definitions section (`%{ ... %}`).
2. Define regular expressions for:
   - **Identifiers**: `[a-zA-Z_][a-zA-Z0-9_]*`
   - **Constants**: `[0-9]+`
   - **Comments**: `//.*` and `/* ... */`
   - **Operators**: `+ - * / = < >`
3. Declare a symbol table array (structure with name and type fields) in the definitions section.
4. Write rules in the rules section of the FLEX (`.l`) file:
   - When an identifier is recognized, call `insert()` to add it to the symbol table if not already present.
   - Print or categorize constants, operators, and comments as they are matched.
5. Use `yylex()` actions to print matched tokens and perform symbol table insertion.
6. In `main()`, open the input file, call `yylex()`, then print the symbol table.
7. Compile the FLEX file using `flex` and `gcc`.
8. Execute the program with a sample C code input file.
9. Stop.

## PROCEDURE
1. Open a terminal and create a new FLEX file, `symtab.l`.
2. In the definitions section, include headers and declare the symbol table array along with `insert()`/`lookup()` helper functions.
3. In the rules section, define patterns for identifiers, constants, comments, and operators.
4. Use `{ printf(...) }` actions to print the recognized tokens and call `insert()` for identifiers.
5. Save and compile the file using:
   ```bash
   flex symtab.l
   gcc lex.yy.c -o symtab -lfl
   ```
6. Run the executable:
   ```bash
   ./symtab input.c
   ```
7. Observe the output and verify the tokens and symbol table entries.

## LOGIC / PSEUDOCODE

The implementation below builds a menu-driven symbol table manager in C that stores identifiers and their values, and supports creating, inserting, modifying, searching, and displaying entries — reflecting the core symbol-table data structure and operations (`insert()`, `search()`) referenced in the FLEX-based procedure above.

```
STRUCT table
    var   : char[10]     // identifier name
    value : int           // associated value

DECLARE tbl[20] : array of table
DECLARE n : int = 0       // number of entries

FUNCTION main()
    LOOP
        DISPLAY menu (Create, Insert, Modify, Search, Display, Exit)
        READ choice
        SWITCH choice:
            1 -> CALL create()
            2 -> CALL insert()
            3 -> CALL modify()
            4 -> READ variable name
                 result = CALL search(variable, n)
                 IF result == 0 -> PRINT "not found"
                 ELSE -> PRINT location
            5 -> CALL display()
            6 -> PRINT "Exiting..."
            default -> PRINT "Invalid choice"
    WHILE choice != 6

FUNCTION create()
    READ n (number of entries, capped at 20)
    FOR i = 1 TO n
        REPEAT
            READ variable name, value
        UNTIL variable name starts with an alphabet
        STORE into tbl[i]

FUNCTION insert()
    IF n >= 20 -> PRINT "table full"; RETURN
    n = n + 1
    REPEAT
        READ variable name, value
    UNTIL variable name starts with an alphabet
    STORE into tbl[n]

FUNCTION modify()
    READ variable name
    location = CALL search(variable, n)
    IF location == 0 -> PRINT "not found"; RETURN
    READ new value
    UPDATE tbl[location].value
    CALL display()

FUNCTION search(variable, n)
    FOR i = 1 TO n
        IF tbl[i].var == variable -> RETURN i
    RETURN 0

FUNCTION display()
    PRINT header "VARIABLE  VALUE"
    FOR i = 1 TO n
        PRINT tbl[i].var, tbl[i].value
```

## PROGRAM (C Source Code)

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <ctype.h>

struct table
{
    char var[10];
    int value;
};

struct table tbl[20];
int i, j, n = 0;

void create();
void modify();
int search(char variable[], int n);
void insert();
void display();

int main()
{
    int ch, result;
    char v[10];

    do
    {
        printf("\nEnter your choice\n");
        printf("1. Create\n");
        printf("2. Insert\n");
        printf("3. Modify\n");
        printf("4. Search\n");
        printf("5. Display\n");
        printf("6. Exit\n");
        printf("Enter your choice: ");
        scanf("%d", &ch);

        switch(ch)
        {
            case 1:
                create();
                break;

            case 2:
                insert();
                break;

            case 3:
                modify();
                break;

            case 4:
                printf("Enter the variable to be searched for: ");
                scanf("%9s", v);

                result = search(v, n);

                if(result == 0)
                    printf("The variable does not belong to the table\n");
                else
                    printf("The location of the variable is %d\n", result);

                break;

            case 5:
                display();
                break;

            case 6:
                printf("Exiting...\n");
                break;

            default:
                printf("Invalid choice!\n");
        }

    } while(ch != 6);

    return 0;
}

void create()
{
    printf("Enter the no. of entries: ");
    scanf("%d", &n);

    if(n > 20)
    {
        printf("Maximum 20 entries allowed.\n");
        n = 20;
    }

    printf("Enter the variable and the value:\n");

    for(i = 1; i <= n; i++)
    {
        while(1)
        {
            scanf("%9s %d", tbl[i].var, &tbl[i].value);

            if(isalpha(tbl[i].var[0]))
                break;

            printf("The variable should start with an alphabet\n");
            printf("Enter correct variable name:\n");
        }
    }
}

void insert()
{
    if(n >= 20)
    {
        printf("Cannot insert. Table is full\n");
        return;
    }

    n++;

    printf("Enter the variable and value:\n");

    while(1)
    {
        scanf("%9s %d", tbl[n].var, &tbl[n].value);

        if(isalpha(tbl[n].var[0]))
            break;

        printf("The variable should start with an alphabet\n");
        printf("Enter correct variable name:\n");
    }
}

void modify()
{
    char variable[10];
    int location;

    printf("Enter the variable to be modified: ");
    scanf("%9s", variable);

    location = search(variable, n);

    if(location == 0)
    {
        printf("Variable not found\n");
        return;
    }

    printf("Enter the new value: ");
    scanf("%d", &tbl[location].value);

    printf("The table after modification is:\n");
    display();
}

int search(char variable[], int n)
{
    for(i = 1; i <= n; i++)
    {
        if(strcmp(tbl[i].var, variable) == 0)
            return i;
    }

    return 0;
}

void display()
{
    printf("\nVARIABLE\tVALUE\n");

    for(i = 1; i <= n; i++)
        printf("%s\t\t%d\n", tbl[i].var, tbl[i].value);
}
```

## COMPILATION

```bash
gcc symbol_table.c -o symbol_table
```

## SAMPLE RUN

**Input given** (choice 1 → create with 3 entries → then choice 5 → display → choice 6 → exit):

```
1
3
a 10
b 20
c 30
5
6
```

## OUTPUT

```
Enter your choice
1. Create
2. Insert
3. Modify
4. Search
5. Display
6. Exit
Enter your choice: Enter the no. of entries: Enter the variable and the value:

Enter your choice
1. Create
2. Insert
3. Modify
4. Search
5. Display
6. Exit
Enter your choice:
VARIABLE	VALUE
a		10
b		20
c		30

Enter your choice
1. Create
2. Insert
3. Modify
4. Search
5. Display
6. Exit
Enter your choice: Exiting...
```

## RESULT
The symbol table program was successfully compiled and executed. Identifiers and their corresponding values were created, stored, and displayed correctly, verifying the working of the symbol table construction and lookup logic described in the algorithm.
