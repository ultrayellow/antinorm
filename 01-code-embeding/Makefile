CC		:= clang
CFLAGS	+= -Wall -Wextra -Werror
LD_DRIVER_FLAGS	+= -Wl,-segprot -Wl,__TEXT -Wl,rwx -Wl,rwx

NAME := push_swap
SRCS := push_swap.c

all : $(NAME)

$(NAME) : $(SRCS)
	$(CC) $(CFLAGS) $(LD_DRIVER_FLAGS) -o $@ $^

clean :

fclean : clean
	rm -f $(NAME)

re : fclean
	$(MAKE) $(NAME)

.PHONY : all clean fclean re