# L02 指令下達行為與基礎檔案管理
###### tags: `Linux`
[TOC]
## 2.1 文字介面終端機操作行為的建立
文字模式登入後所取得的程式被稱為殼(Shell)這是因為這支程式負責最外面跟使用者溝通。
CentOS 7 的預設殼程式為bash
### 2.1.1 文字模式指令下達的方式
```
command [-options] [parameter1...]
```
- 一行指令第一個輸入的部分是指令(command)或可執行檔案。
- command為指令的名稱，例如變換工作目錄的指令為cd等等
- 中括號[]不在實際的指令中，僅作為一個提示，可有可無的資料之意
- -options為選項，通常前面帶有-號，例如-h
- options有時會提供長選項，此時會使用兩個減號，例如--help
- -help通常代表-h-e-l-p之意，與--help的單一長選項不同
- parameter1...: 參數，為依附在選項後面的參數，或者是command的參數
- 指令、選項、參數之間都以空格或tab作為區分，不論空幾格都視為一格，故空白是特殊字元，
- enter按鍵代表著一行指令的開始啟動
- linux的世界中，英文大小寫為不同的字元，例如cd和CD是不一樣的指令

如果想要知道目前的時間 
```
data
```
```
data +%Y/%m/%d
```
上述的選項資料基本上不太需要背誦，使用線上查詢方式處理即可。
```
data --help
```
### 2.1.2 身分切換 su - 的使用
### 2.1.3 語系功能的切換
- LANG語系變數的設定功能:
```
LANG=en_US.utf8
```
- 台灣常見的語系:
    1. zn_TW.utf8
    2. zh_TW.big5
    3. en_US.utf8
- 查閱目前語系的方法:
    ```
     echo ${LANG}
    ```
### 2.1.4 常見的熱鍵與組合按鍵
純文字模式下建議熟計的組合按鍵
- \[tab]: 命令補齊、變數名稱補齊
- [ctrl]+c: 中斷一個運作中的指令
- shift+PageUp, shift+PageDown
- 例題
  1. 系統中以if及ls為開頭的指令各有哪些
用tabtab顯示
  3. 有個以ifco為開頭的指令
用tabtab
  5. 中斷指令
ctrl+c
  7. ls '如何中斷
ctrl+c
  9. 使用ll-d 去看一下/etc/sec開頭的檔案有哪些，該如何操作
tabtab
  11. 到底有多少變數是由H開頭的，如何使用echo查閱
  ```
  echo $H[tab][tab]
  ```
### 2.1.5. 線上求助方式
有一個man指令可以呼叫(代表manual)
/keyword找關鍵字
n: 向整份文件的下方繼續找關鍵字
N: 向整份文件的上方繼續找關鍵字
man man指令
```
echo "scale=50; 4*a(1)" | bc -l
```
man -k
man -f
### 2.1.6 管線命令的應用
|稱作管線，它的目的是"將前一個指令輸出的資料，交由後面的指令來處理"
```
echo "scale=10; 4*a(1)" | bc -l
```
echo這個指令很單純地將後續的資料當成文字訊息傳輸
因此意思就是在bc -l環境下執行echo傳輸的內容

- 大量資料輸出的片段展示:
more與less
```
ll /etc | more
ll /etc | less
```
透過grep取得關鍵字ifconfig
## 2.2 檔案管理初探
### 2.2.1 目錄樹系統簡介
```
ll \
```
最左側的第一個字元，就是這個檔名的類型，三種類型：
\- ：一般檔案
d ：目錄檔案
l ：連結檔案
- 查看資料夾檔案容量
  ```
  ll -d /proc /sys
  ```
### 2.2.2 工作目錄的切換與相對/絕對路徑
ls、cd、pwd
幾個特殊目錄:
```
cd ..
cd ~
cd -
```
### 2.2.3 簡易檔案管理練習

cp可以在目的資料夾使用
cp 想要複製檔案的絕對路徑
cp會自動忽略目錄檔案的複製
cp -r 可以複製整個資料夾
## 2.3: 課後練習操作
### 簡答題
1. (a)什麼指令可以切換語系成為 en_US.utf8，並且(b)如何確認語系為正確設定了；
- a: 改變LANG變數
    ```
    LANG=en_US.utf8
    ```
- b: 
    ```
    echo ${LANG}
    ```

2. Linux 的日期設定其實與 Unix 相同，都是從 1970/01/01 開始計算時間而來。若有一個密碼資料，該資料告訴你密碼修改的日期是在 16849，請問如何使用 date 這個指令計算出該日期其實是西元年月日？(寫下完整指令)
    ```
    date --date='@16849'
    ```
3. 用 cal 輸出 2016/04/29 這一天的月曆與直接看到該日為星期幾？(寫下完整指令)
    ```
    cal 4 2016
    ```
4. 承上，當天是這一年當中由 1 月 1 日算起來的第幾天？ (註：該日期稱為 julian date)，(a)寫下完整指令與(b)執行結果顯示第幾天
- a: 
    ```
    cal -j 4 2016
    ```
- b: 120
5. 若為 root 的身份，使用 su - student 切換成為 student 時，需不需要輸入密碼？
- 實作起來不需要
6. 呼叫出 HOME 這個變數的指令為何？
- ~
7. 使用那一個指令可以查出 /etc/group 這個檔案的第三個欄位意義為何？(寫下指令)
查看
```
man ls
```
提示查看SEE ALSO
```
info coreutils 'ls invocation'
```
8. 請查出 /dev/null 這個裝置的意義為何？(寫下指令)
```
man null
```
9. 如何透過管線命令與 grep 的功能，透過 find /etc 找出檔名含有 passwd 的檔名資料？(a)寫下指令與(b)執行結果的檔名有哪幾個？
- a
    ```
    find /etc | grep 'passwd*'
    ```
- b
    /etc/passwd
    /etc/paddwd-
    /etc/security/opassed
    /etc/pam.d/passwd
10. 承上，將一堆錯誤訊息丟棄，我只需要顯示正確的檔名而已。(寫下指令)
    ```
    find /etc/ 2> /dev/null | grep 'passwd'
    ```
11. 根目錄下，哪兩個目錄主要在放置使用者與管理員常用的指令？
    - /bin和/sbin
12. 根目錄下，那兩個目錄其實是記憶體內的資料，本身並不佔硬碟空間？
    ```
    ll -d /sys /proc 
    ```
13. 根目錄下，那一個目錄主要在放置設定檔？
    - /etc
    - [完整目錄功能補充]()
14. 上網找出， /lib/modules/ 這個目錄的內容主要在放置什麼東西？
    - http://cn.linux.vbird.org/linux_basic/0540kernel_4.php
15. 有個指令名稱為 /usr/bin/mount，請使用『絕對路徑』與『工作目錄下的指令』來執行該指令
```
/usr/bin/mount
cd ~
../../usr/bin/mount
```
### 實作題
1. 使用 student 身分，在自己的家目錄底下，建立名為 ./20xx/unit02 的目錄
2. 使用 student 身分，將 /etc/X11 這個資料複製到上述的目錄內
3. 使用 root 的身分，刪除 /opt/myunit02 檔名。
4. 使用 root 的身分，建立名為 /mnt/myunit02 目錄
5. 使用 root 的身分，透過 find /etc 指令，找出檔名含有 passwd 的檔案資料，並將這些檔案資料複製到 /mnt/myunit02 去。

## 補充資料
[一定要會的指令](https://linux.vbird.org/linux_basic/redhat6.1/linux_06command.php)