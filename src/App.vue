<script>
  import TodoHeader from './components/TodoHeader.vue';
  import TodoInput from './components/TodoInput.vue';
  import TodoList from './components/TodoList.vue';

  export default{
    components: {
      TodoHeader,
      TodoInput,
      TodoList,
    },
    data(){
      return {
        todo: [],
        current: 'all',
        darkMode: false,
        searchQuery: '',
        selectedTags: [],
        selectedPriority: null,
      }
    },
    computed:{
      filteredTodos() {
        let filtered = [...this.todo];
        
        // 검색어로 필터링
        if (this.searchQuery) {
          const query = this.searchQuery.toLowerCase();
          filtered = filtered.filter(todo => {
            // 메인 태스크 검색
            const mainTaskMatch = todo.text.toLowerCase().includes(query);
            
            // 태그 검색
            const tagMatch = todo.tags.some(tag => 
              tag.toLowerCase().includes(query)
            );
            
            // 서브태스크 검색
            const subtaskMatch = todo.subtasks.some(subtask => 
              subtask.text.toLowerCase().includes(query)
            );
            
            return mainTaskMatch || tagMatch || subtaskMatch;
          });
        }
        
        // 우선순위로 필터링
        if (this.selectedPriority) {
          filtered = filtered.filter(todo => 
            todo.priority === this.selectedPriority
          );
        }
        
        // 탭으로 필터링
        switch (this.current) {
          case 'active':
            filtered = filtered.filter(todo => !todo.completed);
            break;
          case 'completed':
            filtered = filtered.filter(todo => todo.completed);
            break;
        }
        
        return filtered;
      },
      allTags() {
        const tags = new Set();
        this.todo.forEach(todo => {
          if (Array.isArray(todo.tags)) {
            todo.tags.forEach(tag => tags.add(tag));
          }
        });
        return Array.from(tags);
      }
    },
    methods: {
      addTodo(inputData) {
        console.log('App.vue - addTodo 호출됨');
        console.log('App.vue - 입력 데이터:', inputData);
        console.log('App.vue - 입력 데이터의 msg:', inputData.msg);
        console.log('App.vue - 입력 데이터의 태그:', inputData.tags);
        console.log('App.vue - 입력 데이터의 서브태스크:', inputData.subtasks);
        
        const item = {
          id: Date.now(),
          text: inputData.msg,
          completed: false,
          priority: inputData.priority || 'medium',
          tags: inputData.tags || [],
          subtasks: inputData.subtasks || [],
          dueDate: inputData.dueDate,
          createdAt: new Date().toISOString()
        };
        
        console.log('App.vue - 저장할 아이템:', item);
        console.log('App.vue - 저장할 아이템의 text:', item.text);
        this.todo.push(item);
        console.log('App.vue - 저장 후 todo 배열:', this.todo);
        
        // localStorage에 저장
        localStorage.setItem('todos', JSON.stringify(this.todo));
      },
      saveTodo() {
        localStorage.setItem('todos', JSON.stringify(this.todo));
      },
      updateTab(tab) {
        this.current = tab;
      },
      deleteTodo(id) {
        this.todo = this.todo.filter((v)=> v.id !== id);
      },
      updateTodo(id, updates) {
        this.todo = this.todo.map((v) =>
          v.id === id ? { ...v, ...updates } : v
        );
      },
      toggleDarkMode() {
        this.darkMode = !this.darkMode;
        document.body.classList.toggle('dark-mode');
      },
      updateSearch(query) {
        this.searchQuery = query.trim();
      },
      updateTags(tags) {
        this.selectedTags = Array.isArray(tags) ? [...tags] : [];
      },
      updatePriority(priority) {
        this.selectedPriority = priority || null;
      },
      reorderTodos(oldIndex, newIndex) {
        const item = this.todo.splice(oldIndex, 1)[0];
        this.todo.splice(newIndex, 0, item);
      },
      updateOrder(newOrder) {
        this.todo = newOrder;
        this.saveTodo();
      },
    },
  }
  
</script>

<template>
  <div class="todo" :class="{ 'dark-mode': darkMode }">
    <div class="theme-toggle" @click="toggleDarkMode">
      <i :class="darkMode ? 'fas fa-sun' : 'fas fa-moon'"></i>
    </div>
    <TodoHeader 
      :current="current" 
      :dark-mode="darkMode"
      :all-tags="allTags"
      :selected-tags="selectedTags"
      :selected-priority="selectedPriority"
      @update-tab="updateTab"
      @update-search="updateSearch"
      @update-tags="updateTags"
      @update-priority="updatePriority"
    />
    <TodoList 
      :computed-todo="filteredTodos"
      :dark-mode="darkMode"
      @delete-todo="deleteTodo"
      @update-todo="updateTodo"
      @reorder-todos="reorderTodos"
      @update-order="updateOrder"
    />
    <TodoInput 
      :dark-mode="darkMode"
      @add-todo="addTodo"
    />
  </div>
</template>

<style>
:root {
  --primary-color: #6c5ce7;
  --secondary-color: #a29bfe;
  --background-color: #f5f6fa;
  --text-color: #2d3436;
  --card-background: #ffffff;
  --border-color: #dfe6e9;
  --success-color: #00b894;
  --warning-color: #fdcb6e;
  --danger-color: #d63031;
}

.dark-mode {
  --primary-color: #a29bfe;
  --secondary-color: #6c5ce7;
  --background-color: #2d3436;
  --text-color: #f5f6fa;
  --card-background: #353b48;
  --border-color: #636e72;
}

.todo {
  min-height: 100vh;
  background: linear-gradient(135deg, var(--background-color), var(--secondary-color));
  padding: 2rem;
  transition: all 0.3s ease;
  max-width: 1400px;
  margin: 0 auto;
}

.theme-toggle {
  position: fixed;
  top: 1rem;
  right: 1rem;
  background: var(--card-background);
  border-radius: 50%;
  width: 40px;
  height: 40px;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
  transition: all 0.3s ease;
}

.theme-toggle:hover {
  transform: scale(1.1);
}

.theme-toggle i {
  color: var(--text-color);
  font-size: 1.2rem;
}

@media (max-width: 1440px) {
  .todo {
    margin: 0 2rem;
  }
}

@media (max-width: 768px) {
  .todo {
    padding: 1rem;
    margin: 0 1rem;
  }
}
</style>