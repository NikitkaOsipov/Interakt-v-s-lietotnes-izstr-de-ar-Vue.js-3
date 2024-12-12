<script setup>
import { ref, onMounted, computed, watch } from 'vue'

const todos = ref([])
const name = ref('')

const input_content = ref('')
const input_category = ref(null)
const filter_status = ref('')
const filter_category = ref('')
const filter_name = ref('')


const todos_asc = computed(() => todos.value.sort((a,b) =>{
	return  b.createdAt - a.createdAt // sort by date created
}))

const filteredTodos = computed(() => {
	return todos_asc.value.filter(todo => {
        return (
			(filter_status.value ? todo.done === (filter_status.value === 'true') : true) &&
          (filter_category.value ? todo.category === filter_category.value : true) &&
          (filter_name.value ? todo.content.includes(filter_name.value) : true)
        );
	})
})

const clearFilters = () => {
  filter_status.value = ''
  filter_category.value = ''
  filter_name.value = ''
}

watch(name, (newVal) => {
	localStorage.setItem('name', newVal) // when name changes, save to local storage
})

watch(todos, (newVal) => {
	localStorage.setItem('todos', JSON.stringify(newVal)) //
}, {
	deep: true
})

const addTodo = () => {
	if (input_content.value.trim() === '' || input_category.value === null) {
		return
	}

	todos.value.push({
		content: input_content.value,
		category: input_category.value,
		done: false,
		editable: false,
		createdAt: new Date().getTime()
	})
}

const removeTodo = (todo) => {
	todos.value = todos.value.filter((t) => t !== todo)
}

onMounted(() => {
	name.value = localStorage.getItem('name') || '' // when component mounts, get name from local storage
	todos.value = JSON.parse(localStorage.getItem('todos')) || []
})
</script>

<template>
	<main class="app">
		
		<section class="greeting">
			<h2 class="title">
				What's up, <input type="text" id="name" placeholder="Name here" v-model="name">
			</h2>
		</section>

		<section class="create-todo">
			<h3>CREATE A TODO</h3>

			<form id="new-todo-form" @submit.prevent="addTodo">
				<h4>What's on your todo list?</h4>
				<input 
					type="text" 
					name="content" 
					id="content" 
					placeholder="e.g. do homework"
					v-model="input_content" />
				
				<h4>Pick a category</h4>
				<div class="options">

					<label>
						<input 
							type="radio" 
							name="category" 
							id="category1" 
							value="business"
							v-model="input_category" />
						<span class="bubble business"></span>
						<div>Business</div>
					</label>

					<label>
						<input 
							type="radio" 
							name="category" 
							id="category2" 
							value="personal"
							v-model="input_category" />
						<span class="bubble personal"></span>
						<div>Personal</div>
					</label>

					<label>
						<input 
							type="radio" 
							name="category" 
							id="category3" 
							value="study"
							v-model="input_category" />
						<span class="bubble study"></span>
						<div>Study</div>
					</label>
				</div>

				<input type="submit" value="Add todo" />
			</form>
		</section>

		<section class="todo-list">
			<h3>TODO LIST</h3>
			<div class="list" id="todo-list">
				<h3>Filter Todos</h3>
				<div class="filters">
					<input 
						type="text" 
						placeholder="Filter by name" 
						v-model="filter_name" />
					<select v-model="filter_status">
						<option value="">All</option>
						<option value="true">Completed</option>
						<option value="false">Pending</option>
					</select>
					<select v-model="filter_category">
						<option value="">All</option>
						<option value="business">Business</option>
						<option value="personal">Personal</option>
						<option value="study">Study</option>
					</select>
					<button @click="clearFilters">Clear Filters</button>
				</div>                                                                   
				
				<div v-for="todo in filteredTodos" :class="`todo-item ${todo.done && 'done'}`">
					<label>
						<input type="checkbox" v-model="todo.done" />
						<span :class="`bubble ${todo.category}`">	</span>
					</label>

					<div class="todo-content">
						<input type="text" v-model="todo.content" />
					</div>

					<div class="actions">
						<button class="delete" @click="removeTodo(todo)">Delete</button>
					</div>
				</div>

			</div>
		</section>

	</main>
</template>