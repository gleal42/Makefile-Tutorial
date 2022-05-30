## Makefile Tutorial

Some things I wish I knew about Makefiles when I first started using them. Please feel free to pull request and suggest improvements.

## Topics that will be covered

1. [Introduction](#1-introduction)
2. [Implicit Rules](#2-implicit-rules)
   - [Example 1 - The Simplest Makefile](#example-1---the-simplest-makefile)
   - [Example 2 - Multiple Sources](#example-2---multiple-sources)
   - [Example 3 - 42 Requirements](#example-3---42-requirements)

### 1. Introduction

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


### 2. Implicit Rules

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
- % is a wildcard, meaning it can be replaced by any combination of characters. When there are 2 % signs, the second one will be replace by the same combination of characters as the first one (i.e. they must match).


### Example 1 - The Simplest Makefile

File: main.c
Program to be created: main

**main.c**
```
#include <stdio.h>

int	main(void)
{
	printf("Hello\n");
}
```

**Makefile**
```
main:
```

If you go to the terminal and write "make", you will see that a program test will be created from our main.c file
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

On the following chapter I will show you different ways of compiling more than one source and explain which one I believe is the best and why.



### Example 2 - Multiple Sources


### Example 3 - 42 Requirements

- Compiler - gcc
- Flags - -Wall -Wextra -Werror (you can add more if you want)
- There must exist the rules all clean fclean re
- The program must not relink

The rules given are just the standard [.PHONY conventions](https://www.oreilly.com/library/view/managing-projects-with/0596006101/ch02.html)<br>

According to [this link](https://docs.oracle.com/cd/E19504-01/802-5880/6i9k05dhg/index.html), `LINK.c` will be replaced by:<br>
`$(CC) $(CFLAGS) $(CPPFLAGS) $(LDFLAGS)`, which are by default:
CC=cc

This means that we can simply change the Variables CC, CPPFLAGS and CFLAGS to fit our needs:
**Makefile**
```
CC=gcc
CFLAGS= -Wall -Wextra -Werror
test:
```

will now compile
`gcc -Wall -Wextra -Werror test.c -c -o test`

which means that we can create something useful without writing much.

> Tip: If you ever want to see what the Makefile is doing under the hood make sure to type `make -d` (d stands for debug)


```
CC = c++
CFLAGS := -Wall -Wextra -Werror -std=c++98
CXXFLAGS := -Wall -Wextra -Werror -std=c++98
LDFLAGS := -Wall -Wextra -Werror -std=c++98
SRCS := megaphone.cpp
OBJS := $(SRCS:.cpp=.o)
INC := $(SRCS:.cpp=.hpp)
NAME := megaphone
all: $(NAME)
$(NAME): $(OBJS)
$(OBJS): $(SRCS) Makefile $(INC)
clean:
	$(RM) $(OBJS)
fclean: clean
	$(RM) $(NAME)
re: fclean all
.PHONY: all clean fclean re
.SUFFIXES: .o .cpp
```
(https://stackoverflow.com/questions/55727692/how-does-make-build-an-executable-file-from-a-c-source-file-when-the-implicit-ru)

### Other References:

- https://stackoverflow.com/questions/36209876/generating-dependency-files-in-separate-folder
- https://stackoverflow.com/questions/66118766/what-is-include-in-gnu-make-and-how-it-works
- http://make.mad-scientist.net/papers/advanced-auto-dependency-generation/
- https://stackoverflow.com/questions/52034997/how-to-make-makefile-recompile-when-a-header-file-is-changed
