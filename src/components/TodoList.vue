<script>
import draggable from 'vuedraggable';

export default {
    components: {
        draggable
    },
    props: {
        computedTodo: Array,
        darkMode: Boolean,
    },
    data() {
        return {
            localTodos: [],
            expandedTodos: [],
        }
    },
    watch: {
        computedTodo: {
            handler(newVal) {
                console.log('TodoList - 받은 데이터:', newVal);
                console.log('TodoList - 첫 번째 아이템의 태그:', newVal[0]?.tags);
                console.log('TodoList - 첫 번째 아이템의 서브태스크:', newVal[0]?.subtasks);
                this.localTodos = [...newVal];
            },
            deep: true
        }
    },
    methods: {
        toggleComplete(id) {
            this.$emit('update-todo', id, { completed: !this.localTodos.find(t => t.id === id).completed });
        },
        deleteTodo(id) {
            this.$emit('delete-todo', id);
        },
        toggleExpand(id) {
            const index = this.expandedTodos.indexOf(id);
            if (index === -1) {
                this.expandedTodos.push(id);
            } else {
                this.expandedTodos.splice(index, 1);
            }
        },
        toggleSubtask(todoId, subtaskIndex) {
            const todo = this.localTodos.find(t => t.id === todoId);
            if (todo) {
                const subtasks = [...todo.subtasks];
                subtasks[subtaskIndex].completed = !subtasks[subtaskIndex].completed;
                this.$emit('update-todo', todoId, { subtasks });
            }
        },
        calculateProgress(todo) {
            if (!todo.subtasks.length) return todo.completed ? 100 : 0;
            const completed = todo.subtasks.filter(st => st.completed).length;
            return Math.round((completed / todo.subtasks.length) * 100);
        },
        formatDate(dateString) {
            if (!dateString) return '';
            const date = new Date(dateString);
            return date.toLocaleDateString('ko-KR', {
                year: 'numeric',
                month: 'long',
                day: 'numeric'
            });
        },
        onDragEnd() {
            this.$emit('update-order', this.localTodos);
        }
    }
}
</script>
<template>
    <div class="todo-list">
        <draggable 
            v-model="localTodos" 
            item-key="id"
            handle=".drag-handle"
            @end="onDragEnd"
        >
            <template #item="{ element: todo }">
                <div 
                    class="todo-item"
                    :class="{ 
                        'completed': todo.completed,
                        'priority-high': todo.priority === 'high',
                        'priority-medium': todo.priority === 'medium',
                        'priority-low': todo.priority === 'low'
                    }"
                >
                    <div class="todo-content">
                        <div class="todo-header">
                            <div class="drag-handle">
                                <i class="fas fa-grip-vertical"></i>
                            </div>
                            <input 
                                type="checkbox" 
                                :checked="todo.completed"
                                @change="toggleComplete(todo.id)"
                            >
                            <span class="todo-text">{{ todo.text }}</span>
                            <div class="todo-actions">
                                <button class="action-btn expand" @click="toggleExpand(todo.id)">
                                    <i class="fas" :class="expandedTodos.includes(todo.id) ? 'fa-chevron-up' : 'fa-chevron-down'"></i>
                                </button>
                                <button class="action-btn delete" @click="deleteTodo(todo.id)">
                                    <i class="fas fa-trash"></i>
                                </button>
                            </div>
                        </div>
                        
                        <div v-if="expandedTodos.includes(todo.id)" class="todo-details">
                            <div class="detail-section">
                                <h4>태그</h4>
                                <div class="tags" v-if="todo.tags && todo.tags.length">
                                    <span v-for="(tag, index) in todo.tags" :key="index" class="tag">
                                        {{ tag }}
                                    </span>
                                </div>
                                <p v-else class="no-data">태그가 없습니다.</p>
                            </div>
                            
                            <div class="detail-section">
                                <h4>서브태스크</h4>
                                <div class="subtasks" v-if="todo.subtasks && todo.subtasks.length">
                                    <div v-for="(subtask, index) in todo.subtasks" :key="index" class="subtask">
                                        <input 
                                            type="checkbox" 
                                            :checked="subtask.completed"
                                            @change="toggleSubtask(todo.id, index)"
                                        >
                                        <span :class="{ 'completed': subtask.completed }">{{ subtask.text }}</span>
                                    </div>
                                </div>
                                <p v-else class="no-data">서브태스크가 없습니다.</p>
                            </div>

                            <div class="detail-section" v-if="todo.dueDate">
                                <h4>마감일</h4>
                                <p class="due-date">{{ formatDate(todo.dueDate) }}</p>
                            </div>
                            
                            <div class="progress-bar" v-if="todo.subtasks && todo.subtasks.length">
                                <div class="progress" :style="{ width: calculateProgress(todo) + '%' }"></div>
                                <span class="progress-text">{{ calculateProgress(todo) }}% 완료</span>
                            </div>
                        </div>
                    </div>
                </div>
            </template>
        </draggable>
    </div>
</template>

<style scoped>
.todo-list {
    display: flex;
    flex-direction: column;
    gap: 0rem;
    margin-bottom: 3rem;
}

.todo-item {
    display: flex;
    gap: 1rem;
    background: var(--card-background);
    padding: 1.2rem;
    border-radius: 0.8rem;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
    transition: all 0.3s ease;
    border-left: 4px solid transparent;
    cursor: default;
}

.todo-item:hover {
    transform: translateY(-2px);
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.15);
}

.todo-item.completed {
    opacity: 0.7;
}

.todo-item.priority-high {
    border-left-color: var(--danger-color);
}

.todo-item.priority-medium {
    border-left-color: var(--warning-color);
}

.todo-item.priority-low {
    border-left-color: var(--success-color);
}

.todo-item.dragging {
    opacity: 0.5;
    background: var(--background-color);
}

.todo-content {
    flex: 1;
}

.todo-header {
    display: flex;
    align-items: center;
    gap: 1.5rem;
}

.drag-handle {
    cursor: move;
    padding: 0.5rem;
    color: var(--text-color-light);
    transition: color 0.3s ease;
}

.drag-handle:hover {
    color: var(--primary-color);
}

.todo-text {
    flex: 1;
    color: var(--text-color);
    font-size: 1.1rem;
}

.todo-text.completed {
    text-decoration: line-through;
    opacity: 0.5;
}

.todo-actions {
    display: flex;
    gap: 1rem;
}

.action-btn {
    background: none;
    border: none;
    color: var(--text-color);
    opacity: 0.5;
    cursor: pointer;
    padding: 0.5rem;
    transition: all 0.3s ease;
    font-size: 1.1rem;
}

.action-btn:hover {
    opacity: 1;
}

.action-btn.delete:hover {
    color: var(--danger-color);
}

.todo-details {
    margin-top: 2rem;
    padding-top: 2rem;
    border-top: 1px solid var(--border-color);
}

.detail-section {
    margin-bottom: 1.5rem;
}

.tags {
    display: flex;
    flex-wrap: wrap;
    gap: 0.8rem;
    margin-bottom: 1.5rem;
}

.tag {
    display: inline-flex;
    align-items: center;
    gap: 0.5rem;
    padding: 0.5rem 1rem;
    background: var(--primary-color);
    color: white;
    border-radius: 2rem;
    font-size: 0.9rem;
}

.no-data {
    color: var(--text-color);
    opacity: 0.5;
    font-size: 0.9rem;
    font-style: italic;
}

.subtasks {
    display: flex;
    flex-direction: column;
    gap: 1rem;
    margin-bottom: 1.5rem;
}

.subtask {
    display: flex;
    align-items: center;
    gap: 1rem;
    padding: 0.8rem 1rem;
    background: var(--background-color);
    border-radius: 0.5rem;
    transition: all 0.3s ease;
}

.subtask:hover {
    transform: translateX(5px);
}

.subtask input[type="checkbox"] {
    width: 1.2rem;
    height: 1.2rem;
    cursor: pointer;
}

.subtask span {
    flex: 1;
    color: var(--text-color);
}

.subtask span.completed {
    text-decoration: line-through;
    opacity: 0.5;
}

.progress-bar {
    position: relative;
    height: 1.2rem;
    background: var(--background-color);
    border-radius: 1rem;
    overflow: hidden;
}

.progress {
    position: absolute;
    top: 0;
    left: 0;
    height: 100%;
    background: var(--primary-color);
    transition: width 0.3s ease;
}

.progress-text {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    color: var(--text-color);
    font-size: 0.8rem;
    font-weight: bold;
}

.due-date {
    color: var(--text-color);
    font-size: 0.9rem;
    margin-top: 0.5rem;
    padding: 0.5rem;
    background: var(--background-color);
    border-radius: 0.5rem;
    display: inline-block;
}
</style>