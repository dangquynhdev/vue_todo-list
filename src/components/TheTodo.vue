<script setup>
    import {ref, computed} from 'vue'
    import draggable from 'vuedraggable'
    // status
    const msgTodo = ref('')
    const filterKey = ref('')
    const input = ref(null)
    const editedTodo = ref()
    const todos = ref([
        {
            id: 1,
            text: 'todo01',
            done: false,
        },
        {
            id: 2,
            text: 'todo02',
            done: false,
        },
        {
            id: 3,
            text: 'todo03',
            done: false,
        }
    ])

    const checkIsEmptyAdd = computed(function() {
        return msgTodo.value.trim() === ''
    })
   
    const filteredTodos = computed(function() {
        let data = null
        if(filterKey.value) {
            data = todos.value.filter(function(item) {
                return item.text.toLowerCase().includes(filterKey.value.toLowerCase())
            })
        } 
        return data
    })

    // function 
    function addTodo() {
        if(!checkIsEmptyAdd.value) {
            todos.value.push({
                id: Date.now(),
                text: msgTodo.value,
                done: false
            })
            msgTodo.value = ""
            input.value.focus()
        }
    }
    function deleteTodo(todo) {
        todos.value = todos.value.filter(function(item) {
            return item.id != todo.id;
        })
    }
    let cachedBeforeEditTodo = '';
    function editTodo(todo) {
        editedTodo.value = todo
        cachedBeforeEditTodo = todo.text
    }
    
    function cancelEditTodo(todo) {
        todo.text = cachedBeforeEditTodo
        editedTodo.value = null
    }
    function doneEditTodo(todo) {
        if(editedTodo.value) {
            todo.text = todo.text.trim()
            if(!todo.text) {
                deleteTodo(todo)
            }
            editedTodo.value = null
        }
    }
    function deleteFilterKey() {
        filterKey.value = ''
    }
    
</script>

<template>
    <div class="p-todoList">
        <h2 class="p-todoList__title">
            TODO LIST
        </h2>
        <div class="p-todoList__form">
            <input 
                class="p-todoList__form-input"
                autofocus
                placeholder="What needs to be done?"
                v-model="msgTodo"
                ref="input"
                @keyup.enter = "addTodo()"
                @focus="deleteFilterKey()"
            >
            <button 
                class="p-todoList__form-btn"
                :disabled="checkIsEmptyAdd"
                @click="addTodo()"
            >
                Add 
            </button>
        </div>
        <div class="p-todoList__form">
            <input 
                class="p-todoList__form-input"
                placeholder="Search for todo by title"
                v-model="filterKey"
            >
            <button 
                v-show="filterKey.length != 0"
                class="p-todoList__form-btn" 
                @click="deleteFilterKey()"
            >x</button>
        </div>
        <draggable 
            v-model="todos" 
            tag="ul"
            class="p-todoList__list" 
            v-if="filteredTodos == null"
            item-key="id"
        >
            <template #item="{ element: item }">
                <li 
                    :class="{'is-complete': item.done, 'is-editing': item == editedTodo}"
                    class="p-todoList__item"
                    
                >
                    <div class="p-todoList__item-view">
                        <input 
                            class="p-todoList__item-toggle"
                            type="checkbox"
                            v-model="item.done"
                        >
                        <label @dblclick="editTodo(item)" class="p-todoList__item-msg">{{item.text}}</label>
                        <button class="p-todoList__item-editBtn" @click="editTodo(item)">edit</button>
                        <button class="p-todoList__item-deleteBtn" @click="deleteTodo(item)">remove</button>
                    </div>
                    <div class="p-todoList__item-edit" v-if="editedTodo == item">
                        <input 
                            v-model="item.text"
                            type="text" 
                            class="p-todoList__item-editInput"
                            @blur="doneEditTodo(item)"
                            @keyup.enter="doneEditTodo(item)"
                            @keyup.esc="cancelEditTodo(item)"
                            @vue:mounted="({ el }) => el.focus()"
                        >
                    </div>
                </li>
            </template>
        </draggable>
        <ul 
            class="p-todoList__list" 
            v-show="todos.length > 0" 
            v-else
        >
            <li 
                v-for="item in filteredTodos"
                :key="item.id"
                :class="{'is-complete': item.done, 'is-editing': item == editedTodo}"
                class="p-todoList__item"
                
            >
                <div class="p-todoList__item-view">
                    <input 
                        class="p-todoList__item-toggle"
                        type="checkbox"
                        v-model="item.done"
                    >
                    <label @dblclick="editTodo(item)" class="p-todoList__item-msg">{{item.text}}</label>
                    <button class="p-todoList__item-editBtn" @click="editTodo(item)">edit</button>
                    <button class="p-todoList__item-deleteBtn" @click="deleteTodo(item)">remove</button>
                </div>
                <div class="p-todoList__item-edit" v-if="editedTodo == item">
                    <input 
                        v-model="item.text"
                        type="text" 
                        class="p-todoList__item-editInput"
                        @blur="doneEditTodo(item)"
                        @keyup.enter="doneEditTodo(item)"
                        @keyup.esc="cancelEditTodo(item)"
                        @vue:mounted="({ el }) => el.focus()"
                    >
                </div>
            </li>
        </ul> 
    </div>
</template>

<style>
    .p-todoList {
        max-width: 700px;
        margin: 0 auto;
    }
    .p-todoList__title {
        color: #d42140;
        font-size: 26px;
        text-align: center;
        font-weight: bold;
    }
    .p-todoList__form {
        display: flex;
        margin-top: 20px;
        gap: 10px;
        position: relative;
    }
    .p-todoList__form-input {
        width: 100%;
        display: block;
        height: 50px;
        padding: 5px 100px 5px 20px;
        font-size: 18px;
        border: 1px solid #000;
    }
    .p-todoList__form-input:focus, .p-todoList__form-input:focus-visible {
        outline: 1px solid #000
    }
    .p-todoList__form-btn {
        border: 1px solid #d42140;
        flex-shrink: 0;
        cursor: pointer;
        background-color: #d42140;
        color: #fff;
        font-weight: bold;
        padding: 5px 10px;
        width: 80px;
        position: absolute;
        right: 0;
        top: 0;
        bottom: 0;

    }
    .p-todoList__form-btn:disabled {
        opacity: 0.5;
        cursor: auto;
    }
    .p-todoList__list {
        width: 100%;
        display: block;
        padding: 0;
        margin-top: 40px;
        box-shadow: 0 0 5px #e6e6e6;
    }
    .p-todoList__item {
        list-style: none;
       
        border-bottom: 2px solid #e6e6e6;
        padding: 15px 10px;
        position: relative;
    }
    .p-todoList__item-view {
        display: flex;
        gap: 10px;
    }
    .p-todoList__item.is-complete .p-todoList__item-msg{
        text-decoration: line-through;
    }
    .p-todoList__item.is-editing .p-todoList__item-view{
        opacity: 0;
        pointer-events: none;
        visibility: hidden;
    }
    .p-todoList__item-toggle {
        width: 26px;
        height: 26px;
        position: absolute;
        top: 50%;
        left: 20px;
        transform: translateY(-50%);
        opacity: 0;
        cursor: pointer;

    }
    .p-todoList__item-toggle:checked + .p-todoList__item-msg {
        background-image: url("data:image/svg+xml,%3Csvg viewBox='0 0 24 24' fill='none' xmlns='http://www.w3.org/2000/svg'%3E%3Cg id='SVGRepo_bgCarrier' stroke-width='0'%3E%3C/g%3E%3Cg id='SVGRepo_tracerCarrier' stroke-linecap='round' stroke-linejoin='round'%3E%3C/g%3E%3Cg id='SVGRepo_iconCarrier'%3E%3Cpath fill-rule='evenodd' clip-rule='evenodd' d='M3 12C3 7.02944 7.02944 3 12 3C16.9706 3 21 7.02944 21 12C21 16.9706 16.9706 21 12 21C7.02944 21 3 16.9706 3 12ZM12 1C5.92487 1 1 5.92487 1 12C1 18.0751 5.92487 23 12 23C18.0751 23 23 18.0751 23 12C23 5.92487 18.0751 1 12 1ZM16.8 8.6C17.1314 8.15817 17.0418 7.53137 16.6 7.2C16.1582 6.86863 15.5314 6.95817 15.2 7.4L9.89181 14.4776L7.70711 12.2929C7.31658 11.9024 6.68342 11.9024 6.29289 12.2929C5.90237 12.6834 5.90237 13.3166 6.29289 13.7071L9.29289 16.7071C9.49788 16.9121 9.78173 17.018 10.0709 16.9975C10.3601 16.9769 10.6261 16.8319 10.8 16.6L16.8 8.6Z' fill='%2321B721'%3E%3C/path%3E%3C/g%3E%3C/svg%3E");
    }
    .p-todoList__item-msg {
        font-weight: bold;
        font-size: 20px;
        margin-right: auto;
        padding-left: 50px;
        background-image: url("data:image/svg+xml,%3Csvg viewBox='0 0 24 24' fill='none' xmlns='http://www.w3.org/2000/svg'%3E%3Cg id='SVGRepo_bgCarrier' stroke-width='0'%3E%3C/g%3E%3Cg id='SVGRepo_tracerCarrier' stroke-linecap='round' stroke-linejoin='round'%3E%3C/g%3E%3Cg id='SVGRepo_iconCarrier'%3E%3Cpath d='M21 12C21 16.9706 16.9706 21 12 21C7.02944 21 3 16.9706 3 12C3 7.02944 7.02944 3 12 3C16.9706 3 21 7.02944 21 12Z' stroke='%2321B721' stroke-width='2' stroke-linecap='round' stroke-linejoin='round'%3E%3C/path%3E%3C/g%3E%3C/svg%3E");
        background-size:  26px 26px;
        background-position: 10px center;
        background-repeat: no-repeat;
    }
    .p-todoList__item-deleteBtn {
        appearance: none;
        background-color: #fff;
        border: none;
        cursor: pointer;
        flex-shrink: 0;
        font-size: 0;
        width: 30px;
        height: 30px;
        align-self: center;
        cursor: pointer;
        position: relative;
        background: #000;
        mask-image: url("data:image/svg+xml,%3Csvg viewBox='0 -0.5 25 25' fill='none' xmlns='http://www.w3.org/2000/svg'%3E%3Cg id='SVGRepo_bgCarrier' stroke-width='0'%3E%3C/g%3E%3Cg id='SVGRepo_tracerCarrier' stroke-linecap='round' stroke-linejoin='round'%3E%3C/g%3E%3Cg id='SVGRepo_iconCarrier'%3E%3Cpath d='M6.96967 16.4697C6.67678 16.7626 6.67678 17.2374 6.96967 17.5303C7.26256 17.8232 7.73744 17.8232 8.03033 17.5303L6.96967 16.4697ZM13.0303 12.5303C13.3232 12.2374 13.3232 11.7626 13.0303 11.4697C12.7374 11.1768 12.2626 11.1768 11.9697 11.4697L13.0303 12.5303ZM11.9697 11.4697C11.6768 11.7626 11.6768 12.2374 11.9697 12.5303C12.2626 12.8232 12.7374 12.8232 13.0303 12.5303L11.9697 11.4697ZM18.0303 7.53033C18.3232 7.23744 18.3232 6.76256 18.0303 6.46967C17.7374 6.17678 17.2626 6.17678 16.9697 6.46967L18.0303 7.53033ZM13.0303 11.4697C12.7374 11.1768 12.2626 11.1768 11.9697 11.4697C11.6768 11.7626 11.6768 12.2374 11.9697 12.5303L13.0303 11.4697ZM16.9697 17.5303C17.2626 17.8232 17.7374 17.8232 18.0303 17.5303C18.3232 17.2374 18.3232 16.7626 18.0303 16.4697L16.9697 17.5303ZM11.9697 12.5303C12.2626 12.8232 12.7374 12.8232 13.0303 12.5303C13.3232 12.2374 13.3232 11.7626 13.0303 11.4697L11.9697 12.5303ZM8.03033 6.46967C7.73744 6.17678 7.26256 6.17678 6.96967 6.46967C6.67678 6.76256 6.67678 7.23744 6.96967 7.53033L8.03033 6.46967ZM8.03033 17.5303L13.0303 12.5303L11.9697 11.4697L6.96967 16.4697L8.03033 17.5303ZM13.0303 12.5303L18.0303 7.53033L16.9697 6.46967L11.9697 11.4697L13.0303 12.5303ZM11.9697 12.5303L16.9697 17.5303L18.0303 16.4697L13.0303 11.4697L11.9697 12.5303ZM13.0303 11.4697L8.03033 6.46967L6.96967 7.53033L11.9697 12.5303L13.0303 11.4697Z' fill='%2321B721'%3E%3C/path%3E%3C/g%3E%3C/svg%3E");
        -webkit-mask-image: url("data:image/svg+xml,%3Csvg viewBox='0 -0.5 25 25' fill='none' xmlns='http://www.w3.org/2000/svg'%3E%3Cg id='SVGRepo_bgCarrier' stroke-width='0'%3E%3C/g%3E%3Cg id='SVGRepo_tracerCarrier' stroke-linecap='round' stroke-linejoin='round'%3E%3C/g%3E%3Cg id='SVGRepo_iconCarrier'%3E%3Cpath d='M6.96967 16.4697C6.67678 16.7626 6.67678 17.2374 6.96967 17.5303C7.26256 17.8232 7.73744 17.8232 8.03033 17.5303L6.96967 16.4697ZM13.0303 12.5303C13.3232 12.2374 13.3232 11.7626 13.0303 11.4697C12.7374 11.1768 12.2626 11.1768 11.9697 11.4697L13.0303 12.5303ZM11.9697 11.4697C11.6768 11.7626 11.6768 12.2374 11.9697 12.5303C12.2626 12.8232 12.7374 12.8232 13.0303 12.5303L11.9697 11.4697ZM18.0303 7.53033C18.3232 7.23744 18.3232 6.76256 18.0303 6.46967C17.7374 6.17678 17.2626 6.17678 16.9697 6.46967L18.0303 7.53033ZM13.0303 11.4697C12.7374 11.1768 12.2626 11.1768 11.9697 11.4697C11.6768 11.7626 11.6768 12.2374 11.9697 12.5303L13.0303 11.4697ZM16.9697 17.5303C17.2626 17.8232 17.7374 17.8232 18.0303 17.5303C18.3232 17.2374 18.3232 16.7626 18.0303 16.4697L16.9697 17.5303ZM11.9697 12.5303C12.2626 12.8232 12.7374 12.8232 13.0303 12.5303C13.3232 12.2374 13.3232 11.7626 13.0303 11.4697L11.9697 12.5303ZM8.03033 6.46967C7.73744 6.17678 7.26256 6.17678 6.96967 6.46967C6.67678 6.76256 6.67678 7.23744 6.96967 7.53033L8.03033 6.46967ZM8.03033 17.5303L13.0303 12.5303L11.9697 11.4697L6.96967 16.4697L8.03033 17.5303ZM13.0303 12.5303L18.0303 7.53033L16.9697 6.46967L11.9697 11.4697L13.0303 12.5303ZM11.9697 12.5303L16.9697 17.5303L18.0303 16.4697L13.0303 11.4697L11.9697 12.5303ZM13.0303 11.4697L8.03033 6.46967L6.96967 7.53033L11.9697 12.5303L13.0303 11.4697Z' fill='%2321B721'%3E%3C/path%3E%3C/g%3E%3C/svg%3E");
        mask-size: contain;
        -webkit-mask-size: contain;
        mask-repeat: no-repeat;
        -webkit-mask-repeat: no-repeat;
        mask-position: center;
        -webkit-mask-position: center;
    }
    .p-todoList__item-deleteBtn:hover {
        background: #d42140;
    }
    .p-todoList__item-editBtn {
        appearance: none;
        background-color: #fff;
        border: none;
        cursor: pointer;
        flex-shrink: 0;
        font-size: 0;
        width: 30px;
        height: 30px;
        align-self: center;
        cursor: pointer;
        position: relative;
        background: #000;
        mask-image: url("data:image/svg+xml,%3Csvg viewBox='0 0 24 24' fill='none' xmlns='http://www.w3.org/2000/svg'%3E%3Cg id='SVGRepo_bgCarrier' stroke-width='0'%3E%3C/g%3E%3Cg id='SVGRepo_tracerCarrier' stroke-linecap='round' stroke-linejoin='round'%3E%3C/g%3E%3Cg id='SVGRepo_iconCarrier'%3E%3Cpath fill-rule='evenodd' clip-rule='evenodd' d='M20.8477 1.87868C19.6761 0.707109 17.7766 0.707105 16.605 1.87868L2.44744 16.0363C2.02864 16.4551 1.74317 16.9885 1.62702 17.5692L1.03995 20.5046C0.760062 21.904 1.9939 23.1379 3.39334 22.858L6.32868 22.2709C6.90945 22.1548 7.44285 21.8693 7.86165 21.4505L22.0192 7.29289C23.1908 6.12132 23.1908 4.22183 22.0192 3.05025L20.8477 1.87868ZM18.0192 3.29289C18.4098 2.90237 19.0429 2.90237 19.4335 3.29289L20.605 4.46447C20.9956 4.85499 20.9956 5.48815 20.605 5.87868L17.9334 8.55027L15.3477 5.96448L18.0192 3.29289ZM13.9334 7.3787L3.86165 17.4505C3.72205 17.5901 3.6269 17.7679 3.58818 17.9615L3.00111 20.8968L5.93645 20.3097C6.13004 20.271 6.30784 20.1759 6.44744 20.0363L16.5192 9.96448L13.9334 7.3787Z' fill='%230F0F0F'%3E%3C/path%3E%3C/g%3E%3C/svg%3E");;
        -webkit-mask-image: url("data:image/svg+xml,%3Csvg viewBox='0 0 24 24' fill='none' xmlns='http://www.w3.org/2000/svg'%3E%3Cg id='SVGRepo_bgCarrier' stroke-width='0'%3E%3C/g%3E%3Cg id='SVGRepo_tracerCarrier' stroke-linecap='round' stroke-linejoin='round'%3E%3C/g%3E%3Cg id='SVGRepo_iconCarrier'%3E%3Cpath fill-rule='evenodd' clip-rule='evenodd' d='M20.8477 1.87868C19.6761 0.707109 17.7766 0.707105 16.605 1.87868L2.44744 16.0363C2.02864 16.4551 1.74317 16.9885 1.62702 17.5692L1.03995 20.5046C0.760062 21.904 1.9939 23.1379 3.39334 22.858L6.32868 22.2709C6.90945 22.1548 7.44285 21.8693 7.86165 21.4505L22.0192 7.29289C23.1908 6.12132 23.1908 4.22183 22.0192 3.05025L20.8477 1.87868ZM18.0192 3.29289C18.4098 2.90237 19.0429 2.90237 19.4335 3.29289L20.605 4.46447C20.9956 4.85499 20.9956 5.48815 20.605 5.87868L17.9334 8.55027L15.3477 5.96448L18.0192 3.29289ZM13.9334 7.3787L3.86165 17.4505C3.72205 17.5901 3.6269 17.7679 3.58818 17.9615L3.00111 20.8968L5.93645 20.3097C6.13004 20.271 6.30784 20.1759 6.44744 20.0363L16.5192 9.96448L13.9334 7.3787Z' fill='%230F0F0F'%3E%3C/path%3E%3C/g%3E%3C/svg%3E");;
        mask-size: 18px auto;
        -webkit-mask-size: 18px auto;
        mask-repeat: no-repeat;
        -webkit-mask-repeat: no-repeat;
        mask-position: center;
        -webkit-mask-position: center;
    }
    .p-todoList__item-editBtn:hover {
        background: #d42140;
    }

    .p-todoList__item-edit {
        position: absolute;
        left: 50px;
        top: 0;
        bottom: 0;
        right: 0;
    }
    .p-todoList__item-editInput {
        display: block;
        width: 100%;
        height: 100%;
        box-shadow: 0 0 3px #d22945;
        border: none;
        outline: none;
        padding: 10px;
        font-size: 20px;
        font-weight: bold;
        font-family: inherit;
    }
</style>