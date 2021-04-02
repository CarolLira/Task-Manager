<template>
  <div v-show="showAddTask">
    <AddTask @add-task="addTask" />
  </div>
  <Tasks
    @toggle-reminder="toggleReminder"
    @delete-task="deleteTask"
    :tasks="tasks"
  />
</template>

<script>
import Tasks from '@/components/Tasks';
import AddTask from '@/components/AddTask';

export default {
  name: 'Home',
  props: {
    showAddTask: Boolean,
  },
  components: {
    Tasks,
    AddTask,
  },
  data() {
    return {
      tasks: [],
    };
  },
  methods: {
    async addTask(task) {
      const resp = await fetch('api/tasks', {
        method: 'POST',
        headers: {
          'Content-type': 'application/json',
        },
        body: JSON.stringify(task),
      });

      const data = await resp.json();
      this.tasks = [...this.tasks, data];
    },
    async deleteTask(id) {
      if (confirm('Are you sure?')) {
        const resp = await fetch(`api/tasks/${id}`, {
          method: 'DELETE',
        });
        resp.status === 200
          ? (this.tasks = this.tasks.filter((task) => task.id !== id))
          : alert('Error: Task could not be deleted');
      }
    },
    async toggleReminder(id) {
      const taskToToggle = await this.getTask(id);
      const updateTask = { ...taskToToggle, reminder: !taskToToggle.reminder };
      const resp = await fetch(`api/tasks/${id}`, {
        method: 'PUT',
        headers: {
          'Content-type': 'application/json',
        },
        body: JSON.stringify(updateTask),
      });

      const data = await resp.json();

      this.tasks = this.tasks.map((task) =>
        task.id === id ? { ...task, reminder: data.reminder } : task
      );
    },
    async getTasks() {
      const resp = await fetch('api/tasks');
      const data = await resp.json();
      return data;
    },
    async getTask(id) {
      const resp = await fetch(`api/tasks/${id}`);
      const data = await resp.json();
      return data;
    },
  },
  async created() {
    this.tasks = await this.getTasks();
  },
};
</script>
