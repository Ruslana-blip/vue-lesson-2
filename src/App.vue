<template>
	<div class="container">
		<div class="container__input">
			<input v-model="newTodo" type="text" />
			<button @click="addList">Add</button>
		</div>
		<br />
		<span>Total: {{ count + todos.length - this.limit }}</span>
		<br />
		<div class="container__filter">
			<span>Filter by: </span>
			<select v-model="filter" id="select">
				<option value="All">All</option>
				<option value="Selected">Selected</option>
				<option value="Uncompleted">Uncompleted</option>
			</select>
		</div>
		<div class="container__limit">
			<span>Limit</span>
			<select v-model="limit" @change="addTodo">
				<option value="10">10</option>
				<option value="15">15</option>
				<option value="20">20</option>
			</select>
		</div>
		<ul class="list">
			<li v-for="todo in isFiltered" :key="todo.id" class="list__item">
				<input type="checkbox" :checked="todo.completed" />
				<span> {{ todo.title }}</span>
				<button @click="deleteItem(todo.id)" class="delete">X</button>
			</li>
		</ul>
		<div v-if="count" class="pagination">
			<button
				v-for="page in Math.ceil(count / limit)"
				:key="page"
				:class="{ active: page === this.page }"
				@click="this.page = page"
			>
				{{ page }}
			</button>
		</div>
	</div>
</template>

<script>
import axios from 'axios';
export default {
	data() {
		return {
			baseUrl: 'https://jsonplaceholder.typicode.com/',
			todos: [],
			newTodo: '',
			filter: 'all',
			limit: 10,
			page: 1,
			count: 0,
		};
	},
	mounted() {
		this.counter();
		this.showData();
	},
	computed: {
		isFiltered() {
			if (this.filter === 'Selected') {
				return this.todos.filter(todo => todo.completed);
			} else if (this.filter === 'Uncompleted') {
				return this.todos.filter(todo => !todo.completed);
			}
			return this.todos;
		},
	},
	watch: {
		page() {
			this.addTodo();
		},
		limit() {
			this.addTodo();
		},
		filter() {
			this.saveFilter();
		},
	},
	methods: {
		async addTodo() {
			try {
				const { data } = await axios.get(`${this.baseUrl}todos`, {
					params: {
						_limit: this.limit,
						_page: this.page,
					},
				});
				this.todos = data;
				this.saveData();
			} catch (err) {
				this.errorMessage = 'Виникла помилка';
			} finally {
			}
		},
		addList() {
			if (!this.newTodo) return;
			this.todos.unshift({
				id: Date.now(),
				completed: false,
				title: this.newTodo,
			});
			this.newTodo = '';
			this.saveData();
		},
		deleteItem(id) {
			this.todos = this.todos.filter(el => el.id !== id);
			this.saveData();
		},
		async counter() {
			try {
				const { data } = await axios.get(`${this.baseUrl}todos`, {
					params: {},
				});
				this.count = data.length;
			} catch (err) {
				this.errorMessage = 'Виникла помилка';
			} finally {
			}
		},
		saveData() {
			localStorage.setItem('list', JSON.stringify(this.todos));
		},
		showData() {
			const data = JSON.parse(localStorage.getItem('list'));
			if (data && data.length) {
				this.todos = data;
			} else {
				this.addTodo();
			}
		},
		saveFilter() {
			localStorage.setItem('filter', this.filter);
		},
		restoreFilter() {
			const savedFilter = localStorage.getItem('filter');
			if (savedFilter) {
				this.filter = savedFilter;
			}
		},
	},
};
</script>

<style scoped lang="scss">
.container {
	box-sizing: border-box;
	width: 900px;
	height: 100vh;
	margin: 0 auto;
	display: flex;
	flex-direction: column;
	justify-content: center;
	font-size: 18px;
	&__input {
		display: flex;
		position: relative;
		& input {
			flex-basis: 80%;
			height: 40px;
			border: 1px solid grey;
			padding-left: 10px;
			font-size: 20px;
			border-radius: 20px;
		}
		& button {
			position: absolute;
			top: 0;
			right: 0;
			width: 200px;
			height: 44px;
			font-size: 26px;
			background-color: orangered;
			border: none;
			color: white;
			border-radius: 20px;
			cursor: pointer;
		}
	}
	&__filter,
	span,
	&__limit {
		margin: 0 auto;
	}

	&__limit {
		margin-top: 20px;
	}
}
.list {
	padding: 0;
	list-style: none;
	// .list__item
	&__item {
		&:not(:last-child) {
			margin-bottom: 10px;
		}
		display: flex;
		justify-content: space-between;
		& input {
			flex-basis: 10%;
		}
		& span {
			flex-basis: 80%;
		}
		& button {
			flex-basis: 5%;
		}
	}
}
.pagination {
	margin: 0 auto;
	margin-top: 30px;
	& button {
		width: 40px;
		height: 40px;
		background-color: orangered;
		border: none;
		color: white;
		font-size: 26px;
		cursor: pointer;
		&:not(:last-child) {
			margin-right: 5px;
		}
	}
}
</style>
