### 1. Listing All Functions in a C File

Yes, there are several excellent ways to do this in VS Code, ranging from built-in features to powerful extensions.

#### **Method A: The Built-in "Outline" View (Easiest)**

This is the simplest and is already part of VS Code. You don't need any extensions.

1.  Open the C file you want to inspect.
2.  Look at the **Explorer** panel on the left side of your screen. At the very bottom of that panel, there is a collapsible section called **OUTLINE**.
3.  Expand it.

The Outline view will show you a complete, navigable tree of all the symbols in the current file, including macros, global variables, structs, and, most importantly, **all the functions**. Clicking on any function name in the outline will immediately jump your cursor to its definition in the code.

You can also access this with the command palette (`Ctrl+Shift+P`) and typing `Go to Symbol in Editor...` (shortcut is `Ctrl+Shift+O`). This opens a searchable dropdown list of all symbols.

#### **Method B: Using a Dedicated Extension for Advanced Features**

If you want more power, like a dedicated side-panel view of your code's structure across the entire project, then an extension is the way to go.

*   **Recommended Extension:** **C/C++ TestMate** or **C++ TestMate Adapter**. While their primary purpose is for running tests, they come with a fantastic "Test Explorer" side-panel. This panel provides a hierarchical view of your entire codebase, organized by files, namespaces, and classes, showing all the functions within them. It's like the Outline view on steroids.
*   **Another popular one is CodeMap**, which creates a visual "minimap" of your code's structure, including functions.

**Recommendation:** Start with the built-in **Outline view**. It does 95% of what most people need and requires zero setup.
