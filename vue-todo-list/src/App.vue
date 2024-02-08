<template>
  <div id="outercover">
    <div id="app">
      <h1 class="title">TODO LIST</h1>
      <div class="input-container">
        <input
          type="text"
          v-model="newTaskName"
          placeholder="Add new task"
          class="input-field"
        />
        <input type="date" v-model="newTaskDate" class="input-field" />
        <button @click="addTask" class="add-button">Add Task</button>
      </div>

      <!-- Open Tasks -->
      <h2 class="section-title">Open Tasks</h2>
      <ul v-if="openTasks.length > 0">
        <li v-for="task in openTasks" :key="task.id" class="task-item">
          <span>{{ task.name }} - Created on: {{ task.date }}</span>
          <button @click="completeTask(task)" class="complete-button">
            Complete
          </button>
          <button @click="editTask(task)" class="edit-button">Edit</button>
        </li>
      </ul>
      <!-- Completed Tasks -->
      <h2 class="section-title">Completed Tasks</h2>
      <ul v-if="completedTasks.length > 0">
        <li v-for="task in completedTasks" :key="task.id" class="task-item">
          <input
            type="checkbox"
            v-model="selectedCompletedTasks"
            :value="task.id"
            class="checkbox"
          />
          <span class="completed"
            >{{ task.name }} - Completed on: {{ task.date }}</span
          >
        </li>
      </ul>
      <button
        @click="deleteSelectedCompletedTasks"
        v-if="selectedCompletedTasks.length > 0"
        class="delete-button"
      >
        Delete Selected
      </button>
    </div>
  </div>
</template>

<script>
import axios from "axios";

export default {
  name: "App",
  data() {
    return {
      newTaskName: "",
      newTaskDate: "",
      tasks: [],
      selectedCompletedTasks: [],
      baseURL: "http://localhost:4000", // Update base URL to use port 4000
    };
  },
  computed: {
    openTasks() {
      return this.tasks.filter((task) => !task.completed);
    },
    completedTasks() {
      return this.tasks.filter((task) => task.completed);
    },
  },
  methods: {
    fetchTasks() {
      axios
        .get(`${this.baseURL}/tasks`) // Use baseURL for endpoint URL
        .then((response) => {
          this.tasks = response.data;
        })
        .catch((error) => {
          console.error("Error fetching tasks:", error);
        });
    },
    addTask() {
      const newTask = {
        name: this.newTaskName,
        completed: false,
        date: this.newTaskDate,
      };

      axios
        .post(`${this.baseURL}/tasks`, newTask) // Use baseURL for endpoint URL
        .then((response) => {
          this.tasks.push(response.data);
          this.newTaskName = "";
          this.newTaskDate = "";
        })
        .catch((error) => {
          console.error("Error adding task:", error);
        });
    },
    completeTask(task) {
      task.completed = true;
      axios
        .put(`${this.baseURL}/tasks/${task.id}`, task) // Use baseURL for endpoint URL
        .then(() => {
          // Optionally, you can remove the task from the open tasks list
          // this.tasks = this.tasks.filter(t => t.id !== task.id);
        })
        .catch((error) => {
          console.error("Error completing task:", error);
        });
    },
    deleteCompletedTask(taskId) {
      axios
        .delete(`${this.baseURL}/tasks/${taskId}`) // Use baseURL for endpoint URL
        .then(() => {
          this.tasks = this.tasks.filter((task) => task.id !== taskId);
        })
        .catch((error) => {
          console.error(
            `Error deleting completed task with ID ${taskId}:`,
            error
          );
        });
    },
    deleteSelectedCompletedTasks() {
      this.selectedCompletedTasks.forEach((id) => {
        axios
          .delete(`${this.baseURL}/tasks/${id}`) // Use baseURL for endpoint URL
          .then(() => {
            this.tasks = this.tasks.filter((task) => task.id !== id);
          })
          .catch((error) => {
            console.error(`Error deleting task with ID ${id}:`, error);
          });
      });
      this.selectedCompletedTasks = [];
    },
    // editTask(task) {
    // Implement task editing functionality here
    // For example, you can open a modal or a form to edit the task
    //},
  },
  mounted() {
    this.fetchTasks();
  },
};
</script>

<style scoped>
#app {
  font-family: Arial, sans-serif;
  margin: 0 auto;
  padding: 20px;
  max-width: 600px;
  background-color: #ccc; /* Grey outer background */
}

.title {
  text-align: center;
  margin-bottom: 20px;
  color: #7055a5; /* Green color */
}

.input-container {
  display: flex;
  margin-bottom: 20px;
}

.input-field {
  flex: 1;
  padding: 8px;
  font-size: 16px;
  border: 1px solid #ccc;
  border-radius: 4px;
  margin-right: 10px;
}

.add-button {
  padding: 8px 16px;
  font-size: 16px;
  background-color: #4caf50; /* Green color */
  color: #fff; /* White text color */
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.complete-button {
  padding: 6px 12px;
  font-size: 14px;
  background-color: #ffc107; /* Yellow color */
  color: #000; /* Black text color */
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.delete-button {
  padding: 6px 12px;
  font-size: 14px;
  background-color: #ff5722; /* Red color */
  color: #fff; /* White text color */
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.section-title {
  font-size: 20px;
  margin-top: 30px;
  margin-bottom: 10px;
}

.task-item {
  margin-bottom: 10px;
}

.completed {
  text-decoration: line-through;
}

.checkbox {
  margin-right: 10px;
}
</style>
