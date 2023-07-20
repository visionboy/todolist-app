<template>
    <div class="container">
        <Header />
        <router-view />
        <Loading v-if="states.isLoading"/>
    </div>
</template>

<script setup>
import { reactive, computed, provide } from 'vue'
import Header from '@/components/Header.vue'
import Loading from './components/Loading.vue'
import axios from 'axios';

const owner = "gdhong";
const BASEURI = "/api/react/";
const states = reactive({ todoList:[], isLoading:false })

//TodoList 목록을 조회합니다.
const fetchTodoList = async () => {
  states.isLoading = true;
  try {
    const response = await axios.get(BASEURI+"list");
    if (response.status === 200) {
        states.todoList = response.data;
        console.log(response.data);
    } else {
        alert('데이터 조회 실패');
    }
  } catch(error) {
    alert('에러발생 :' + error);
  }
  states.isLoading = false;

}

const addTodo = async ({ todo, desc }, successCallback) => {
  states.isLoading = true;
  try {
    const payload = {
      id:"ID_"+(states.todoList.length+1),
      title: todo,
      todo: todo, 
      desc: desc};
    console.log(payload);
    console.log(states.todoList.length);
    const response = await axios.post('/api/react/add',payload);
    console.log(response);

    if (response.data.status ==="o") {
      states.todoList.push({id: payload.id, todo, desc, done:false});
    } else {
      alert("Todo 추가 실패:" + response.data.message);
    }
  } catch(error) {
    alert('에러발생 :' + error);
  }
  states.isLoading = false;
  
};

const updateTodo = async ({ id, todo, desc, done, no }) => {
  states.isLoading = true;
  try {
    const payload = {
      id:id,
      todo: todo, 
      desc: desc,
      no : no,
      done:done
    };
    console.log(payload);

    const response = await axios.put('/api/react/update',payload);
    console.log(response);

    if (response.data.status ==="o") {
      let index = states.todoList.findIndex((todo) => todo.id === id);
      states.todoList[index] = { ...states.todoList[index], todo, desc, done };
    } else {
      alert("Todo 추가 실패:" + response.data.message);
    }
  } catch(error) {
    alert('에러발생 :' + error);
  }
  states.isLoading = false;
};

const deleteTodo = async (id) => {
  states.isLoading = true;
  if (window.confirm("삭제하시겠습니까?")) {
    try {
      const payload = {
        data:{
          id:id,
          title: "a"
        }
      };
      console.log(payload);
      const response = await axios.delete('/api/react/del',payload);
      console.log(response);

      if (response.data.status ==="o") {
        let index = states.todoList.findIndex((todo) => todo.id === id);
        states.todoList.splice(index, 1);
      } else {
        alert("Todo 추가 실패:" + response.data.message);
      }
    } catch(error) {
      alert('에러발생 :' + error);
    }
  }
  states.isLoading = false;
}

const toggleDone = (id) => {
  let index = states.todoList.findIndex((todo) => todo.id === id);
  states.todoList[index].done = !states.todoList[index].done;
}

provide('todoList', computed(()=>states.todoList))
provide('actions', { addTodo, deleteTodo, toggleDone, updateTodo,fetchTodoList })
fetchTodoList();
</script>
