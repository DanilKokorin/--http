<template>
  <div class="container">
    <app-alert :alert="alert" @close="alert = null"></app-alert>

    <form class="card" @submit.prevent="createPerson"> 
      <h2>Работа с базой данных</h2>

      <div class="form-control">
        <label for="name">Введите имя</label>
        <input type="text" id="name" v-model.trim="name">
      </div>

      <button class="btn prymary" :disabled="name.length === 0">Создать человека</button>
    </form>

    <app-loader v-if="loading"></app-loader>

    <app-people-list
      v-else
      :people="people"
      @load="loadPeople"
      @remove="removePerson"
    ></app-people-list>
  </div>
</template>

<script>
import AppPeopleList from './AppPeopleList.vue'
import AppAlert from './AppAlert.vue'
import AppLoader from './AppLoader.vue'
import axios from 'axios'

export default {
  data () {
    return {
      name: '',
      people: [],
      alert: null,
      loading: false
    }
  },
  mounted () {
    this.loadPeople()
  },
  methods: {
    async createPerson () {
      const responce = await fetch( 'https://vue-with-http-7b807-default-rtdb.firebaseio.com/people.json', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify({
          firstName: this.name 
        })
      })

      const firebaseData = await responce.json()

      this.people.push({
        firstName: this.name,
        id: firebaseData.name
      })
      this.name = ''
    },
    async loadPeople () {
      try {
        this.loading = true
        const {data} = await axios.get('https://vue-with-http-7b807-default-rtdb.firebaseio.com/people.json')
        if (!data) {
          throw new Error('Список людей пуст')
        }
        // setTimeout( () => {
          this.people = Object.keys(data).map(key => {
            return {
              id: key,
              ...data[key]
            }
          }) 
        // }, 1500)
        this.loading = false
      } catch (e) {
        this.alert = {
          type: 'danger',
          title: 'Ошибка!',
          text: e.message
        }
        this.loading = false
      }
    },
    async removePerson (id) {
      try {
        const name = this.people.find(person => person.id === id).firstName
        await axios.delete(`https://vue-with-http-7b807-default-rtdb.firebaseio.com/people/${id}.json`)
        this.people = this.people.filter(person => person.id !== id)
        this.alert = {
          type: 'primary',
          title: 'Успешно!',
          text: `Пользователь с именем "${name}" удален`
        }
      } catch (e) {

      }
    }
  },
  components: {AppPeopleList, AppAlert, AppLoader}
}
</script>

<style>

</style>
