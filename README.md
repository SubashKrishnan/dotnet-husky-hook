# README

## Table of Contents

1. [How to Configure Husky Pre-Commit](#how-to-configure-husky-pre-commit)
   - [Step 1: Create a Tool Manifest](#step-1-create-a-tool-manifest)
   - [Step 2: Install Husky.NET](#step-2-install-huskynet)
   - [Step 3: Initialize Husky.NET](#step-3-initialize-huskynet)
   - [Step 4: Create and Customize the Pre-Commit Hook](#step-4-create-and-customize-the-pre-commit-hook)
   - [Step 5: Handling Formatting Issues](#step-5-handling-formatting-issues)
   - [Step 6: Customizing Pre-Commit Commands](#step-6-customizing-pre-commit-commands)
2. [How to Run Husky Pre-Commit Hooks](#how-to-run-husky-pre-commit-hooks)
   - [Using Git Commands](#using-git-commands)
   - [Skipping Hooks](#skipping-hooks)

---

### 1. How to Configure Husky Pre-Commit

This section will guide you through setting up Husky pre-commit hooks in your .NET project.

#### Step 1: Create a Tool Manifest

Husky.NET must be installed in the root folder of your solution. First, create a tool manifest file in the root folder by running:

```bash
dotnet new tool-manifest
```

This command creates a `dotnet-tools.json` file under the `.config` folder, which will list the external tools used by dotnet.

The `dotnet-tools.json` file should initially contain:

```json
{
  "version": 1,
  "isRoot": true,
  "tools": {}
}
```

#### Step 2: Install Husky.NET

Next, add Husky as a dotnet tool by running:

```bash
dotnet tool install Husky
```

After running the command, the `dotnet-tools.json` file will include the Husky tool:

```json
{
  "version": 1,
  "isRoot": true,
  "tools": {
    "husky": {
      "version": "0.6.2",
      "commands": ["husky"]
    }
  }
}
```

#### Step 3: Initialize Husky.NET

To add Husky to your existing .NET application, run:

```bash
dotnet husky install
```

After running this command, you should see three folders in your root directory:

- **.git**: Contains the Git repository information.
- **.config**: Contains descriptions of the tools, such as `dotnet-tools.json`.
- **.husky**: Contains the files you will use to define your Git hooks.

#### Step 4: Create and Customize the Pre-Commit Hook

You can add a new hook by running:

```bash
dotnet husky add pre-commit -c "echo 'Hello world!'"
```

This command creates a new file named `pre-commit` (without a file extension) under the `.husky` folder. The default content of this file might look like this:

```bash
#!/bin/sh
. "$(dirname "$0")/_/husky.sh"

echo 'Hello world!'
```

To customize the hook for your needs, replace the content with:

```bash
#!/bin/sh
. "$(dirname "$0")/_/husky.sh"

echo 'Building code'
dotnet build

echo 'Formatting code'
dotnet format

echo 'Running tests'
dotnet test
```

This script will:

1. Build the code.
2. Format the code based on the rules defined in the `.editorconfig` file.
3. Run all tests.

Finally, add the `pre-commit` file to Git:

```bash
git add .husky/pre-commit
```

#### Step 5: Handling Formatting Issues

If `dotnet format` modifies the source files after the snapshot has been created, those changes won't be included in the final commit. Here are three approaches to handle this:

1. **Include All Changes Using Git Add:**
   Add `git add .` after `dotnet format` to ensure all changes are included in the commit.

   ```bash
   dotnet build
   dotnet format
   git add .
   dotnet test
   ```

2. **Execute a Dry Run of Dotnet Format:**
   Use `dotnet format --verify-no-changes` to check for formatting issues before running the actual formatting.

   ```bash
   dotnet format --verify-no-changes
   ```

   If issues are found, the commit is aborted, allowing you to fix the formatting and then retry the commit.

3. **Run Dotnet Format Only on Staged Files:**
   Define a task in `task-runner.json` to format only the staged files:

   ```json
   {
     "name": "dotnet-format-staged-files",
     "group": "pre-commit-operations",
     "command": "dotnet",
     "args": ["format", "--include", "${staged}"],
     "include": ["**/*.cs"]
   }
   ```

   Then, update your pre-commit script to run this task:

   ```bash
   dotnet husky run --name dotnet-format-staged-files
   ```

#### Step 6: Customizing Pre-Commit Commands

You can customize the commands that run during the pre-commit process by editing the `.husky/pre-commit` file. Here are some examples of tasks you can add:

1. **Running Test Coverage:**

   You can add a command to check test coverage using a tool like `coverlet`:

   ```bash
   echo 'Checking test coverage'
   dotnet test --collect:"XPlat Code Coverage"
   ```

   This will run the tests and collect code coverage data.

2. **Linting:**

   If you want to lint your code, you can use a tool like `dotnet format` with additional rules or use a specific linter like `StyleCop.Analyzers`:

   ```bash
   echo 'Linting code'
   dotnet format --fix-style
   ```

3. **Static Code Analysis:**

   You can run static code analysis tools to ensure your code meets quality standards:

   ```bash
   echo 'Running static code analysis'
   dotnet build --no-restore /p:RunCodeAnalysis=true
   ```

4. **Custom Scripts:**

   You can add any custom scripts or commands that you need to run before committing the code. Just include them in the `.husky/pre-commit` file:

   ```bash
   echo 'Running custom script'
   ./scripts/my-custom-script.sh
   ```

   Ensure the script is executable and located in the correct path.

By customizing the pre-commit hook, you can ensure that your codebase maintains high standards of quality and consistency before any changes are committed.

---

### 2. How to Run Husky Pre-Commit Hooks

Once Husky.NET is configured, it will automatically run the pre-commit hooks whenever you attempt to commit changes.

#### Using Git Commands

To commit your changes, use the following command:

```bash
git commit -m "Your commit message"
```

During the commit process, Husky.NET will:

1. Format the staged files (`dotnet husky run --name dotnet-format-staged-files`).
2. Build the project (`dotnet build --no-restore`).
3. Run all unit tests (`dotnet test --no-restore`).

If any of these steps fail, the commit will be aborted, and you’ll need to fix the issues before trying again.

#### Skipping Hooks

If you need to skip the pre-commit validation (e.g., during an emergency or when an external dependency is down), you can bypass the hooks using:

```bash
git commit -m "my message" --no-verify
```

This allows you to commit the changes without running the pre-commit hooks.

---
