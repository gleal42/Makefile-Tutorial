## Makefile-Tutorial

Some things I which I new about Makefile when I started

### Makefile Defaults

If you're compiling .c files the Makefile itself already has implicit rules that will try to compile the programs and the .o files that you require:

### Example 1 - One program with same name as file

File: test.c
Program to be created: test

**test.c**
```
#include <stdio.h>

int	main(void)
{
	printf("Hello\n");
}
```

**Makefile**
```
test:
```

If you go to the terminal and write "make", you will see that a program test will be created from our test.c file
`cc test.c -c -o test`

According to GNU.org (https://www.gnu.org/software/make/manual/make.html#Catalogue-of-Rules)
Compiling C programs
n.o is made automatically from n.c with a recipe of the form ‘$(CC) $(CPPFLAGS) $(CFLAGS) -c’.

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


### Other References:

- https://stackoverflow.com/questions/36209876/generating-dependency-files-in-separate-folder
- https://stackoverflow.com/questions/66118766/what-is-include-in-gnu-make-and-how-it-works
- http://make.mad-scientist.net/papers/advanced-auto-dependency-generation/
- https://stackoverflow.com/questions/52034997/how-to-make-makefile-recompile-when-a-header-file-is-changed
