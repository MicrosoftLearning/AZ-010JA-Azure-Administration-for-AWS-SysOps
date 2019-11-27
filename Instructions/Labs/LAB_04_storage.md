---
lab:
    title: 'BLOB、セキュア アクセス ストレージ、サービス エンドポイント、File Storage'
    module: 'モジュール 4: Azure ストレージ'
---

# ラボ 04: Azure ストレージ

ストレージ アカウント、BLOB、セキュア アクセス ストレージ、サービス エンドポイント、File Storage

## 受講者ラボ マニュアル

## シナリオ

このラボでは、Bash インターフェイス上で Cloud Shell の CLI コマンドを使用して Azure Storage を管理します。ストレージ アカウント、Azure BLOB ストレージ、セキュア アクセス ストレージ (SAS)、サブネット サービス エンドポイント、および File Storage を構成します。

## 目的

このラボを完了すると、Azure CLIを使用して、作成および構成できます。

* ストレージ アカウント
* Blob Storage
* 安全アクセス ストレージ (SAS)
* SubNet サービス エンドポイント
* File Storage

## ラボのセットアップ

* **予想時間**: 45 分

## 指示

### 開始する前に

#### セットアップ タスク

1. **前のラボへの依存関係:**
    1. モジュール 1: Azure 管理 - **ラボ: リソース グループの作成**: EastRG および WestRG リソース グループが構成されました。
    1. モジュール 2: Azure ネットワーク - **ラボ仮想ネットワークとピアリング**: サブネットとピアリングが構成された VNet。
    1. モジュール 3: Azure コンピューティング - **ラボ: Azure VM**: EastDebianVM が作成されました。

### エクササイズ 1: BLOB、セキュア アクセス ストレージ、サービス エンドポイント、File Storage

このエクササイズの主なタスクは次のとおりです。

1. Blob Storage の作成と構成
1. セキュリティで保護されたアクセス ストレージ (SAS) を使用して BLOB を構成する
1. SubNet サービス エンドポイントを構成する
1. File Storage の作成と構成

#### タスク 1: Blob Storage でストレージ アカウントを作成する

**ストレージ アカウント変数を作成する**

1. 一意の名前に使用するランダムな文字列を作成する

```sh
myRand=`head /dev/urandom | tr -dc a-z0-9 | head -c 6 ; echo ''`
echo "the random string append will be:  "$myRand
```

2. Set variables


```sh
# 変数を指定
my_resource_group=WestRG
location=westus
my_storage_account=weststore$myRand
my_storage_sku=Standard_RAGRS # デフォルト
my_storage_kind=StorageV2myRand
my_access_tier=hot
my_storage_encryption=blob

echo "Your my_storage_account name will be: " $my_storage_account
```

>上記のコマンドにあるストレージ アカウント名を書き留める

**ストレージ アカウントを作成する**

```sh
# アカウントを作成
az storage account create \
    -n $my_storage_account \
    -g $my_resource_group \
    -l $location \
    --kind $my_storage_kind \
    --access-tier $my_access_tier \
    --sku $my_storage_sku \
    --encryption $my_storage_encryption
```

**Bash CLI で環境変数を設定する**

1. ストレージ アカウントとストレージ アカウント接続文字列環境変数の作成

```sh
# 環境変数を設定
export AZURE_STORAGE_ACCOUNT=$my_storage_account

export AZURE_STORAGE_CONNECTION_STRING="$(az storage account show-connection-string \
-n $AZURE_STORAGE_ACCOUNT -g $my_resource_group --query connectionString -o tsv)"
```

2. ストレージ アカウント キー環境変数の作成

```sh
# キーの表示
az storage account keys list \
    --account-name $AZURE_STORAGE_ACCOUNT \
    --resource-group $my_resource_group \
    --output table

# AZURE_STORAGE_KEY=<storage_account_key1> をエクスポート
# 環境変数として Key 1 を格納する
export AZURE_STORAGE_KEY="$(az storage account keys list --account-name \
$AZURE_STORAGE_ACCOUNT --resource-group $my_resource_group --query [0].value -o tsv)"
```

> ヒント:テキスト ファイル内のすべての変数と環境変数コマンドをキャプチャして、Cloud Shell が 20 分後にタイムアウトになり、変数が一時メモリから失われた場合に簡単に再入力できるようにします。

**Blob Storage に追加するファイルを用意する**

1. `helloAdmin.html` ファイルを作成して Blob Storage にアップロードする

```sh
# BLOB としてコンテナーに追加するファイルを作成する
echo "<h1>Hello Azure Administrators</h1>">helloAdmin.html
```

2. `helloAdmin.html` ファイルが作成されたことを確認する

```sh
ls
```

**West ストレージ アカウントにパブリック BLOB コンテナーを作成する**

1. パブリック アクセスを使用して BLOB コンテナーを作成する

```sh
container_name=westblobcontainerpublic

az storage container create --name $container_name --public-access blob
# --connection-string $AZURE_STORAGE_CONNECTION_STRING
```

**ファイルをパブリック BLOB コンテナーにアップロードする**

```sh
blob_name=helloAdmin

az storage blob upload \
    --container-name $container_name \
    --file helloAdmin.html \
    --name $blob_name
    #--connection-string $AZURE_STORAGE_CONNECTION_STRING
```

#### 出力ファイルのダウンロード

1. `helloAdmin.html` は新しい名前 `helloAdminDownload.html` でダウンロードされます

```sh
az storage blob download \
    --container-name $container_name \
    --name $blob_name \
    --file helloAdminDownload.html
    #--account-name $AZURE_STORAGE_ACCOUNT \
```

#### `ls` を実行してファイル `helloAdminDownload.html` がダウンロードされたことを確認する

```bash
ls
```

**upload-batch**

1. `upload` ディレクトリを作成する

```sh
# アップロード ファイル ディレクトリにファイルを作成する
mkdir uploadfiles
```

2. `upload` ディレクトリにアップロードするファイルを作成する

```sh
# ファイルの作成...
touch uploadfiles/myfile.html
touch uploadfiles/more.html
touch uploadfiles/hi.html
touch uploadfiles/myfile.txt
```

3. `myUploadPath` および `myDownloadPath` で使用する `az_user_name` 変数を作成する

> 注: `az_user_name` を設定するには、以下のコマンドを編集する必要があります。
>
> クラウド シェル プロンプトで `<name>@azure` の名前をキャプチャする
> 例 - クラウド シェル内のプロンプトが `eric@Azure:~$` の場合は `az_user_name='eric'` を設定する

```sh
# REPLACE <name>
az_user_name=<name>
```

4. アップロード パスとフォルダを作成する

```sh
# クラウド シェルの $home へのパス
myUploadPath=/home/$az_user_name/uploadfiles
```

5. ファイルをコンテナーにバッチ アップロードする

```sh
# パスに html ファイルをアップロードする
az storage blob upload-batch -d $container_name \
-s $myUploadPath -o table
```

6. BLOB 内のファイルを一覧表示し、アップロードされたファイルを確認する

```sh
az storage blob list --container-name $container_name -o table
```

**download-batch**

1. `download` ディレクトリを作成する

> 注: `az_user_name` 変数が上記の指示どおりに設定されていることを確認してください。

2. アップロード パスとフォルダを作成する

```sh
mkdir downloadfiles

myDownloadPath=/home/$az_user_name/downloadfiles
```

3. コンテナからファイルをバッチ ダウンロードする

```sh
az storage blob download-batch -d $myDownloadPath -s $container_name
```

4. ダウンロード ディレクトリにファイルを一覧表示する

```sh
# ダウンロード ファイルを一覧表示
ls downloadfiles
```

**BLOB の URL を取得し、パブリック ファイルを表示する**

1. コマンドを実行し、生成されたリンクをクリックします。

```sh
az storage blob url -c $container_name -n helloAdmin -o tsv
```

**パブリック BLOB コンテナーの削除**

1. BLOB コンテナーのクリーンアップの確認が完了した際

```bash
az storage container delete -n $container_name
```

#### タスク 2: ストレージへのセキュリティ保護されたアクセス

**West ストレージ アカウントにプライベート BLOB コンテナーを作成する**

1. 必要に応じて変数を更新する
1. `AZURE_STORAGE_ACCOUNT` を **編集** して、生成されたアカウント名を反映する

> 注: `container_name` の新しい値は `westblobcontainerprivate` です。

```sh
my_resource_group=WestRG
location=westus
container_name=westblobcontainerprivate

export AZURE_STORAGE_ACCOUNT=<*Enter*Storage*Account*> # $my_storage_account

export AZURE_STORAGE_KEY="$(az storage account keys list \
    --account-name $AZURE_STORAGE_ACCOUNT \
    --resource-group $my_resource_group \
    --query [0].value -o tsv)"

export AZURE_STORAGE_CONNECTION_STRING="$(az storage account \
    show-connection-string -n $AZURE_STORAGE_ACCOUNT\
     -g $my_resource_group --query connectionString -o tsv)"
```

**プライベート アクセスを使用して BLOB コンテナーを作成する**

1. プライベートな新しい BLOB コンテナーを作成する

```sh
az storage container create --name $container_name --public-access blob
```

**ファイルをプライベート BLOB コンテナーにアップロードする**

```bash
echo "<h1>Hello Azure Administrators - This is private</h1>">helloAdmin.html
blob_name=helloAdmin

az storage blob upload \
    --container-name $container_name \
    --file helloAdmin.html --name $blob_name
```

**サービス レベルで Shared Access Signature (SAS) を作成する**

1. `end_date` を設定する

> *注: アクセスは 30 分のみに設定されます。*

```sh
# コンテナーに 30 分間の読み取り専用 SAS トークンを作成し、CONTAINER_SAS_KEY として格納する
end_date=`date -u -d "30 minutes" '+%Y-%m-%dT%H:%MZ'`
```

2. コンテナーおよび BLOB キー変数の生成

```sh
CONTAINER_SAS_KEY="$(az storage container generate-sas \
    --name $container_name --https-only --auth-mode key \
    --expiry $end_date --permissions r -o tsv)"

BLOB_SAS_KEY="$(az storage blob generate-sas \
    --name $blob_name \
    --container-name $container_name \
    --permissions r --expiry $end_date \
    --auth-mode key -o tsv)"
```

3. 完全な Blob URI を生成してリンクをテストする

```sh
# 完全な Blob URI を生成してリンクをテストする
private_URI="$(az storage blob generate-sas \
    --name $blob_name \
    --container-name $container_name \
    --permissions r --auth-mode key \
    --expiry $end_date --https-only \
    --full-uri)"

echo $private_URI
```




