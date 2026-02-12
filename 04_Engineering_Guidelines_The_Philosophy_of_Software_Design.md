# Engineering Guidelines: The Philosophy of Software Design by John Ousterhout

> **Core Philosophy:** The most fundamental problem in computer science is managing complexity. Our primary goal as engineers is to reduce the complexity of the systems we build.

---

## 📚 Table of Contents
1. [The Nature of Complexity](#1-the-nature-of-complexity)
2. [Strategic vs. Tactical Programming](#2-strategic-vs-tactical-programming)
3. [Modular Design Principles](#3-modular-design-principles)
4. [Information Hiding & Leakage](#4-information-hiding--leakage)
5. [Error Handling Philosophy](#5-error-handling-philosophy)
6. [Design Process](#6-design-process)
7. [Documentation & Comments](#7-documentation--comments)
8. [Naming Standards](#8-naming-standards)
9. [Modifying Existing Code](#9-modifying-existing-code)
10. [Performance & Optimization](#10-performance--optimization)
11. [Red Flag Checklist](#11-red-flag-checklist)

---

## 1. The Nature of Complexity

**Definition:** Complexity is anything related to the structure of a software system that makes it hard to understand and modify.

### Symptoms of Complexity
1.  **Change Amplification:** A simple change requires code modifications in many different places.
2.  **Cognitive Load:** The amount of information a developer must hold in their mind to complete a task. (More lines of code is not always more complex; sometimes more code reduces cognitive load by making logic explicit).
3.  **Unknown Unknowns:** The worst symptom. It is not obvious which pieces of code must be modified to complete a task, or what information is needed to do so safely.

### Causes of Complexity
*   **Dependencies:** When code cannot be understood or modified in isolation.
*   **Obscurity:** When important information is not obvious.

**Rule:** Complexity is incremental. It does not happen in one big event; it accumulates in thousands of small choices. We must adopt a **Zero Tolerance** policy for complexity.

---

## 2. Strategic vs. Tactical Programming

### 🚫 Tactical Programming (Avoid)
*   **Mindset:** "Get the feature working as quickly as possible."
*   **Result:** Short-term speed, long-term accumulation of "technical debt," eventual paralysis ("Tactical Tornado").
*   **Symptoms:** Patching bugs without fixing the root design, introducing special cases to save time.

### ✅ Strategic Programming (Adopt)
*   **Mindset:** "Working code isn't enough." The goal is a great design that *also* works.
*   **The Investment:** Spend 10–20% of total development time on design and cleanup.
*   **Payoff:** The crossover point where strategic programming becomes faster than tactical programming is typically 6–18 months.
*   **Startup Reality:** Even startups must be strategic. "Move fast and break things" leads to unmaintainable codebases that slow down hiring and product iteration.

---

## 3. Modular Design Principles

### Deep Modules vs. Shallow Modules
We strive for **Deep Modules**.
*   **Concept:** Visualize a module as a rectangle. The top edge is the interface; the area is the functionality.
*   **Deep Module:** Small top edge (simple interface), large area (powerful functionality).
    *   *Example:* Unix I/O (`open`, `read`, `write`, `close`).
    *   *Example:* Garbage Collectors (No interface, massive functionality).
*   **Shallow Module:** Large top edge (complex interface), small area (little functionality).
    *   *Example:* Linked List classes that barely hide the pointer logic.
    *   *Example:* A method `addNullValueForAttribute(String attr)` that just calls `map.put(attr, null)`.

### Classitis
Avoid breaking code into tiny classes just for the sake of "small classes." If a class doesn't hide significant complexity, it creates more interface noise than value.

### General-Purpose vs. Special-Purpose
*   **Guideline:** Make modules "somewhat general-purpose."
*   **Why:** General-purpose interfaces are deeper. They force you to define a clean abstraction that isn't tied to the dirty details of a specific UI or feature.
*   **Separation:** Separate general-purpose code from special-purpose code.
    *   *Example:* A text editor class should handle text mechanics (insert/delete range). The UI class should handle the Backspace key logic.

### Different Layer, Different Abstraction
*   **Pass-Through Methods:** If a method does nothing but call another method with the same signature, it adds no value. This indicates class responsibilities are confused.
*   **Decorators:** Avoid decorators unless they add significant functionality. They often result in shallow pass-through layers.
*   **Pass-Through Variables:** Avoid passing variables down through 4 layers of methods just to reach the bottom. Use a **Context Object** to hold global state/configuration.

---

## 4. Information Hiding & Leakage

**Information Hiding:** Each module should encapsulate a design decision (e.g., a file format, a network protocol, a data structure). This knowledge should be invisible to the interface.

**Information Leakage:** When a design decision is reflected in multiple modules.
*   *Red Flag:* If you change a file format in Class A, and Class B breaks, information has leaked.

### Temporal Decomposition (The Enemy)
Do not break up modules based on the *order* in which things happen (e.g., `FileReader`, `FileParser`, `FileWriter`). This forces the caller to know the order of operations and usually leaks file format knowledge across all three classes.
*   **Solution:** Group by knowledge (e.g., `FileHandler` handles reading, parsing, and writing).

---

## 5. Error Handling Philosophy

**Goal: Define Errors Out of Existence.**

Exceptions increase complexity by creating invisible exit points and forcing the caller to handle awkward states.

### Techniques
1.  **Redefine Semantics:** Change the method so it always succeeds.
    *   *Example:* `unset(variable)` should simply ensure the variable is gone. If it didn't exist, that's a success, not an exception.
    *   *Example:* `substring(index)` where index is out of bounds should return empty, or clamp to valid bounds, rather than throwing.
2.  **Masking:** Handle the error at the lower level so the upper level doesn't know.
    *   *Example:* TCP retransmits lost packets automatically. The application doesn't need to know about packet loss.
    *   *Example:* NFS hangs on server crash rather than aborting, allowing applications to resume seamlessly when the server returns.
3.  **Exception Aggregation:** Handle many different exceptions in a single top-level handler rather than distinct `try/catch` blocks for every specific error.
4.  **Just Crash:** For errors that are impossible to handle (e.g., Out of Memory, corrupted internal state), aborting the program is often cleaner than attempting complex, untestable recovery.

---

## 6. Design Process

### Design It Twice
Never go with your first design idea. It is usually based on the current context (Tactical).
1.  Sketch at least two radically different approaches (e.g., "What if this was a line-oriented API?" vs "What if this was character-oriented?").
2.  List pros and cons.
3.  Pick the best, or combine them.
*   *Cost:* Small (hours). *Benefit:* Massive (prevents months of bad code).

### Pull Complexity Downwards
It is better for the **module developer** to suffer complexity than the **module user**.
*   If a situation is complex, handle it internally. Do not use configuration parameters to push the decision to the user unless absolutely necessary.

---

## 7. Documentation & Comments

**Core Rule:** Comments should describe things that aren't obvious from the code.

### The Excuse "Code is Self-Documenting" is False
Code can show *what* is happening, but it rarely explains *why*, or the *abstraction* (the simplified view).

### Write Comments FIRST
Write the interface comments (header files / Javadoc) **before** writing the code.
*   This serves as a design tool. If the comment is hard to write, the design is likely wrong (shallow or complex).

### Types of Comments
1.  **Interface Comments:** Describe the **abstraction**.
    *   *Do:* Describe what the method does, inputs, outputs, side effects, and constraints.
    *   *Don't:* Describe implementation details ("Loop through array...").
2.  **Implementation Comments:** Inside the method.
    *   *Do:* Explain *why* tricky code exists. Explain the high-level steps ("Phase 1: Scan for candidates").
    *   *Don't:* Repeat the code (`i++ // increment i`).
3.  **Cross-Module Comments:** Use a central `designNotes` file for concepts that span multiple files, and link to it.

---

## 8. Naming Standards

**Goal:** Create an image in the reader's mind.

1.  **Precision:** Names must be precise.
    *   *Bad:* `count`, `result`, `temp`.
    *   *Good:* `numActiveUsers`, `mergedLine`, `swapTemp`.
2.  **Consistency:** Use the same word for the same concept everywhere.
    *   Don't mix `fileBlock` and `diskBlock` if they refer to the same index.
3.  **Avoid Extra Words:** `fileObject` is redundant. `file` is sufficient.
4.  **Loop Variables:** `i`, `j` are fine for short loops. For long loops or nested loops, use `lineIndex`, `charIndex`.

---

## 9. Modifying Existing Code

*   **Stay Strategic:** When fixing a bug, don't just patch it. Ask: "If I designed this from scratch today, how would it look?" Refactor towards that.
*   **Maintain Comments:** Comments must be updated when code changes. Stale comments are worse than no comments.
*   **Commit Logs:** Do not put design documentation in commit logs. Put it in the code.

---

## 10. Performance & Optimization

*   **Simple is Fast:** Clean, simple code is usually efficient.
*   **Measure First:** Intuition about performance is usually wrong. Measure before optimizing.
*   **Critical Path:** Identify the code that runs in the most common case (the critical path). Design this path to be minimal (few checks, few method calls). Handle special cases off the critical path.

---

## 11. Red Flag Checklist

Use this checklist during code reviews. If you see these, the design needs work.

1.  **🚩 Shallow Module:** The interface is complicated compared to the functionality it provides.
2.  **🚩 Information Leakage:** A design decision (e.g., file format) is hardcoded in multiple classes.
3.  **🚩 Temporal Decomposition:** Classes are named after stages of processing (`Parser`, `Reader`) rather than objects.
4.  **🚩 Overexposure:** The API forces users to learn rarely used features to do simple things.
5.  **🚩 Pass-Through Method:** Methods that just call other methods with the same signature.
6.  **🚩 Repetition:** The same code pattern appears over and over.
7.  **🚩 Special-General Mixture:** General-purpose code is polluted with special-case logic for a specific user.
8.  **🚩 Conjoined Methods:** You can't understand method A without reading method B.
9.  **🚩 Comment Repeats Code:** `x = x + 1 // Add 1 to x`.
10. **🚩 Implementation Contaminates Interface:** The Interface comment explains implementation details (e.g., "Uses a hash map to...").
11. **🚩 Vague Name:** `data`, `info`, `manager`, `common`.
12. **🚩 Hard to Describe:** If a class/method is hard to document, the design is flawed.
