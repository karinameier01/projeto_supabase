<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-title>Início</ion-title>
        <ion-buttons slot="end">
          <ion-button @click="sair">Sair</ion-button>
        </ion-buttons>
      </ion-toolbar>
    </ion-header>

    <ion-content class="ion-padding">
      <div class="boas-vindas">
        <ion-avatar class="avatar">
          <img v-if="avatarUrl" :src="avatarUrl" alt="Foto de perfil" />
          <div v-else class="avatar-vazio">Sem foto</div>
        </ion-avatar>

        <h1>Bem-vindo, {{ nomeExibido }}!</h1>
        <p class="email">{{ email }}</p>

        <ion-button expand="block" class="ion-margin-top" router-link="/profile">
          Meu perfil
        </ion-button>
      </div>
    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import { useRouter } from 'vue-router'
import {
  IonPage, IonHeader, IonToolbar, IonTitle, IonButtons,
  IonContent, IonButton, IonAvatar, onIonViewWillEnter,
} from '@ionic/vue'
import { supabase } from '@/lib/supabase'

const router = useRouter()
const nomeExibido = ref('')
const email = ref('')
const avatarUrl = ref('')

// Roda toda vez que a tela aparece (inclusive ao voltar do perfil)
onIonViewWillEnter(carregar)

async function carregar() {
  const { data: { user } } = await supabase.auth.getUser()
  if (!user) return

  email.value = user.email ?? ''

  const { data } = await supabase
    .from('profiles')
    .select('nickname, nome, avatar_url')
    .eq('id', user.id)
    .maybeSingle()

  nomeExibido.value = data?.nickname || data?.nome || user.email || ''
  avatarUrl.value = data?.avatar_url ?? ''
}

async function sair() {
  await supabase.auth.signOut()
  router.replace('/login')
}
</script>

<style scoped>
.boas-vindas {
  display: flex;
  flex-direction: column;
  align-items: center;
  text-align: center;
  margin-top: 32px;
}
.avatar {
  width: 120px;
  height: 120px;
  margin-bottom: 16px;
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
.email {
  color: var(--ion-color-medium);
  margin: 0;
}
</style>