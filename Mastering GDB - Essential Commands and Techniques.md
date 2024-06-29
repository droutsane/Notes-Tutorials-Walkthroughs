### 1. Setting Up GDB

First and foremost thing to do is to set disassembly flavor to intel, because we don't want to deal with at&t syntax which is default in gdb. To set this there are two ways.
1.  after running gdb we can run `"set disassembly-flavor intel"`
2. we can write the above to gdbinit file, gdbinit runs when gdb starts and makes the defined changes. we can do this by 
   `echo "set disassembly-flavor intel" > ~/.gdbinit`
   
### 2.  Running a program and providing arguments

Open a compiled binary named a.out:
`gdb ./a.out`
but this will not start running/executing our program right away.

Use `run` or `r` after running `gdb ./a.out` to starts the programs execution.

just like we send arguments in any binary, we can do it here too.
`gdb ./a.out arg1 arg2.....`
we can also supply the arguments after opening the program in gdb.
`run arg1 arg2 ...`

### 3.  setting breakpoints

we can set breakpoint at main by `break *main` 
we can set breakpoint at any name / address that is in our assembly.
ex: `break my_func`  , `break 0xfffffff`
now if we run the program,  execution will stop right before running the breakpoint. we can verify this is if we look at the $rip, it will be pointing to the address of breakpoint.

we can see all breakpoints that is set by `info break` 
we can delete breakpoint by `del num` . the num here is the number assigned to each breakpoint that we set and we can see the number with `info break` .

also we can set breakpoints using the memory address, but the memory address may change when we rerun the program on same / different system. so we should always do such operations on offsets like `break *my_func + offset_number` , where `my_func` can be any function whose function frame we are setting a breakpoint in.


### 4.  Understanding Registers and Memory Inspection in GDB

#### 4.1  info
we can take look at registers by `info reg` or `info register` , this will show all the registers and its contents.
`info register` shows three columns:
1. register Names
2. Current values stored in corresponding registers in hexadecimal format.
3. values of the register in decimal format

#### 4.2  print
To only see one register (lets say rax ) we can do `print $rax`  and the gdb will make a local variable and assign it the value of $rax and display it . ( we will see how variable works in GDB in later section. )
When you use `print $rax` in GDB, it will display the value stored in the register `rax`.
- If `rax` contains a memory address (a pointer), it will print the value of that address (i.e., the content stored at that memory location).
- If `rax` contains a regular data value (like an integer), it will print that value directly.

GDB treats `$rax` as a register unless you explicitly dereference it. If you want to inspect the memory content at the address stored in `rax`, you would use `print *(long *) $rax`, which explicitly dereferences the address and shows the data at that location.

#### 4.3 inspect
now that we know how to see what is stored in register which can either be a data value or be an address, we would also like to see what those address holds. in those case we can use `inspect $register`, inspect dereferences the address stored in a register or variable and displays the value at that memory location.

#### 4.4 examine (x/)
examine is a command similar to inspect with added functionalities.
 `x/<n><f><u> address` where :
`<n>` : The number of units to display.
`<f>` : The format in which to display the data (e.g., x for hexadecimal, d for decimal, s for string, i for instructions ).
`<u>` : The size of the units to display (b for byte, h for halfword/2 bytes, w for word/4 bytes, g for giant word/8 bytes).
`<address>` : The memory address to examine.

so examine is our go-to command for detailed memory inspection and analysis.

now lets say we have added a break point at main and ran the binary, once it hits the breakpoint, the rip will point to the address of main, we can examine that address by `x/i $rip` , and it will show us the starting of main which in case of non stripped binary will be `endbr64` , and if we do `x/10i $rip` , then we can see 10 lines of assembly from starting of main, but lets say in case we want to see whole main, with our current knowldge we can guess the number of instructions in main if we must see whole main function.

### 5. Disassembling any function

Gdb has another command `disas` (short for `disassemble` command), we can run disas main and it will disassemble the whole main, now since it can disassemble main, similarly we can disassemble any function if we know its name, we can also disassemble a function if we know its starting address.
`disass function_name` / `disass 0xaddress` .

### 6. Understanding Instruction Stepping in GDB: `si`, `ni`, and `finish`

after disassembling we can move to different address relative to rip by either `si (step instruction)` or `ni (next instruction)` command.

In GDB, the commands `si` (step instruction), `ni` (next instruction), and `finish` are crucial for navigating and analyzing your program's execution:
- **`si` vs `ni`:**
    - The main difference between `si` and `ni` is how they handle function calls. If the instruction pointer (`rip`) is pointing to a call to another function and we use `ni`, it will skip over the function call. The `rip` will move to the next instruction in the same function frame.
    - On the other hand, if we use `si` when `rip` is pointing to a function call, `si` will step into the function. The `rip` will move to the beginning of the called function's frame, allowing us to analyze the function's internal execution.
- **Using `finish`:**
    - Once we have stepped into a function using `si`, we can do whatever we want within that function. However, if we want to complete the execution of the called function and return to the calling function, we can use the `finish` command. `finish` will execute the entire current function and break at the instruction immediately after the function call that invoked it.

### 7. Automatically Displaying Instruction with Display

Now since we are mostly insterested in analyzing the code, it would be pretty cool if we can see some instruction automatically once our program stops anywhere (either due to breakpoint or any other commands like si or ni), we can do it with display command.
ex `display /a $rip` will display the address of rip.
we can set multiple display one after another and everytime the program stops all the displays will be printed.

### 8. Using `set` command to Manipulate Variables and Registers in GDB

#### 8.1 Variables in GDB
anything starting with `$` is a variable in GDB.
we can create and set a value in a variable with `set $variable = value`

#### 8.2 Convenience Variable

Convenience variables in GDB are variables that you can create and use to store values while debugging. They are not part of the program's code but are available only within the GDB session. You create and manipulate these using the set command.

#### 8.3 Exploring `set` command

not only we can store values in the variable, but we can directly set/change the value of any standard register by the `set` command.
like `set $rax = 0` and if we do info register, we will see rax set to 0.

we can also do something like this:
- **`set $rsp = 0x1337`:** Directly modifies the stack pointer register.
- **`set *(long *) $rsp = 0x1337`:** Modifies the memory content at the address pointed to by the stack pointer.

We can even move `rip` by setting it to an address. However, this will likely cause a mismatch in the internal state of the program and most probably result in a segmentation fault.

### 9. GDB Scripting.

#### 9.1 Creating and running a GDB script file

we can create a gdb script with `.gdb` extension and write exactly the same command that we write in the terminal. 

we can run this script by gdb - x my_script.gdb ./a.out
or by providing the script as argument to the program.

#### 9.2 Basic Structure of a GDB Script

```gdb
# my_script.gdb

# Set a breakpoint at the start of main
break *main

# Define commands to run when the breakpoint is hit
commands
    # Print the value of the rax register
    p $rax
    # Continue execution
    continue
end

# Run the program with arguments
run arg1 arg2

# Quit GDB
quit
```
- `break *main` sets a breakpoint at the start of the `main` function.
- `commands ... end` specifies a list of commands to execute when the breakpoint is hit.
    - `p $rax` prints the value of the `rax` register.
    - `continue` resumes program execution.
- `run arg1 arg2` starts the program with the specified arguments.
- `quit` exits GDB after the script is executed.
  
#### 9.4 Additional GDB Script Commands

1. Conditional Breakpoints
    `break main if argc > 1`
   
2. Looping in Scripts
   ```
   while $pc != 0
    p $rax
    stepi
	end
   ```

3. Calling Functions

	`call my_function()`


4. Executing Shell Commands

	`shell echo "Hello from GDB script"`

5. Handling Signals

	You can specify how GDB should handle signals during debugging. For example, configuring GDB to ignore SIGINT signals without stopping or printing messages:
	`handle SIGINT pass nostop noprint`
