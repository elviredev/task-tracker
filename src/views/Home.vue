<template>
  <AddTask
      v-show="showAddTask"
      @add-task="addTask"
  />
  <Tasks
      @toggle-reminder="toggleReminder"
      @delete-task="deleteTask"
      :tasks="tasks"
  />
</template>

<script>
  import AddTask from "@/components/AddTask.vue";
  import Tasks from "@/components/Tasks.vue";

  export default {
    props: {
      showAddTask: Boolean
    },
    name: 'Home',
    components: {
      Tasks,
      AddTask
    },
    data() {
      return {
        tasks: [],
      }
    },
    methods: {
      async addTask(task) {
        // requête http pour envoyer les données et les persister dans db.json
        const res = await fetch('api/tasks', {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json',
          },
          body: JSON.stringify(task)
        })
        // récupère la nouvelle tâche en retour
        const data = await res.json()

        // dans l'array de tasks (this.tasks), on se répartit sur les tasks en cours et on ajoute une nouvelle tâche
        this.tasks = [...this.tasks, data]
      },
      async deleteTask(id) {
        // console.log('task', id)
        // requête http à notre api pour persister la suppression dans db.json
        if (confirm('Are you sure ?')) {
          const res = await fetch(`api/tasks/${id}`, {
            method: 'DELETE'
          })
          // si le serveur est OK (200)
          // on veut filtrer sur les tasks qui ne sont pas à supprimer donc on n'aura pas la task avec l'id passé en param car on veut la supprimer
          res.status === 200 ?
              (this.tasks = this.tasks.filter((task) => task.id !== id))
              : alert('Error deleting task')
        }
      },
      async toggleReminder(id) {
        // récupère la tâche pour le rappel toggle
        const taskToToggle = await this.fetchTask(id)
        // on met la tâche à modifier et la modification à faire dans une variable
        const updateTask = {...taskToToggle, reminder: !taskToToggle.reminder}
        // requête http
        const res = await fetch(`api/tasks/${id}`, {
          method: 'PUT',
          headers: {
            'Content-Type' : 'application/json',
          },
          body: JSON.stringify(updateTask)
        })
        // récupérer les données
        const data = await res.json()
        // map retourne un array des tâches mises à jour
        // si la task de l'array (task.id) est = à celle qui a l'id en param (double cliquée) alors on retourne la task courante (...task) et l'état modifié de la propriété "reminder" sinon on retourne la task initiale
        this.tasks = this.tasks.map((task) =>
            task.id === id ? {...task, reminder: data.reminder} : task)
      },
      /* Backend db.json (json-server) */
      // Récupérer toutes les taches
      async fetchTasks() {
        const res = await fetch('api/tasks')
        const data = await res.json()
        return data
      },
      // Récupérer une tache
      async fetchTask(id) {
        const res = await fetch(`api/tasks/${id}`)
        const data = await res.json()
        return data
      }
    },
    async created() {
      this.tasks = await this.fetchTasks()
    }
  }
</script>

<style scoped>

</style>