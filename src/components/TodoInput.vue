<script>
export default {
    props: {
        darkMode: Boolean,
    },
    data(){
        return{
            msg: '', // 데이터 정의
            priority: 'medium',
            dueDate: '',
            tags: [],
            tagInput: '',
            subtasks: [],
            subtaskInput: '',
        };
    },
    emits: ['add-todo'],

    methods: {
        handleSubmit() {
            console.log('handleSubmit 호출됨');
            console.log('현재 msg 값:', this.msg);
            
            if (!this.msg.trim()) {
                console.log('msg가 비어있음');
                return;
            }
            
            console.log('현재 subtasks 상태:', this.subtasks);
            console.log('현재 tags 상태:', this.tags);
            console.log('현재 dueDate 상태:', this.dueDate);
            
            const todoData = {
                msg: this.msg.trim(),
                priority: this.priority,
                tags: [...this.tags],
                subtasks: this.subtasks.map(st => ({
                    text: st.text,
                    completed: st.completed
                })),
                dueDate: this.dueDate
            };
            
            console.log('전송할 데이터:', todoData);
            console.log('전송할 데이터의 msg:', todoData.msg);
            this.$emit('add-todo', todoData);
            
            // 입력 필드 초기화
            this.msg = '';
            this.priority = 'medium';
            this.tags = [];
            this.subtasks = [];
            this.tagInput = '';
            this.subtaskInput = '';
            this.dueDate = '';
        },
        addTag() {
            console.log('addTag 호출됨');
            console.log('현재 tagInput:', this.tagInput);
            
            if (!this.tagInput || !this.tagInput.trim()) {
                console.log('태그 입력이 비어있음');
                return;
            }
            
            const tag = this.tagInput.trim();
            console.log('추가할 태그:', tag);
            console.log('현재 tags 배열:', this.tags);
            
            if (!this.tags.includes(tag)) {
                this.tags = [...this.tags, tag];
                console.log('태그 추가 후 tags 배열:', this.tags);
            } else {
                console.log('이미 존재하는 태그');
            }
            
            this.tagInput = '';
        },
        removeTag(index) {
            console.log('removeTag 호출됨, 인덱스:', index);
            console.log('제거 전 tags:', this.tags);
            this.tags = this.tags.filter((_, i) => i !== index);
            console.log('제거 후 tags:', this.tags);
        },
        addSubtask() {
            console.log('addSubtask 호출됨');
            console.log('현재 subtaskInput:', this.subtaskInput);
            
            if (!this.subtaskInput || !this.subtaskInput.trim()) {
                console.log('서브태스크 입력이 비어있음');
                return;
            }
            
            const subtask = {
                text: this.subtaskInput.trim(),
                completed: false
            };
            
            console.log('추가할 서브태스크:', subtask);
            console.log('현재 subtasks 배열:', this.subtasks);
            
            this.subtasks = [...this.subtasks, subtask];
            console.log('서브태스크 추가 후 subtasks 배열:', this.subtasks);
            
            this.subtaskInput = '';
        },
        removeSubtask(index) {
            console.log('removeSubtask 호출됨, 인덱스:', index);
            console.log('제거 전 subtasks:', this.subtasks);
            this.subtasks = this.subtasks.filter((_, i) => i !== index);
            console.log('제거 후 subtasks:', this.subtasks);
        }
    }
}
</script>

<template>
    <div class="todo-input" :class="{ 'dark-mode': darkMode }">
        <form @submit.prevent="handleSubmit">
            <div class="input-group">
                <input 
                    type="text" 
                    v-model="msg"
                    placeholder="새로운 할 일을 입력하세요..."
                    @keyup.enter="handleSubmit"
                >
                <select v-model="priority">
                    <option value="high">높음</option>
                    <option value="medium">중간</option>
                    <option value="low">낮음</option>
                </select>
                <input 
                    type="date" 
                    v-model="dueDate"
                    class="due-date"
                >
                <button type="submit">추가</button>
            </div>

            <!-- 태그 입력 -->
            <div class="tag-input">
                <input 
                    type="text" 
                    v-model="tagInput"
                    placeholder="태그 입력..."
                    @keyup.enter.prevent="addTag"
                >
                <button type="button" @click="addTag">태그 추가</button>
                <div class="tags">
                    <span v-for="(tag, index) in tags" :key="index" class="tag">
                        {{ tag }}
                        <button type="button" @click="removeTag(index)" class="remove-tag">&times;</button>
                    </span>
                </div>
            </div>

            <!-- 서브태스크 입력 -->
            <div class="subtask-input">
                <input 
                    type="text" 
                    v-model="subtaskInput"
                    placeholder="서브태스크 입력..."
                    @keyup.enter.prevent="addSubtask"
                >
                <button type="button" @click="addSubtask">서브태스크 추가</button>
                <div class="subtasks">
                    <div v-for="(subtask, index) in subtasks" :key="index" class="subtask">
                        <input 
                            type="checkbox" 
                            v-model="subtask.completed"
                        >
                        <span>{{ subtask.text }}</span>
                        <button type="button" @click="removeSubtask(index)" class="remove-subtask">&times;</button>
                    </div>
                </div>
            </div>
        </form>
    </div>
</template>

<style scoped>
.todo-input {
    background: var(--card-background);
    padding: 1.5rem;
    border-radius: 1rem;
    box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
    margin-bottom: 2rem;
}

.input-group {
    display: flex;
    gap: 1rem;
    margin-bottom: 1rem;
}

input[type="text"] {
    flex: 1;
    padding: 0.8rem;
    border: 2px solid var(--border-color);
    border-radius: 0.5rem;
    font-size: 1rem;
    background: var(--card-background);
    color: var(--text-color);
}

select {
    padding: 0.8rem;
    border: 2px solid var(--border-color);
    border-radius: 0.5rem;
    background: var(--card-background);
    color: var(--text-color);
    min-width: 100px;
}

button {
    padding: 0.8rem 1.5rem;
    border: none;
    border-radius: 0.5rem;
    background: var(--primary-color);
    color: white;
    cursor: pointer;
    transition: all 0.3s ease;
}

button:hover {
    background: var(--background-color);
    color: var(--text-color);
    transform: translateY(-1px);
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
}

.dark-mode button:hover {
    background: var(--background-color);
    color: var(--text-color);
}

.tag-input, .subtask-input {
    margin-top: 1rem;
}

.tags, .subtasks {
    display: flex;
    flex-wrap: wrap;
    gap: 0.5rem;
    margin-top: 0.5rem;
}

.tag {
    display: inline-flex;
    align-items: center;
    padding: 0.3rem 0.8rem;
    background: var(--primary-color-light);
    color: var(--primary-color);
    border-radius: 1rem;
    font-size: 0.9rem;
}

.remove-tag, .remove-subtask {
    padding: 0 0.5rem;
    background: none;
    color: var(--primary-color);
    font-size: 1.2rem;
}

.remove-tag:hover, .remove-subtask:hover {
    color: var(--primary-color-dark);
    background: none;
    transform: none;
    box-shadow: none;
}

.subtask {
    display: flex;
    align-items: center;
    gap: 0.5rem;
    padding: 0.5rem;
    background: var(--background-color);
    border-radius: 0.5rem;
    width: 100%;
}

.subtask input[type="checkbox"] {
    width: 1.2rem;
    height: 1.2rem;
}

.dark-mode {
    --card-background: #2d2d2d;
    --text-color: #ffffff;
    --border-color: #404040;
}

.due-date {
    padding: 0.8rem;
    border: 2px solid var(--border-color);
    border-radius: 0.5rem;
    background: var(--card-background);
    color: var(--text-color);
    min-width: 150px;
}
</style>