<script setup>
import { store } from '../store'
import { ref } from 'vue'

import { createClient } from '@supabase/supabase-js/dist/main/index.js'
const { supabaseUrl, supabasePublicKey } = useRuntimeConfig()
const supabase = createClient(supabaseUrl, supabasePublicKey)

const loading = ref(true)
const username = ref('')
const website = ref('')
const avatar_url = ref('')

async function updateProfile() {
  try {
    loading.value = true
    store.user = supabase.auth.user()

    const updates = {
      id: store.user.id,
      username: username.value,
      website: website.value,
      avatar_url: avatar_url.value,
      updated_at: new Date()
    }

    let { error } = await supabase.from('profiles').upsert(updates, {
      returning: 'minimal' // Don't return the value after inserting
    })

    if (error) throw error
  } catch (error) {
    alert(error.message)
  } finally {
    loading.value = false
  }
}

async function signOut() {
  try {
    loading.value = true
    let { error } = await supabase.auth.signOut()
    if (error) throw error
  } catch (error) {
    alert(error.message)
  } finally {
    loading.value = false
  }
}
</script>

<template>
  <form class="form-widget" @submit.prevent="updateProfile">
    <div>
      <label for="email">Email</label>
      <input id="email" type="text" :value="store.user.email" disabled />
    </div>
    <div>
      <label for="username">Name</label>
      <input id="username" type="text" v-model="username" />
    </div>
    <div>
      <label for="website">Website</label>
      <input id="website" type="website" v-model="website" />
    </div>

    <div>
      <input
        type="submit"
        class="button block primary"
        :value="loading ? 'Loading ...' : 'Update'"
        :disabled="loading"
      />
    </div>

    <div>
      <button class="button block" @click="signOut" :disabled="loading">
        Sign Out
      </button>
    </div>
  </form>
</template>
