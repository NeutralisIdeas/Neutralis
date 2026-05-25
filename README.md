\## Neutralis v2.0.3 — "Iron Guard" Release



Welcome to the \*\*Neutralis v2.0.3\*\* architecture update. This release shifts the paradigm of our development environment by converting the parser and compiler into an uncompromising static filter. By moving 90% of our code validation and linting logic from the runtime execution phase to the compilation phase, we eliminate expensive diagnostic checks where they hurt the most—preserving precious CPU cycles and maintaining peak framerates within the `neu3d` engine runtime.



If a script compiles successfully in Neutralis v2.0.3, it is guaranteed to be structurally sound and architecturally efficient by definition.



\---



\### Key Architectural Enhancements



\#### 1. Robust Parser Recovery (Panic Mode Synchronization)

Previously, the parser suffered from an "error blinding" effect: encountering an initial syntax anomaly caused it to lose logical alignment, generating a cascade of false-positive syntax errors throughout the rest of the file.

\* \*\*Implemented `synchronize()` Mechanics:\*\* The parser now handles unexpected tokens gracefully. Upon detecting a syntax error, it enters a structured Panic Mode, discarding malformed tokens until it reaches a safe logical boundary (`if`, `while`, `for`, `end`, `eofl`).

\* \*\*Impact:\*\* The compiler now generates a complete, clean, and accurate multi-error diagnostic report in a single compilation pass, allowing developers to debug batches of errors at once.



\#### 2. Context-Aware Diagnostics (RAII Context Stack)

To support a growing ISA of over 50+ core language statements without nesting overhead or unmaintainable string code, we have centralized our error tracking using an RAII-based `ContextGuard` mechanism.

\* Every compilation error is now paired with an explicit hierarchical block traceback.

\* \*\*Example Diagnostic Output:\*\* `\[Syntax Error] Line 10: Expected statement in block in context: while loop -> block -> if statement -> block -> for loop -> block, found 'eofl'`.



\#### 3. Compile-time Performance Guard (Static Linter)

In alignment with our core philosophy of minimizing hidden allocations in hot execution loops, the compiler now strictly bans sub-optimal memory design patterns before bytecode generation:

\* \*\*Loop Concatenation Ban:\*\* Attempting to use the `+` operator on string types inside a `while` or `for` loop body triggers an immediate compilation abort (`Compiler Error: Inefficient string concatenation (+) inside a loop`). Developers are strictly enforced to use high-performance pre-allocated `buffer()` structures alongside `buf\_write()`.

\* \*\*Enforced Hidden Class / Shape Optimization:\*\* Utilizing runtime bracket property lookup (`obj\["prop"]`) where a static dot-access is viable now triggers a strict `Compile Warning`. This steers developers toward explicit dot-access (`obj.prop`), ensuring our VM's Inline Cache functions at speeds comparable to native C++ structures.



\#### 4. Lexical Integrity \& Strict EOF Semantics

\* Implemented compile-time mutability checks on globally interned string constants. Attempts to mutate static strings via subscript indices (`msg\[0.0] = "H"`) are caught and blocked during parsing.

\* Reworked the end-of-file validation pipeline. The `eofl` keyword is now strictly enforced as the terminal instruction of a script. Any meaningful token parsed past `eofl` triggers a structural error, and an omitted `eofl` correctly flags unclosed block levels.



\#### 5. Deterministic Debug Line Mapping

\* Integrated an internal debug line map table (`IP -> line\_number`) directly into the `Chunk` structure. When unexpected runtime errors occur (such as calling an unallocated symbol), the register-based VM safely unpacks active `CallFrame` bounds to output a precise, human-readable Stack Trace mapping directly to the source script line.



\---



\### Diagnostic Evaluation Logs



\#### Syntax Recovery \& Context Traceback Test:

\# Test Script

for i <<= 10.0 do

&#x20;   print(i)

end



if x == 5.0 do

&#x20;   y = + 

end

eofl



\[Neutralis Error] Parser encountered 3 error(s):

\[Syntax Error] Line 2: Unexpected token in expression in context: for loop -> expression -> comparison, found '<='

\[Syntax Error] Line 4: Unrecognized statement, found 'end'

\[Syntax Error] Line 8: Unexpected token in expression in context: if statement -> block -> identifier statement -> expression -> comparison, found '+'



Loop Performance Guard Test:

data = ""

for i = 1.0 <= 100.0 do

&#x20;   data = data + "a" 

end





v2.0.3 Compiler Output:

\[Neutralis Error] Compiler Error at line 3: Inefficient string concatenation (+) inside a loop. Use buffer() and buf\_write() instead.







Relaxed parameter list boundaries in function definitions: the parser now gracefully compiles trailing/hanging commas in argument definitions (def process(hp, mana, ) do ... end).



Hardened block termination checking: the compiler will explicitly point out which nested statement scope (while, if, or for) was left unclosed if the end of the file or an early eofl token is unexpectedly reached.





The v2.0.3 core stabilization locks down a rigid, reliable ISA. This release prepares the Neutralis environment for seamless integration as the primary script driver for the transition of our engine layer from neu2d into a fully-featured neu3d rendering system. External modding/plugin architectures can safely begin deploying production scripts using the call() environment via the C:\\NeuLibs\\ library channel.

