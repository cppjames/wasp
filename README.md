# WASP
WASP (**W**asp **AS**sembly **P**reprocessor) makes x86-64 assembly language easier and safer through a set of preprocessing directives.

Features of WASP include:
- Code generation
- Aliases
- Powerful macro system
- Predefined macros
- Function calls with parameter syntax
- Modular macro sets

# Usage
WASP enables the developer to reduce complexity by naming constants, registers, instructions, fragments of code, and so on.

Preprocessing directives start with the `@` character.

## Macros
There are two types of macros: single-line and multiline.
```asm
; General syntax for single-line macro
@def name value
```

After the above line, every occurrence of `name` will be expanded to `value`.
```asm
@def num_lines 32
@def add_length add rax, [line_lengths + rcx]

mov rcx, num_lines
length_loop:
    add_length
    loop length_loop
```
After preprocessing, the above snippet will look like this:
```asm
mov rcx, 32
length_loop:
    add rax, [line_lengths + rcx]
    loop length_loop
```

Un-defining a macro is done using the `@undef` command:
```asm
@undef name
; From this point on, name will no longer be expanded.
```
**Note:** Macro names respect the rules of ANSI C identifiers: they must only contain letters a-z, A-Z, digits 0-9 and underscores. They cannot start with a digit.

**Note:** Macros can be defined more than once. The last definition before the occurrence will be picked. `@undef`-ing a macro will cause it not to be expanded until it is re-defined, regardless of how many definitions it had.

### Macro parameters
Parameters can be passed to macros:
```asm
@def name(param1, param2, ...) value
```

Just like in C, parameter names will be expanded inside `value`. The last `...` parameter is optional, and can be accessed using the `@args` directive.
```asm
@def num_lines 32
@def line_len(index) [line_lengths + index]

mov rcx, num_lines
length_loop:
    add rax, line_len(rcx)
    loop length_loop
```

The above will be expanded to:
```asm
mov rcx, 32
length_loop:
    add rax, [line_lengths + rcx]
    loop length_loop
```

### Multiline macros

WASP supports macros that span multiple lines:
```asm
@def name
|   instr1
|   instr2
|   instr3
```

Every line contained by the macro must begin with a `|` character. Multiline macros can have parameters, as well.
```asm
; x86 already has the xchg instruction.
; This example is only for demonstration purposes.

@def swap(x, y)
|   xor x, y
|   xor y, x
|   xor x, y

swap(rax, rbx)
```

This snippet will expand to:
```asm
xor rax, rbx
xor rbx, rax
xor rax, rbx
```

## Extended literals
Character literals can be used by enclosing the character in single quotes (`'`). They will automatically be expanded to their corresponding ASCII code.
```asm
mov rax, 'a'
sub rax, 'A'
; rax now contains 32 ('A' - 'a' = 97 - 65 = 32)
```

## Function call syntax

WASP is capable of passing parameters automatically to System V ABI compliant functions.
```asm
; Print a character N times. The char goes in rdi. N goes in rsi.
print_n_times:
    extern putchar
    mov rcx, rsi
print_loop:
    putchar(rdi)
    loop print_loop
    ret

print_n_times('+', 10)
```

Here is the expanded version:
```asm
print_n_times:
    extern putchar
    mov rcx, rsi
print_loop:
    call putchar
    loop print_loop
    ret

mov rdi, 43
mov rsi, 10
call print_n_times
```
**Note:** Nothing needs to be done before `call putchar`, as the character is already in `rdi`.
