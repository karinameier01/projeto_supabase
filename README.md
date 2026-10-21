# App Ionic + Vue + Capacitor + Supabase

Pesquisa prática sobre desenvolvimento de aplicações móveis com Ionic Framework, Vue.js e Capacitor, integrados aos serviços do Supabase.

## Funcionalidades
- **Supabase Auth:** cadastro, login e recuperação de senha por e-mail
- **Tela de boas-vindas** para o usuário logado (nickname, e-mail e foto)
- **Supabase Database:** perfil com nome, data de nascimento e nickname
- **Supabase Storage:** upload da foto de perfil
- Rotas protegidas (só usuário logado acessa início e perfil)

## Tecnologias
Ionic Framework, Vue 3, TypeScript, Capacitor, Vite, Supabase

## Como rodar
1. `npm install`
2. Copie `.env.example` para `.env` e preencha com a Project URL e a anon/publishable key do seu projeto Supabase
3. Execute o SQL abaixo no SQL Editor do Supabase
4. Crie o bucket público `avatars` em Storage
5. Em Authentication → URL Configuration, adicione `http://localhost:8100/reset-password` às Redirect URLs
6. `ionic serve`

## SQL

### Tabela profiles
```sql
create table public.profiles (
  id uuid primary key references auth.users(id) on delete cascade,
  nome text,
  data_nascimento date,
  nickname text unique,
  avatar_url text,
  updated_at timestamptz default now()
);

alter table public.profiles enable row level security;

create policy "Ler próprio perfil"
  on public.profiles for select
  using (auth.uid() = id);

create policy "Criar próprio perfil"
  on public.profiles for insert
  with check (auth.uid() = id);

create policy "Atualizar próprio perfil"
  on public.profiles for update
  using (auth.uid() = id)
  with check (auth.uid() = id);
```

### Policies do Storage (bucket avatars)
```sql
create policy "Avatars: leitura pública"
  on storage.objects for select
  using (bucket_id = 'avatars');

create policy "Avatars: upload na própria pasta"
  on storage.objects for insert to authenticated
  with check (
    bucket_id = 'avatars'
    and (storage.foldername(name))[1] = auth.uid()::text
  );

create policy "Avatars: atualizar na própria pasta"
  on storage.objects for update to authenticated
  using (
    bucket_id = 'avatars'
    and (storage.foldername(name))[1] = auth.uid()::text
  );

create policy "Avatars: apagar na própria pasta"
  on storage.objects for delete to authenticated
  using (
    bucket_id = 'avatars'
    and (storage.foldername(name))[1] = auth.uid()::text
  );
```

## Capacitor
O projeto inclui o `capacitor.config.ts`. Para gerar a versão Android: `ionic build`, `npx cap add android` e `npx cap sync`. Para a recuperação de senha no celular seria necessário configurar um deep link (`@capacitor/app` com o evento `appUrlOpen`).

## Autora
Karina Meier