<template>
    <div class="app-container">
        <div class="todo-app">
            <h1 class="app-title">Todo App</h1>
            
            <!-- Gestion des catégories -->
            <div class="categories-section">
                <h2>Catégories</h2>
                <div class="categories-list">
                    <div 
                        v-for="category in categories" 
                        :key="category.id"
                        class="category-item"
                        :style="{ borderColor: category.color }"
                    >
                        <span class="category-name">{{ category.name }}</span>
                        <div class="category-actions">
                            <button @click="editCategory(category)" class="edit-button">✏️</button>
                            <button @click="deleteCategory(category)" class="delete-button">🗑️</button>
                        </div>
                    </div>
                </div>
                <div class="category-form">
                    <input 
                        type="text" 
                        v-model="newCategory.name" 
                        placeholder="Nouvelle catégorie"
                        class="category-input"
                    >
                    <input 
                        type="color" 
                        v-model="newCategory.color"
                        class="color-picker"
                    >
                    <button @click="addCategory" class="add-button">Ajouter</button>
                </div>
            </div>

            <div class="todo-input-container">
                <input 
                    type="text" 
                    v-model="newTodo.title" 
                    @keyup.enter="addTodo" 
                    placeholder="Que souhaitez-vous faire ?"
                    class="todo-input"
                >
                <select v-model="newTodo.category_id" class="category-select">
                    <option value="">Sans catégorie</option>
                    <option v-for="category in categories" :key="category.id" :value="category.id">
                        {{ category.name }}
                    </option>
                </select>
                <button @click="addTodo" class="add-button">
                    <span class="plus-icon">+</span>
                </button>
            </div>

            <div class="todo-stats">
                <div class="stats-item">
                    <span class="stats-number">{{ remainingCount }}</span>
                    <span class="stats-label">tâches restantes</span>
                </div>
                <div class="stats-item">
                    <span class="stats-number">{{ todos.length - remainingCount }}</span>
                    <span class="stats-label">tâches complétées</span>
                </div>
            </div>

            <div class="filters">
                <button 
                    :class="{ active: filter === 'all' }" 
                    @click="filter = 'all'"
                    class="filter-button"
                >
                    Toutes
                </button>
                <button 
                    :class="{ active: filter === 'active' }" 
                    @click="filter = 'active'"
                    class="filter-button"
                >
                    Actives
                </button>
                <button 
                    :class="{ active: filter === 'completed' }" 
                    @click="filter = 'completed'"
                    class="filter-button"
                >
                    Complétées
                </button>
            </div>

            <transition-group name="todo-list" tag="ul" class="todo-list">
                <li v-for="todo in filteredTodos" :key="todo.id" class="todo-item" :style="getTodoStyle(todo)">
                    <div class="todo-content">
                        <label class="checkbox-container">
                            <input 
                                type="checkbox" 
                                v-model="todo.completed" 
                                @change="updateTodo(todo)"
                                class="checkbox-input"
                            >
                            <span class="checkmark"></span>
                        </label>
                        <div class="todo-info">
                            <span 
                                :class="{ completed: todo.completed }"
                                @dblclick="startEditing(todo)"
                                v-if="!todo.editing"
                                class="todo-text"
                            >
                                {{ todo.title }}
                            </span>
                            <input 
                                v-else
                                type="text" 
                                v-model="todo.title" 
                                @blur="finishEditing(todo)"
                                @keyup.enter="finishEditing(todo)"
                                @keyup.esc="cancelEditing(todo)"
                                ref="editInput"
                                class="edit-input"
                            >
                            <span v-if="todo.category" class="todo-category" :style="{ color: todo.category.color }">
                                {{ todo.category.name }}
                            </span>
                        </div>
                    </div>
                    <div class="todo-actions">
                        <select v-model="todo.category_id" @change="updateTodo(todo)" class="category-select">
                            <option value="">Sans catégorie</option>
                            <option v-for="category in categories" :key="category.id" :value="category.id">
                                {{ category.name }}
                            </option>
                        </select>
                        <button @click="deleteTodo(todo)" class="delete-button">
                            <span class="trash-icon">🗑️</span>
                        </button>
                    </div>
                </li>
            </transition-group>

            <div v-if="todos.length === 0" class="empty-state">
                <p>Vous n'avez pas encore de tâches</p>
                <p class="empty-state-hint">Commencez par en ajouter une !</p>
            </div>
        </div>
    </div>
</template>

<script>
import axios from 'axios';

export default {
    data() {
        return {
            newTodo: {
                title: '',
                category_id: ''
            },
            todos: [],
            filter: 'all',
            editingTodo: null,
            categories: [],
            newCategory: {
                name: '',
                color: '#4CAF50'
            }
        }
    },
    computed: {
        filteredTodos() {
            switch (this.filter) {
                case 'active':
                    return this.todos.filter(todo => !todo.completed);
                case 'completed':
                    return this.todos.filter(todo => todo.completed);
                default:
                    return this.todos;
            }
        },
        remainingCount() {
            return this.todos.filter(todo => !todo.completed).length;
        }
    },
    methods: {
        async fetchTodos() {
            try {
                const response = await axios.get('/api/todos');
                this.todos = response.data;
            } catch (error) {
                console.error('Erreur lors de la récupération des tâches:', error);
            }
        },
        async fetchCategories() {
            try {
                const response = await axios.get('/api/categories');
                this.categories = response.data;
            } catch (error) {
                console.error('Erreur lors de la récupération des catégories:', error);
            }
        },
        async addTodo() {
            if (this.newTodo.title.trim()) {
                try {
                    const response = await axios.post('/api/todos', this.newTodo);
                    this.todos.push(response.data);
                    this.newTodo.title = '';
                    this.newTodo.category_id = '';
                } catch (error) {
                    console.error('Erreur lors de l\'ajout de la tâche:', error);
                }
            }
        },
        async updateTodo(todo) {
            try {
                const response = await axios.put(`/api/todos/${todo.id}`, todo);
                const index = this.todos.findIndex(t => t.id === todo.id);
                this.todos[index] = response.data;
            } catch (error) {
                console.error('Erreur lors de la mise à jour de la tâche:', error);
                todo.completed = !todo.completed;
            }
        },
        async deleteTodo(todo) {
            try {
                await axios.delete(`/api/todos/${todo.id}`);
                this.todos = this.todos.filter(t => t.id !== todo.id);
            } catch (error) {
                console.error('Erreur lors de la suppression de la tâche:', error);
            }
        },
        async addCategory() {
            if (this.newCategory.name.trim()) {
                try {
                    const response = await axios.post('/api/categories', this.newCategory);
                    this.categories.push(response.data);
                    this.newCategory.name = '';
                    this.newCategory.color = '#4CAF50';
                } catch (error) {
                    console.error('Erreur lors de l\'ajout de la catégorie:', error);
                }
            }
        },
        async updateCategory(category) {
            try {
                const response = await axios.put(`/api/categories/${category.id}`, category);
                const index = this.categories.findIndex(c => c.id === category.id);
                this.categories[index] = response.data;
            } catch (error) {
                console.error('Erreur lors de la mise à jour de la catégorie:', error);
            }
        },
        async deleteCategory(category) {
            if (confirm(`Voulez-vous vraiment supprimer la catégorie "${category.name}" ?`)) {
                try {
                    await axios.delete(`/api/categories/${category.id}`);
                    this.categories = this.categories.filter(c => c.id !== category.id);
                    // Mettre à jour les todos qui avaient cette catégorie
                    this.todos.forEach(todo => {
                        if (todo.category_id === category.id) {
                            todo.category_id = null;
                            this.updateTodo(todo);
                        }
                    });
                } catch (error) {
                    console.error('Erreur lors de la suppression de la catégorie:', error);
                }
            }
        },
        startEditing(todo) {
            this.editingTodo = todo;
            todo.editing = true;
            this.$nextTick(() => {
                this.$refs.editInput[0].focus();
            });
        },
        async finishEditing(todo) {
            if (todo.title.trim()) {
                try {
                    await this.updateTodo(todo);
                    todo.editing = false;
                    this.editingTodo = null;
                } catch (error) {
                    console.error('Erreur lors de la mise à jour de la tâche:', error);
                }
            } else {
                this.deleteTodo(todo);
            }
        },
        cancelEditing(todo) {
            todo.editing = false;
            this.editingTodo = null;
        },
        getTodoStyle(todo) {
            if (todo.category) {
                return {
                    borderLeft: `4px solid ${todo.category.color}`
                };
            }
            return {};
        },
        editCategory(category) {
            const newName = prompt('Nouveau nom de la catégorie:', category.name);
            const newColor = prompt('Nouvelle couleur (format hex):', category.color);
            if (newName !== null && newColor !== null) {
                category.name = newName;
                category.color = newColor;
                this.updateCategory(category);
            }
        }
    },
    mounted() {
        this.fetchCategories();
        this.fetchTodos();
    }
}
</script>

<style>
:root {
    --primary-color: #4CAF50;
    --primary-hover: #45a049;
    --danger-color: #ff4444;
    --danger-hover: #cc0000;
    --text-color: #333;
    --text-light: #666;
    --border-color: #ddd;
    --background-color: #f5f5f5;
    --white: #ffffff;
    --shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}

body {
    font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
    background-color: var(--background-color);
    color: var(--text-color);
}

.app-container {
    min-height: 100vh;
    display: flex;
    justify-content: center;
    align-items: center;
    padding: 20px;
}

.todo-app {
    width: 100%;
    max-width: 600px;
    background: var(--white);
    border-radius: 12px;
    box-shadow: var(--shadow);
    padding: 30px;
}

.app-title {
    text-align: center;
    color: var(--primary-color);
    margin-bottom: 30px;
    font-size: 2.5rem;
    font-weight: 600;
}

.todo-input-container {
    display: flex;
    gap: 10px;
    margin-bottom: 20px;
}

.todo-input {
    flex: 1;
    padding: 12px 15px;
    border: 2px solid var(--border-color);
    border-radius: 8px;
    font-size: 16px;
    transition: border-color 0.3s;
}

.todo-input:focus {
    outline: none;
    border-color: var(--primary-color);
}

.add-button {
    width: 45px;
    height: 45px;
    border: none;
    background-color: var(--primary-color);
    color: white;
    border-radius: 8px;
    cursor: pointer;
    display: flex;
    align-items: center;
    justify-content: center;
    transition: background-color 0.3s;
}

.add-button:hover {
    background-color: var(--primary-hover);
}

.plus-icon {
    font-size: 24px;
    font-weight: bold;
}

.todo-stats {
    display: flex;
    justify-content: space-around;
    margin-bottom: 20px;
    padding: 15px;
    background-color: var(--background-color);
    border-radius: 8px;
}

.stats-item {
    text-align: center;
}

.stats-number {
    display: block;
    font-size: 24px;
    font-weight: bold;
    color: var(--primary-color);
}

.stats-label {
    font-size: 14px;
    color: var(--text-light);
}

.filters {
    display: flex;
    gap: 10px;
    margin-bottom: 20px;
}

.filter-button {
    flex: 1;
    padding: 10px;
    border: 2px solid var(--border-color);
    background: var(--white);
    border-radius: 8px;
    cursor: pointer;
    transition: all 0.3s;
}

.filter-button.active {
    background: var(--primary-color);
    color: var(--white);
    border-color: var(--primary-color);
}

.todo-list {
    list-style: none;
}

.todo-item {
    display: flex;
    align-items: center;
    justify-content: space-between;
    padding: 15px;
    border-bottom: 1px solid var(--border-color);
    transition: all 0.3s;
}

.todo-content {
    display: flex;
    align-items: center;
    gap: 15px;
    flex: 1;
}

.checkbox-container {
    display: block;
    position: relative;
    cursor: pointer;
}

.checkbox-input {
    position: absolute;
    opacity: 0;
    cursor: pointer;
    height: 0;
    width: 0;
}

.checkmark {
    position: relative;
    display: inline-block;
    height: 20px;
    width: 20px;
    background-color: var(--white);
    border: 2px solid var(--border-color);
    border-radius: 4px;
    transition: all 0.3s;
}

.checkbox-input:checked ~ .checkmark {
    background-color: var(--primary-color);
    border-color: var(--primary-color);
}

.checkmark:after {
    content: "";
    position: absolute;
    display: none;
    left: 6px;
    top: 2px;
    width: 5px;
    height: 10px;
    border: solid white;
    border-width: 0 2px 2px 0;
    transform: rotate(45deg);
}

.checkbox-input:checked ~ .checkmark:after {
    display: block;
}

.todo-text {
    flex: 1;
    font-size: 16px;
}

.todo-text.completed {
    text-decoration: line-through;
    color: var(--text-light);
}

.edit-input {
    flex: 1;
    padding: 8px;
    border: 2px solid var(--primary-color);
    border-radius: 4px;
    font-size: 16px;
}

.delete-button {
    background: none;
    border: none;
    color: var(--danger-color);
    cursor: pointer;
    padding: 5px;
    transition: color 0.3s;
}

.delete-button:hover {
    color: var(--danger-hover);
}

.empty-state {
    text-align: center;
    padding: 40px;
    color: var(--text-light);
}

.empty-state-hint {
    margin-top: 10px;
    font-size: 14px;
}

/* Animations */
.todo-list-enter-active,
.todo-list-leave-active {
    transition: all 0.3s ease;
}

.todo-list-enter-from,
.todo-list-leave-to {
    opacity: 0;
    transform: translateX(30px);
}

.todo-list-move {
    transition: transform 0.3s ease;
}

.categories-section {
    margin-bottom: 30px;
    padding: 20px;
    background-color: var(--background-color);
    border-radius: 8px;
}

.categories-section h2 {
    margin-bottom: 15px;
    color: var(--text-color);
    font-size: 1.2rem;
}

.categories-list {
    display: flex;
    flex-wrap: wrap;
    gap: 10px;
    margin-bottom: 15px;
}

.category-item {
    display: flex;
    align-items: center;
    padding: 8px 12px;
    background: var(--white);
    border-radius: 6px;
    border-left: 4px solid;
    box-shadow: var(--shadow);
}

.category-name {
    margin-right: 10px;
}

.category-actions {
    display: flex;
    gap: 5px;
}

.category-form {
    display: flex;
    gap: 10px;
    align-items: center;
}

.category-input {
    flex: 1;
    padding: 8px 12px;
    border: 2px solid var(--border-color);
    border-radius: 6px;
}

.color-picker {
    width: 40px;
    height: 40px;
    padding: 2px;
    border: 2px solid var(--border-color);
    border-radius: 6px;
    cursor: pointer;
}

.category-select {
    padding: 8px;
    border: 2px solid var(--border-color);
    border-radius: 6px;
    background-color: var(--white);
    margin-left: 10px;
}

.todo-category {
    font-size: 0.8rem;
    padding: 2px 6px;
    border-radius: 4px;
    background-color: var(--background-color);
    margin-left: 10px;
}

.todo-info {
    display: flex;
    align-items: center;
    flex: 1;
}

.todo-actions {
    display: flex;
    align-items: center;
    gap: 10px;
}

.edit-button {
    background: none;
    border: none;
    cursor: pointer;
    padding: 5px;
    font-size: 1rem;
}
</style> 