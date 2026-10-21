<template>
  <ion-page>
    <ion-content class="ion-padding">
      <h1>Login</h1>

      <ion-input v-model="email" type="email" label="E-mail" label-placement="floating" fill="outline" />
      <ion-input v-model="password" type="password" label="Senha" label-placement="floating" fill="outline" class="ion-margin-top" />

      <ion-text color="danger" v-if="erro"><p>{{ erro }}</p></ion-text>

      <ion-button expand="block" class="ion-margin-top" :disabled="loading" @click="entrar">
        Entrar
      </ion-button>
      <ion-button expand="block" fill="clear" router-link="/forgot-password">
  Esqueci minha senha
</ion-button>
      <ion-button expand="block" fill="clear" router-link="/register">
        Criar conta
      </ion-button>
    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import { useRouter } from 'vue-router'
import { IonPage, IonContent, IonInput, IonButton, IonText } from '@ionic/vue'
import { supabase } from '@/lib/supabase'

const router = useRouter()
const email = ref('')
const password = ref('')
const erro = ref('')
const loading = ref(false)

async function entrar() {
  loading.value = true
  erro.value = ''
  const { error } = await supabase.auth.signInWithPassword({
    email: email.value,
    password: password.value,
  })
  loading.value = false
  if (error) erro.value = error.message
  else router.replace('/home')
}
</script>