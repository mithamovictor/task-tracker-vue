<template>
  <div>
    <AddTask v-show="showAddTask" @add-task="addTask" />
    <TasksComponent
      @toggle-reminder="toggleReminder"
      @delete-task="deleteTask"
      :tasks="tasks"
    />
  </div>
</template>

<script>
import TasksComponent from "../components/Tasks.vue";
import AddTask from "../components/AddTask.vue";

export default {
  name: "HomeComponent",
  props: {
    showAddTask: Boolean,
  },
  components: {
    TasksComponent,
    AddTask,
  },
  data() {
    return {
      tasks: [],
    };
  },
  methods: {
    async addTask(task) {
      const res = await fetch("api/tasks", {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify(task),
      });
      const data = await res.json();
      this.tasks = [...this.tasks, data];
    },
    async deleteTask(id) {
      if (confirm("Are you sure?")) {
        const res = await fetch(`api/tasks/${id}`, {
          method: "DELETE",
        });
        res.status === 200
          ? (this.tasks = this.tasks?.filter((x) => x.id != id))
          : alert("Error deleting task");
      }
    },
    async toggleReminder(id) {
      const taskToToggle = await this.fetchTask(id);
      const updTask = { ...taskToToggle, reminder: !taskToToggle.reminder };
      const res = await fetch(`api/tasks/${id}`, {
        method: "PUT",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify(updTask),
      });
      const data = await res.json();
      this.tasks = this.tasks.map((x) =>
        x.id === id ? { ...x, reminder: data.reminder } : x
      );
    },
    async fetchTasks() {
      const result = await fetch("api/tasks");
      return await result.json();
    },
    async fetchTask(id) {
      const result = await fetch(`api/tasks/${id}`);
      return await result.json();
    },
  },
  async created() {
    this.tasks = await this.fetchTasks();
  },
};
</script>
