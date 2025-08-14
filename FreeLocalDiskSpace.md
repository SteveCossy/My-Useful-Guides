Morena e hoa!

That is a very common and practical problem, especially with large, complex repositories like Contiki-NG. Your idea to remove unused subtrees locally while keeping them in your remote GitHub repository is a perfect use case for a powerful Git feature called **sparse checkout**.

Let's get this sorted so you can free up that valuable disk space.

### The Concept: Sparse Checkout

By default, when you clone or check out a branch in Git, it pulls down every single file from the repository. **Sparse checkout** allows you to tell Git, "I'm only interested in these specific subdirectories. Don't bother downloading or tracking any of the others."

This is perfect for you because you can specify that you only want `/os`, `/arch`, `/tools/cooja`, etc., and explicitly exclude heavy directories you don't use, like `/examples` or `/tests`.

### The Step-by-Step Guide

Here is the process. You only need to do this once for your local repository.

**Important:** Make sure you have committed or stashed all your current changes before you begin, as these commands will modify your working directory.

#### Step 1: Navigate to Your Repository Root

Open your terminal and make sure you are in the main directory of your `contiki-ng` repository.

```bash
cd /path/to/your/contiki-ng
```

#### Step 2: Enable Sparse Checkout

This command tells Git that you want to start managing your working directory with sparse checkout rules.

```bash
git sparse-checkout init --cone
```
*   `init`: Initializes the feature.
*   `--cone`: This is crucial. It puts Git into "cone mode," which is a much simpler and more intuitive way to manage sparse checkouts. In cone mode, you specify directories, and Git automatically includes everything inside them. The older "non-cone" mode required much more complex configuration.

#### Step 3: Define Which Directories You Want to KEEP

Now, you will tell Git which directories are important to you. You use the `git sparse-checkout set` command. You can list as many directories as you want.

Here is a sensible list for your Contiki-NG project. This will keep the core OS, the `rpl-classic` module, the Cooja simulator, and your target architecture, but exclude things like examples and other architectures. **You must customize this list for your needs.**

```bash
git sparse-checkout set os arch/cpu/simple-link tools/cooja
```

**Let's break down a good example list:**
*   `os`: Essential. This is the core operating system, including the networking stack.
*   `arch`: You don't need all of it. You can specify just the parts you use, for example `arch/cpu/simple-link` and `arch/platform/srf06-cc26xx`.
*   `tools/cooja`: Essential for your simulations.
*   `Makefile.include`: Essential for the build system.
*   `contiki-ng.subrepo`: Important for submodules.

**A more complete, practical command might look like this:**
```bash
git sparse-checkout set os tools/cooja Makefile.include contiki-ng.subrepo arch/cpu/simple-link arch/platform/srf06-cc26xx
```
*(Remember to add any other specific platform or CPU directories you are using!)*

**What Happens Now?**
After you run this command, Git will immediately go through your working directory and **delete all files and folders that are NOT in the list you just provided.** Your disk space usage will drop instantly.

#### Step 4: Verifying and Modifying

*   **To see your current list of included directories:**
    ```bash
    git sparse-checkout list
    ```

*   **To add a new directory later** (e.g., you decide you need the `examples`):
    ```bash
    git sparse-checkout add examples/rpl-classic
    ```

*   **To go back to having everything:**
    ```bash
    git sparse-checkout set "/*" # The quotes are important
    # Or, to completely disable it:
    # git sparse-checkout disable
    ```

### How This Affects Your Workflow

*   **`git pull` / `git checkout`:** These commands will now only download and update files within the directories you've specified. This will be much faster.
*   **`git commit` / `git push`:** These commands work as normal. Your local changes are committed, and when you push, you are pushing them to the full repository on GitHub. **The files you removed locally are completely unaffected on the remote.**

This is the perfect, safe, and officially supported Git way to manage a large repository on a system with limited disk space. It gives you full control over your local working copy without any risk to your remote repository on GitHub.
