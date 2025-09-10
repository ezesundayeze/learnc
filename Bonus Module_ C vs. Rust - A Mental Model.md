# Bonus Module: C vs. Rust - A Mental Model

## Philosophy Focus: Knowledge Graph

You already know Rust. This isn\'t a new topic, but rather a way to
connect your existing knowledge graph to the new one you\'re building
for C. Understanding these comparisons will make C\'s design choices
\"click\" and give you a deeper appreciation for both languages.

### **1. Memory Management: The Core Difference**

This is the single biggest distinction and influences everything else.

**Rust: The Guardian**

-   **Core Idea:** The compiler enforces memory safety at compile time.

-   **Mechanism:** Ownership, Borrowing, and Lifetimes. The borrow
    > checker is a static analysis tool that proves your memory access
    > is safe *before* the program runs.

-   **You Write:**\
    > let mut s = String::from(\"hello\");\
    > let r1 = &s;\
    > let r2 = &s; // OK: Multiple immutable borrows\
    > println!(\"{} and {}\", r1, r2);\
    > let r3 = &mut s; // ERROR: Cannot borrow as mutable because it\'s
    > already borrowed as immutable

-   **Result:** No data races, no use-after-free bugs, no null pointer
    > dereferences (thanks to Option\<T\>). Memory is freed
    > automatically when the owner goes out of scope (drop is called).
    > It feels restrictive at first, but provides immense safety.

**C: The Trusting Anarchist**

-   **Core Idea:** The programmer is in complete control and is trusted
    > to do the right thing.

-   **Mechanism:** Manual memory management via malloc (allocate),
    > realloc (resize), and free (deallocate).

-   **You Write:**\
    > char \*s = malloc(6); // Allocate 6 bytes for \"hello\\0\"\
    > strcpy(s, \"hello\");\
    > \
    > char \*r1 = s; // Just a copy of the pointer\
    > char \*r2 = s; // Another copy of the pointer\
    > \
    > // You can read from r1 and r2\...\
    > \
    > free(s); // Deallocate the memory s points to\
    > \
    > // DANGER ZONE!\
    > printf(\"%c\\n\", r1\[0\]); // UNDEFINED BEHAVIOR:
    > Use-after-free.\
    > // r1 still holds the address, but the memory is no longer valid.\
    > // This might crash, or it might print garbage.

-   **Result:** Immense power and flexibility. You can implement complex
    > data structures exactly as you see fit. However, this power comes
    > with great responsibility. Forgetting to free, using a pointer
    > after it\'s been freed (use-after-free), or writing past the end
    > of allocated memory (buffer overflow) are common, serious bugs
    > that C allows.

### **2. Strings: A Perfect Example of the Core Difference**

**Rust:**

-   String: A growable, heap-allocated, UTF-8 encoded string. It owns
    > its data. It\'s a \"fat pointer\" containing a pointer to the
    > data, a capacity, and a length. It\'s easy and safe to use.

-   &str: An immutable \"string slice\" or \"view\" into some string
    > data. It\'s just a pointer and a length.

**C:**

-   **No string type!** A \"string\" in C is a **convention**, not a
    > type.

-   The convention is: **A sequence of characters in memory terminated
    > by a special null character (\\0)**.

-   A char \* variable is used to point to the beginning of this
    > character sequence.

-   All the \"string\" functions in C (strcpy, strlen, strcmp, etc.)
    > work by iterating over memory from the starting address until they
    > find a \\0. This is why buffer overflows are so common: if you
    > strcpy a long string into a short buffer, strcpy doesn\'t know the
    > buffer\'s size and will just keep writing past the end, corrupting
    > other data.

### **3. Error Handling**

**Rust:**

-   **Result\<T, E\> and Option\<T\>:** Errors and optionality are
    > encoded in the type system. The compiler *forces* you to handle
    > the Err or None case, typically with match or unwrap. This makes
    > it impossible to forget to handle a potential failure.

**C:**

-   **Return Codes and errno:** Functions typically signal errors by
    > returning a special value (like -1 or a NULL pointer). The
    > specific error code is often stored in a global variable called
    > errno.

-   **It is the programmer\'s responsibility to check the return value
    > after every single function call that could fail.** Forgetting to
    > do this is a common source of bugs.\
    > FILE \*f = fopen(\"non_existent_file.txt\", \"r\");\
    > if (f == NULL) {\
    > // MUST check for the error case!\
    > perror(\"Failed to open file\");\
    > // Handle error\...\
    > }

### **Summary: A Mental Map**

  -----------------------------------------------------------------------
  **Feature**             **C (Manual &           **Rust (Safe &
                          Explicit)**             Abstracted)**
  ----------------------- ----------------------- -----------------------
  **Memory**              You manage it           The compiler manages it
                          (malloc/free). Unsafe   (Ownership). Safe by
                          by default.             default.

  **Strings**             Convention (char\* +    A built-in, safe String
                          \\0). Prone to          type.
                          overflows.

  **Errors**              Return codes (-1/NULL)  Result\<T, E\> that the
                          that you **must**       compiler **forces** you
                          check.                  to handle.

  **Build System**        Simple compiler (gcc).  Integrated package
                          Build systems like Make manager and build
                          are a separate tool.    system (cargo).

  **Philosophy**          \"Trust the             \"Trust the compiler.\"
                          programmer.\" Provides  Provides safe
                          direct, low-level       abstractions.
                          access.
  -----------------------------------------------------------------------

**Why Learn C if you know Rust?**

1.  **Understand the \"Why\":** Learning C shows you all the problems
    > that Rust was designed to solve. It gives you a profound
    > appreciation for the borrow checker.

2.  **Understand the Foundation:** The world runs on C. The Linux
    > kernel, Git, Python interpreters, and countless other foundational
    > tools are written in C. Understanding it means you understand how
    > your computer *really* works.

3.  **Interoperability (FFI):** To have Rust call a C library (or
    > vice-versa), you need to understand C\'s types and memory model to
    > create a safe boundary.
