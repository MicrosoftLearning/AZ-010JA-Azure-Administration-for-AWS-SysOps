# ラボ 04 解答キー

## 指示

1. 以下の CLI スクリプトをメモ帳などのエディタにコピーする
   1. `# ----実行する前にこれらの値を編集する----` と名付けられたロケーション セクション
   1. **値を編集する**と、**エラー**が発生します
   1. ファイルをローカルに保存する
1. Azure ポータルにログインし、bash Cloud Shell を開きます。
1. 依存関係が整っているかどうかを確認します (スクリプトのコメントの上部を参照)。
1. ローカル ファイルから CLI スクリプトをコピーし、Bash Cloud Shell にスクリプトを貼り付けます。
1. 一部のタスクは 1 分以上かかる場合があります。スクリプトが完了するのを待ち、出力を確認します。
1. エラーが発生した場合は講師と話しましょう

> `これらの値を編集する` セクションに関する注意事項
>
> `myUploadPath` および `myDownloadPath` で使用する `az_user_name` 変数を作成する
>
> `az_user_name` を設定するには、以下のコマンドを編集する必要があります
>
> Cloud Shell プロンプトで `<name>@azure` の**名前**をキャプチャする
> 例 - クラウド シェル内のプロンプトが `eric@Azure:~$` の場合は **`az_user_name='eric'`** を設定する

```sh
## AZ-010 LAB4 Solution
# ---------------
# 依存関係: 
#   LAB1 - ソリューションを設定済み (WestRG & EastRG 作成済み)
#   LAB2 - ソリューションを設定済み (VNets, SubNets & Peering 作成済み)
#   LAB3 - ソリューションを設定済み (EastDebianVM 作成済み)
# ---------------
# 警告: これらのステップは 1 分以上かかります
# エラーが発生した場合は、講師に相談してください

# ----------開始----------

# ----実行する前にこれらの値を編集する----
subscriptionID=<**subscription ID to use for labs**>

az_user_name=<name>

# ----主なスクリプト----
# ----デフォルトのサブスクリプションを設定する----
az account set --subscription $subscriptionID

# ----一意の名前に使用するランダムな文字列を作成する----

myRand=`head /dev/urandom | tr -dc a-z0-9 | head -c 6 ; echo ''`
echo "the random string append will be:  "$myRand

# ----変数の設定----
my_resource_group=WestRG
location=westus
my_storage_account=weststore$myRand
my_storage_sku=Standard_RAGRS # default
my_storage_kind=StorageV2
my_access_tier=hot
my_storage_encryption=blob

# **************************************************************
echo "Your my_storage_account name will be: " $my_storage_account
# **************************************************************

# ----ストレージ アカウントの作成----
az storage account create \
    -n $my_storage_account \
    -g $my_resource_group \
    -l $location \
    --kind $my_storage_kind \
    --access-tier $my_access_tier \
    --sku $my_storage_sku \
    --encryption $my_storage_encryption

# ----Bash CLI で環境変数を設定する----
export AZURE_STORAGE_ACCOUNT=$my_storage_account
export AZURE_STORAGE_CONNECTION_STRING="$(az storage account show-connection-string \
-n $AZURE_STORAGE_ACCOUNT -g $my_resource_group --query connectionString -o tsv)"

# ----ストレージ アカウント キー環境変数の作成----
# キーの表示
az storage account keys list \
    --account-name $AZURE_STORAGE_ACCOUNT \
    --resource-group $my_resource_group \
    --output table
# ----export AZURE_STORAGE_KEY=<storage_account_key1>----
# ----環境変数として Key 1 を格納する----
export AZURE_STORAGE_KEY="$(az storage account keys list --account-name \
$AZURE_STORAGE_ACCOUNT --resource-group $my_resource_group --query [0].value -o tsv)"

# ----`helloAdmin.html` ファイルを作成して Blob Storage にアップロードする
echo "<h1>Hello Azure Administrators</h1>">helloAdmin.html

# ----パブリック アクセスを使用して BLOB コンテナーを作成する----
container_name=westblobcontainerpublic

az storage container create --name $container_name --public-access blob

# ----ファイルをパブリック BLOB コンテナーにアップロードする----
blob_name=helloAdmin

az storage blob upload \
    --container-name $container_name \
    --file helloAdmin.html \
    --name $blob_name

# ----公開ファイルのダウンロード----
az storage blob download \
    --container-name $container_name \
    --name $blob_name \
    --file helloAdminDownload.html

# ---------upload-batch---------
mkdir uploadfiles

# ----`upload` ディレクトリにアップロードするファイルを作成する----
touch uploadfiles/myfile.html
touch uploadfiles/more.html
touch uploadfiles/hi.html
touch uploadfiles/myfile.txt

# ***********************************************************
# 上記の「編集された変数]セクションで <name> 置き換えられると仮定する
# az_user_name=<name>
# ***********************************************************

# path to $home in cloud shell
myUploadPath=/home/$az_user_name/uploadfiles

# パスに html ファイルをアップロードする
az storage blob upload-batch -d $container_name \
-s $myUploadPath -o table

#----アップロード パスとフォルダを作成する----
mkdir downloadfiles

#----ダウンロード用のパスを作成する----
myDownloadPath=/home/$az_user_name/downloadfiles

#----コンテナからファイルをバッチ ダウンロードする----
az storage blob download-batch -d $myDownloadPath -s $container_name

#----BLOB の URL を取得し、パブリック ファイルを表示する----
#----生成されたリンクをクリックしてファイルを表示する
az storage blob url -c $container_name -n helloAdmin -o tsv

# ----タスク 2: ストレージへのセキュリティ保護されたアクセス----
# ----West ストレージ アカウントにプライベート BLOB コンテナーを作成する----

# ----変数の更新----
container_name=westblobcontainerprivate

# ----ストレージ キー----
export AZURE_STORAGE_KEY="$(az storage account keys list \
    --account-name $AZURE_STORAGE_ACCOUNT \
    --resource-group $my_resource_group \
    --query [0].value -o tsv)"

# ----ストレージ接続キー----
export AZURE_STORAGE_CONNECTION_STRING="$(az storage account \
    show-connection-string -n $AZURE_STORAGE_ACCOUNT\
     -g $my_resource_group --query connectionString -o tsv)"

# ----プライベートな新しい BLOB コンテナーを作成する
az storage container create --name $container_name --public-access blob
# ----ファイルを作成し、プライベート BLOB コンテナーにアップロードする----
echo "<h1>Hello Azure Administrators - This is private</h1>">helloAdmin.html
blob_name=helloAdmin

az storage blob upload \
    --container-name $container_name \
    --file helloAdmin.html --name $blob_name

# ----サービス レベルで Shared Access Signature (SAS) を作成する----
end_date=`date -u -d "30 minutes" '+%Y-%m-%dT%H:%MZ'`

CONTAINER_SAS_KEY="$(az storage container generate-sas \
    --name $container_name --https-only --auth-mode key \
    --expiry $end_date --permissions r -o tsv)"

BLOB_SAS_KEY="$(az storage blob generate-sas \
    --name $blob_name \
    --container-name $container_name \
    --permissions r --expiry $end_date \
    --auth-mode key -o tsv)"


# ----完全な Blob URI を生成してリンクをテストする----
private_URI="$(az storage blob generate-sas \
    --name $blob_name \
    --container-name $container_name \
    --permissions r --auth-mode key \
    --expiry $end_date --https-only \
    --full-uri)"

echo $private_URI
# ***********************************************************************
# ----上記で生成されたリンクをテストする----
# ***********************************************************************


# ---------タスク 3: SubNet サービス エンドポイントを作成する---------
# ----既定のルールを設定してネットワーク アクセスを拒否する (必要な場合)----
az storage account update --resource-group $my_resource_group \
    -n $AZURE_STORAGE_ACCOUNT --default-action Deny

# ----サブネット ルールを更新して、WestVNet - WestSubnet1 上のストレージを有効にします。----
az network vnet subnet update --resource-group $my_resource_group \
    --vnet-name WestVNet --name WestSubnet1 \
    --service-endpoints "Microsoft.Storage"

# ----'subnet_id' 変数の作成
subnet_id="$(az network vnet subnet show \
    --resource-group $my_resource_group \
    --vnet-name WestVNet --name WestSubnet1 \
    --query id --output tsv)"

# ----ネットワーク サブネット ルールを追加します。
az storage account network-rule add --resource-group $my_resource_group \
    -n $AZURE_STORAGE_ACCOUNT --subnet $subnet_id

# ----ストレージ アカウント ネットワーク ルールの更新を一覧表示する----
az storage account network-rule list --resource-group $my_resource_group \
    -n $AZURE_STORAGE_ACCOUNT
# **********************************************************
# ----前のタスクでアクセス可能だったページをテストする
# ----ネットワーク ルールは **アクセスを拒否する**必要があります
echo $private_URI
# 上記リンクをテストする
# *********************************************************

# ----タスク 4 – ファイル ストレージの作成----
# ----Azure CLI を使用して eastus ストレージ アカウントを作成する----

# ----変数の作成----
my_resource_group=EastRG
location=eastus
my_storage_account=eaststore$myRand

#----eastus ストレージ アカウントの作成----
az storage account create --name $my_storage_account \
    --resource-group $my_resource_group --location $location \
    --sku Standard_LRS --kind StorageV2

#----Https 以外のトラフィックを許可する (Linux ドライブをマウントする際に「error 13]を回避する)----
az storage account update -n $my_storage_account --https-only false

#---------ファイル共有を作成してファイルをアップロードする---------
#----環境変数の設定----
export AZURE_STORAGE_ACCOUNT=$my_storage_account
export AZURE_STORAGE_KEY="$(az storage account keys list --account-name $AZURE_STORAGE_ACCOUNT --resource-group $my_resource_group --query [0].value -o tsv)"
export AZURE_STORAGE_CONNECTION_STRING="$(az storage account show-connection-string -n $AZURE_STORAGE_ACCOUNT -g $my_resource_group --query connectionString -o tsv)"
#----`eastfiles` ファイル共有の作成----
file_share_name=eastfiles
#----ストレージ ファイル共有の作成----
az storage share create --name $file_share_name --quota 2048
#----アップロードする 'myFileShareFile.html' ファイルを作成する
echo "File Shares Share Files">myFileShareFile.html
#----共有するファイルをアップロードする----
az storage file upload --share-name $file_share_name --source ~/myFileShareFile.html

# ++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
# ******************************************************************
# ---------------------------手動ステップ---------------------------
#----Linux Virtual Machine からファイル共有をマウントする----
#----Linux VM への SSH 接続----
#----これらのコマンドを Cloud Shell Bash に入力して SSH セッションを開始する----
#> east_vm_ip="$(az vm show -d -g $my_resource_group -n eastdebianvm --query publicIps -o tsv)"
#> ssh azuser@$east_vm_ip
#
#----これらのコマンドを SSH セッションに入力する (または lab04 のタスク 4 を参照)----
#----ディレクトリを作成し、Azure Storage にアタッチする----
#ssh> mkdir -p $my_storage_account/eastfiles
#ssh> sudo apt-get update
#ssh> sudo apt-get install cifs-utils
# ==================================================================
#----Linux マシンにストレージをアタッチするコードをポータルから取得する---
#
#    1. ポータルで次の操作を行います。
#        * eaststorage*123af4* (類似名) ストレージ アカウントを開きます。
#        * eastfiles ファイル共有に移動します。
#    1. 「eastfiles ]ブレードから 「接続] をクリックします。
#    1. **Linux** タブに変更し、接続文字列をコピーします。
#    1. EastDebianVM ssh セッションに戻ります。
#    1. ssh セッションに接続文字列を貼り付ける
#
# ==================================================================
# ====新しい Azure Portal Web ページ タブを開き、Cloud Shell を起動する====
#----共有がマウントされたら、https 以外のトラフィックの許可を停止する (CLI)
#----**Azure CLI** (SSH セッションではありません) にコマンドを入力する
#
#> az storage account update -n $my_storage_account --https-only true
#
# ==================================================================
# =================**Linux SSH セッションに戻る================
#----Linux SSH セッションでは、次のコマンドを続行する----
#
#ssh> cd /mnt/$my_storage_account
#ssh> echo "I am from eastDebianVM">newFile.txt
#----"ls"コマンドを使用して Linux マシン上の共有内のファイルを表示する
#----SSH セッションタイプ "exit" を終了する----
#----ファイル (newFile.txt) が Azure ポータルに存在することを確認する

# =================================================================
# -----------準備ができたらストレージ ラボをクリーンアップする-----------
#----------CLI に戻る (ssh セッションを終了する)---------
#---次のコマンドを使用してファイル ストレージ アカウントを削除する----
#> az storage account delete -n eaststore$myRand -g EastRG
#> az storage account delete -n weststore$myRand -g WestRG
#

> 上記のコマンド スクリプトを確認する
> * 検証手順
> * SSH の手動ステップ
> * 手動クリーンアップ手順

