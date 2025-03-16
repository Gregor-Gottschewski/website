---
layout: post
title:  "Writing a Lexer using Flex"
date:   2025-03-08 11:15:00 +0100
categories: compiler lexer
---

A lexical analyzer (or, simply _lexer_) is a program used in compiler development. Lexers generate tokens from input text, mostly source code, which is essential for syntax analysis. You could write a lexer from scratch using pure C. However, that approach is inefficient and mistakes can happen quickly.

When looking to some source code, you will notice that a keyword cannot be identified by looking at just the first character of a given input. Often, you need to consider both the preceding and following characters.

Let's take a look at a simple C snippet:

{%highlight C%}
main(void) {
  int my_num1 = 5;
  float my_num2 =   2.2;
  int floating = 3;
  printf("int float void %i", my_num1);
}
{% endhighlight %}

If you build a lexer that does not account for surrounding characters, some problems may arise:
1. **Whitespace:** the lexer must ignore whitespace, since `int i = 3` is functionally identical to `int i=3`.
2. **Identifiers:** most programming languages allow numbers in variable or function names. A lexer that ignores surrounding characters might interpret `my_num2` as `my_num` and `2`.
3. **Strings:** a lexer must handle string content, which is impossible without analyzing both the preceding and following characters.

## Tokens

Let's use the given C code for the following example. The output of a C-lexer for the given source code above could be:

```
IDENT(main) LPAREN VOID RPAREN LBRACE INT IDENT(my_num1) ASSIGN NUMBER(5) SEMICOLON FLOAT IDENT(my_num2) ASSIGN NUMBER(2.2) SEMICOLON INT IDENT(floating)...
```

All whitespace are ignored, keywords are separated from other identifiers (notice `floating`, which contains the keyword `float` but is correctly identified as an identifier).

> **tl;dr** Tokens provide a structured representation of source code. They are the output of a lexer.

## A Language

The source code above is valid C code. But let's take a look at some invalid C code:

{%highlight C%}
main(i_love_pizza) {
  float float float;
  int int magic_muhaha;
  what the hell happens here
}
{% endhighlight %}

This code doesn't compile, but the keywords a valid C keywords. Also, `what the hell happens here` is valid; `IDENT(what) IDENT(the) IDENT(hell) IDENT(happens) IDENT(here)` are all valid identifiers.
It is extremely important to know that a lexer doesn't check for syntax or semantics. It only generates tokens.
However, if a sequence of characters does not produce a valid token, then it is not part of the language’s lexicon.
For example, `°` is not valid in C. The C-lexer cannot classify it as a token, meaning it is not part of the language. 

> **tl;dr** The lexer doesn't check for semantics or syntax, however the lexer can determine whether a given input is part of a specified language lexicon.

That was enough theory for now. Let's start with something more practical.

## Hell no... regex

Yep, I am sorry. However, we have to talk about regex. Regex is nothing new, it exists since more than 50 years. The theory behind regex is huge and way too big to cover it up here. If you don't know regex, I recommend using [Regex One](https://www.regexone.com/) or other online tutorials to learn it.

Here's a small cheat sheet with the most important expressions to follow the next section:

* `.`: matches any character
* `^`: not
* `[abc]`: matches `a` or `b` or `c`
* `\d`: matches any character that is a digit
* `\D`: matches any character that is not a digit
* `\w`: matches any alphanumeric and underscore
* `\W`: matches any character that is not alphanumeric or underscore
* `*`: **zero** or more repetitions
* `+`: **one** or more repetitions
* `?`: optional
* `()`: group
* `|`: or

**Example:** In most programming languages, strings are marked by single or double quotes. By using `".*"` we are able to match all double-quoted strings (empty or not).

I created a small cheat sheet over the most important regex expressions as PDF. You can download it [directly](https://raw.githubusercontent.com/Gregor-Gottschewski/regex-cheat-sheet/refs/heads/main/cheat_sheet.pdf) or visit the [GitHub Repository page](https://github.com/Gregor-Gottschewski/regex-cheat-sheet).

## What is Flex?

To follow this tutorial you need [Flex](https://github.com/westes/flex). Follow the [installation instructions](https://github.com/westes/flex/blob/master/INSTALL.md), or use `brew install flex` if you're using Homebrew package manager to install it. For editing the files for Flex, I recommend using Xcode on Mac with disabled indentation, Visual Studio on Windows (not tested) and Vim on Linux. If you prefer Visual Studio Code, there are several plugins for available.

## Flex input file

The flex input file (e.g. `my_lexer.l`) consists of three sections:

```
definitions
%%
rules
%%
user code
```

The code inside these sections is written in C. The `definitions`-section contains includes and regex constants. The rules-section contain regex code and "actions". User code can contain everything you want. The user code is copied by Flex to the generated C-file.

## Creating Your First Lexer

I started this project to create a full C-compiler by myself.
However, I - and probably also you - have to start with something more simple.
So I created a small lexer that generates tokens for simple mathematical calculations:

```
5+3*5/6
5     +6.1 -   3
```

The lexer ignores whitespace, detects integers, floats,  mathematical operators (`+`, `-`, `*` and `/`) and new lines.

First, we define some regex constants that make our code more readable later. These constants can be used in the rules section by putting them between curly braces.
I created seven of these constants for all characters we want to recognize later:

```
DIGIT       \d
WHITESPACE  \w
PLUS        \+
MINUS       \-
STAR        \*
SLASH       \/
NEWLINE     \n
```

Now we have to define some C preprocessor constants. Be aware that they have nothing to do with the regex constants defined above.

```c
%{
#define NEWLINE_CODE    1
#define NUMBER_CODE     2
#define PLUS_CODE       3
#define MINUS_CODE      4
#define STAR_CODE       5
#define SLASH_CODE      6
%}
```

Afterward, we start specifying the ruleset.
For now, we define five rules:

1. ignore all types of whitespace
2. return `PLUS_CODE` when `+` found
3. return `MINUS_CODE` when `-` found
4. return `STAR_CODE` when `*` found
5. return `SLASH_CODE` when `/` found

A rule is defined by a regex expression followed by an action that is written in C and executed when the regex expression is matched.
You are free to write everything you want into the action code.
We will write simple return statements into these actions.
Later, we identify tokens by their return value to give feedback to the user.

```c
(your regex here) {
    // C code here...
}
```

That's everything we have to know for now. Let's start defining the five rules defined above:

```c
{WHITESPACE}+   { /* ignore */ }

{PLUS} {
    return PLUS_CODE;
}

{MINUS} {
    return MINUS_CODE;
}

{STAR} {
    return STAR_CODE;
}

{SLASH} {
    return SLASH_CODE;
}
```

Here I used the predefined regex constants. If you don't want to define these constants,  write:

```c
\w+   { /* ignore */ }
```

The rule to detect both integers and floating numbers is a little bit more complicated:

```c
{DIGIT}+(\.{DIGIT}+)? {
    printf("DIGIT found: %s\n", yytext);
    return NUMBER_CODE;
}
```

Let's take this apart by replacing `DIGIT` with the corresponding regex expression: `\d+(\.\d+)?`.
* `\d+`: one or more digits
* `(...)?`: group is optional
* `\.\d+`: string begins with `.` followed by one or more digits

**In natural language:** a number consist of one digit or more that can be followed by a period, which must be followed by one or more digits.

We've specified all rules we need! The last rule we have to add is: if no other rule can be applied, the character is not part of our language:

```c
. {
    fprintf(stderr, "unexpected char: '%s'\n", yytext);
    return -1;
}
```

## Adding User Code

To make the code executable, we have to add some user code to the file.
Add a `yywrap()`-function and a main-function that prints log messages:

```c
int yywrap() {
    return 1;
}
    
int main(void) {
    int token;

    do {
        token = yylex();
        switch (token) {
            case NEWLINE_CODE:
                printf("NEWLINE found\n");
                break;
            case PLUS_CODE:
                printf("PLUS found\n");
                break;
            case STAR_CODE:
                printf("STAR found\n");
                break;
            case MINUS_CODE:
                printf("MINUS found\n");
                break;
            case SLASH_CODE:
                printf("SLASH found\n");
                break;
            case NUMBER_CODE:
                printf("NUMBER found\n");
                break;
            case -1:
                return EXIT_FAILURE;
        }
    } while (token != 0);

    return EXIT_SUCCESS;
}
```

## Compiling the Executeable

Here the complete file to copy paste it:

{% highlight c %}
%{
#define NEWLINE_CODE    1
#define NUMBER_CODE     2
#define PLUS_CODE       3
#define MINUS_CODE      4
#define STAR_CODE       5
#define SLASH_CODE      6

#include <stdio.h> 
%}

DIGIT       [0-9]
WHITESPACE  [ \t\r]
PLUS        \+
MINUS       \-
STAR        \*
SLASH       \/
NEWLINE     \n

%%
{WHITESPACE}+   { /* ignore */ }

{PLUS} {
    return PLUS_CODE;
}

{MINUS} {
    return MINUS_CODE;
}

{DIGIT}+(\.{DIGIT}+)? {
    printf("DIGIT found: %s\n", yytext);
    return NUMBER_CODE;
}

{STAR} {
    return STAR_CODE;
}

{SLASH} {
    return SLASH_CODE;
}

{NEWLINE}+ {
    return NEWLINE_CODE;
}

. {
    fprintf(stderr, "unexpected char: '%s'\n", yytext);
    return 7;
}

%%
    
int yywrap() {
    return 1;
}

int main(void) {
    int token;

    do {
        token = yylex();
        switch (token) {
            case NEWLINE_CODE:
                printf("NEWLINE found\n");
                break;
            case PLUS_CODE:
                printf("PLUS found\n");
                break;
            case STAR_CODE:
                printf("STAR found\n");
                break;
            case MINUS_CODE:
                printf("MINUS found\n");
                break;
            case SLASH_CODE:
                printf("SLASH found\n");
                break;
            case NUMBER_CODE:
                printf("NUMBER found\n");
                break;
            case -1:
                return EXIT_FAILURE;
        }
    } while (token != 0);

    return EXIT_SUCCESS;
}
{% endhighlight %}

In the terminal, run `flex` in the same dictionary where the flex-file is stored: `flex lexer.l`.
Afterward, a file named `lex.yy.c` appears.
This file contains pure C code. Take a look at it.
The generated code has around 1800 lines of code. It would be hard to write this code without using Flex.
The last lines of the file contain our user code.

Now compile the generated C-code with your favorite compiler: `gcc lex.yy.c -o lexer` and run the executable: `./lexer`.

The output should look like that:

```
$ ./lexer
$ 4*4
DIGIT found: 4
STAR found
DIGIT found: 4

$ 4     +5.9
NEWLINE found
DIGIT found: 4
PLUS found
DIGIT found: 5.9

$ abracadabra
NEWLINE found
unexpected char: 'a'
unexpected char: 'b'
unexpected char: 'r'
unexpected char: 'a'
unexpected char: 'c'
unexpected char: 'a'
unexpected char: 'd'
unexpected char: 'a'
unexpected char: 'b'
unexpected char: 'r'
unexpected char: 'a'
```

Tadadaaa... you created your first lexer!

## What Next?
You can extend the lexer's functionality by adding support for braces and factorial.
In the next post, we will take a look at more complex lexers.
We will create a lexer that recognizes C code and outputs the tokens.