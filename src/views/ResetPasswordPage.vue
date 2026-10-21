<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-title>Nova senha</ion-title>
      </ion-toolbar>
    </ion-header>

    <ion-content class="ion-padding">
      <ion-input v-model="senha" type="password" label="Nova senha" label-placement="floating" fill="outline" />

      <ion-input v-model="confirmar" type="password" label="Confirmar nova senha" label-placement="floating" fill="outline" class="ion-margin-top" />

      <ion-text color="danger" v-if="erro"><p>{{ erro }}</p></ion-text>

      <ion-button expand="block" class="ion-margin-top" :disabled="loading" @click="salvar">
        {{ loading ? 'Salvando...' : 'Salvar nova senha' }}
      </ion-button>
    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import { useRouter } from 'vue-router'
import {
  IonPage, IonHeader, IonToolbar, IonTitle,
  IonContent, IonInput, IonButton, IonText,
} from '@ionic/vue'
import { supabase } from '@/lib/supabase'

const router = useRouter()
const senha = ref('')
const confirmar = ref('')
const erro = ref('')
const loading = ref(false)

async function salvar() {
  erro.value = ''

  if (senha.value.length < 6) { erro.value = 'A senha deve ter pelo menos 6 caracteres.'; return }
  if (senha.value !== confirmar.value) { erro.value = 'As senhas não conferem.'; return }

  const { data: { session } } = await supabase.auth.getSession()
  if (!session) {
    erro.value = 'Link inválido ou expirado. Peça um novo e-mail de recuperação.'
    return
  }

  loading.value = true
  const { error } = await supabase.auth.updateUser({ password: senha.value })
  loading.value = false

  if (error) erro.value = error.message
  else router.replace('/home')
}
</script>