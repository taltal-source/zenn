---
title: "ViteをWSLのDevcontainerで入門してみた"
emoji: "🚀"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["vite","wsl2","typescript","react","docker"]
published: false
---

### はじめに
久しぶりにReact触る機会があったのでせっかくならと思ってVite入門してみました。
最小構成意識したつもりです。多分。
ファイル名ごとにどんな設定にしたか書いているので必要に応じてジャンプしてみてください。
(完全にすべての項目を理解しているかといわれるとちょっと自信なくしますが・・。)
WSLやDevcontainer内でうまく動かない・ホットリロード問題についてはvite.config.tsで触れてます。


Twitter(X)やってますのでよかったらお話しましょう。
https://x.com/\_taltalsource\_

## 使うものたち
粒度気にせず列挙しておきます。
- WSL2(Windows11)
- DockerDesktop
- Devcontainer (VSCode)
- Vite
- React
- TailwindCSS


## とりあえず動かしてみる
nodeは22.7.0を使用。

作成。
```shell
$ npm create vite@latest
```

初めて使ったのでインストールする。
```shell
Need to install the following packages:
create-vite@5.5.2
Ok to proceed? (y)
```

諸々設定する。ReactでTypeScriptを使用。
```shell
√ Project name: ... example-react
√ Select a framework: » React
√ Select a variant: » TypeScript + SWC
```

次はこれを実行してねっていうのが出てるのでそのまま実行していく
```shell
$ cd example-react
$ npm install
$ npm run dev
```

```shell
  VITE v5.4.3  ready in 1181 ms

  ➜  Local:   http://localhost:5173/
  ➜  Network: use --host to expose
  ➜  press h + enter to show help
  ```

アクセスしてみる
![DjangoFormをカスタマイズせずFormViewで表示](/images/react-dev-environment/helloworld.png)
こんな画面が表示されていたのでok。

ヘルプが出てくるらしく、ちょっと気になったのでhを押してenterを押すとこんなコマンドが使えるってことがわかった。
```shell
Shortcuts
  press r + enter to restart the server
  press u + enter to show server url
  press o + enter to open in browser
  press c + enter to clear console
  press q + enter to quit
```
試しにo + enterでやってみると確かにブラウザが開いた。楽。
q + enterでいったん止めてもろもろの設定をする。


## 開発環境の設定
### tsconfig.app.json
インポート時に `../../..` とかなるのは嫌なのでパスエイリアスの設定をする。
楽したいので以下を開発環境にだけインストールする。
```shell
npm install -D vite-tsconfig-paths
```

tsconfigが何種類かあるが、追加の設定はtsconfig.app.jsonに書けばいいみたい
ついでにTypeScriptを強制するのと、戻り値のチェックを厳しくしておく。


```diff json:tsconfig.app.json
"compilerOptions": {

...
"include": ["src"],
+  "allowJs": false,
+  "noImplicitReturns": true,
+  "baseUrl": "./",
+  "paths": {
+    "@/*": ["src/*"]
+  }
}
```

### vite.config.ts 
同様にパスの設定。

自分の環境はWSL+DevContainer想定なので、以下のようにする。

```diff typescript:vite.config.ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react-swc';
+ import tsconfigPaths from 'vite-tsconfig-paths';

// https://vitejs.dev/config/
export default defineConfig({
-  plugins: [react()],
+  plugins: [react(), tsconfigPaths()],
+  server: {
+    watch: {
+      usePolling: true,
+      interval: 1000,
+    },
+    host: '127.0.0.1',
+  },
});
```

usePollingとintervalはWSLからはホットリロードが効かない問題対策、
hostはDockerDesktopの特定のバージョンで起きているポート転送がうまくされない問題対策です。

とても助かった記事を貼っておきます。
https://zenn.dev/onozaty/articles/docker-desktop-portforward-not-working

### Devcontainer.json
今回はDevcontainerを使うのでフォーマッタあたりはそこに書いていく。

.devcontainerフォルダにdevcontainer.jsonを作成し、設定を記述する。
serviceとかfolderと拡張機能はいい感じにしてください。
```json:devcontainer.json
{
  "name": "React Example App",
  "dockerComposeFile": ["../docker-compose.yml"],
  "service": "coffee",
  "workspaceFolder": "/home/node/app",
  "customizations": {
    "vscode": {
      "extensions": [
        "MS-CEINTL.vscode-language-pack-ja", // Visual Studio Codeの日本語言語パック
        "formulahendry.auto-close-tag", // HTML Auto Close Tag
        "bradlc.vscode-tailwindcss", // Tailwind CSS IntelliSense
        "dbaeumer.vscode-eslint", // ESLint
        "esbenp.prettier-vscode", // prettier
        "PKief.material-icon-theme", // iconパック
        "xabikos.ReactSnippets", // React Code Snippets
        "xabikos.JavaScriptSnippets" // JS Code Snippets
      ]
    }
  },
  "remoteUser": "node"
}


```

### settings.json
vscodeの拡張機能設定。お好みで。
TailWindCSSを使っているのでそのあたりとかフォーマッタ設定とか。

```json:settings.json
{
  "files.associations": {
    "*.css": "tailwindcss"
  },
  "editor.quickSuggestions": {
    "strings": true
  },
  "editor.inlineSuggest.enabled": true,
  "tailwindCSS.classAttributes": ["class", "className", "ngClass", ".*Styles.*", ".*Style.*"],
  "editor.formatOnSave": true,
  "editor.formatOnPaste": true,
  "editor.formatOnType": true,
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": "explicit"
  }
}

```

### .prettierrc
Prettierの設定。お好みで。
```json:.prettierrc
{  
    "trailingComma": "all",
    "tabWidth": 2,
    "semi": true,
    "singleQuote": true,
    "jsxSingleQuote": true,
    "printWidth": 100,
    "plugins": ["prettier-plugin-tailwindcss"]
}   
```


### Dockerfile
ここは普通。開発環境用なのでいったんマルチステージビルドとかはいいかなという感じ。
Devcontainer内で起動とかいろいろ自由にしたいので起動コマンドとかは書かず、composeから永続化コマンドを実行させます。

```Dockerfile:Dockerfile
FROM node:22-bookworm

USER node
WORKDIR /home/node/app

COPY package.json /home/node/app/
COPY package-lock.json /home/node/app/
RUN npm install
```

### docker-compose.yml
これも普通に設定する。
commandで永続化設定。
ソースコードはバインドマウントで同期し、node_moduleだけはボリュームマウントとします。
ボリューム名とかはいい感じにしてください。

```yaml:docker-compose.yml
services:
  coffee:
    build:
      context: .
      dockerfile: containers/app/Dockerfile.development
    command: >
      sh -c "sleep infinity"
    tty: true
    volumes:
      - node_modules:/home/node/app/node_modules
      - .:/home/node/app

volumes:
    node_modules:
```

### package.json
いろいろ追加して最終的にこんな感じになりました。
devDependenciesからいりそうなもの選んだりとか足したりしてください。

```json:package.json
"dependencies": {
    "react": "^18.3.1",
    "react-dom": "^18.3.1"
  },
  "devDependencies": {
    "@eslint/js": "^9.9.0",
    "@types/react": "^18.3.3",
    "@types/react-dom": "^18.3.0",
    "@vitejs/plugin-react-swc": "^3.5.0",
    "autoprefixer": "^10.4.20",
    "eslint": "^9.9.0",
    "eslint-plugin-react-hooks": "^5.1.0-rc.0",
    "eslint-plugin-react-refresh": "^0.4.9",
    "eslint-plugin-tailwindcss": "^3.17.4",
    "globals": "^15.9.0",
    "postcss": "^8.4.45",
    "prettier-plugin-tailwindcss": "^0.6.6",
    "tailwindcss": "^3.4.10",
    "typescript": "^5.5.3",
    "typescript-eslint": "^8.0.1",
    "vite": "^5.4.1",
    "vite-tsconfig-paths": "^5.0.1"
  }
  ```

## TailwindCSSの設定
TailwindCSSを使用しない場合は読み飛ばしてください。

公式のCLIからのインストールを参考にしつつ準備。

### tailwind.config.js
exportの記法に注意する。

```js:tailwind.config.js
export default {
  content: ['./index.html', './src/**/*.{js,ts,jsx,tsx}'],
  theme: {
  },
  variants: {},
};
```

追加設定とかあればお好みで。

### postcss.config.js
同様にexportの記法に注意。
```js:postcss.config.js
export default {
  plugins: {
    tailwindcss: {},
    autoprefixer: {},
  },
}
```

### index.css
動きは確認できたしもういいか...と思って書き換えました。
```css:indec.css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

## 動かしてみる
Ctrl+Shift+PからRebuild Containerで動かしてみる。
Tailwindで上書きしたせいで微妙な感じのトップページが出たら完了。
そのまま少し触ってホットリロードの確認とかフォーマッタ設定とか確認する。


### あとがき
最小構成とは・・・？
いったん動かし始めれそうです！