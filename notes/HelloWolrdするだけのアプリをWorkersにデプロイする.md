## 構成
- TypeScript
- Hono
- Wrangler

## 手順
1. `package.json` を作成する

```sh
$ pnpm init
```

2. `wrangler` を開発依存パッケージに追加する

```sh
$ pnpm add -D wrangler
```

3. Hono を依存パッケージに追加する

```sh
$ pnpm add hono
```

4. `wrangler.jsonc` を作成する

```jsonc
{
    // node_modulesにある wrangler のスキーマを参照させる
    "schema": "node_modules/wrangler/config-schema.json",
    
    // Worker の名前。Cloudeflareのダッシュボードで表示される名前になる
    "name": "sample-worker",

    // アプリのエントリーポイント。Worker が呼ばれた際に最初に実行される
    "main": "src/app.ts",

    // ランタイムで実行される Worker のバージョンを指定する。書式は `yyyy-mm-dd`。特に事情がなければ今日の日付を入力すればok
    "compatibility_date": "2026-01-01"
}
```

※ アプリを最初から作るなら `pnpm create hono@latest` や `pnpm wrangler init` した時に自動で作成される。ここでは学習目的のため、手で作る場合の手順を記載

5. `Hello world` するだけのアプリを作る

```sh
$ mkdir src && touch src/app.ts
```

```typescript
// app.ts
const app = new Hono();

app.get('/', (c) => c.text('Hello Hono!'));

export default app;
```

作成後、ローカルで起動に成功することを確認する。

```sh
$ pnpm wrangler dev
```

6. TBD

## 参考
- https://developers.cloudflare.com/workers/wrangler/configuration/
- https://hono.dev/docs/getting-started/basic