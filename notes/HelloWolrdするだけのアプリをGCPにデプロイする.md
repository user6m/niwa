## 構成
- TypeScript
- Express.js
- Google Cloud

## 手順

### 1. アプリケーションの作成
1. `pnpm init` で pnpm を初期化

いらないものは削っておく

```json
{
	"name": "webapp-202609121700",
	"scripts": {},
	"license": "ISC",
	"packageManager": "pnpm@10.33.2"
}
```

ESMを使えるように `type: "module"`を追記しておく。 * `import xxx` で記述できるようになる。

```json
{
	...,
	"type": "module"
}
```

クラウドにデプロイした際のランタイム環境の指定として `engines` を追記しておく。

```json
{
	...,
	"engines": {"node": ">=22.18.0"},
}
```

2. `express` を依存パッケージに追加

```sh
$ pnpm add express
```

3. Typescript 関連のパッケージを開発依存パッケージに追加

```sh
$ pnpm add -D typescript @types/express @types/node
```

4. `tsconfig.json` を作成する

```sh
$ pnpm tsc --init
```

[Express のクイックスタート](https://expressjs.com/ja/5x/starter/installing/#typescript)に記載してる `tsconfig.json` をコピペする

```json
{
  "compilerOptions": {
    "target": "esnext",
    "module": "nodenext",
    "rewriteRelativeImportExtensions": true,
    "erasableSyntaxOnly": true,
    "verbatimModuleSyntax": true,
    "noEmit": true,
    "strict": true,
    "skipLibCheck": true
  }
}
```

ビルドが必要になるので、`outDir` と `rootDir` を設定しておき、
`noEmit` はコメントアウトする

```json
{
  "compilerOptions": {
    ...,
		// "noEmit": true,
		"outDir": "./dist",
		"rootDir": "./src"
  }
}
```


5. `Hello world`するだけのアプリを適当に作成

```ts
import express, { type Express } from "express";

const app: Express = express();

app.get("/", (_req, res) => {
	res.send("Hello world!");
});

const port = process.env.PORT || 8080;
app.listen(port, () => {
	console.log(`🚀Start listening on port: ${port}...`);
});
```

6. ビルドスクリプト、起動スクリプトを `package.json` に追加する。追加後、アプリの軌道に成功することを確認する

```json
{
	...,
	"scripts": {
		"build": "tsc",
		"start": "node dist/app.js"
	},
}
```

### 2. Google Cloud Project の作成
#### CLIの場合
1. `gcloud projects create` で Google Cloud Projects を作成する

```sh
$ gcloud projects create $PID --name="project-name"
```

2. CLI のデフォルトのプロジェクトを新規作成したものに変更する

```sh
$ gcloud config set project $PID
```

使いまわすこと多いのでローカル変数にしておく。
```sh
$ PID=$(gcloud config get-value project)
```

3. 請求先アカウントのIDを入手

請求先アカウントが未作成の場合は先に作っておく

```sh
$ gcloud billing accounts list
ACCOUNT_ID            NAME              OPEN   MASTER_ACCOUNT_ID
billing-account-id  billing-account-name  True
```

4. プロジェクトに請求先アカウントを紐づける

```sh
$ gcloud billing projects link $PID --billing-account=billing-account-id
```

5. デプロイコマンドを実行する *この段階では必ず失敗する

途中APIの有効化を促されるので、全部 yes で実行する

```sh
$ gcloud run deploy hello-world \
  --source . \
  --region asia-northeast1 \
  --allow-unauthenticated \
  --project=$PID
```

以下のようにビルド実行時に `PERMISSION_DENIED` で失敗することを確認できればOK。

```sh
X Building and deploying new service... Uploading sources.             
  ✓ Creating Container Repository...                                   
  ✓ Validating configuration...                                        
  ✓ Uploading sources...                                               
  . Building Container...                                              
  . Creating Revision...                                               
  . Routing traffic...                                                 
  . Setting IAM Policy...                                              
Deployment failed                                                      
ERROR: (gcloud.run.deploy) PERMISSION_DENIED: Build failed because the default service account is missing required IAM permissions.
```

6. サービスアカウントが作成されていることを確認する

```sh
$ gcloud iam service-accounts list --project=$PID
DISPLAY NAME                     EMAIL                                               DISABLED
Default compute service account  sample-compute@developer.gserviceaccount.com  False
```

確認出来たら `EMAIL` を控えておく

```sh
$ BUILDER=sample-compute@developer.gserviceaccount.com
```

7. サービスアカウントにビルドを実行できる権限を付与する

```sh
$ gloud projects add-iam-policy-binding $PID \
	--member="serviceAccount:${BUILDER}" \
	--role="roles/run.builder"
```

8. 再度デプロイを実行する

```sh
$ gcloud run deploy hello-world \
  --source . \
  --region asia-northeast1 \
  --allow-unauthenticated \
  --project=$PID
```

サービスURLが発行されることを確認出来たら完了

```sh
Service [hello-world] revision [hello-world-00001-pjl] has been deployed and is serving 100 percent of traffic.
Service URL: https://hello-world-sample.asia-northeast1.run.app
```

## 参考
- https://expressjs.com/ja/5x/starter/installing/
