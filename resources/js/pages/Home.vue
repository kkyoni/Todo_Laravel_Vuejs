<template>
    <div class="w-6/12 p-10 mx-auto">
        <div class="flex justify-between">
            <h1 class="text-2xl"> Todo </h1>
            <span class="capitalize">Welcome {{ user && user.name }}, <button
                class="text-orange-500 underline hover:no-underline rounded-md"
                @click="handleLogout">Logout</button></span>
        </div>
        <input type="text" class="p-2 w-64 border rounded-md" placeholder="Search Todo" v-model="searchQuery"/>
        <button class="bg-blue-600 text-white px-5 py-2 rounded-md ml-2 hover:bg-blue-400" @click="openModal" style="float: right;">Add Todo</button>
        <Loader v-if="isLoading"/>
        <ul class="border-t mt-3 cursor-pointer">
            <li :class="`py-3 border-b text-gray-600 ${val.has_completed ? 'line-through' : ''}`"
                v-for="(val, idx) in filteredTodos" :key="idx">
                <input type="checkbox" :checked="val.has_completed" @click="checked(idx)"/>
                <span @click="checked(val, idx)" class="pl-3">{{ val.title }} </span>
                <button class="float-right bg-red-400 px-2 text-white font-bold rounded-md hover:bg-red-600"
                        @click="deleteTodo(val, idx)">&times;
                </button>
            </li>
        </ul>


        <!-- Popup/Modal -->
        <div v-if="isModalOpen" class="fixed top-0 left-0 w-full h-full flex items-center justify-center bg-gray-800 bg-opacity-50">
            <div class="bg-white p-8 rounded-md shadow-md relative">
                <h2 class="text-2xl mb-4">Add Todo</h2>
                <input type="text" class="p-2 w-64 border rounded-md" v-model="todo" placeholder="Enter your todo"/>
                <button class="bg-blue-600 text-white px-5 py-2 rounded-md ml-2 hover:bg-blue-400 mr-4" @click="addTodo">Add</button>
                <!-- Close button -->
                <button @click="closeModal" class="absolute top-2 right-2 text-gray-600 hover:text-gray-800 focus:outline-none">
                    <svg class="h-6 w-6" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
                        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12"></path>
                    </svg>
                </button>
            </div>
        </div>

    </div>
</template>
<script>
import { ref, onMounted,computed } from 'vue'
import { useRouter } from "vue-router";
import { request } from '../helper'
import Loader from '../components/Loader.vue';

export default {
    components: {
        Loader,
    },
    setup() {
        const todo = ref('')
        const todos = ref([])
        const user = ref()
        const isLoading = ref()
        const isModalOpen = ref(false); // Boolean ref for controlling popup/modal visibility
        const searchQuery = ref('');

        let router = useRouter();
        onMounted(() => {
            authentication()
            handleTodos()
        });

        const authentication = async () => {
            isLoading.value = true
            try {
                const req = await request('get', '/api/user')
                user.value = req.data
            } catch (e) {
                await router.push('/')
            }
        }

        const handleTodos = async () => {
            try {
                const req = await request('get', '/api/todos')
                todos.value = req.data.data
            } catch (e) {
                await router.push('/')
            }
            isLoading.value = false
        }

        const handleNewTodo = async (title) => {
            try {
                const data = { title: title }
                const req = await request('post', '/api/todos', data)
                if (req.data.message) {
                    isLoading.value = false
                    return alert(req.data.message)
                }
                todos.value.push(req.data.data)
            } catch (e) {
                await router.push('/')
            }
            isLoading.value = false
        }

        const handleLogout = () => {
            localStorage.removeItem('APP_DEMO_USER_TOKEN')
            router.push('/')
        }

        const addTodo = () => {
            if (todo.value === "") {
                return alert("Todo cannot be empty");
            }
            isLoading.value = true
            isModalOpen.value = false
            handleNewTodo(todo.value)
            todo.value = ""
        }

        const checked = async (val, index) => {
            try {
                const data = { has_completed: !val.has_completed }
                const req = await request('put', `/api/todos/${val.id}`, data)
                if (req.data.message) {
                    isLoading.value = false
                    return alert(req.data.message)
                }
                todos.value[index].has_completed = !val.has_completed
            } catch (e) {
                await router.push('/')
            }
            isLoading.value = false
        }

        const deleteTodo = async (val, index) => {
            if (window.confirm("Are you sure")) {
                try {
                    const req = await request('delete', `/api/todos/${val.id}`)
                    if (req.data.message) {
                        isLoading.value = false
                        todos.value.splice(index, 1)
                    }
                } catch (e) {
                    await router.push('/')
                }
                isLoading.value = false
            }
        }

        // Method to open the modal
        const openModal = () => {
            isModalOpen.value = true;
        }

        // Method to close the modal
        const closeModal = () => {
            isModalOpen.value = false;
        }

        // Computed property to filter todos based on the search query
        const filteredTodos = computed(() => {
            return todos.value.filter(todo => {
                return todo.title.toLowerCase().includes(searchQuery.value.toLowerCase());
            });
        });

        return {
            todo,
            todos,
            user,
            checked,
            addTodo,
            isLoading,
            deleteTodo,
            handleLogout,
            isModalOpen,
            openModal, // Expose the method to open the modal
            closeModal, // Expose the method to close the modal
            searchQuery,
            filteredTodos // Expose the filtered todos
        }
    },
}
</script>
