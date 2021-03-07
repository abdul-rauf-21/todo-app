<template>
  <div id="app-wrapper">
    <div id="app-header">
      <input 
        type="text"
        v-model="todo"
        @keyup.enter="addTodo"
      >
      <button @click="addTodo">+</button>
    </div>
    <div 
      id="app-todo-list"
      v-if="todos.length"
    >
      <div 
        class="app-todo"
        v-for="(todo, index) in todos"
        :key="index" 
      >
        <p :ref="el => { if (el) todoText[index] = el }">{{ index + 1 }}. {{ todo.text }}</p>
        <div class="action">
          <button 
            class="check"
            @click="checkTodo(index)"
          >
          ✔
          </button>
          <button 
            class="delete" 
            @click="deleteTodo(todo.id)"
          >
          x
          </button>
        </div>
      </div>
    </div>
    <div
      id="empty-todo" 
      v-else>
      <p>Empty</p>
    </div>
  </div>
</template>

<script>
import { ref, onMounted } from 'vue'
import Localbase from 'localbase'

export default {
  setup () {
    let db;
    let todo = ref('')
    let todos = ref([])
    let todoText = ref([])

    const generateRandomCombination = () => {
      const combination = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 'a', 'b', 'c', 'd', 'e', 'f', 'g', 'h', 'i', 'j', 'k', 'l', 'm', 'n', 'o', 'p', 'q', 'r', 's', 't', 'u', 'v', 'w', 'x', 'y', 'z']
      let key = ''

      Array(10).fill().forEach(_ => {
        key += combination[Math.floor(Math.random() * (combination.length - 1))]
      })

      return key
    }

    const addTodo = async () => {
      if ((/\w/).test(todo.value)) {
        await db.collection('todos').add({ 
          id: generateRandomCombination(),
          text: todo.value 
        })
        await db.collection('todos').get().then( data => {
          todos.value = data
          todo.value = ''
        })
      } 
    }

    const deleteTodo = async targetId => {
      await db.collection('todos').doc({ id: targetId }).delete()
      await db.collection('todos').get().then( data => {
        todos.value = data
      })
    }

    const checkTodo = index => {
      if (todoText.value[index].style.textDecoration !== 'line-through') {
        todoText.value[index].style.textDecoration = 'line-through'
      } else {
        todoText.value[index].style.textDecoration = 'none'
      }
      todoText.value[index].nextElementSibling.firstChild.classList.toggle('check-done')
    }

    onMounted(() => {
      db = new Localbase('db')
      db.config.debug = false
      db.collection('todos').get()
        .then( data => {
          if (data) {
            todos.value = data
          }
        })
    })

    return {
      todo,
      todos,
      todoText,
      addTodo,
      deleteTodo,
      checkTodo
    }
  }
}
</script>

<style lang="scss">

$red: #e74c3c;
$green: #3BD16F;
$root-bg: #000000;
$gray: #1F1F1F;

body {
  font-family: verdana, arial;
  display: flex;
  flex-direction: row;
  justify-content: center;
  align-items: center;
  background-color: $root-bg;
}

#app-wrapper {
  width: 60vw;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}

#app-header {
  width: 100%;
  padding: 1rem;
  display: flex;
  flex-direction: row;
  justify-content: center;

  & > * {
    margin: 1rem;
  }

  input {
    width: 75%;
    padding: 0.5rem 1rem;
    font-family: inherit;
    border: none;
    border-radius: 10px;
    outline: none;
    background-color: $gray;
    color: white;
  }

  & > button {
    padding: 0.5rem 0.8rem;
    font-size: 1rem;
    font-family: inherit;
    font-weight: 900;
    border: none;
    border-radius: 5px;
    color: $root-bg;
    background-color: $green;
    cursor: pointer;
    outline: none;

    &:hover {
      background-color: darken($green, 5%);
    }
  }
}

#app-todo-list {
  width: 100%;
  padding: 1rem;
  text-align: center;
  color: white;
  border-radius: 5px;
  background-color: $gray;
  box-sizing: border-box;

  .app-todo {
    display: flex;
    flex-direction: row;
    justify-content: space-between;
    align-items: center;
    width: 100%;
    padding: 1rem 0;

    button {
      width: 30px;
      height: 30px;
      margin: 0 1rem;
      padding: 0.25rem;
      font-size: 0.8rem;
      font-family: inherit;
      font-weight: 900;
      border: none;
      border-radius: 50%;
      background-color: $root-bg;
      cursor: pointer;
      outline: none;
    }

    .check {
      @extend button;
      color: $green;

      &:hover {
        background-color: darken($green, 30%);
      }
    }

    .check-done {
      color: $gray;
    }

    .delete {
      @extend button;
      color: $red;

      &:hover {
        background-color: darken($red, 30%);
      }
    }
  }
}

#empty-todo {
  @extend #app-todo-list;
}

@media only screen and (max-width: 768px) {
  #app-wrapper {
    width: 90vw;
  }
}

</style>
