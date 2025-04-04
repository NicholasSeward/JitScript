# JitScript

JitScript is a unique 2D programming language developed in-house at **Jitter**. Crafted by **Hector Sere, Employee #2** (the go-to fixer who knows the Jitter codebase better than his own children, albeit a tad burnt out), this language is designed so that _new interns_ can maintain and extend the system alongside Hector.

The language files use the save icon for the extension `program.💾`.

---

## Overview

JitScript is built around a dynamic 2D grid where a movable cursor executes code spread across rectangular blocks. Key features include:

- **Multiple Blocks:** A program consists of one or more blocks (rectangular grids of characters). Blocks are separated by blank lines and padded to form perfect rectangles.
  - The **first block** is the main block, executed at startup.
  - **Subsequent blocks** are addressable by index (e.g., using `1` to call the second block, `2` for the third, etc.). When returning from a subblock, the cursor resumes from the call point in the parent block. (The direction may have changed.)
- **Cursor Movement:** The cursor moves around within a block. Its initial position in the main block is specified below in **Cursor Initialization**.
- **Deque & Registers:** There is an unbounded deque (supporting left/right operations), three registers (`A`, `B`, and `C`), and a 2 slot shift register for temporary storage.
- **8-bit Arithmetic:** Every stored value is between `0x00` and `0xFF` (with wrap-around on overflow/underflow).
- **Printing:** The print command outputs only characters with ASCII codes from `0x20` (space) to `0x7E` (`~`).

---

## Block Structure and Cursor Placement

- **Block Layout:**  
  After stripping comments and trimming each line, every block must be a rectangle with a blank line both above and below. Lines are right-padded as necessary.
  
- **Cursor Initialization:**  
  When entering a block, the cursor is positioned based on the direction from which it is entering (It is roughly the middle of the side you need to enter from given your direction.):
  
  - **Right (`>`):** `[0, len(rows)//2]`
  - **Down (`v`):** `[len(rows[0])//2, 0]`
  - **Left (`<`):** `[len(rows[0])-1, len(rows)//2]`
  - **Up (`^`):** `[len(rows[0])//2, len(rows)-1]`
```
# The entrances are labeled below.  In practice, no need for an movement charcter at the entrance.
+----v----+
|         |
>         <
|         |
+----^----+
```
```
# Example with even dimensions.
+----v---+
|        |
>        <
+----^---+
```
  When leaving a block, the cursor moves away from the call character in the direction it exited. The interpreter maintains a stack of subblocks, positions, and directions to resume execution correctly.

- **Program Termination:**  
  Exiting the main block ends the program.

---

## Commands Reference

| Command | Description |
| ------- | ----------- |
| `#`     | Comment (rest of line ignored by the interpreter) |
| `?`     | Conditional jump: compares the second-to-last value (x) with the last value (y) from the 2-element shift register and jumps: 0 if `x < y`, 1 if `x == y`, 2 if `x > y` |
| `<`     | Set direction to left |
| `>`     | Set direction to right |
| `^`     | Set direction to up |
| `v`     | Set direction to down |
| `"`     | Literal toggle: store the next character literally |
| `x`     | Hex Literal toggle: store the next hex digit |
| `.`     | Print the previous value (ignores unprintable characters) |
| `X`     | Print the hexidecimal representation of the previous value |
| `,`     | Print the decimal representation of the previous value |
| `a`     | Store last value into register A |
| `b`     | Store last value into register B |
| `c`     | Store last value into register C |
| `*`     | Multiple the last value by 16 (or bitshift left 4) |
| `/`     | Divide the last value by 16 (or bitshift right 4) |
| `+`     | Increment the last value |
| `-`     | Decrement the last value |
| `$`     | Add the second-to-last value to the last value |
| `_`     | Subtract the second-to-last value from the last value |
| `[`     | Push the last value on the left side of the deque |
| `]`     | Push the last value on the right side of the deque |
| `(`     | Peek (read without removing) from the left side of the deque |
| `)`     | Peek (read without removing) from the right side of the deque |
| `{`     | Pop (remove and read) the left side of the deque |
| `}`     | Pop (remove and read) the right side of the deque |
| `E`     | isEmpty: Store 0xFF if the deque is empty else store 0x00 in the last value |
| `A`, `B`, `C` | Read a new value from the respective register (pushing the current last value to the shift register) |
| `1`–`9` | Enter the corresponding subblock |

**Additional Notes:**
- Commands `"(){}ABC` read a new value and push it to the shift register.
- Commands `*/+-$_` modify the last value.
- Commands `?.X,[]` access the last value(s) without modification.

---

## Arithmetic and Value Handling

- **8-bit Range:** All values are stored as 8-bit numbers (from `0x00` to `0xFF`).  
- **Overflow Behavior:** Incrementing `0xFF` wraps around to `0x00`; decrementing `0x00` wraps around to `0xFF`.
- **Printing Constraints:** Only values between `0x20` (space) and `0x7E` (`~`) are printed.
- **Empty Deque:** Peeking or popping from an empty deque will.
- **Last Value:** Every time a new last value is stored, the previous last value becomes the second-to-last value.

---

## Examples

Below are 10 example tasks demonstrating various operations in JitScript. For each, we describe the task, specify any required deque input, and outline the expected output (either on the console or as a final deque state).

### 1. Simple Output
```jit
"H."e."l."l."o.
```
- **Task:** Prints "Hello".
- **Deque Input:** None.
- **Expected Output:**
  - **Console:** `Hello`
  - **Deque:** Unchanged.
- **Description:** Use the literal toggle (`"`) to store a character and then the print command (`.`) to output it.

### 2. Deque Manipulation
```jit
"A["B["C[}.}.}.
```
- **Task:** Use the deque to reverse a string.
- **Deque Input:** `[]` empty
- **Expected Output:**
  - **Console:** `CBA`
  - **Deque:** `[]` empty.
- **Description:** Use `[` and `]` to push values on the left and right, and `{` and `}` to pop them off.

### 3. Cursor Movement in a Block
- **Task:** Navigate the cursor across a block.
- **Deque Input:** None.
- **Expected Output:**
  - **Console:** No output (unless a print command is executed during movement).
  - **Deque:** Unchanged.
- **Description:** Demonstrate cursor movement using `<`, `>`, `^`, and `v` commands within the block.

### 4. Arithmetic Operations
- **Task:** Increment and decrement a value, demonstrating wrap-around behavior.
- **Deque Input:** Begin with a value (e.g., starting at `0xFF`).
- **Expected Output:**
  - **Console:** (Optional) Printed results after arithmetic.
  - **Deque:** Final value reflecting correct wrap-around (e.g., decrementing `0x00` yields `0xFF`).
- **Description:** Use `+` and `-` to modify values, noting the 8-bit overflow/underflow behavior.

### 5. Register Storage and Retrieval
- **Task:** Store a value into register A and retrieve it.
- **Deque Input:** A value to be stored.
- **Expected Output:**
  - **Console:** The value retrieved from register A (when printed).
  - **Deque:** The shift register updates accordingly.
- **Description:** Use `a` to store a value in register A and `A` to retrieve and push it onto the shift register.

### 6. Looping with Conditional Jump
- **Task:** Implement a loop using the conditional jump (`?`) command.
- **Deque Input:** A sequence of numbers for comparison.
- **Expected Output:**
  - **Console:** Iterative output demonstrating the loop’s progress.
  - **Deque:** Updated state depending on operations performed inside the loop.
- **Description:** Compare two values with `?` to conditionally jump between sections of code, effectively creating a loop.

### 7. Subblock Execution
- **Task:** Enter a subblock to perform a distinct operation, then return to the main block.
- **Deque Input:** Depends on subblock logic.
- **Expected Output:**
  - **Console:** Combined output from both the subblock and main block.
  - **Deque:** Intermediate states reflecting subblock operations.
- **Description:** Use a digit command (e.g., `1`) to call a subblock; observe how execution resumes in the calling block upon subblock exit.

### 8. Literal Toggle Usage
- **Task:** Use the literal toggle to store a character that might otherwise be interpreted as a command.
- **Deque Input:** None.
- **Expected Output:**
  - **Console:** The literal character is printed.
  - **Deque:** Remains unchanged.
- **Description:** Prefix a command character with `"` to ensure it is treated as data rather than an instruction.

### 9. Overflow Behavior
- **Task:** Demonstrate arithmetic underflow by decrementing a value of `0x00`.
- **Deque Input:** Start with `0x00`.
- **Expected Output:**
  - **Console:** If printed, the result should show `0xFF`.
  - **Deque:** The value becomes `0xFF` due to underflow.
- **Description:** Use the `-` command on `0x00` to illustrate the wrap-around behavior.

### 10. Combined Operation Example
- **Task:** Create a mini-program that combines deque operations, cursor movement, arithmetic, and register usage to generate a sequence.
- **Deque Input:** Initialize with specific values as required by the algorithm.
- **Expected Output:**
  - **Console:** A sequence of characters or numbers output by the program.
  - **Deque:** The final state reflects all intermediate manipulations.
- **Description:** Integrate multiple commands (e.g., pushing/popping on the deque, performing arithmetic, storing/retrieving registers, and calling subblocks) to demonstrate a full cycle of operations.

---

## JITR (JitScript Interactive Transversal Runtime)

Within the `JITR` folder, you'll find a single-file web sandbox designed to let you experiment with JitScript in your browser. This interactive tool allows you to:
- Write and edit JitScript code.
- Visualize the cursor's path through the 2D block grid.
- Monitor the state of the deque and registers in real time.

---

## Getting Started

1. **Clone the Repository:**
   ```bash
   git clone https://github.com/NicholasSeward/JitScript.git
   cd JitScript```
2. **Run JITR:** Open the file in the JITR folder using your preferred web browser to launch the interactive sandbox.
3. **Contribute:** This project is maintained collaboratively by Hector Sere and our team of enthusiastic interns. Bug reports, feature requests, and contributions are welcome—please adhere to our contribution guidelines.
