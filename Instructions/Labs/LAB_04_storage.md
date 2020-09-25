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

* **予想時間**: 60 分

## 指示

### 開始する前に

#### セットアップ タスク

1. **前のラボへの依存関係:**
    1. モジュール 1: Azure 管理 - **ラボ: リソース グループの作成**: EastRG および WestRG リソース グループが構成されました。
    1. モジュール 2: Azure ネットワーク - **ラボ仮想ネットワークとピアリング**: サブネットとピアリングが構成された VNet。
    1. モジュール 3: Azure コンピューティング - **ラボ: Azure VM**: EastDebianVM が作成されました。

## エクササイズ 1: BLOB、セキュア アクセス ストレージ、サービス エンドポイント、File Storage

このエクササイズの主なタスクは次のとおりです。

1. Blob Storage の作成と構成
1. セキュリティで保護されたアクセス ストレージ (SAS) を使用して BLOB を構成する
1. SubNet サービス エンドポイントを構成する
1. File Storage の作成と構成

### エクササイズ 1 - タスク 1: Blob Storage でストレージ アカウントを作成する

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
my_storage_kind=StorageV2
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
    --encryption-services $my_storage_encryption
```

**Bash CLI で環境変数を設定する**

1. 以下の環境変数の作成
   * ストレージ アカウント
   * ストレージ アカウント接続文字列


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

**出力ファイルのダウンロード**

1. 以前にアップロードしたファイルをダウンロードする
2. `helloAdmin.html` が新しい名前 `helloAdminDownload.html` でダウンロードされることを確認する

```sh
az storage blob download \
    --container-name $container_name \
    --name $blob_name \
    --file helloAdminDownload.html
    #--account-name $AZURE_STORAGE_ACCOUNT \
```

3. `ls` を実行してファイル `helloAdminDownload.html` がダウンロードされたことを確認する

```sh
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
az_user_name='<name>'
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

1. コマンドを実行する
2. 生成されたリンクをクリックして、ファイルを表示する

```sh
az storage blob url -c $container_name -n helloAdmin -o tsv
```

**パブリック BLOB コンテナーの削除**

1. BLOB コンテナーのクリーンアップの確認が完了した際

```sh
az storage container delete -n $container_name
```

### タスク 2: ストレージへのセキュリティ保護されたアクセス

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

```sh
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
```

3. BLOB キー変数の生成

```sh
BLOB_SAS_KEY="$(az storage blob generate-sas \
    --name $blob_name \
    --container-name $container_name \
    --permissions r --expiry $end_date \
    --auth-mode key -o tsv)"
```

4. 完全な Blob URI を生成してリンクをテストする

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

5. 前の手順で生成された URI リンクをテストして、正常に動作することを確認す
### タスク 3: SubNet サービス エンドポイントを作成する
**ストレージ アカウントの既定のルールの状態を表示する**
1. ストレージ アカウントの既定のルールの状態を表示するには、次の CLI コマンドを入力します。

```sh
az storage account show --resource-group $my_resource_group \
    -n $AZURE_STORAGE_ACCOUNT --query networkRuleSet.defaultAction
```

**既定のルールを設定してネットワーク アクセスを拒否する (必要な場合)**

1. 現在のルールが `Allow` に設定されている場合は、ルールを `Deny` に設定する

```sh
az storage account update --resource-group $my_resource_group \
    -n $AZURE_STORAGE_ACCOUNT --default-action Deny
```

**ストレージ アカウント ネットワーク ルールを一覧表示する**

1. ネットワーク ルールの一覧を表示し、確認します。

```sh
az storage account network-rule list --resource-group $my_resource_group \
    -n $AZURE_STORAGE_ACCOUNT
```

**既存の vnet (WestVNet) およびサブネット (WestSubnet1) 上の Azure Storage のサービス エンドポイントを有効にする**

1. サブネット ルールを更新して、WestVNet - WestSubnet1 上のストレージを有効にします。

```sh
az network vnet subnet update --resource-group $my_resource_group \
    --vnet-name WestVNet --name WestSubnet1 \
    --service-endpoints "Microsoft.Storage"
```

**仮想ネットワークとサブネットのネットワーク ルールを追加する**

> **注**: 事前に既定のルールを拒否するように設定しておく必要があります。そうでない場合、ネットワーク ルールは適応されません。

1. 'subnet_id' 変数の作成

```sh
subnet_id="$(az network vnet subnet show \
    --resource-group $my_resource_group \
    --vnet-name WestVNet --name WestSubnet1 \
    --query id --output tsv)"
```

2. ネットワーク サブネット ルールを追加します。

```sh
az storage account network-rule add --resource-group $my_resource_group \
    -n $AZURE_STORAGE_ACCOUNT --subnet $subnet_id
```

**ストレージ アカウント ネットワーク ルールの更新を一覧表示する**

1. ネットワーク ルールの一覧を表示し、もう一度確認します。

```sh
az storage account network-rule list --resource-group $my_resource_group \
    -n $AZURE_STORAGE_ACCOUNT
```

**以前のタスクでアクセス可能だったページをテストします。問題がなければネットワーク ルールはアクセスを拒否します。**

1. テストするリンクをクリックします。
2. 結果は "access denied" となる必要があります。

```sh
echo $private_URI
```

**ストレージ アカウントのクリーンアップ (West)**

1. 確認が完了したら、ストレージ アカウント リソースを削除します。

```sh
az storage account delete -n $my_storage_account -g my_resource_group
```

### タスク 4 – File Storageの作成

**Azure CLI を使用してストレージ アカウントを作成する**

1. 一意の名前に使用するランダムな文字列を作成する (まだメモリにない場合)

```sh
myRand=`head /dev/urandom | tr -dc a-z0-9 | head -c 6 ; echo ''`
echo "the random string append will be:  "$myRand
```

2. 変数の作成

```sh
my_resource_group=EastRG
location=eastus
my_storage_account=eaststore$myRand
```

3. eastus ストレージ アカウントを作成する

```sh
az storage account create --name $my_storage_account \
    --resource-group $my_resource_group --location $location \
    --sku Standard_LRS --kind StorageV2
```

4. Https 以外のトラフィックを許可する (Linux ドライブをマウントする際に「error 13]を回避する)

```sh
az storage account update -n $my_storage_account --https-only false
```

5. ストレージ アカウントを一覧表示する
6. 新しく作成したアカウントをメモする

```sh
az storage account list -o table
```

**ファイル共有を作成してファイルをアップロードする**

1. 環境変数を設定する

```sh
export AZURE_STORAGE_ACCOUNT=$my_storage_account
export AZURE_STORAGE_KEY="$(az storage account keys list --account-name $AZURE_STORAGE_ACCOUNT --resource-group $my_resource_group --query [0].value -o tsv)"
export AZURE_STORAGE_CONNECTION_STRING="$(az storage account show-connection-string -n $AZURE_STORAGE_ACCOUNT -g $my_resource_group --query connectionString -o tsv)"
```

2. `eastfiles` ファイル共有を作成する

```sh
file_share_name=eastfiles

az storage share create --name $file_share_name --quota 2048
```

3. アップロードする `myFileShareFile.html` ファイルを作成する

```sh
echo "File Shares Share Files">myFileShareFile.html
```

4. 共有するファイルをアップロードする

```sh
az storage file upload --share-name $file_share_name --source ~/myFileShareFile.html
```

5. Azure File Storage 内のファイルの一覧を出力する
6. 'myFileShareFile.html' が File Storage に格納されます

```sh
az storage file list --share-name $file_share_name -o table
```

**Linux Virtual Machine からファイル共有をマウントする**

1. Linux VM への SSH 接続

```sh
east_vm_ip="$(az vm show -d -g $my_resource_group -n eastdebianvm --query publicIps -o tsv)"

ssh azuser@$east_vm_ip
```

> 注: VM SSH セッションでコマンドを実行して、ストレージアタッチの準備を行う

2. Linux VM 上に eastfiles 用ディレクトリを作成する

```sh
mkdir -p $my_storage_account/eastfiles
```

3. Linux VM に cifs-utils をインストールする

```sh
sudo apt-get update

sudo apt-get install cifs-utils
```

4. Linux マシンにストレージをアタッチするコードをポータルから取得する

    1. ポータルで次の操作を行います。
        * eaststorage*123af4* (類似名) ストレージ アカウントを開きます。
        * eastfiles ファイル共有に移動します。
    1. 「eastfiles ]ブレードから 「接続] をクリックします。
    1. **Linux** タブに変更し、接続文字列をコピーします。
    1. EastDebianVM ssh セッションに戻ります。
    1. ssh セッションに接続文字列を貼り付ける

> ```sh
> # ポータルからの接続文字列は次のようになります
> sudo mkdir /mnt/eaststorage123af4
> if [ ! -d "/etc/smbcredentials" ]; then
> sudo mkdir /etc/smbcredentials
> fi
> if [ ! -f "/etc/smbcredentials/eaststorage123af4.cred" ]; then
>     sudo bash -c 'echo "username=eaststorage123af4" >> /etc/smbcredentials/eaststorage123af4.cred'
>     sudo bash -c 'echo "password=EEBKMxGPWyTHe5CKEU58d1PCEdU2gOMHWb4nRZM07RlT5dpk6yiY0nFHCeN4HQOqNr8HLuKhQqa05m4qrAXJrg==" >> /etc/smbcredentials/eaststorage123af4.cred'
> fi
> sudo chmod 600 /etc/smbcredentials/eaststorage123af4.cred
> 
> sudo bash -c 'echo "//eaststorage123af4.file.core.windows.net/eastfiles /mnt/eaststorage123af4 cifs nofail,vers=3.0,credentials=/etc/smbcredentials/eaststorage123af4.cred,dir_mode=0777,file_mode=0777,serverino" >> /etc/fstab'
> 
> sudo mount -t cifs //eaststorage123af4.file.core.windows.net/eastfiles /mnt/eaststorage123af4 -o vers=3.0,credentials=/etc/smbcredentials/eaststorage123af4.cred,dir_mode=0777,file_mode=0777,serverino
> ```

**ファイルが Azure から VM に共有されていることを確認する**

1. 共有がマウントされたら、https 以外のトラフィックの許可を停止する (CLI)
    * **新しい Azure Portal Web ページ**インスタンスを開き、Cloud Shell を起動する
    * **Azure CLI** (SSH セッションではありません) にコマンドを入力する

```sh
az storage account update -n $my_storage_account --https-only true
```

> 注: 次のコマンドの **Linux SSH セッション**に戻る

2. Linux SSH セッションでマウントディレクトリに移動する

```sh
# eastDebianVM SSH から
cd /mnt/eastfiles
```

3. Azure File Storage から VM に共有されているファイルを表示する

```sh
# 共有ファイルを確認する
ls
```

4. VM から新しいファイルを追加する

```sh
# VM から新しいファイルを作成する
echo "I am from eastDebianVM">newFile.txt
```

5. SSH セッションを `終了` する

```sh
# VM ssh セッションから
exit
```

6. Azure Portal でファイル (newFile.txt) を確認し、共有が両方向に動作することを確認します。

**File Storage リソースをクリーンアップする**

1. ストレージ アカウントを削除する

```sh
az storage account delete -n $my_storage_account -g $my_resource_group
```


