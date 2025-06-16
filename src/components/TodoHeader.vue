<script>
export default{
    props:{
        current:{
            type: String,
            default() {
                return 'all';
            },
        },
        darkMode: Boolean,
        selectedPriority: String,
    },
    emits: ['update-tab', 'update-search', 'update-priority'],
    data() {
        return {
            searchQuery: '',
            localSelectedPriority: '',
        }
    },
    watch: {
        selectedPriority: {
            immediate: true,
            handler(newVal) {
                this.localSelectedPriority = newVal;
            }
        }
    },
    methods:{
        updateTab(tab) {
            this.$emit('update-tab', tab);
        },
        updateSearch() {
            this.$emit('update-search', this.searchQuery);
        },
        handlePriorityChange(event) {
            this.localSelectedPriority = event.target.value;
            this.$emit('update-priority', event.target.value);
        },
    },
}
</script>

<template>
  <header class="todo-header">
    <h1>할 일 관리</h1>
    <div class="search-bar">
      <input 
        type="text" 
        v-model="searchQuery" 
        placeholder="할 일, 태그, 서브태스크 검색..."
        @input="updateSearch"
      >
      <div class="search-hint" v-if="searchQuery">
        검색 중: "{{ searchQuery }}"
      </div>
    </div>
    <div class="filters">
      <div class="priority-filter">
        <select 
          :value="localSelectedPriority" 
          @change="handlePriorityChange"
        >
          <option value="">모든 우선순위</option>
          <option value="high">높음</option>
          <option value="medium">중간</option>
          <option value="low">낮음</option>
        </select>
      </div>
    </div>
    <div class="tabs">
      <button 
        :class="{ active: current === 'all' }"
        @click="updateTab('all')"
      >
        전체
      </button>
      <button 
        :class="{ active: current === 'active' }"
        @click="updateTab('active')"
      >
        진행중
      </button>
      <button 
        :class="{ active: current === 'completed' }"
        @click="updateTab('completed')"
      >
        완료
      </button>
    </div>
  </header>
</template>

<style scoped>
.todo-header {
  background: var(--card-background);
  padding: 1.5rem;
  border-radius: 1rem;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  margin-bottom: 2rem;
}

h1 {
  color: var(--primary-color);
  margin-bottom: 1rem;
  font-size: 2rem;
  text-align: center;
}

.search-bar {
  margin-bottom: 1rem;
  position: relative;
}

.search-bar input {
  width: 100%;
  padding: 0.8rem;
  border: 2px solid var(--border-color);
  border-radius: 0.5rem;
  font-size: 1rem;
  transition: all 0.3s ease;
  background: var(--card-background);
  color: var(--text-color);
}

.search-bar input:focus {
  border-color: var(--primary-color);
  outline: none;
  box-shadow: 0 0 0 3px rgba(108, 92, 231, 0.1);
}

.search-hint {
  position: absolute;
  top: 100%;
  left: 0;
  right: 0;
  padding: 0.5rem;
  background: var(--card-background);
  border: 1px solid var(--border-color);
  border-top: none;
  border-radius: 0 0 0.5rem 0.5rem;
  font-size: 0.9rem;
  color: var(--text-color-light);
}

.filters {
  display: flex;
  gap: 1rem;
  margin-bottom: 1rem;
}

.priority-filter select {
  padding: 0.5rem;
  border: 2px solid var(--border-color);
  border-radius: 0.5rem;
  background: var(--card-background);
  color: var(--text-color);
  min-width: 150px;
}

.tabs {
  display: flex;
  gap: 0.5rem;
}

.tabs button {
  flex: 1;
  padding: 0.8rem;
  border: none;
  border-radius: 0.5rem;
  background: var(--background-color);
  color: var(--text-color);
  cursor: pointer;
  transition: all 0.3s ease;
}

.tabs button:hover {
  background: var(--primary-color);
  color: white;
}

.tabs button.active {
  background: var(--primary-color);
  color: white;
}

@media (max-width: 768px) {
  .filters {
    flex-direction: column;
  }
  
  .priority-filter select {
    width: 100%;
  }
}
</style>