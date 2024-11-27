<template>
  <div class="todo-container">
    <h2 class="title">My To-Do List</h2>

    <!-- Form to Add a Task -->
    <form @submit.prevent="addTask" class="task-form">
      <input
        v-model="newTask"
        type="text"
        placeholder="Add a new task"
        required
        class="task-input"
      />
      <button type="submit" class="add-task-button">Add Task</button>
    </form>

    <!-- Display Tasks -->
    <ul class="task-list">
      <li v-for="task in tasks" :key="task.id" class="task-item">
        <input
          type="checkbox"
          v-model="task.completed"
          @change="updateTask(task)"
          class="task-checkbox"
        />

        <!-- Inline Edit/Display Toggle -->
        <span v-if="task.id !== editingTaskId" :class="{ completed: task.completed }" class="task-text">
          {{ task.task }}
        </span>
        <input
          v-else
          type="text"
          v-model="editingTaskText"
          @keyup.enter="saveEdit(task.id)"
          class="task-input-edit"
        />

        <!-- Edit/Save Button -->
        <button
          v-if="task.id !== editingTaskId"
          @click="editTask(task.id, task.task)"
          class="edit-button"
        >
          Edit
        </button>
        <button
          v-else
          @click="saveEdit(task.id)"
          class="save-button"
        >
          Save
        </button>

        <!-- Delete Button -->
        <button @click="deleteTask(task.id)" class="delete-button">Delete</button>
      </li>
    </ul>
  </div>
</template>

<script>
import axios from "axios";

export default {
  data() {
    return {
      tasks: [], // List of tasks
      newTask: "", // Holds the new task input
      editingTaskId: null, // Tracks the ID of the task being edited
      editingTaskText: "", // Holds the new text for the task being edited
    };
  },
  mounted() {
    this.loadTasksFromLocalStorage(); // Load tasks from localStorage on mount
    this.fetchTasks(); // Fetch existing tasks from the backend (optional)
  },
  methods: {
    // Load tasks from localStorage
    loadTasksFromLocalStorage() {
      const storedTasks = localStorage.getItem("tasks");
      if (storedTasks) {
        this.tasks = JSON.parse(storedTasks);
      }
    },

    // Save tasks to localStorage
    saveTasksToLocalStorage() {
      localStorage.setItem("tasks", JSON.stringify(this.tasks));
    },

    // Fetch tasks from the API
    fetchTasks() {
      axios
        .get("/api/tasks")
        .then((response) => {
          this.tasks = response.data;
          this.saveTasksToLocalStorage();
        })
        .catch((error) => {
          console.error("Error fetching tasks:", error);
        });
    },

    // Function to validate input
    validateInput(input) {
  const forbiddenPatterns = /<[^>]+>|[{}[\]()<>;*%$#@!&]/g;
  if (forbiddenPatterns.test(input)) {
    alert("Your input contains invalid characters or scripts. Please enter valid text.");
    this.newTask = ""; // Reset the input field by updating the bound property
    return false;
  }
  return true;
}
,

    // Add a new task
    addTask() {
      if (!this.newTask.trim()) {
        alert("Task cannot be empty!");
        return;
      }

      // Validate the new task input
      if (!this.validateInput(this.newTask)) {
        return;
      }

      const newTask = {
        id: Date.now(),
        task: this.newTask,
        completed: false,
      };

      this.tasks.push(newTask);
      this.saveTasksToLocalStorage();
      this.newTask = "";

      // Simulated backend update
      axios
        .post("/api/tasks", { task: newTask.task })
        .then((response) => {
          newTask.id = response.data.id;
          this.saveTasksToLocalStorage();
        })
        .catch((error) => {
          console.error("Error adding task to backend:", error);
        });
    },

    // Update task completion status
    updateTask(task) {
      this.saveTasksToLocalStorage();
      // Simulate backend update
      axios
        .patch(`/api/tasks/${task.id}`, { completed: task.completed })
        .catch((error) => {
          console.error("Error updating task:", error);
        });
    },

    // Delete a task
    deleteTask(id) {
      this.tasks = this.tasks.filter((task) => task.id !== id);
      this.saveTasksToLocalStorage();
      // Simulate backend deletion
      axios
        .delete(`/api/tasks/${id}`)
        .catch((error) => {
          console.error("Error deleting task:", error);
        });
    },

    // Start editing a task
    editTask(id, text) {
      this.editingTaskId = id;
      this.editingTaskText = text;
    },

    saveEdit(id) {
  const task = this.tasks.find((task) => task.id === id);

  if (!task || !this.editingTaskText.trim()) {
    alert("Task cannot be empty!");
    return;
  }

  // Update the task locally
  task.task = this.editingTaskText.trim();
  this.editingTaskId = null;
  this.editingTaskText = "";
  this.saveTasksToLocalStorage();

  // Send the updated task to the backend
  axios
    .patch(`/api/tasks/${id}`, { task: task.task }) // Only send the updated field
    .then((response) => {
      console.log("Task updated successfully:", response.data);
    })
    .catch((error) => {
      console.error("Error updating task:", error);
    });
}
  },
};
</script>

<style scoped>
.todo-container {
  max-width: 600px;
  margin: 0 auto;
  padding: 20px;
  background-color: #f9fafb;
  border-radius: 8px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
}

.title {
  text-align: center;
  font-size: 2rem;
  color: #333;
  margin-bottom: 20px;
}

.task-form {
  display: flex;
  margin-bottom: 20px;
  gap: 10px;
}

.task-input {
  flex-grow: 1;
  padding: 10px;
  font-size: 1rem;
  border: 1px solid #ddd;
  border-radius: 4px;
}

.add-task-button {
  padding: 10px 15px;
  background-color: #4CAF50;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 1rem;
  transition: background-color 0.3s ease;
}

.add-task-button:hover {
  background-color: #45a049;
}

.task-list {
  list-style-type: none;
  padding: 0;
  margin: 0;
}

.task-item {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 10px 0;
  border-bottom: 1px solid #ddd;
}

.task-checkbox {
  margin-right: 10px;
}

.task-text {
  flex-grow: 1;
  font-size: 1rem;
  color: #333;
  transition: text-decoration 0.3s ease;
}

.completed {
  text-decoration: line-through;
  color: #888;
}

.task-input-edit {
  flex-grow: 1;
  padding: 5px;
  font-size: 1rem;
  border: 1px solid #ddd;
  border-radius: 4px;
  margin-right: 10px;
}

.edit-button,
.save-button {
  background-color: #007bff;
  color: white;
  border: none;
  padding: 5px 10px;
  border-radius: 4px;
  cursor: pointer;
  font-size: 0.875rem;
  transition: background-color 0.3s ease;
}

.edit-button:hover,
.save-button:hover {
  background-color: #0056b3;
}

.delete-button {
  background-color: #ff4d4d;
  color: white;
  border: none;
  padding: 5px 10px;
  border-radius: 4px;
  cursor: pointer;
  font-size: 0.875rem;
  transition: background-color 0.3s ease;
}

.delete-button:hover {
  background-color: #e74c3c;
}
</style>
