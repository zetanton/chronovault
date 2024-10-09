<template>
  <v-app>
    <v-app-bar color="primary" dark app>
      <v-app-bar-nav-icon v-if="isLoggedIn" @click="drawer = !drawer"></v-app-bar-nav-icon>
      <v-app-bar-title>ChronoVault</v-app-bar-title>
      <v-spacer></v-spacer>
      <v-btn v-if="isLoggedIn" icon @click="toggleView">
        <v-icon>{{ isTableView ? 'mdi-view-grid' : 'mdi-table' }}</v-icon>
      </v-btn>
      <v-btn v-if="isLoggedIn" @click="logout">Logout</v-btn>
    </v-app-bar>

    <v-navigation-drawer v-model="drawer" app temporary>
      <v-list>
        <v-list-item @click="currentView = 'collection'">
          <v-list-item-title>My Collection</v-list-item-title>
        </v-list-item>
        <v-list-item @click="currentView = 'wishlist'">
          <v-list-item-title>Wishlist</v-list-item-title>
        </v-list-item>
      </v-list>
    </v-navigation-drawer>

    <v-main>
      <v-container v-if="!isLoggedIn">
        <v-card class="mx-auto mt-5" max-width="400">
          <v-card-title>Login</v-card-title>
          <v-card-text>
            <v-form @submit.prevent="login">
              <v-text-field v-model="username" label="Username" required></v-text-field>
              <v-text-field v-model="password" label="Password" type="password" required></v-text-field>
              <v-btn type="submit" color="primary" block class="mt-2">Login</v-btn>
            </v-form>
          </v-card-text>
        </v-card>
      </v-container>

      <v-container v-else>
        <h2 class="text-h4 mb-4">{{ currentView === 'collection' ? 'My Watch Collection' : 'My Wishlist' }}</h2>
        
        <v-data-table
          v-if="isTableView && currentView === 'collection'"
          :headers="headers"
          :items="watches"
          class="elevation-1"
        >
          <template v-slot:item.photo="{ item }">
            <v-img :src="item.photo" height="100" width="100" contain></v-img>
          </template>
          <template v-slot:item.actions="{ item }">
            <v-btn color="error" @click="removeWatch(item.id)">Remove</v-btn>
          </template>
        </v-data-table>

        <v-row v-else>
          <v-col v-for="watch in displayedItems" :key="watch.id" cols="12" sm="6" md="4">
            <v-card>
              <v-img :src="watch.photo" height="200px" cover></v-img>
              <v-card-title>{{ watch.manufacturer }} {{ watch.model }}</v-card-title>
              <v-card-text>
                <p>Price: ${{ watch.price.toFixed(2) }}</p>
              </v-card-text>
              <v-card-actions>
                <v-btn v-if="currentView === 'wishlist'" color="primary" @click="moveToCollection(watch)">Add to Collection</v-btn>
                <v-btn v-else color="error" @click="removeWatch(watch.id)">Remove</v-btn>
              </v-card-actions>
            </v-card>
          </v-col>
        </v-row>
      </v-container>
    </v-main>

    <v-navigation-drawer v-model="addDrawer" location="right" temporary>
      <v-list-item>
        <v-list-item-title>Add New Watch</v-list-item-title>
      </v-list-item>
      <v-divider></v-divider>
      <v-form @submit.prevent="addWatch" class="pa-4">
        <v-text-field v-model="newWatch.manufacturer" label="Manufacturer" required></v-text-field>
        <v-text-field v-model="newWatch.model" label="Model" required></v-text-field>
        <v-text-field v-model="newWatch.price" label="Price" type="number" prefix="$" required></v-text-field>
        <v-text-field v-model="newWatch.photo" label="Photo URL"></v-text-field>
        <v-btn type="submit" color="primary" block class="mt-2">Add Watch</v-btn>
      </v-form>
    </v-navigation-drawer>

    <v-btn
      v-if="isLoggedIn"
      color="primary"
      dark
      fixed
      bottom
      right
      fab
      @click="addDrawer = !addDrawer"
    >
      <v-icon>mdi-plus</v-icon>
    </v-btn>
  </v-app>
</template>

<script setup>
import { ref, computed } from 'vue'

const isLoggedIn = ref(false)
const username = ref('')
const password = ref('')
const drawer = ref(false)
const addDrawer = ref(false)
const isTableView = ref(false)
const currentView = ref('collection')

const watches = ref([])
const wishlist = ref([])
const newWatch = ref({ manufacturer: '', model: '', price: '', photo: '' })

const headers = [
  { title: 'Photo', key: 'photo', sortable: false },
  { title: 'Manufacturer', key: 'manufacturer' },
  { title: 'Model', key: 'model' },
  { title: 'Price', key: 'price' },
  { title: 'Actions', key: 'actions', sortable: false }
]

function login() {
  if (username.value === 'testuser' && password.value === 'password123') {
    isLoggedIn.value = true
    username.value = ''
    password.value = ''
  } else {
    alert('Invalid credentials')
  }
}

function logout() {
  isLoggedIn.value = false
}

function addWatch() {
  if (newWatch.value.manufacturer && newWatch.value.model) {
    const watch = {
      id: Date.now(),
      ...newWatch.value,
      price: parseFloat(newWatch.value.price) || 0
    }
    if (currentView.value === 'collection') {
      watches.value.push(watch)
    } else {
      wishlist.value.push(watch)
    }
    newWatch.value = { manufacturer: '', model: '', price: '', photo: '' }
    addDrawer.value = false
  }
}

function removeWatch(id) {
  if (currentView.value === 'collection') {
    watches.value = watches.value.filter(watch => watch.id !== id)
  } else {
    wishlist.value = wishlist.value.filter(watch => watch.id !== id)
  }
}

function moveToCollection(watch) {
  wishlist.value = wishlist.value.filter(w => w.id !== watch.id)
  watches.value.push(watch)
}

function toggleView() {
  isTableView.value = !isTableView.value
}

const displayedItems = computed(() => {
  return currentView.value === 'collection' ? watches.value : wishlist.value
})

const totalValue = computed(() => {
  return watches.value.reduce((sum, watch) => sum + watch.price, 0).toFixed(2)
})
</script>