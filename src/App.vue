  <template>
    <div class="container">
      
      <h2>To-Do List</h2>
      <input 
        class="form-control"
        type="text" 
        v-model="searchText"
        placeholder="Search">
        <hr/>
      <TodoSimpleForm @add-todo="addTodo"/>
      <div style="red">{{error}}</div>
      <div v-if="!filteredTodos.length">
      There is nothing to display
    </div>
    <TodoList 
    :todos="filteredTodos" 
    @toggle-todo="toggleTodo"
    @delete-todo="deleteTodo"/>
    
    <hr/>
    <nav aria-label="Page navigation example">
    <ul class="pagination">
    <li v-if="currentPage !== 1"  class="page-item">
      <a style="cursor: pointer" class="page-link" @click="getTodos(currentPage -1)">
        Previous
        </a>
        </li>
    <li 
    v-for="page in numberOfPages"
    :key="page"
    class="page-item"
    :class="currentPage === page ? 'active' : ''">
    
      <a style="cursor: pointer" class="page-link" @click="getTodos(page)">{{page}}</a>
      </li>
    <li v-if="numberOfPage !== currentPage" class="page-item">
      <a style="cursor: pointer" class="page-link" @click="getTodos(currentPage +1)">Next</a>
      </li>
  </ul>
</nav>
</div>
  </template>

<script>
  import {ref, computed } from 'vue';
  import TodoSimpleForm from './components/TodoSimpleForm.vue';
  import TodoList from './components/TodoList.vue';
  import axios from 'axios';

  export default {
    components: {
    // eslint-disable-next-line vue/no-unused-components
    TodoSimpleForm,
    // eslint-disable-next-line vue/no-unused-components
    TodoList,
  
},
    setup() {
      
      const todos = ref([]);
      const error = ref('')
      const numberOfTodos = ref(0)
      const limit = 5
      const currentPage = ref(1)
      
      const numberOfPages = computed(() => {
        return Math.ceil(numberOfTodos.value/limit)
      })

      const getTodos = async (page = currentPage.value) => {
        currentPage.value = page
        try {
        const res = await axios.get(
          `http://localhost:3000/todos?_page=${page}&_limit=${limit}`)
          numberOfTodos.value = res.headers['x-total-count']
        todos.value = res.data
      }catch (err) {
        console.log(err)
        error.value = 'Somthing went wrong'
        }
      }
      
      getTodos()
      
      const todoStyle = {
        textDecoration: 'line-through',
        color: 'gray'
      }
  
      const addTodo = async (todo) => {
        //?????????????????? ????????? ??????
        error.value='';
        try {
        const res = await axios.post('http://localhost:3000/todos', {
          subject: todo.subject,
          completed: todo.completed,
        })
        todos.value.push(res.data)
      }catch (err) {
        console.log(err);
        error.value = 'Somthing went wrong'
      }
      
  }

  //const greeting = (name) => { 
   // return 'Hello, ' + name;
  // };
  
  // const greet = greeting(name);
  const toggleTodo = async (index) => {
    error.value = ''
    const id = todos.value[index].id
    try{
      await axios.patch('http://localhost:3000/todos/' + id, {
        completed: !todos.value[index].completed
      })

      todos.value[index].completed = !todos.value[index].completed;
    }catch(err) {
      console.log(err)
      error.value = "Somthing went wrong"
    }
  };

  const deleteTodo = async (index) => {
      error.value = ''
      const id = todos.value[index].id
      try {
      await axios.delete('http://localhost:3000/todos/' + id)
      
      todos.value.splice(index,1)
    }catch (err) {
      console.log(err)
      error.value = "Somthing went wrong"
    }
  }

  const searchText = ref('');
  const filteredTodos = computed(() => {
  if(searchText.value) {
  return todos.value.filter(todo => {
      return todo.subject.includes(searchText.value);
  });
  }

  return todos.value;
  })

  return {
    todos,
    addTodo,
    todoStyle,
    deleteTodo,
    toggleTodo,
    searchText,
    filteredTodos,
    error,
    numberOfPages,
    currentPage,
    getTodos,
  };
}
  }
</script>

<style>
.todo {
  color : gray;
  text-decoration: line-through ;
}
</style>