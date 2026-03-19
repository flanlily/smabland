# スマブランド — みんなの掲示板 セットアップガイド

このリポジトリには、GitHub Pages 上で動作するブログ形式の掲示板ページ (`blog.html`) が含まれています。  
投稿・コメントのデータは **Firebase Firestore** に保存され、全ユーザー間で共有されます。

---

## 1. Firebase プロジェクトの作成

1. [Firebase Console](https://console.firebase.google.com/) を開く
2. 「プロジェクトを追加」をクリック
3. プロジェクト名を入力（例: `smabland-blog`）し、画面の指示に従って作成する

---

## 2. Firestore Database の有効化

1. Firebase Console の左メニューから「構築」→「Firestore Database」を選択
2. 「データベースを作成」をクリック
3. モードは「**テストモード**」を選択（後でセキュリティルールを更新する）
4. ロケーションを選択（例: `asia-northeast1`（東京））し、「有効にする」をクリック

---

## 3. `firebaseConfig` の設定値を取得する

1. Firebase Console の左上にある歯車アイコン（⚙️）→「プロジェクトの設定」を開く
2. 「マイアプリ」セクションで「**ウェブ**」アイコン（`</>`）をクリックしてウェブアプリを登録する
3. アプリのニックネームを入力（例: `smabland-web`）し、「アプリを登録」をクリック
4. 表示される `firebaseConfig` オブジェクトをコピーする

---

## 4. `blog.html` に設定値を反映する

`blog.html` 内の以下のプレースホルダーを、取得した実際の値に置き換えてください：

```javascript
const firebaseConfig = {
    apiKey: "YOUR_API_KEY",                        // ← 実際の値に変更
    authDomain: "YOUR_PROJECT_ID.firebaseapp.com", // ← 実際の値に変更
    projectId: "YOUR_PROJECT_ID",                  // ← 実際の値に変更
    storageBucket: "YOUR_PROJECT_ID.appspot.com",  // ← 実際の値に変更
    messagingSenderId: "YOUR_SENDER_ID",           // ← 実際の値に変更
    appId: "YOUR_APP_ID"                           // ← 実際の値に変更
};
```

> ⚠️ **セキュリティに関する注意**: Firebase の `apiKey` などの設定値はクライアントサイドのコードに含まれるため、ページを閲覧したユーザーが参照できます。これは Firebase の仕様として想定されており、`apiKey` は Firebase プロジェクトを識別するためのものです。実際のアクセス制御は **Firestore セキュリティルール**（手順 5）で行ってください。パブリックなリポジトリに設定値をコミットする場合は、Firestore セキュリティルールで書き込み内容の検証を必ず行うことを推奨します。

---

## 5. Firestore セキュリティルールの設定

Firebase Console の「Firestore Database」→「ルール」タブを開き、以下のルールを設定してください。

### 誰でも読み書き可能にする場合（テスト用）

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /posts/{postId} {
      allow read, write: if true;
      match /comments/{commentId} {
        allow read, write: if true;
      }
    }
  }
}
```

> ⚠️ **注意**: 上記のルールは誰でも読み書き可能な設定です。本番運用では、より厳格なルールの設定を検討してください。

### 読み取りは誰でも可能、書き込みはデータの検証を行う場合（推奨）

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /posts/{postId} {
      allow read: if true;
      allow create: if request.resource.data.keys().hasAll(['nickname', 'title', 'content', 'createdAt'])
                   && request.resource.data.title is string
                   && request.resource.data.title.size() <= 100
                   && request.resource.data.content is string
                   && request.resource.data.content.size() <= 2000;
      allow update, delete: if false;

      match /comments/{commentId} {
        allow read: if true;
        allow create: if request.resource.data.keys().hasAll(['nickname', 'content', 'createdAt'])
                     && request.resource.data.content is string
                     && request.resource.data.content.size() <= 1000;
        allow update, delete: if false;
      }
    }
  }
}
```

---

## 6. GitHub Pages へのデプロイ

`blog.html` を編集・保存したら、`main` ブランチにプッシュするだけで GitHub Pages に自動的に反映されます。

```bash
git add blog.html
git commit -m "Firebase設定を更新"
git push origin main
```

---

## Firestore コレクション構成

| コレクション | フィールド | 説明 |
|------------|-----------|------|
| `posts` | `nickname` (string) | 投稿者のニックネーム |
| | `title` (string) | 投稿タイトル |
| | `content` (string) | 投稿本文 |
| | `createdAt` (timestamp) | 投稿日時 |
| `posts/{id}/comments` | `nickname` (string) | コメント投稿者のニックネーム |
| | `content` (string) | コメント本文 |
| | `createdAt` (timestamp) | コメント日時 |
