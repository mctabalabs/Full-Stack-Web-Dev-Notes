# Project: Task Manager

Build a Task Manager application that runs in the browser. This project focuses on using JavaScript functions to manage a private list of tasks, supporting addition (individual or bulk), removal, and listing. It also simulates asynchronous data saving and utilizes Git branching for feature management.

---

## Step 1: Set up the Git repository

1.  **Create a new repository on GitHub** named `task-manager`.
2.  **Clone it locally** using VS Code's built-in Git support.
3.  **Branching Strategy**: Keep the `main` branch clean. All new features must live in dedicated feature branches.
4.  **Workflow**: For each feature, create a new branch using:
    ```bash
    git checkout -b <branch-name>
    ```
    Push the branch when the feature is complete.

---

## Step 2: Implement the core logic using closures

1.  **File Creation**: Create a file named `taskManager.js`.
2.  **Function Definition**: Define a function `createTaskManager()` that returns an object exposing the following methods:
    *   `addTask`
    *   `addTasks`
    *   `listTasks`
    *   `removeTask`
    *   `saveTasks`
3.  **Closures and Private State**:
    Inside `createTaskManager()`, define a private array `let tasks = []`.
    
    > **Technical Note: Closures**
    > Because the returned methods access `tasks`, they form **closures**. They "remember" and maintain access to the outer `tasks` array even after `createTaskManager()` has finished execution. This effectively bundles a function with references to its lexical environment.

4.  **Scope Validation**: Ensure that `tasks` is only accessible through these public methods.

---

## Step 3: Parameters, Default Values, and Rest Parameters

1.  **Rest Parameters**: Implement `addTasks(...items)`.
    *   The `...` syntax gathers any number of arguments into an array.
    *   *Constraint*: A function can only have one rest parameter, and it must be the last parameter.
2.  **Default Parameters**: Implement `addTask(task, priority = 'normal')`.
    *   If `priority` is omitted or `undefined`, it defaults to `'normal'`.
    *   *Rule*: Place default parameters before rest parameters. Rest parameters cannot have default values.

```javascript
// taskManager.js Template
function createTaskManager() {
  let tasks = [];

  return {
    addTask(task, priority = 'normal') {
      // Implementation
    },
    addTasks(...items) {
      // Implementation
    },
    listTasks() {
      // Implementation
    },
    removeTask(index) {
      // Implementation
    },
    saveTasks(callback) {
      // Implementation
    }
  };
}
```

---

## Step 4: Callback Pattern and Asynchronous Actions

1.  **Simulate Async Saving**: The `saveTasks(callback)` method should simulate a server request. Use `setTimeout` to delay execution for 1 second, then invoke the callback with the current `tasks` array.
2.  **Event Loop and Job Queue**:
    `setTimeout` schedules a task on the **Task Queue** (or Callback Queue). The event loop processes these once the call stack is empty. Each job runs to completion before the next begins, preventing race conditions.
3.  **Experiment**: Try calling `saveTasks` multiple times to observe the order of execution and the difference between synchronous and asynchronous code.

---

## Step 5: Build a Simple HTML Interface (Optional)

Create an `index.html` file with a form and buttons to interact with the task manager.

1.  **User Interface**: Include inputs for task names and buttons for adding/listing tasks.
2.  **DOM Events**: Use event listeners (e.g., `click`, `submit`) to trigger your JavaScript functions. These event listeners are themselves callbacks managed by the browser.

```html
<!-- index.html Template -->
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Task Manager</title>
</head>
<body>
    <h1>Task Manager</h1>
    <input type="text" id="taskInput" placeholder="New Task">
    <button id="addBtn">Add Task</button>
    <div id="taskList"></div>
    <script src="taskManager.js"></script>
    <script>
        // DOM logic here
    </script>
</body>
</html>
```

---

## Step 6: Practice Git Branching and Merging

1.  **Feature Isolation**: Create a separate branch for each major addition (e.g., `feature-rest-params`, `feature-callbacks`).
2.  **Merging**:
    *   Switch to main: `git checkout main`
    *   Merge: `git merge <branch-name>`
    *   *Note*: Git will perform a "fast-forward" merge if possible, or create a "three-way" merge commit if histories have diverged.
3.  **Cleanup**: Delete the feature branch after a successful merge:
    ```bash
    git branch -d <branch-name>
    ```

---

## Deliverables

*   **`taskManager.js`**: Demonstrating closures, rest/default parameters, and callbacks.
*   **`README.md`**: Including explanations for:
    *   Closures and their use in state management.
    *   The benefits of rest and default parameters.
    *   Asynchronous callbacks and the event loop.
    *   Summary of Git branching commands.
*   **Git History**: Showing clean feature branches and merge commits.
