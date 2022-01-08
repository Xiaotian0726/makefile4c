# makefile4c
一个关于 C Project 的 makefile 模板

## Explanation
首先，确保你的 C Project 为如下结构：
```shell
root
├── include/
│   └── all .h files here
├── lib/
│   └── all third-party library files (.a/.so files) here
├── src/
│   └── all .c files here
└── Makefile
```

定义源文件、目标文件、可执行文件的目录：
```makefile
SRC_DIR := src
OBJ_DIR := obj
BIN_DIR := bin   # or . if you want it in the current directory
```

将最终的可执行文件（以 main 为例）定义为 `EXE`：
```makefile
EXE := $(BIN_DIR)/main
```

列出所有的源文件，定义为 `SRC`：
```makefile
SRC := $(wildcard $(SRC_DIR)/*.c)
```

根据源文件列出所有的目标文件，定义为 `OBJ`：
```makefile
OBJ := $(SRC:$(SRC_DIR)/%.c=$(OBJ_DIR)/%.o)
# or
OBJ := $(patsubst $(SRC_DIR)/%.c, $(OBJ_DIR)/%.o, $(SRC))
```

定义预处理选项 `CPPFLAGS`、编译选项 `CFLAGS`、链接选项 `LDFLAGS`、包含的库 `LDLIBS`：
```makefile
CPPFLAGS := -Iinclude -MMD -MP  # -I is a preprocessor flag, not a compiler flag
CFLAGS   := -Wall               # some warnings about bad code
LDFLAGS  := -Llib               # -L is a linker flag
LDLIBS   := -lm                 # Left empty if no libs are needed
```

其中，预处理选项 `-MMD` 和 `-MP` 用于自动生成头文件依赖，这样仅头文件发生改变时也可以触发编译。

默认的 make 目标一般被命名为 `all`：
```makefile
all: $(EXE)
```

不过，`Make` 可能会错误地认为你想要创建一个名为 `all` 的文件或者目录，因此需要加入以下处理：
```makefile
.PHONY: all
```

在链接的过程中列出构建可执行文件需要的所有目标文件：
```makefile
$(EXE): $(OBJ)
    $(CC) $(LDFLAGS) $^ $(LDLIBS) -o $@
```
其中，`$(CC)` 是默认的 C 编译器 `gcc`。预处理选项 `$(CPPFLAGS)` 和编译选项 `$(CFLAGS)` 在这里未被用到，因为这两个选项是在编译时期被用到的

另外，注意到 `$(BIN_DIR)` 可能还未创建，因此链接过程需要加一步检查：
```makefile
$(EXE): $(OBJ) | $(BIN_DIR)
    $(CC) $(LDFLAGS) $^ $(LDLIBS) -o $@

$(BIN_DIR):
    mkdir -p $@
```

编译的过程需要列出所有的源文件，并加入预处理选项 `$(CPPFLAGS)` 和编译选项 `$(CFLAGS)`：
```makefile
$(OBJ_DIR)/%.o: $(SRC_DIR)/%.c
    $(CC) $(CPPFLAGS) $(CFLAGS) -c $< -o $@
```

`$(OBJ_DIR)` 可能也仍未创建，因此同样加入一步检查，并将两个创建规则合并：
```makefile
$(OBJ_DIR)/%.o: $(SRC_DIR)/%.c | $(OBJ_DIR)
    $(CC) $(CPPFLAGS) $(CFLAGS) -c $< -o $@

$(BIN_DIR) $(OBJ_DIR):
    mkdir -p $@
```

至此，最终的可执行文件已经可以被成功构建。添加 `clean` 规则来清理构建产物，并把 `clean` 加入 `.PHONY` 中：
```makefile
.PHONY: all clean

...

clean:
    @$(RM) -rv $(BIN_DIR) $(OBJ_DIR) # The @ disables the echoing of the command
```

最后，由于自动头文件依赖的生成， `GCC` 和 `Clang` 会根据你的 `.o` 文件来生成 `.d` 文件，所以在需要这里包含它：
```makefile
-include $(OBJ:.o=.d) # The dash is used to silence errors if the files don't exist yet
```

# References
https://stackoverflow.com/questions/30573481/how-to-write-a-makefile-with-separate-source-and-header-directories