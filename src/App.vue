<template>
  <div class="container">
    <h2>To-Do List</h2>
    <input class="form-control" type="text" v-model="searchText" placeholder="Search">
    <hr>
    <TodoSimpleForm @add-todo="addTodo" />
    <div style="color:red">{{error}}</div>
      
      <div v-if="!todos.length">
        추가된 todo가 없습니다.
      </div>
      <TodoList :todos="filteredTodos" @toggle-todo="toggleTodo" @delete-todo="deleteTodo"/>
      
  </div>
</template>

<script>
import {ref, computed} from 'vue';
import TodoSimpleForm from './components/TodoSimpleForm.vue';
import TodoList from './components/TodoList.vue';
import axios from 'axios';

export default {
  components: {
    TodoSimpleForm,
    TodoList,
  },
  setup() {
   
    const todos = ref([]);
    const error = ref('');
    
    const getTodos = async () => {
      try{
        const res = await axios.get('http://localhost:3000/todos');
        todos.value = res.data;
      } catch(err){
        console.log(err);
        error.value = "something is wrong";
      }
    };

    getTodos();

    const addTodo = async (todo) => {
      error.value=''
      try{
        const res = await axios.post('http://localhost:3000/todos',{
        subject: todo.subject,
        completed: todo.completed,
        });
        todos.value.push(res.data);
      }catch(err){
        console.log(err);
        error.value = "something is wrong";
      }
      
      // .then(res => {
      //   console.log(res);
      //   todos.value.push(todo);
      // }).catch(err => {
      //   console.log(err);
      //   error.value = "something is wrong";
      // });
    }   

    const deleteTodo = async (index) => {
      error.value= '';
      const id = todos.value[index].id;
      try{
        await axios.delete('http://localhost:3000/todos/'+id);
        todos.value.splice(index, 1);
      }catch(err){
        console.log(err);
        error.value = "something is wrong";
      }
      
    }

    const toggleTodo = async (index) => {
      const id = todos.value[index].id;
      try{
        await axios.patch('http://localhost:3000/todos/'+id,{
          completed: !todos.value[index].completed
        });
        todos.value[index].completed = !todos.value[index].completed;
      }catch(err){
        console.log(err);
        error.value = "something is wrong";
      }
      
    }

    const searchText = ref('');
    const filteredTodos = computed(() => {
      if(searchText.value){
        return todos.value.filter(todo => {
          return todo.subject.includes(searchText.value);
        });
      }

      return todos.value;
    });

    return{
      addTodo,
      todos,
      deleteTodo,
      toggleTodo,
      searchText,
      filteredTodos,
      error,
    };
  }
}
</script>

<style>
  .todo{
    color: gray;
    text-decoration: line-through;
  }
</style>
