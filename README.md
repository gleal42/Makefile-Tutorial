## Makefile Tutorial
Some things I wish I knew about Makefiles when I first started using them. Please feel free to pull request and suggest improvements.

## Topics that will be covered

1. [Introduction](#1-introduction)
2. [The Empty/Simple Makefile - Intro to Implicit Rules](#2-the-emptysimple-makefile---intro-to-implicit-rules)
3. [Multiple Sources](#3-multiple-sources)
4. [42 Requirements](#4-42-requirements)
   - [Compiler (gcc or cc)](#41-compiler)
   - [Flags (-Wall -Wextra -Werror)](#42-flags)
   - [Rules $(NAME) all clean fclean re](#43-rules-name-all-clean-fclean-re)
   - [Program must not relink](#44-the-program-must-not-relink)

___

## 1. Introduction

In compiled languages like c or cpp, the source files (aka .c and .cpp files) are turned into binaries (low level, assembly code), which are then linked to form an executable (or archived into a static library).

The Makefile is a script-like tool made to help us with this process allowing us to:
 - Compile programs with templated compilation rules.
 - Change those templates(i.e. recipes) to fit our needs
 - Recognize changed source files and compile only the necessary binaries
 - Create variables with regex-like tools
 - Search for source files in different directories
 - See if directory exists or not and create it if it doesn't
 - Analyze how header files affect source files and compile only the necessary binaries (Complex topic)

Even though it may appear as a blank document we will see that, without writing much, we will be able to compile even programs with complex folder structures.

Once we create a Makefile document, when we type the command `make`, it will execute the same way a bash script would.

I will focus on **C** and **CPP** compilation (using gcc and c++ compilers), since these are taught at the coding school I'm studying in (School 42) however the contents might still apply to the compilation of other languages.

___

## 2. The Empty/Simple Makefile - Intro to Implicit Rules

The Makefile was made already with [certain programming languages](http://gnu.ist.utl.pt/software/autoconf/manual/make/Catalogue-of-Rules.html) in mind.

This means that if we don't write our own rules, Makefile will use its own, in order to show this I will make the simples Makefile I can.

> Tip: To [see the list of implicit rules](https://www.oreilly.com/library/view/managing-projects-with/0596006101/ch02.html) use the command `make -p`

Usually an implicit rule will look something like:

% : %.c <br>
% : %.o <br>
%.o : %.c <br>

Let's break these down:

The sintax is usually:
```
target: normal pre-req | order-only pre-req
	recipes
```

- Target is the thing we want to create (or the name of the rule).<br>
- Pre-requisites are the thing on which the target depends on:<br>
  - For normal pre-requisites the target needs to be rebuilt if they are updated.<br>
  - For order-only pre-requisites, these rules only run once, in case these requirements don't exist (e.g. directories only need to be built once).<br>
- Recipes are a series of commands we want to run.
- % is a wildcard, meaning it can be replaced by any combination of characters. When there are [2 % signs](https://stackoverflow.com/questions/7404444/what-does-a-percent-symbol-do-in-a-makefile), the second one will be replace by the same combination of characters as the first one (i.e. they must match).

File: main.c <br>
Program to be created: main

**main.c**
```
#include <stdio.h>

int	main(void)
{
	printf("Hello\n");
}
```

**Makefile (EMPTY)**
```

```

If you go to the terminal and write "make main", you will see that a program test will be created from our main.c file  <br>
`cc main.c -o main`

This is because there is an implicit rule to create an executable from .c files (`% : %.c`):<br>

```
%: %.c
#  commands to execute (built-in):
	$(LINK.c) $^ $(LOADLIBES) $(LDLIBS) -o $@
```
Where the defaults are:<br>
LINK.c = cc <br>
$^ = main.c (all pre-requirements) <br>
$@ = main (target)<br>

> Tip: `$^ $@ $<..` are called [automatic variables](https://www.gnu.org/software/make/manual/make.html#Automatic-Variables)

If you would like to just type "make" instead of "make main" you can just add the rule to the Makefile.

**Makefile**
```
main:
```

When you run `make` the Makefile will run the first rule (the most at the top of the page)

On the following chapter I will show you different ways of compiling more than one source and explain which one I believe is the best and why.

___

## 3. Multiple Sources

Files: main.c test.c
Header files: test.h
Program to be created: main

**main.c** <br>

```
#include "test.h"

int	main(void)
{
	test_print("Hello\n");
}
```

**test.h** <br>

```
#ifndef TEST_H
#define TEST_H

#include <stdio.h>
void	test_print();

#endif
```
**test.c**<br>

```
#include "test.h"

void	test_print()
{
	printf("Hello\n");
}
```

So now we have a more complex situation, because main.c calls for a function that is in test.c

And we can choose one of 2 simple makefiles:

**Makefile 1**<br>

```
main: test.c
```

**Makefile 2**<br>
```
main: main.o test.o
```

> Note: For some reason if we didn't add main.o it would execute `cc     -c -o test.o test.c` `cc     main.c test.o   -o main`, not matching the example given in [make documentation](https://www.gnu.org/software/make/manual/html_node/Catalogue-of-Rules.html#Catalogue-of-Rules) when referring to "Linking a single object file".

Now at first glance it may appear that the first option is better since we need to write less and even the Makefile [seems to prefer it](https://stackoverflow.com/questions/55727692/how-does-make-build-an-executable-file-from-a-c-source-file-when-the-implicit-ru).<br>

However, right now we are dealing with only 2 source files. 

Now imagine a project with 20 files `a.c b.c c.c d.c ...` <br>
Choosing the first option would mean that we would always recompile source files `a.c b.c c.c d.c ...`, even if only one of those sources was changed.

By using the 2nd option, if we change a.c, only a.o will be created and then we relink all o files.

___

## 4. 42 Requirements

1. [Compiler (gcc or cc)](#41-compiler)
2. [Flags (-Wall -Wextra -Werror)](#42-flags)
3. [Rules $(NAME) all clean fclean re](#43-rules-name-all-clean-fclean-re)
4. [Program must not relink](#44-the-program-must-not-relink)

### 4.1. Compiler

So now we are using 2 different implicit rules:<br>

```
%.o: %.c <br>
	$(COMPILE.c) $(OUTPUT_OPTION) $<<br>
```
```
%: %.o <br>
	$(LINK.o) $^ $(LOADLIBES) $(LDLIBS) -o $@<br>
```
`COMPILE.c = $(CC) $(CFLAGS) $(CPPFLAGS) $(TARGET_ARCH) -c` <br>
`OUTPUT_OPTION = -o $@` <br>
`LINK.o = $(CC) $(LDFLAGS) $(TARGET_ARCH)` <br>
`CC=cc` <br>

And the remaining rules are blank.

So instead of writing our own rules, we can simply modify the variables that already exist:

If we want to change the compiler from cc to gcc:

**Makefile**<br>

```
CC=gcc
main: main.o test.o
```
Will output:<br>
`gcc    -c -o main.o main.c`<br>
`gcc    -c -o test.o test.c`<br>
`gcc   main.o test.o   -o main`<br>


### 4.2. Flags

If we want to add the 42 required flags:<br>
**Makefile**
```
CC=gcc
CFLAGS= -Wall -Wextra -Werror
LDFLAGS = $(CFLAGS)
main: main.o test.o
```

Will output:<br>
`gcc -Wall -Wextra -Werror   -c -o main.o main.c`<br>
`gcc -Wall -Wextra -Werror   -c -o test.o test.c`<br>
`gcc -Wall -Wextra -Werror  main.o test.o   -o main`<br>



### 4.3. rules $(NAME) all clean fclean re

Most of these rules are [Standard Conventions](https://www.oreilly.com/library/view/managing-projects-with/0596006101/ch02.html)<br>

- $(NAME) - Name of the program<br>
- all - Do all tasks to build program<br>
- clean - Delete binaries generated from sources (.o files)<br>
- fclean - Equivalent to distclean (remove all files produced by Makefile)<br>
- re - Though not standard rule means to remove all files and rebuild the program

$(NAME) represents a file (executable) <br>
all, clean, fclean, re don't represent files.

In order to stop Makefile from trying to create files with these names we must add them as Phony Targets: <br>

**Makefile**
```
CC=gcc
CFLAGS= -Wall -Wextra -Werror
LDFLAGS = $(CFLAGS)
$(NAME)=main
all: $(NAME)
$(NAME): main.o test.o
clean:
	$(RM) *.o
fclean: clean
	$(RM) $(NAME)
re: fclean all
.PHONY: all clean fclean re
```

> Tip: If you ever want to see what the Makefile is doing under the hood make sure to type `make -d` (d stands for debug)


### 4.4. "The Program must not relink"

What this essentially means is that we type the command `make` multiple times there won't be a new recompilation and linkage.

`make: Nothing to be done for 'rule'.`

This one of the main things that distinguishes Makefile from a bash script. If we don't need to recompile then we won't do it.

The main reasons why I see this happening is:

##### 1. Deleting or moving object files

I get it.. .o files are hard to look at. However, if you always get rid of them by default then might as well compile directly (like we saw on previous chapters).

**Makefile that deletes .o files**
```
...
all: $(NAME) clean
...
```

**Makefile that moves .o files**
```
...
$(NAME): main.o test.o | $(OBJ_DIR)
	$(CC) $(CFLAGS) $^ -o $@
	mv *.o $(OBJ_DIR)
$(OBJ_DIR):
	mkdir $@
...
```

### Other References:

- https://stackoverflow.com/questions/36209876/generating-dependency-files-in-separate-folder
- https://stackoverflow.com/questions/66118766/what-is-include-in-gnu-make-and-how-it-works
- http://make.mad-scientist.net/papers/advanced-auto-dependency-generation/
- https://stackoverflow.com/questions/52034997/how-to-make-makefile-recompile-when-a-header-file-is-changed
- https://docs.oracle.com/cd/E19504-01/802-5880/6i9k05dhg/index.html
