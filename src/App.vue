<template>
  <div class="container">
    <Header @toggle-add-task="toggleAddTask" title="Task Tracker" :showAddTask="showAddTask" />
    <div v-show="showAddTask">
      <AddTask @add-task="addTask" />
    </div>
    <Tasks @toggle-reminder="toggleReminder" @delete-task="deleteTask" :tasks="tasks" />
    <Footer />
  </div>
</template>

<script>
import Header from '@/components/Header'
import AddTask from '@/components/AddTask'
import Tasks from '@/components/Tasks'
import Footer from '@/components/Footer'


export default {
  name: 'App',
  components: {
    Header,
    AddTask,
    Tasks,
    Footer,
  },
  data() {
    return {
      tasks: [],
      showAddTask: false,
    }
  },
  /*
    methods: {
      deleteTask(id) {
        if(confirm('Are you sure?')) {
          this.tasks = this.tasks.filter((task) => task.id !== id)
        }
      }
    }
  */
  methods: {
    toggleAddTask (){
      this.showAddTask = !this.showAddTask
    },
    async addTask(task) { // task coming from AddTask.vue -> newTask from $emit
        Swal.fire({
          position: 'top-end',
          icon: 'success',
          title: 'Your Task has been Added',
          showConfirmButton: false,
          timer: 1250
        })

        const res = await fetch('api/tasks', {
          method: 'POST',
          headers: {
            'Content-type': 'application/json',
          },
            body: JSON.stringify(task)
        })

        const data = await res.json()

      this.tasks = [...this.tasks, data]
    },
    async deleteTask(id){
      if (confirm('Are You Sure?')){
        const res = await fetch(`api/tasks/${id}`, {
              method: 'DELETE'
            })

            // .filter((task)) <= task: for each task
            res.status === 200 ? (this.tasks = this.tasks.filter((task) => task.id !== id)) : alert('Error deleting task')
      }
    },
    async toggleReminder(id) {
      const taskToToggle = await this.fetchTask(id)
      const updateTask = {...taskToToggle, reminder: !taskToToggle.reminder}

      const res = await fetch(`api/tasks/${id}`, {
        method: 'PUT', // PUT for Update
        headers: {
          'Content-type': 'application/json'
        },
        body: JSON.stringify(updateTask)
      })

      const data = await res.json()

      this.tasks = this.tasks.map((task) => task.id === id ? { ...task, reminder: data.reminder} : task)
    },
    async fetchTasks() {
      const res = await fetch('api/tasks') // api -> file/vue.config.js => http://localhost:5000

      const data = await res.json()

      return data
    },
    async fetchTask(id) {
      const res = await fetch(`api/tasks/${id}`) // api -> file/vue.config.js => http://localhost:5000

      const data = await res.json()

      return data
    }
  },
  async created() {
    this.tasks = await this.fetchTasks()
  }
}
</script>

<style>
@import url('https://fonts.googleapis.com/css2?family=Poppins:wght@300;400&display=swap');
* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}
body {
  font-family: 'Poppins', sans-serif;
}
.container {
  max-width: 500px;
  margin: 30px auto;
  overflow: auto;
  min-height: 300px;
  border: 3px solid rgb(200, 200, 200);
  padding: 30px;
  border-radius: 5px;
}
.btn {
  display: inline-block;
  background: green;
  color: #fff;
  border: none;
  padding: 10px 15px;
  margin: 5px;
  border-radius: 5px;
  cursor: pointer;
  text-decoration: none;
  font-size: 15px;
  font-family: inherit;
}
.btn:focus {
  outline: none;
}
.btn:active {
  transform: scale(0.98);
}
.btn-block {
  display: block;
  width: 100%;
}
</style>
