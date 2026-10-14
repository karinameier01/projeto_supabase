<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-buttons slot="start">
          <ion-back-button default-href="/welcome" />
        </ion-buttons>
        <ion-title>Meu perfil</ion-title>
      </ion-toolbar>
    </ion-header>

    <ion-content class="ion-padding">
      <ion-input v-model="nome" label="Nome" label-placement="floating" fill="outline" />

      <ion-input v-model="nickname" label="Nickname" label-placement="floating" fill="outline" class="ion-margin-top" />

      <ion-input v-model="dataNascimento" type="date" label="Data de nascimento" label-placement="stacked" fill="outline" class="ion-margin-top" />

      <ion-text color="danger" v-if="erro"><p>{{ erro }}</p></ion-text>
      <ion-text color="success" v-if="sucesso"><p>Perfil salvo!</p></ion-text>

      <ion-button expand="block" class="ion-margin-top" :disabled="loading" @click="salvar">
        Salvar
      </ion-button>
    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue'
import {
  IonPage, IonHeader, IonToolbar, IonTitle, IonButtons, IonBackButton,
  IonContent, IonInput, IonButton, IonText,
} from '@ionic/vue'
import { supabase } from '@/lib/supabase'

const nome = ref('')
const nickname = ref('')
const dataNascimento = ref('')
const erro = ref('')
const sucesso = ref(false)
const loading = ref(false)

onMounted(carregar)

async function carregar() {
  const { data: { user } } = await supabase.auth.getUser()
  if (!user) return

  const { data, error } = await supabase
    .from('profiles')
    .select('nome, data_nascimento, nickname')
    .eq('id', user.id)
    .maybeSingle()

  if (error) { erro.value = error.message; return }
  if (data) {
    nome.value = data.nome ?? ''
    nickname.value = data.nickname ?? ''
    dataNascimento.value = data.data_nascimento ?? ''
  }
}

async function salvar() {
  loading.value = true
  erro.value = ''
  sucesso.value = false

  const { data: { user } } = await supabase.auth.getUser()

  const { error } = await supabase.from('profiles').upsert({
    id: user!.id,
    nome: nome.value,
    nickname: nickname.value,
    data_nascimento: dataNascimento.value || null,
    updated_at: new Date().toISOString(),
  })

  loading.value = false
  if (error) {
    // 23505 = violação de unique (nickname já em uso)
    erro.value = error.code === '23505' ? 'Esse nickname já está em uso.' : error.message
  } else {
    sucesso.value = true
  }
}
</script>