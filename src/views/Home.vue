<template>
	<AddTask v-show="showAddTask" @add-task="addTask" />
	<Tasks
		@toggle-reminder="toggleReminder"
		@delete-task="deleteTask"
		:tasks="tasks"
	/>
</template>

<script>
import AddTask from '../components/AddTask.vue';
import Tasks from '../components/Tasks.vue';

export default {
	name: 'Home',
	props: {
		showAddTask: Boolean,
	},
	components: {
		AddTask,
		Tasks,
	},
	data() {
		return {
			tasks: [],
		};
	},
	methods: {
		async addTask(task) {
			const response = await fetch(`api/tasks`, {
				method: 'POST',
				headers: {
					'Content-type': 'application/json',
				},
				body: JSON.stringify(task),
			});
			const data = await response.json();

			this.tasks = [...this.tasks, data];
		},
		async toggleReminder(id) {
			const taskToToggle = await this.fetchTask(id);
			const updateTask = {
				...taskToToggle,
				reminder: !taskToToggle.reminder,
			};

			const response = await fetch(`api/tasks/${id}`, {
				method: 'PUT',
				headers: {
					'Content-type': 'application/json',
				},
				body: JSON.stringify(updateTask),
			});
			const data = await response.json();

			this.tasks = this.tasks.map((task) =>
				task.id === id ? { ...task, reminder: data.reminder } : task
			);
		},
		async deleteTask(id) {
			if (confirm('Are you sure?')) {
				const response = await fetch(`api/tasks/${id}`, {
					method: 'DELETE',
				});

				if (response.status === 200) {
					this.tasks = this.tasks.filter((task) => task.id !== id);
				} else {
					alert('Error deleting task.');
				}
			}
		},
		async fetchTasks() {
			const response = await fetch('api/tasks');
			const data = await response.json();

			return data;
		},
		async fetchTask(id) {
			const response = await fetch(`api/tasks/${id}`);
			const data = await response.json();

			return data;
		},
	},
	async created() {
		this.tasks = await this.fetchTasks();
	},
};
</script>
