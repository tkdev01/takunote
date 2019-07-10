+++
author = "Taku"
categories = ["VM"]
date = "2019-07-09"
description = "VID / VMDKファイルの圧縮"
featured = "pic02.jpg"
featuredalt = "Pic 2"
featuredpath = "date"
linktitle = " fff"
title = "VirtualBox 仮想ディスクの圧縮"
type = "post"

+++

# VirtualBox：仮想ディスクの圧縮(vmdkファイル圧縮したい方向け)

Vagrant+VirtualBox(Winsows10)の使用で仮想ディスクのサイズが膨れ上がってきたので、
ダイエット開始。

## 流れ
【ゲストOS側】  
1. ディスククリーンアップ  
2. ドライブの空き領域を0埋め  

【ホストOS側】  
ファイル形式が**VDI**の場合  
1. VBoxManageコマンドで仮想ドライブの圧縮

ファイル形式が**VMDK**の場合 
1. vmdkをvdiに変換  
2. vdiファイル圧縮  
3. 圧縮されたvdiファイルをvmdkに変換  
4. 設定  
5. 既存ファイルの削除  

## 注意
スナップショットを撮っている場合はこの方法はあまり効果がありません。
可能であればスナップショットを削除することをおすすめします。

参考：[VirtualBoxのスナップショット使用時は肥大化するディスク容量に気をつけられたし](https://www.lanches.co.jp/blog/2881)

## 【ゲストOS作業】
1. ディスククリーンアップを利用する。  
参考：[WindowWindowsのディスククリーンアップの使い方](https://freesoft.tvbok.com/tips/optimise_vista/disk_cleanup.html)

2. SDeleteのダウンロード(Linuxの場合は不要　ddコマンド使用)  
OSがファイルを削除する場合、実際にはファイルのインデックス情報を削除するだけで、
ファイル自体は削除されないそうです。SDeleteツールを用いれば、仮想ディスクの空き領域を0埋めすることができます。  
[SDelete](https://technet.microsoft.com/ja-jp/sysinternals/sdelete.aspx)  

3. zipファイルを展開し、そのディレクトリでコマンドプロンプトを開き、以下コマンドを入力  
結構時間がかかります。(´・ω・｀)

```java
#Cドライブの空き領域を0埋めする

＃x86
sdelete.exe -z C:
＃x64
sdelete64.exe -z C:

#linux ddコマンド使用
$ dd if=/dev/zero of=zero bs=4k; \rm zero
```
完了したら、仮想OSのシャットダウン。
ホストOS作業手順を実行する。

## 【ホストOS作業】
以下の圧縮作業にはVBoxManageコマンドを使用します。
1. 圧縮する仮想ドライブのUUID取得  
「C:\Program Files\Oracle\VirtualBox」でコマンドプロンプトを開き、以下のコマンドを入力。  

```
#Windows
VBoxManage.exe list hdds

＃linux
$ vboxmanage list hdds

#以降Linuxの場合は「vboxmanage [コマンド]」と読み替えてください。
```

2. 仮想ドライブの圧縮  
UUIDを指定して仮想ドライブを圧縮します。
```
VBoxManage.exe modifyhd [UUID] --compact
```
vdiファイル圧縮であれば以上です。

※vmdkファイルの場合  
vmdkファイルは対応していないと怒らえます。
```
0%...
Progress state: VBOX_E_NOT_SUPPORTED
VBoxManage.exe: error: Compact medium operation for this format is not implemented yet!
```
別の手法を取りましょう。  
以下手順に進む。

3. vmdkからvdiへ変換  
以下コマンドでvmdkからvdiファイルのクローンを行う。
```
#vdiに変換(パス、ファイル名は適宜変更してください。)
VBoxManage clonehd "F:\VirtualBox VMs\win_10_x64/[ファイル名].vmdk" "F:\VirtualBox VMs\win_10_x64/clone.vdi" --format vdi
```
クローンが完了すると、UUIDが表示される。(後の手順で使用)
```
Clone medium created in format 'vdi'. UUID: 9f86084a-95a9-4a22-a430-76c68983f49c
```

4. 仮想ドライブの圧縮    
UUIDを指定して仮想ドライブを圧縮します。
```
VBoxManage.exe modifyhd [UUID] --compact
```

5. 元ファイルのリネーム
```
rename [ファイル名].vmdk [ファイル名].vmdk_bk
```

6. クローンしたvdiファイルをvmdkに変換
```
VBoxManage.exe clonehd "F:\VirtualBox VMs\win_10_x86/clone.vdi" "F:\VirtualBox VMs\win_10_x86/[ファイル名].vmdk" --format vmdk
```

7. VM VirtualBoxマネージャーを起動し、設定を開く。  
対象VM選択＞設定＞ストレージ  
//TODO このあたりもコマンドでやりたい。  
　時間があれば調べる。

8. 既存の仮想ハードディスクを削除

9. 今回作成した仮想ハードディスクを追加  
追加アイコン＞既存のディスク選択＞[ファイル名].vmdk選択＞OK  
※起動するか確かめたほうが良いかも

10. 元のvmdkとクローンしたvdiファイルを削除

以上。

自分自身の内臓ストレージもどうにかしたい..
