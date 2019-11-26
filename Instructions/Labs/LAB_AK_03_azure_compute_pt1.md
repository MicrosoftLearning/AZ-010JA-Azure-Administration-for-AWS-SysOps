# ラボ 03 pt 1 解答キー

## 指示

1. 以下の CLI スクリプトをメモ帳などのエディタにコピーする
1. `# ----実行する前にこれらの値を編集する----` と名付けられたロケーション セクション
1. 環境を表す値を編集し、ファイルをローカルに保存します。
1. Azure ポータルにログインし、bash Cloud Shell を開きます。
1. 依存関係が整っているかどうかを確認します (スクリプトのコメントの上部を参照)。
1. ローカル ファイルから CLI スクリプトをコピーし、Bash Cloud Shell にスクリプトを貼り付けます。
1. 一部のタスクは 1 分以上かかる場合があります。スクリプトが完了するのを待ち、出力を確認します。
1. エラーが発生した場合は講師と話しましょう

> 注: 受講生は、ラボ 01 で説明されているように、既定のサブスクリプションを設定する必要があります。

```sh
# AZ-010 LAB3 Solution
# ---------------
# 依存関係: 
#   LAB1 - ソリューションを設定済み (WestRG & EastRG 作成済み)
#   LAB2 - ソリューションを設定済み (VNets, SubNets & Peering 作成済み)
# ---------------
# 警告: これらのステップは 1 分以上かかります
# エラーが発生した場合は、講師に相談してください

# ----------開始----------

# ----実行する前にこれらの値を一意の値になるよう編集----
adminUserName='azuser'
adminPassword='UniqueP@$$w0rd-Here'

# ----変数を指定----
resourceGroupName='WestRG'
location='westus'
vmName='WestWinVM'
vmSize='Standard_D1'
availabilitySet='WestAS'

# ----主なスクリプト----

# ----可用性セットを作成----
az vm availability-set create \
  --name $availabilitySet \
  --resource-group $resourceGroupName \
  --location $location

# ----WestWinVM VM を作成----
az vm create --name $vmName --resource-group $resourceGroupName \
  --image win2016datacenter \
  --admin-username $adminUserName \
  --admin-password $adminPassword \
  --location $location \
  --size $vmSize \
  --availability-set $availabilitySet

# ----ポートを開く----
az vm open-port -g WestRG -n $vmName --port 80 --priority 1500
az vm open-port -g WestRG -n $vmName --port 3389 --priority 2000

# ----ICMPv4-In を許可する (PowerShell を実行している Bash CLI)----
az vm extension set --publisher Microsoft.Compute \
--version 1.8 --name CustomScriptExtension \
--vm-name $vmName --resource-group $resourceGroupName \
--settings '{"commandToExecute":"powershell.exe New-NetFirewallRule –DisplayName “Allow ICMPv4-In” –Protocol ICMPv4"}'

# ----IIS のインストール (PowerShell を実行している Bash CLI)----
az vm extension set --publisher Microsoft.Compute \
--version 1.8 \
--name CustomScriptExtension \
--vm-name $vmName \
--resource-group $resourceGroupName \
--settings '{"commandToExecute":"powershell.exe Install-WindowsFeature -name Web-Server -IncludeManagementTools"}'

# ----IIS サーバーが VM Public IP Address で実行されていることを確認する----
az vm show -d -g $resourceGroupName -n $vmName --query publicIps -o tsv

# ***********************************************************
# ----ブラウザーで IP アドレスの上に貼り付けて、IIS が実行されているかどうかを確認します----
# ***********************************************************

# ----DNS で構成された Debian 仮想マシンを作成する----
# ----WestDebianVM を作成する----
az vm create \
--image credativ:Debian:8:latest \
--admin-username azuser \
--resource-group WestRG \
--vnet-name WestVNet \
--subnet WestSubnet1 \
--availability-set WestAS \
--size 'Standard_D1' \
--location westus \
--name WestDebianVM \
--generate-ssh-keys

# ----EastAS 可用性セットの作成----
az vm availability-set create --name EastAS --resource-group EastRG

# ----EastDebianVM を作成する----
az vm create \
--image credativ:Debian:8:latest \
--admin-username azuser \
--resource-group EastRG \
--vnet-name EastVNet \
--subnet EastSubnet2 \
--availability-set EastAS \
--size 'Standard_D1' \
--location eastus \
--name EastDebianVM \
--generate-ssh-keys

# ----Debian マシンの DNS を構成する----
myRand=`head /dev/urandom | tr -dc a-z0-9 | head -c 6 ; echo ''`
echo "the random string append will be:  "$myRand

# ----DNS の構成----
az network public-ip update --resource-group WestRG --name WestDebianVMPublicIP --dns-name westdebianvm$myRand

az network public-ip update --resource-group EastRG --name EastDebianVMPublicIP --dns-name eastdebianvm$myRand

# ----両方の Debian VM が実行されていることを確認する----
az vm get-instance-view --name WestDebianVM --resource-group WestRG --query instanceView.statuses[1] --output table

az vm get-instance-view --name EastDebianVM --resource-group EastRG --query instanceView.statuses[1] --output table

# ----SSH と Ping テスト WestWinVM を使用して WestDebianVM に接続する。----
# ----WestRG の Debian 仮想マシンに接続します----

# ----VM の Public IP Addresses と Private IP Addresses を取得します----
WestDebianIP=$(az vm list-ip-addresses -n WestDebianVM --query "[*].virtualMachine.network.publicIpAddresses[0].ipAddress" -o tsv)

EastDebianIP=$(az vm list-ip-addresses -n EastDebianVM --query "[*].virtualMachine.network.publicIpAddresses[0].ipAddress" -o tsv)

WestWinIP=$(az vm list-ip-addresses -n WestWinVM --query "[*].virtualMachine.network.publicIpAddresses[0].ipAddress" -o tsv)

#----WestDebianVM 仮想マシンへの SSH----

# *******************************************
# SSH セッションを開始するには、以下のコマンドを使用します。
echo "For SSH to WestDebianVM use: " 'ssh' $adminUserName'@'$WestDebianIP
echo "EastDebianVM IP: " $EastDebianIP
echo "WestWinVM IP: " $WestWinIP
# *******************************************

# *******************************************
# --------SSH セッション コマンドを開始します--------
# --ラボ 03 演習 1 タスク 4 を参照してください。 Ping VMs--
# --SSH 接続と IP 値に注意してください--
# *******************************************
```

**演習1 タスク 4 を完了します: SSH と Ping**

**ラボ 03 回答キー ソリューション パート2を LAB_AK_03_azure_compute_pt2.md で続行する**
