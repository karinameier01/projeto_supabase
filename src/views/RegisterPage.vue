<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-buttons slot="start">
          <ion-back-button default-href="/login" />
        </ion-buttons>
        <ion-title>Criar conta</ion-title>
      </ion-toolbar>
    </ion-header>

    <ion-content class="ion-padding">
      <ion-input
        v-model="email"
        type="email"
        label="E-mail"
        label-placement="floating"
        fill="outline"
      />

      <ion-input
        v-model="password"
        type="password"
        label="Senha"
        label-placement="floating"
        fill="outline"
        class="ion-margin-top"
      />

      <ion-input
        v-model="confirmar"
        type="password"
        label="Confirmar senha"
        label-placement="floating"
        fill="outline"
        class="ion-margin-top"
      />

      <ion-text color="danger" v-if="erro"><p>{{ erro }}</p></ion-text>
      <ion-text color="success" v-if="mensagem"><p>{{ mensagem }}</p></ion-text>

      <ion-button expand="block" class="ion-margin-top" :disabled="loading" @click="cadastrar">
        {{ loading ? 'Criando...' : 'Criar conta' }}
      </ion-button>

      <ion-button expand="block" fill="clear" router-link="/login">
        Já tenho conta
      </ion-button>
    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import { useRouter } from 'vue-router'
import {
  IonPage, IonHeader, IonToolbar, IonTitle, IonButtons, IonBackButton,
  IonContent, IonInput, IonButton, IonText,
} from '@ionic/vue'
import { supabase } from '@/lib/supabase'

const router = useRouter()
const email = ref('')
const password = ref('')
const confirmar = ref('')
const erro = ref('')
const mensagem = ref('')
const loading = ref(false)

async function cadastrar() {
  erro.value = ''
  mensagem.value = ''

  if (!email.value || !password.value) {
    erro.value = 'Preencha e-mail e senha.'
    return
  }
  if (password.value.length < 6) {
    erro.value = 'A senha deve ter pelo menos 6 caracteres.'
    return
  }
  if (password.value !== confirmar.value) {
    erro.value = 'As senhas não conferem.'
    return
  }

  loading.value = true
  const { data, error } = await supabase.auth.signUp({
    email: email.value,
    password: password.value,
  })
  loading.value = false

  if (error) {
    erro.value = error.message
    return
  }

  if (data.session) {
    // "Confirm email" desligado: já entra logado
    router.replace('/home')
  } else {
    // "Confirm email" ligado: precisa confirmar pelo e-mail
    mensagem.value = 'Conta criada! Confirme seu e-mail para poder entrar.'
  }
}
</script>