---
author: "bloodnighttw"
title: "Arch Linux Installation"
date: "2021-02-10"
description: "arch linux what i though"
tags: ["arch","archLinux","linux"]
categories: ["arch"]
series: ["archlinux"]
ShowToc: true
TocOpen: true
weight: 2
---

# Arch Linux 安裝紀錄 


## 前言

總之我成功了
但 直播軟體出點狀況
無法用 YT 做紀錄
就先用 Hackmd 記錄著
順便檢驗自己對安裝的熟悉度

![](https://i.imgur.com/VxuXLni.png)

By the way 我一直卡在 bootloader 的設定安裝
總共失敗 六次 我都快把指令背下來了
不過我克服了 感謝網友 Arch Wiki 和 網路上提供較學的 DistroTube
然後，熟悉過後應該會裝到實體電腦上

**那麼，進入正題**
- - -
## 第一步

下載 Arch Linux 的 ISO
接著就是讓他能開機
因為我是用 VirturalBox 做練習
如果是要裝在實體電腦上
我推薦用下面這個軟體製作開機碟
[balenaEtcher](https://www.balena.io/etcher/)

## 第二步
開始打指令(幹話)
先設定時間的東西
(照著Arch Wiki 打就對了)
```
$ timedatectl set-ntp true
```
ntp 是什麼 可以可以參考這篇[維基百科](https://zh.wikipedia.org/zh-tw/%E7%B6%B2%E8%B7%AF%E6%99%82%E9%96%93%E5%8D%94%E5%AE%9A)

## 第三步
切割分割區
我是使用cfdisk
因為是使用 efi 開機
所以切三個分割區

    /dev/sda1 EFI System
    /dev/sda2 Swap
    /dev/sda3 Linux File System

接著就是格式化這些分割區

分別打入
```
$ mkfs.vfat /dev/sda1
$ mkswap /dev/sda2
$ mkfs.ext4 /dev/sda3
```

接著掛載
```
$ mount /dev/sda3 /mnt
$ swapon /dev/sda2
```
***!!注意!!EFI 分割區 在這邊還不需要掛載***

## 第四步
設定 Mirror
我這邊跟Arch Wiki 不太一樣
我有用到 ISO 內提供的腳本 reflector 取得來自台灣的 mirror

```
$ reflector --country Taiwan > /etc/pacman.d/mirrorlist
```

另外再加入國網中心提供的 mirror (比較快)
把 ``Server = http://free.nchc.org.tw/arch/$repo/os/$arch `` 這段文字 加入 ``/etc/pacman.d/mirrorlist`` 的最前面

## 第五步
安裝 linux 和其他東東

``` 
$ pacstrap /mnt base linux linux-firmware
```

**等安裝完後**
打這個
(好像是開機自動掛載的東東)

```
$ genfstab -U /mnt >> /mnt/etc/fstab
```

可參考[這個](https://dywang.csie.cyut.edu.tw/dywang/rhcsaNote/node59.html) 

## 第六步

進入新系統
```
$ arch-chroot /mnt
```

## 第七步
開始設定有的沒的
先安裝一些文字編輯器
我習慣用 vim
所以 我就安裝了 vim

```
$ pacman -S vim
```

這時候是root 所以不用管權限的問題 (因該可以這樣說吧)



接著設定時區 我家在台灣 所以這樣設定
```
$ ln /usr/share/zoneinfo/Asia/Taipei /etc/localtime
```
並打入這個

```
$ hwclock --systohc
```

然後
```
$ echo "en_US.UTF-8 UTF-8" > /etc/locale.gen
$ echo "zh_TW.UTF-8 UTF-8" >> /etc/locale.gen
$ echo "LANG=en_US.UTF-8" > /etc/locale.conf
```

這邊用 vim 編輯也行 但打指令比較快 XD

在這之後，還有其他東西要設定

設定 hostname
我把 hostname 取名叫做 ARCH

```
$ echo "Arch" > /etc/hostname
```


還有 hosts
使用vim 編輯
把以下打入 ``/etc/hosts``
```
127.0.0.1	localhost
::1		localhost
127.0.1.1	ARCH.localdomain	ARCH
```

## 第八步

設定 root 密碼

``` 
$ passwd 
```
接著就輸入密碼

也可以在這地方就新增使用者 不過我不是這樣做

## 第九步
設定 bootloader
我是使用 grub 作為 bootloader
這邊我卡最久

```
$ pacman -S grub efibootmgr os-prober mtools
```

接著
掛載EFI分割區
我習慣掛載在
/boot/efi 下
所以

```
$ mkdir /boot/efi # 建立這個目錄
$ mount /dev/sda1 /boot/efi # 掛載
```

安裝 grub 到EFI

```
$ grub-install --target=x86_64-efi --bootloader-id=frub+uefi
```
接著

```
$ grub-mkconfig -o /boot/grub/grub.cfg
```

就用 Tab 找 

安裝完成，但還沒完

## 第十步
安裝網路的東東
不然重開後沒網路
(第一次裝成功就犯這樣的錯誤)

我是這樣做
```
$ pacman -S dhcpcd
$ systemctl enable dhcpcd.service
```

DONE!


- - -

#### 參考資料 AGAIN
1. Arch Wiki https://wiki.archlinux.org/index.php/installation_guide 
2. 網友提供的協助 https://ray-fish.me/Arch-Newbie-Diary/posts/2019/06/19/%E9%83%BD%E6%98%AF%E5%BE%9E%E9%80%99%E8%A3%A1%E9%96%8B%E5%A7%8B%E7%9A%84%E5%90%A7/
3. 網路上看到的教學影片 在 bootloader 那一邊提供了很多幫助 https://www.youtube.com/watch?v=PQgyW10xD8s