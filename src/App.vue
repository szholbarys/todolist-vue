<script lang="ts">
import { defineComponent, ref, computed, watch, nextTick } from "vue";
import type { Task } from "./types";

export default defineComponent({
  name: "TodoList",
  setup() {
    const tasks = ref<Task[]>([]);
    const newTaskTitle = ref("");
    const newTaskPriority = ref<"low" | "medium" | "high">("medium");
    let nextId = 1;

    const pendingTasksCount = computed(() => {
      return tasks.value.filter((task) => !task.completed).length;
    });

    const sortedTasks = computed(() => {
      return [...tasks.value].sort((a, b) => {
        const priorityOrder = { high: 3, medium: 2, low: 1 };
        return priorityOrder[b.priority] - priorityOrder[a.priority];
      });
    });

    const addTask = async () => {
      if (newTaskTitle.value.trim()) {
        const task: Task = {
          id: nextId++,
          title: newTaskTitle.value,
          priority: newTaskPriority.value,
          completed: false,
        };
        tasks.value.push(task);
        newTaskTitle.value = "";

        await nextTick();
        const taskElement = document.getElementById(`task-${task.id}`);
        taskElement?.scrollIntoView({ behavior: "smooth" });
      }
    };

    const deleteTask = (taskId: number) => {
      const index = tasks.value.findIndex((task) => task.id === taskId);
      if (index !== -1) {
        tasks.value.splice(index, 1);
      }
    };

    const toggleTask = (taskId: number) => {
      const task = tasks.value.find((task) => task.id === taskId);
      if (task) {
        task.completed = !task.completed;
      }
    };

    watch(
      tasks,
      (newTasks, oldTasks) => {
        if (newTasks.length > oldTasks.length) {
          console.log("Task added:", newTasks[newTasks.length - 1]);
        } else if (newTasks.length < oldTasks.length) {
          console.log("Task deleted");
        } else {
          const completedTask = newTasks.find(
            (task, index) => task.completed !== oldTasks[index].completed
          );
          if (completedTask) {
            console.log(
              `Task "${completedTask.title}" ${
                completedTask.completed ? "completed" : "uncompleted"
              }`
            );
          }
        }
      },
      { deep: true }
    );

    return {
      tasks,
      newTaskTitle,
      newTaskPriority,
      pendingTasksCount,
      sortedTasks,
      addTask,
      deleteTask,
      toggleTask,
    };
  },
});
</script>

<template>
  <div class="todo-app">
    <div class="header">
      <h1 class="app-title">Todo List</h1>
      <img src="/public//download.jpeg" alt="todolist-logo" class="logo" />
    </div>
    <p class="pending-tasks">Pending tasks: {{ pendingTasksCount }}</p>

    <div class="add-task">
      <input
        v-model="newTaskTitle"
        @keyup.enter="addTask"
        placeholder="Add new task..."
        class="task-input"
      />
      <select v-model="newTaskPriority" class="priority-select">
        <option value="low">Low</option>
        <option value="medium">Medium</option>
        <option value="high">High</option>
      </select>
      <button @click="addTask" class="add-button">Add Task</button>
    </div>

    <transition-group name="task-list" tag="ul" class="task-list">
      <li
        v-for="task in sortedTasks"
        :key="task.id"
        :id="'task-' + task.id"
        :class="{
          'task-item': true,
          completed: task.completed,
          [`priority-${task.priority}`]: true,
        }"
      >
        <input
          type="checkbox"
          :checked="task.completed"
          @change="toggleTask(task.id)"
          class="task-checkbox"
        />
        <span class="task-title">{{ task.title }}</span>
        <span class="task-priority" :class="`priority-badge-${task.priority}`">
          {{ task.priority }}
        </span>
        <button @click="deleteTask(task.id)" class="delete-button">
          Delete
        </button>
      </li>
    </transition-group>
  </div>
</template>

<style scoped>
.header {
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 10px;
}

.logo {
  width: 50px;
  height: 50px;
}

.todo-app {
  max-width: 800px;
  margin: 2rem auto;
  padding: 2rem;
  background-color: #ffffff;
  border-radius: 12px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}

.app-title {
  font-size: 2.5rem;
  color: #2c3e50;
  margin-bottom: 1.5rem;
  text-align: center;
}

.pending-tasks {
  font-size: 1.2rem;
  color: #666;
  margin-bottom: 1.5rem;
}

.add-task {
  display: flex;
  gap: 1rem;
  margin-bottom: 2rem;
}

.task-input {
  flex: 1;
  padding: 1rem;
  font-size: 1.1rem;
  border: 2px solid #ddd;
  border-radius: 8px;
  transition: border-color 0.3s ease;
}

.task-input:focus {
  border-color: #4caf50;
  outline: none;
}

.priority-select {
  padding: 0.5rem 1rem;
  font-size: 1.1rem;
  border: 2px solid #ddd;
  border-radius: 8px;
  min-width: 120px;
  cursor: pointer;
}

.add-button {
  padding: 0.75rem 1.5rem;
  font-size: 1.1rem;
  background-color: #4caf50;
  color: white;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  transition: background-color 0.3s ease;
}

.add-button:hover {
  background-color: #45a049;
}

.task-list {
  list-style: none;
  padding: 0;
}

.task-item {
  display: flex;
  align-items: center;
  padding: 1rem 1.5rem;
  margin: 1rem 0;
  background-color: #f8f9fa;
  border-radius: 8px;
  gap: 1rem;
  transition: transform 0.2s ease, box-shadow 0.2s ease;
}

.task-item:hover {
  transform: translateY(-2px);
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.task-checkbox {
  width: 1.5rem;
  height: 1.5rem;
  cursor: pointer;
}

.task-title {
  flex: 1;
  font-size: 1.1rem;
  color: #2c3e50;
}

.completed .task-title {
  text-decoration: line-through;
  color: #999;
}

.task-priority {
  padding: 0.5rem 1rem;
  border-radius: 6px;
  font-size: 0.9rem;
  text-transform: uppercase;
  color: white;
  font-weight: 500;
}

.priority-badge-high {
  background-color: #ff4444;
}

.priority-badge-medium {
  background-color: #ffbb33;
  color: #333;
}

.priority-badge-low {
  background-color: #00c851;
}

.priority-high {
  border-left: 6px solid #ff4444;
}

.priority-medium {
  border-left: 6px solid #ffbb33;
}

.priority-low {
  border-left: 6px solid #00c851;
}

.delete-button {
  padding: 0.5rem 1rem;
  font-size: 1rem;
  background-color: #ff4444;
  color: white;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  transition: background-color 0.3s ease;
}

.delete-button:hover {
  background-color: #cc0000;
}

.task-list-enter-active,
.task-list-leave-active {
  transition: all 0.5s ease;
}

.task-list-enter-from,
.task-list-leave-to {
  opacity: 0;
  transform: translateX(30px);
}

.task-list-move {
  transition: transform 0.5s ease;
}

@media (max-width: 600px) {
  .todo-app {
    padding: 1rem;
    margin: 1rem;
  }

  .add-task {
    flex-direction: column;
  }

  .priority-select {
    width: 100%;
  }
}
</style>
