<template>
  <div class="todo">
    <div v-if="token">
      <div class="form-group" style="width: 40%; margin: 30px auto; padding: 13px; border-radius: 10px; background-color: white;">
        <div class="form-group">
          <label for="todo">Todo:</label>
          <input class="form-control" v-model="todo" type="text" placeholder="something to do">
        </div>
        <div class="form-group">
          <label for="date">Estimated Time:</label>
          <input class="form-control" id="date" v-model="estimated_time" type="date">
        </div>
        <div class="form-group">
          <button class="btn" style="background-color: pink; color: white;" @click="addTodo()">Add Todo</button>
        </div>
      </div>
    </div>
    <div class="container-fluid" v-if="list.length !== 0">
      <div class="row" style="margin-left: 150px;">
        <div class="card col col-sm-2 mr-2 mb-2" v-for="(todo, index) in list" :key="index" style="width: 18rem;" align="center">
          <div class="card-header">
            <div>{{ todo.estimated_time }}</div>
          </div>
          <div class="card-body">
            <div v-if="todo.status === 'uncomplete'">
              <div class="card-text">{{ todo.todo }}</div>
            </div>
            <div v-else>
              <strike class="card-text">{{ todo.todo }}</strike>
            </div>
          </div>
          <div class="card-footer">
            <a @click="deleteTodo(todo._id)"><i class="far fa-window-close" style="cursor: pointer; color: red;"></i></a>
            <a @click="completeTodo(todo._id)" v-if="todo.status === 'uncomplete'"><i class="far fa-check-square" style="cursor: pointer; color: green;"></i></a>
            <a @click="uncompleteTodo(todo._id)" v-else><i class="far fa-check-square" style="cursor: pointer; color: green;"></i></a>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
  import axios from 'axios'
  import swal from 'sweetalert2'

  export default {
    data () {
      return {
        token: localStorage.getItem('token'),
        list: [],
        todo: '',
        estimated_time: ''
      }      
    },

    methods: {
      addTodo () {
        let self = this

        axios
          .post('https://safe-plains-12311.herokuapp.com/todos', {
            user: localStorage.getItem('id'),
            todo: self.todo,
            estimated_time: self.estimated_time
          }, {
            headers: {
              token: localStorage.getItem('token')
            }
          })
          .then(response => {
            let date = response.data.todo.estimated_time.split('T')

            response.data.todo.estimated_time = date[0]

            self.list.push(response.data.todo)
          })
          .catch(err => {
            console.log(err)
          })
      },

      completeTodo (id) {
        let self = this

        axios
          .put(`https://safe-plains-12311.herokuapp.com/todos/update/${id}`, {
            status: 'complete'
          }, {
            headers: {
              user: localStorage.getItem('id'),
              token: localStorage.getItem('token')
            }
          })
          .then(response => {
            let index

            self.list.forEach((todo, i) => {
              if (todo._id === response.data.todoUpdated._id) {
                todo.status = 'complete'
              }
            })

            swal({
              position: 'center',
              type: 'success',
              title: 'You have complete your task',
              showConfirmButton: false,
              timer: 1500
            })
          })
          .catch(err => {
            console.log(err)
          })
      },

      uncompleteTodo (id) {
        let self = this
        
        axios
          .put(`https://safe-plains-12311.herokuapp.com/todos/update/${id}`, {
            status: 'uncomplete'
          }, {
            headers: {
              user: localStorage.getItem('id'),
              token: localStorage.getItem('token')
            }
          })
          .then(response => {
            let index

            self.list.forEach((todo, i) => {
              if (todo._id === response.data.todoUpdated._id) {
                todo.status = 'uncomplete'
              }
            })

            swal({
              position: 'center',
              type: 'info',
              title: 'You have uncomplete your task',
              showConfirmButton: false,
              timer: 1500
            })
          })
          .catch(err => {
            console.log(err)
          })
      },

      deleteTodo (id) {
        let self = this
        const swalWithBootstrapButtons = swal.mixin({
          confirmButtonClass: 'btn btn-success',
          cancelButtonClass: 'btn btn-danger',
          buttonsStyling: false,
        })

        swalWithBootstrapButtons({
          title: 'Are you sure?',
          text: "You won't be able to revert this!",
          type: 'warning',
          showCancelButton: true,
          confirmButtonText: 'Yes, delete it!',
          cancelButtonText: 'No, cancel!',
          reverseButtons: true
        }).then((result) => {
          if (result.value) {
            axios
              .delete(`https://safe-plains-12311.herokuapp.com/todos/delete/${id}`, {
                headers: {
                  user: localStorage.getItem('id'),
                  token: localStorage.getItem('token')
                }
              })
              .then(response => {
                let index

                if (response.data.deletedTodo) {
                  self.list.forEach((todo, i) => {
                    if (todo._id === response.data.deletedTodo._id) {
                      console.log(i)
                      index = i
                    }
                  })

                  self.list.splice(index, 1)
                }
              })
              .catch(err => {
                console.log(err)
              })
              swalWithBootstrapButtons(
                'Deleted!',
                'Your file has been deleted.',
                'success'
              )
          } else if (
            result.dismiss === swal.DismissReason.cancel
          ) {
            swalWithBootstrapButtons(
              'Cancelled',
              'Your imaginary file is safe :)',
              'error'
            )
          }
        })
      }
    },

    created () {
      let self = this

      axios
        .get('https://safe-plains-12311.herokuapp.com/todos', {
          headers: {
            token: localStorage.getItem('token'),
            user: localStorage.getItem('id')
          }
        })
        .then(response => {
          let todoList = []

          response.data.todos.forEach(todo => {
            let date = todo.estimated_time.split('T')

            todo.estimated_time = date[0]

            todoList.push(todo)
          })

          self.list = todoList
        })
        .catch(err => {
          console.log(err)
        })
    }
  }
</script>