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
      <!-- Foto de perfil -->
      <div class="avatar-box">
        <ion-avatar class="avatar" @click="escolherFoto">
          <img v-if="avatarUrl" :src="avatarUrl" alt="Foto de perfil" />
          <div v-else class="avatar-vazio">Sem foto</div>
        </ion-avatar>
        <ion-button fill="clear" size="small" :disabled="enviandoFoto" @click="escolherFoto">
          {{ enviandoFoto ? 'Enviando...' : 'Alterar foto' }}
        </ion-button>
        <input ref="inputArquivo" type="file" accept="image/*" hidden @change="enviarFoto" />
      </div>

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
  IonContent, IonInput, IonButton, IonText, IonAvatar,
} from '@ionic/vue'
import { supabase } from '@/lib/supabase'

const nome = ref('')
const nickname = ref('')
const dataNascimento = ref('')
const avatarUrl = ref('')
const erro = ref('')
const sucesso = ref(false)
const loading = ref(false)
const enviandoFoto = ref(false)
const inputArquivo = ref<HTMLInputElement | null>(null)

onMounted(carregar)

async function carregar() {
  const { data: { user } } = await supabase.auth.getUser()
  if (!user) return

  const { data, error } = await supabase
    .from('profiles')
    .select('nome, data_nascimento, nickname, avatar_url')
    .eq('id', user.id)
    .maybeSingle()

  if (error) { erro.value = error.message; return }
  if (data) {
    nome.value = data.nome ?? ''
    nickname.value = data.nickname ?? ''
    dataNascimento.value = data.data_nascimento ?? ''
    avatarUrl.value = data.avatar_url ?? ''
  }
}

function escolherFoto() {
  inputArquivo.value?.click()
}

async function enviarFoto(event: Event) {
  const input = event.target as HTMLInputElement
  const file = input.files?.[0]
  if (!file) return

  erro.value = ''
  enviandoFoto.value = true

  const { data: { user } } = await supabase.auth.getUser()
  if (!user) { enviandoFoto.value = false; return }

  const ext = file.name.split('.').pop()?.toLowerCase() || 'jpg'
  const caminho = `${user.id}/avatar.${ext}`

  // 1) Upload no Storage
  const { error: uploadError } = await supabase.storage
    .from('avatars')
    .upload(caminho, file, { upsert: true, contentType: file.type })

  if (uploadError) {
    erro.value = uploadError.message
    enviandoFoto.value = false
    input.value = ''
    return
  }

  // 2) URL pública (com ?t= para evitar cache da foto antiga)
  const { data } = supabase.storage.from('avatars').getPublicUrl(caminho)
  const url = `${data.publicUrl}?t=${Date.now()}`

  // 3) Salvar a URL na tabela profiles
  const { error: dbError } = await supabase.from('profiles').upsert({
    id: user.id,
    avatar_url: url,
    updated_at: new Date().toISOString(),
  })

  if (dbError) erro.value = dbError.message
  else avatarUrl.value = url

  enviandoFoto.value = false
  input.value = ''
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
    erro.value = error.code === '23505' ? 'Esse nickname já está em uso.' : error.message
  } else {
    sucesso.value = true
  }
}
</script>

<style scoped>
.avatar-box {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin-bottom: 16px;
}
.avatar {
  width: 120px;
  height: 120px;
  cursor: pointer;
}
.avatar-vazio {
  width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  background: var(--ion-color-light);
  color: var(--ion-color-medium);
  font-size: 14px;
}
</style>