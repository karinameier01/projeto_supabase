<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-buttons slot="start">
          <ion-back-button default-href="/login" />
        </ion-buttons>
        <ion-title>Recuperar senha</ion-title>
      </ion-toolbar>
    </ion-header>

    <ion-content class="ion-padding">
      <p>Informe seu e-mail e enviaremos um link para criar uma nova senha.</p>

      <ion-input v-model="email" type="email" label="E-mail" label-placement="floating" fill="outline" />

      <ion-text color="danger" v-if="erro"><p>{{ erro }}</p></ion-text>
      <ion-text color="success" v-if="mensagem"><p>{{ mensagem }}</p></ion-text>

      <ion-button expand="block" class="ion-margin-top" :disabled="loading" @click="enviar">
        {{ loading ? 'Enviando...' : 'Enviar link' }}
      </ion-button>
    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import {
  IonPage, IonHeader, IonToolbar, IonTitle, IonButtons, IonBackButton,
  IonContent, IonInput, IonButton, IonText,
} from '@ionic/vue'
import { supabase } from '@/lib/supabase'

const email = ref('')
const erro = ref('')
const mensagem = ref('')
const loading = ref(false)

async function enviar() {
  erro.value = ''
  mensagem.value = ''
  if (!email.value) { erro.value = 'Informe o e-mail.'; return }

  loading.value = true
  const { error } = await supabase.auth.resetPasswordForEmail(email.value, {
    redirectTo: `${window.location.origin}/reset-password`,
  })
  loading.value = false

  if (error) erro.value = error.message
  else mensagem.value = 'Se o e-mail existir, você receberá um link para redefinir a senha.'
}
</script>