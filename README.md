# linuxjailonbsd

無聊測試在 FreeBSD 安裝 Debian Stretch Jail 的成功或失敗筆記

* [隨手紀錄版本](https://hackmd.io/@holishing/HJfp1mhKS)

* [穩定(ㄐㄧㄡˋ)版](https://github.com/holishing/linuxjailonbsd)


## 參考資料

[Setting up a (Debian) Linux jail on FreeBSD](https://forums.freebsd.org/threads/68434)

## 動機?

* Debian GNU/kFreeBSD 已經被放生
* Linux Binary 相容層雖然無法相關最新格式的 Linux binary，但大概能支援到 GNU/Linux 2.6.32 格式的 binary (可能部份 syscall 還無法完整支援)，因此想測試這功能能支援到的極限在哪邊

## 預計跟參考資料不一樣的地方

* 直接上 Debian + `sysv-rc` 等套件 (不刻意拔 `systemd`，~~反正也用不到~~)
* 整理 `apt update` 遇到 mmap 問題的 workaround
* 整理用以上方法會遇到的疑點
* 嘗試整理出相關自動化腳本或是可以匯進 iocage

## 步驟

* 檔案系統選擇：
    * UFS -> 建立一般的 jail
    * ZFS -> 建立一般 jail 或搭配其他管理工具，如：`iocage`...等

* 選定 jail 所在 root 目錄
* `pkg install debootstrap`
* `debootstrap --foreign --arch=amd64 stretch /zroot/iocage/jail/debian9  http://free.nchc.org.tw/debian/` (自行改成你喜歡的 Debian 鏡像站，可以加上 `--include="sysv-rc locales locales-all"` 等套件)
* 建立預計掛載的 dev, proc, sys 目錄，掛上去
* chroot 進去 debian root，利用 dpkg 把設定跑完
* 設定 jail 後，把 Debian jail 打開
* 修改 apt.conf 設定，使 `apt update` 可以繞過 mmap 相關問題
* 修改 `/etc/apt/source.list` ，加入 `stretch-backports`，使 Debian Stretch 可以測試使用較新的部份套件
* `kldload pty` 使 Debian 上面的 sshd 可以被連線

## 問題

* 為何非得要用 `pty` 這個模組才能讓 Debian 9 jail 可以被連線?
* 一些 `/dev` 上面的東西在 Debian jail 仍無法設定正常使用，影響到 OpenSSH server (DropBear 相對問題較少)，是 FreeBSD jail 的安全機制? 有方法可以設定?
* 編譯 [ptt/pttbbs](https://github.com/ptt/pttbbs) 這個開源專案會找不到 `explicit_bzero()` 相關函式，可能少裝哪些套件或是少設定某些參數?