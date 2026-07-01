# 完全無料でスマホとPCを同期する

完全無料で使う場合は、次の構成にします。

- 画面: GitHub Pages
- 同期・ログイン: Supabase Free
- AI機能: オフ

OpenAI APIは料金が発生する可能性があるため、完全無料版ではAI提案ボタンは使いません。

## 1. Supabaseを作る

1. Supabaseで無料プロジェクトを作る。
2. Authentication > Providers > Email を有効にする。
3. Authentication > Users で自分のメール/パスワードのユーザーを作る。
4. SQL Editorで下のSQLを実行する。

```sql
create table if not exists public.journal_states (
  user_id uuid primary key references auth.users(id) on delete cascade,
  state jsonb not null default '{}'::jsonb,
  updated_at timestamptz not null default now()
);

alter table public.journal_states enable row level security;

create policy "select own journal state"
on public.journal_states
for select
using (auth.uid() = user_id);

create policy "insert own journal state"
on public.journal_states
for insert
with check (auth.uid() = user_id);

create policy "update own journal state"
on public.journal_states
for update
using (auth.uid() = user_id)
with check (auth.uid() = user_id);
```

5. Project Settings > API から Project URL と anon public key を控える。

## 2. cloud-config.jsを書く

`cloud-config.js` を開いて、Supabaseの値に置き換えます。

```js
window.GTJ_SUPABASE = {
  url: "https://YOUR_PROJECT_ID.supabase.co",
  anonKey: "YOUR_SUPABASE_ANON_KEY",
};
```

`anonKey` は公開用キーです。ただし、上のRLSポリシーを必ず設定してください。

## 3. GitHub Pagesへアップロード

GitHubに新しいPublic repositoryを作り、次のファイルをアップロードします。

```text
index.html
app.js
styles.css
manifest.webmanifest
service-worker.js
icon.svg
cloud-config.js
cloud-config.example.js
SYNC_DEPLOY.md
```

`ai-server.js`、`package.json`、`.env`、`sync-data.json`、文字化け名ファイル、`.app` フォルダは完全無料版では不要です。

## 4. GitHub Pagesを有効にする

Repository Settings > Pages で、Deploy from a branch を選びます。

- Branch: `main`
- Folder: `/root`

公開URLは `https://ユーザー名.github.io/リポジトリ名/` になります。

## 5. スマホで使う

1. スマホでGitHub PagesのURLを開く。
2. 同期欄にSupabaseで作ったメールとパスワードを入れる。
3. PCでも同じURLを開き、同じメールとパスワードでログインする。

これで同じWi-Fiでなくても同期できます。
