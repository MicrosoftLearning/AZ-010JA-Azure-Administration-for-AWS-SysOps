---
lab:
    title: 'Azure VM と スケール セット'
    module: 'モジュール 3: Azure コンピューティング'
---
    
# ラボ 03: Azure コンピューティング

## 受講者ラボ マニュアル

## シナリオ

このラボでは、Bash インターフェイス上で Cloud Shell の CLI コマンドを使用して、可用性セット内で Azure Windows と Debian VM を管理します。また、スケール インとスケール アウトを示すオプションのテストを含む Ubuntu スケール セットを作成します。

## 目的

この実習ラボを完了すると、Azure CLI を使用して次の操作を行うことができるようになります。

* スケール セットを作成する
* Windows および Linux VM の作成と構成
* Ubuntu スケール セットを構成する

## ラボのセットアップ

* **予想時間**: 60 分

## 指示

### 開始する前に

#### セットアップ タスク

1. **前のラボへの依存関係:**
    1. モジュール 1: Azure 管理 - **ラボ: リソース グループの作成**。EastRG および WestRG リソース グループが構成されました。
    1. モジュール 2: Azure ネットワーク - **ラボ仮想ネットワークとピアリング**: サブネットとピアリングが構成された VNet。

### エクササイズ 1: 可用性セット内で構成された VM を作成する

このエクササイズの主なタスクは次のとおりです。

1. 可用性セットの作成
1. Windows 仮想マシンを作成する
1. Linux Debian 仮想マシンの作成

#### タスク 1: Windows 仮想マシンを作成する

* 可用性セットの作成
* Create Windows Server 2016 DataCenter VM
* WebServer として構成する

**変数の準備**

1. **Cloud Shell** コマンド プロンプトで、次のコマンドを入力し、**Enter** キーを押して、次のスクリプトで使用する変数を作成します。

> **注**: 一意のパスワードを作成し、書き留めましょう

```sh
resourceGroupName='WestRG'
location='westus'
adminUserName='azuser'
adminPassword='UniqueP@$$w0rd-Here' # 一意の値を作成する
vmName='WestWinVM'
vmSize='Standard_D1'
availabilitySet='WestAS'
```

**可用性セットの作成**

1. 次のコマンドを入力し、**WestAS** 可用性セットを作成します。

```sh
az vm availability-set create \
  --name $availabilitySet \
  --resource-group $resourceGroupName \
  --location $location
```

**WestWinVM VM の作成**

1. 次のコマンドを入力 して、**WestWinVM** Windows Server VM を作成します。

```sh
az vm create --name $vmName --resource-group $resourceGroupName \
  --image win2016datacenter \
  --admin-username $adminUserName \
  --admin-password $adminPassword \
  --size $vmSize \
  --location $location \
  --availability-set $availabilitySet
```

**ポートを開く**

1. 次のコマンドを入力して、WestWinVM でポートを開きます

```sh
az vm open-port -g WestRG -n $vmName --port 80 --priority 1500
az vm open-port -g WestRG -n $vmName --port 3389 --priority 2000
```

**作業を確認する**

1. Azure Portal を起動して WestRG に移動します
2. WestWinVM を含むリソースに注意してください
3. Azure Advisor を起動し、推奨事項に注意してください

#### タスク 2: WestWinVM を Web サーバーとして構成し、Ping を許可する

**ICMPv4-In を許可する (PowerShell を実行している Bash CLI)**

1. **Cloud Shell** コマンド プロンプトで、次のコマンドを入力し、**Enter** キーを押して、PowerShell コマンドを WestWinVM Windows Server に送信して ICMPv4-In を許可します。

> **注**: Cloud Shell がタイムアウトした場合は、タスク 1 から変数を更新する必要があります。

```sh
az vm extension set --publisher Microsoft.Compute \
--version 1.8 --name CustomScriptExtension \
--vm-name $vmName --resource-group $resourceGroupName \
--settings '{"commandToExecute":"powershell.exe New-NetFirewallRule –DisplayName “Allow ICMPv4-In” –Protocol ICMPv4"}'
```

**IIS のインストール (PowerShell を実行している Bash CLI)**

1. 次のコマンドを入力して、PowerShell コマンドを WestWinVM Windows サーバーに送信する IIS をインストールします。

```sh
az vm extension set --publisher Microsoft.Compute \
--version 1.8 \
--name CustomScriptExtension \
--vm-name $vmName \
--resource-group $resourceGroupName \
--settings '{"commandToExecute":"powershell.exe Install-WindowsFeature -name Web-Server -IncludeManagementTools"}'
```

**IIS サーバーが VM Public IP Address で実行されていることを確認する**

1. 次のコマンドを入力して、WestWSinVM Public IP Address をキャプチャします。

```sh
az vm show -d -g $resourceGroupName -n $vmName --query publicIps -o tsv

# ----ブラウザーで IP アドレスの上に貼り付けて、IIS が実行されているかどうかを確認します----
```

2. 結果の IP アドレスを Web ブラウザーに貼り付け、IIS の既定ページが存在することを確認します。

#### タスク 3: DNS で構成された Debian 仮想マシンを作成する

Cloud Shell で 2 つの Debian 仮想サーバーを作成し、SSH を使用してサーバーに接続する

マシン 1: **WestDebianVM**

> - WestRG リソース グループ
>   - WestVNet
>      - WestSubnet1

および

マシン 2: **EastDebianVM**

> - East リソース グループ
>   - EastVNet
>     - EastSubnet2 サブネット

**WestDebianVM を作成する**

1. 次のコマンドを入力して、WestDebianVM を作成します。

```sh
az vm create \
--image credativ:Debian:8:latest \
--size 'Standard_D1' \
--admin-username azuser \
--resource-group WestRG \
--vnet-name WestVNet \
--subnet WestSubnet1 \
--availability-set WestAS \
--location westus \
--name WestDebianVM \
--generate-ssh-keys
```

2. 注: shh キーが生成され (存在しない場合)、`~/ssh` ディレクトリに保存されます。Cloud Shell に次のコマンドを入力すると、`~/.ssh` ディレクトリの内容が表示されます。

```sh
ls .ssh
```

**EastAS 可用性セットの作成**

1. 次のコマンドを入力して、EastAS を作成します。

```sh
az vm availability-set create --name EastAS --resource-group EastRG
```

**EastDebianVM を作成する**

1. 次のコマンドを入力して、EastDebianVM を作成します。

```sh
az vm create \
--image credativ:Debian:8:latest \
--size 'Standard_D1' \
--admin-username azuser \
--resource-group EastRG \
--vnet-name EastVNet \
--subnet EastSubnet2 \
--availability-set EastAS \
--location eastus \
--name EastDebianVM \
--generate-ssh-keys
```

**Debian マシンの DNS を構成する**

1. 次のコマンドを入力して、Bash を使用して一意の DNS 名を割り当てる場合に使用するランダム文字列を作成します。

```sh
myRand=`head /dev/urandom | tr -dc a-z0-9 | head -c 6 ; echo ''`
echo "the random string append will be:  "$myRand
```

**DNS の構成**

1. Linux Debian VM に一意の DNS 名を割り当てるには、次のコマンドを入力します。

```sh
az network public-ip update --resource-group WestRG --name WestDebianVMPublicIP --dns-name westdebianvm$myRand

az network public-ip update --resource-group EastRG --name EastDebianVMPublicIP --dns-name eastdebianvm$myRand
```

2. Linux Debian VM の DNS 名に注意してください。

**両方の Debian VM が実行されていることを確認する**

1. 次のコマンドを入力すると、Linux Debian VM のステータスが表示されます (通常は実行中)。

```sh
az vm get-instance-view --name WestDebianVM --resource-group WestRG --query instanceView.statuses[1] --output table

az vm get-instance-view --name EastDebianVM --resource-group EastRG --query instanceView.statuses[1] --output table
```

#### タスク 4: SSH と Ping テスト WestWinVM を使用して WestDebianVM に接続する。

**West リソース グループの Debian 仮想マシンに接続する**

1. 次のコマンドを入力して、VM の Public IP Addresses と Private IP Addresses を取得します。

```sh
az vm list-ip-addresses --resource-group WestRG

az vm list-ip-addresses --resource-group EastRG
```

2. EastDebianVM、WestDebianVM、WestWinVM IPアドレスの値に注意し、**IP を記録します** (またはポータルの各 VM の概要ページで検索します)。

**WestDebianVM 仮想マシンへの SSH**

1. WestDebianVM IP アドレスを使用して、次のコマンドを編集して SSH セッションを開始します。

```sh
ssh azuser@<PUBLIC IP address of West Debian VM>
```

**SSH セッションから Windows VM プライベート アドレスを Ping する**

> *両方同じ **非公開** VNet上にあるため、これは通常機能します。*

1. WestWinVM IP アドレスを使用して、次のコマンドを編集し、Cloud Shell SSH セッションに入力して、WestWinVM に Ping を実行します。

```sh
ping <PRIVATE IP address of the Windows server>
```

**Ping EastDebianVM**

1. EastDebianVM IP アドレスを使用して、次のコマンドを編集し、Cloud Shell SSH セッションに入力して、WestWinVM に Ping を実行します。

> *これは、既に VNet ピアリングが構成されているため、通常機能します。*

```sh
ping <PRIVATE IP address of eastdebianvm>
```

> **注**: `exit` を入力して、SSH を終了します。

### エクササイズ 2: Ubuntu スケール セットの作成とテスト

このエクササイズの主なタスクは次のとおりです。

1. Ubuntu スケール セットと自動スケーリング プロファイルを作成する
1. ルールで自動スケーリング アウトと自動スケーリング インを作成する
1. Ubuntu スケール セットをテストする (オプション)

#### タスク 1: Ubuntu スケール セットを作成する

**スケール セット リソース グループを作成する**

1. 次のコマンドを入力して、スケール セットのリソース グループを作成します。

```sh
az group create --name EastScaleRG --location eastus
```

**スケール セットを作成する**

1. 次のコマンドを入力して、スケールセットを作成します

```sh
az vmss create \
  --resource-group EastScaleRG \
  --name EastUbuntuServers \
  --image UbuntuLTS \
  --upgrade-policy-mode automatic \
  --instance-count 2 \
  --admin-username azuser \
  --generate-ssh-keys
```

ポータル ダッシュボードを表示し、スケール ルール前の 2 つのサーバーをメモする

> - リソース グループ
>   - EastScaleRG
>     - EastUbuntuServers > インスタンス

**自動スケーリング プロファイルの定義**

1. 次のコマンドを入力して、自動スケーリング プロファイルを定義します。

```sh
az monitor autoscale create \
  --resource-group EastScaleRG \
  --resource EastUbuntuServers \
  --resource-type Microsoft.Compute/virtualMachineScaleSets \
  --name autoscale \
  --min-count 1 \
  --max-count 3 \
  --count 1
```

> 「スケーリングルールを追加するには、`az monitor autoscale rule create` でフォローアップしてください」というメッセージが表示されます。

**自動スケーリング アウト ルールを作成する**

1. 次のコマンドを入力 して、**自動スケーリング アウト** ルールを作成します。

```sh
az monitor autoscale rule create \
  --resource-group EastScaleRG \
  --autoscale-name autoscale \
  --condition "Percentage CPU > 50 avg 2m" \
  --scale out 1
```

**自動スケーリング イン ルールを作成する**

1. 次のコマンドを入力して、**自動スケーリング イン** ルールを作成します。

```sh
az monitor autoscale rule create \
  --resource-group EastScaleRG \
  --autoscale-name autoscale \
  --condition "Percentage CPU < 30 avg 1m" \
  --scale in 1
```

> *注: 「scale in」は最短 2 分で発生しうるので、以下の結果は最初の数分で異なると予想されます*

#### タスク 2: Ubuntu スケール セットをテストする (オプション タスク)

> このタスクは、スケール セットの動作を示すテストとして、サーバーに対する CPU 負荷を生成します。

**スケール セットで実行されているサーバーを一覧表示する**

* 最初の 1 分間に、スクリプトに 2 つのサーバー インスタンス (0、1 など) が一覧表示されます。
* 1 分後に 1 つのサーバー インスタンス (0 など) が一覧表示されます。

1. 次のコマンドを入力して、サーバーを一覧表示します
1. 1 つのサーバー インスタンスのみが表示されるまで、30 秒後にコマンドを **繰り返します**。

```sh
az vmss list-instance-connection-info \
--resource-group EastScaleRG \
--name EastUbuntuServers
```

**SSH を使用してインスタンス 0 に接続する**

1. ポータルを使用して、スケール セットのインスタンス 0 の IP アドレスを取得します。
1. 次の編集コマンドを入力して SSH に接続する

```sh
# 例: ssh azuser@13.92.224.66 -p 50000  
ssh azuser@<instance 0 IP> -p 50000
```

**4 分間のストレスを実行する (240 秒)**

1. SSH セッションで次のコマンドを入力して、ストレス アプリケーションをインストールします (240 秒でタイムアウト)

```sh
sudo apt-get -y install stress
sudo stress --cpu 10 --timeout 240 &
```

**SSH セッションでストレスを確認する**

1. SSH で次のコマンドを入力して、ストレスを監視します。

```sh
top
```

**終了および SSH**

```sh
# ctrl-c
exit
```

**自動スケーリング アウトと自動スケーリング インを監視する**

1. 次のコマンドを入力して、自動スケーリングで監視を実行します。

```sh
watch az vmss list-instances \
  --resource-group EastScaleRG \
  --name EastUbuntuServers \
  --output table
```

> **注**: ストレスが 50% 以上の負荷の登録を開始するまで数分かかる場合があります。
>
> **注**: ctrl + c を押して「watch」を閉じる

**クリーンアップ スケール セット デモ**

```sh
az group delete --name EastScaleRG --yes --no-wait
```
