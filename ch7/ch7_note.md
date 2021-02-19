# ch7_note
###### tags: `Linux`
[TOC]
# 7認識bash基礎與系統救援
## 7.1 bash shell基礎認識
- 登入系統取得的文字形互動介面就稱為shell
- shell的操作環境能夠依據使用者的喜好設定
- shell最重要的就是變數
### 7.1.1 系統與使用者的shell
- 系統所有合法的shell都在`/etc/shells`檔案內
- recall:
    - `/etc : `一堆系統設定檔、包括帳號、密碼與各式服務軟體的設定檔大多在此目錄內
- `cat /etc/shells`
- ![](https://i.imgur.com/jLHxYIh.png)
    - `/bin/sh`(已經被`/bin/bash`所取代)
    - `/bin/bash`(就是linux預設的shell)
    - `/bin/tcsh`(整合Cshell，提供更多功能)
    - `/bin/csh`(已經被`/bin/tcsh`取代)
- 在文字界面登入後系統會給予一個shell
- 在圖形介面時藉由按下終端機也可以取得shell
- 從使用者設定資料搜尋預設取得的shell
- `man cut`
![](https://i.imgur.com/FUKfpbq.png)
![](https://i.imgur.com/WGZsKka.png)
![](https://i.imgur.com/H52vV0t.png)
- `cat /ect/passwd`
![](https://i.imgur.com/ufsKE06.png)
- `cur -d ':' -f 1, 7 /etc/passwd`
![](https://i.imgur.com/pat6bsW.png)
#### 例題
- 1 請使用cut這個指令，將`/etc/passwd`這個檔案的內容中，以冒號(:)為分隔字元，將第一及第七欄位輸出到螢幕上
    - `cut -d ':' -f 1, 7 /etc/passwd`
- 2. 承上，找到關鍵字為daemon的那一行，daemon用戶所使用的shell是什麼
    - `/sbin/nologin`
- 使用者可以自由切換所需shell，不過不同shell使用的方式語法都有差異，舉例來說: bash使用的變數設定方式為`var = 'content'`，但csh使用的是`set var = 'content'`
#### 例題
- 1. 請使用student身分登入系統，取得終端機後，使用echo $bach的方式查閱有沒有這個變數以及其輸出的內容
    - `echo $BASH`
    ![](https://i.imgur.com/dx9mOzE.png)

- 2. 請輸入`echo #shell`觀察有沒有資料輸出
    ![](https://i.imgur.com/O7FuCGz.png)
- 3. 使用`/bin/csh`切換shell成為c shell
    ![](https://i.imgur.com/D68Hb3K.png)
- 4. 分別使用`echo $BASH`與`echo $shell`觀察輸出的資料為何
    ![](https://i.imgur.com/jnHh1E0.png)
- 5. 使用echo $0觀察輸出的資料是什麼
    ![](https://i.imgur.com/lHAdjz8.png)
- 6. 先透過exit離開c shell之後，再次以`echo $0`觀察目前的shell名稱為何
    ![](https://i.imgur.com/pHdGEbn.png)
- 7. 執行`/sbin/nologin`看看輸出的資料為何
    ![](https://i.imgur.com/3behiby.png)
- 使用者透過輸入shell執行檔(`/bin/csh`)直接切換shell
- 想確認shell是什麼，最簡單的方式是使用echo $0
- 寫入在`/ect/shells`內有個名為`/sbin/nologin`的shell，就是給系統帳號預設使用的不可互動的合法shell
#### 例題
- 1. 請使用usermod來修改student的shell變成`sbin/nologin`
    - `man usermod`
    ![](https://i.imgur.com/YWxxLLb.png)
    - `grep student /etc/passwd`
    - `usermod -s /sbin/nologin student`
    - ![](https://i.imgur.com/7GbPJdp.png)
    - 
- 2. 修改完畢後，請到`tty3`的終端機，嘗試使用student的帳號登入，看看會出現什麼狀況
- 3. 請再次以usermod的方式將student的shell改回來`/bin/bash`
    - `usermod -s /bin/bash student`
- 為何需要設定`/sbin/nologin`呢
- 許多系統預設要執行的軟體，例如mail的郵件分析、WWW的網頁回應等等，系統不希望該軟體使用root的權限，因為擔心網路會被惡意人士攻擊，因此系統就會依據該軟體的特性給予系統帳號，這些系統帳號就是有特殊的任務而產生的。因此系統帳號通常就是使用`/sbin/nologin`作為預設shell
- 某些伺服器的帳號，例如郵件伺服器，FTP伺服器等，這些伺服器的帳號就在收發email或是傳輸檔案，這些帳號無須登入系統來取得互動shell，因此這些帳號就不需要可互動的shell，此時就能給`/sbin/nologin`
#### 例題
- 1. 使用id這個指令檢查系統有無bin與student這兩個帳號的存在
    - `man id`
    ![](https://i.imgur.com/LwjXnwJ.png)
- 2. 能不能在不知道密碼的情況下，使用root切換成student這個帳號?為什麼?
    - `su -l student`
- 3. 能不能在不知道密碼的情況下，使用root切換成bin這個帳號?為什麼?
    - `su -l bin`
        - This account is currently not available
    - ![](https://i.imgur.com/eDsC52D.png)
    - 他`/sbin/nologin`
- 4. 建立一個不可登入系統取得互動shell的帳號，帳號名稱為puser1，密碼為MyPuser1
    - `man useradd`
    - ![](https://i.imgur.com/BvV8Vkl.png)
    - ![](https://i.imgur.com/AZo0ksw.png)
    - `echo MyPuser1 | passwd --stdin puser1`
- 5. 嘗試在tty3登入該帳號，結果是?
    - 打完密碼後無法登入
### 7.1.2 變數設定規則
- echo $BASH就是變數的功能
- bash shell會主動建立BASH這個變數，且其內容就是`/bin/bash`
    ```
    變數="變數內容"
    echo $變數
    echo ${變數}
    ```
- 變數的設定規則:
    - [x] 變數與變數內容以一個等號=來連結
    - [x] 等號兩邊不能直接接空白字元
    - [x] 變數名稱只能是英文字母與數字，但是開頭字元不能是數字
    - [x] 變數內容若有空白字元可使用雙引號`"`或單引號`'`將變數內容結合起來
    - [x] 雙引號內的特殊字元如$等可以保有原本的特性
    - [x] 單引號內的特殊字元僅為一般字元(純文字)
    - [ ] 可用跳脫字元`\`將特殊符號如`[Enter], $, \, 空白, '`等變成一般字元
    - [x] 在一串指令的執行中，還需要藉由其他額外的指令所提供的資訊時，可以使用反單引號`'指令'`或`$指令`
    - [ ] 若該變數為擴增變數內容時，則可用"$變數名稱"或`${變數}`累加內容
    - [ ] 若該變數為擴增變數內容時，則需要以export來使變數變成環境變數
    - [ ] 通常大寫字元為系統預設變數，自行設定變數可以使用小寫字元，方便判斷(純粹依照使用者興趣與嗜好)
    - [ ] 取消變數的方法為使用unset: unset變數名稱
#### 例題:
- 1. 設定一個名為myname的變數，變數的內容為`peter pan`
- 2. 使用echo呼叫出myname的內容
    - `echo $myname`
- 3. 是否能夠設定2myname的內容為peter pan呢
    - 變數名稱開頭不能是數字
    - `2myname=peter`
        - 2myname=peter: command not found
- 4. 設定varsymbo變數的內容為`$var`，`$var`就是純文字資料不是變數。設定完畢後呼叫出來
    - `var='$'var`
    - `echo $var`
- 5. 設定hero變數的內容為`I am $myname`，其中`$myname`會依據`myname`變數的內容而變化。請設定完畢後乎叫出來
    - `hero="I am $myname"`
    - ![](https://i.imgur.com/zEbCAOp.png)
- 6. 使用`uname -r`秀出目前的核心版本
    - `uname -r`
    ![](https://i.imgur.com/Jq7SpWg.png)
- 7. 設定`kver`變數，內容為my kernel version is 3.xx，其中3.xx為`uname -r`輸出的資訊。請注意，kver變數設定過程中，需要用到`uname -r`這個指令的協助
- `kver="my kernel version is $uname -r"`
![](https://i.imgur.com/MMe6Ax9.png)

- 變數設定的過程當中，使用子指令`$(command)`的操作為相當重要的。底下的案例中，管理員可以很快速的找到前一堂課談到的特殊權限檔案並列出該檔案的權限
#### 例題:
- 1. 使用man find找出`-perm`的功能為何?
    - `man find`
    - `/perm`
    - `n` and `N`
- ![](https://i.imgur.com/mIYSMGg.png)
- 2. 使用`find /usr/bin /usr/sbin -perm /6000`
- 3. 使用`ls -l$(find /usr/bin /usr/sbin -perm /6000)`將所有檔名的權限列出
- ==子指令==
#### 例題:
- 1. 使用find的功能，找出在`/usr/sbin`及`/usr/bin`底下權限為4755的檔案
    - `ll $(find /usr/sbin /usr/bin -perm 4755)`
- 2. 建立`/root/findfile`目錄
- 3. 將步驟1找到的檔案連同權限複製到`/root/findfile`目錄下
    - `cp -a $(find /usr/sbin /usr/bin -perm 4755) /root/findfile/`
### 7.1.3 影響操作行為的變數
- 某些變數會影響到使用者的操作行為，許多變數之前曾經題及
- `LANG`, `LC_ALL`: 語系資料，例如使用date輸出資訊時透過LANG可以修改輸出的訊息
- `PATH`: 執行檔搜尋的路徑`~目錄`與目錄中間以冒號分隔，由於執行檔`/指令`的搜尋是依序由`PATH`的變數內的目錄來查詢，所以，目錄的順序也是重要的
- `HOME`: 使用者的家目錄
- `MAIL`: 當我們使用mail這個指令在收信時，系統會去讀取的郵件信箱檔案
- `HISTSIZE`: 這個與歷史命令有關。我們曾經下達過的指令可以被系統記錄下來，而記錄的筆數則是由這個值來設定的
- `RANDOM`: 隨機亂數的變數。目前大多數的distributions都會有亂數產生器，亦即`/dev/random`檔案。你只要`echo /dev/random`系統就會主動的隨機取出一個介於0~32767的數值
- `PS1`: 命令提示字元，可使用`man bash`搜尋`PS1`關鍵字，即可了解提示字元的設定方式
- `?`這個變數內容為指令的回傳值，當回傳值為0代表指令正常運作結數，當不為0則代表指令有錯誤
- 比較需要注意到的變數是`PATH`路徑搜尋變數，他會影響到使用者操作的行為，設定錯誤會有相當嚴重的後果
#### 例題:
- 關於PATH的重要性
- 1. 請用root的身分來處理底下的任務。
    - `su -`
- 2. 印出PATH這個變數的內容，並觀察每個項目中間的分隔符號為何?
    - `echo $PATH`
    ![](https://i.imgur.com/Ln933cD.png)

- 3. 設定一個名為`oldpath`的變數，內容就是`${PATH}`
    - `oldpath="$PATH"`
- 4. 設定PATH的內容成為`/bin`而已(非常重要，不可設錯)
    - `PATH=/bin`
- 5. 此時輸入以前曾操作過的`useradd --help`及`usermod --help`等指令，螢幕顯示的訊息為何
    - command not found
- 6. 若使用`/sbin/usermod --help`可以正常顯示嗎?
    - yes
- 7. 請設定PATH的內容成為`${oldpath}`，恢復正常的路徑資料
    - `PATH=oldpath`
- 8. 請改用student的身分來執行下列練習
    - `su -l student`
- 9. 建立`~student/cmd/`目錄，且將`/bin/cat`複製成為`~student/cmd/scat`
    - `mkdir ~student/cmd`
    - `cp /bin/cat ~student/cmd/scat`
- 10. 輸入`~student/cmd/scat/etc/hosts`確認指令正常無誤
- 11. 輸入`scat/etc/hosts`會發生什麼問題
    - command not found
- 12. 如何讓student用戶直接使用scat而不須使用`~student/cmd/scat`來執行
    - ==累加PATH==
    - `PATH=$PATH:~/cmd`
- TIPs:
    - 由於PATH設定錯誤時，可能會導致系統的crash狀態，尤其是當PATH並未含有`/bin`這個搜尋路徑時，有相當高的機率會造成Linux系統的當機。因此，在上述的練習中，PATH的設定請務必小心謹慎。
- 命令提示字元在每個系統中都不一樣，但那是可以修改的，就透過PS1這個變數來修改即可
#### 例題:
- 1. 呼叫出PS1這個變數的內容
    - `echo $PS1`
    ![](https://i.imgur.com/O5EICqI.png)
- 2. 請查詢上述變數內容當中`\W`及`\$`的意義為何(請man bash)透過PS1關鍵字查詢
    - `man bash`
    ![](https://i.imgur.com/nrBU2vU.png)
    ![](https://i.imgur.com/8pQpARj.png)
    - `/u`: the username
    - `/W`: the basename of the current working directory
    - `/$`: if the effective UID is 0. a #. otherwise a $
- 3. 假設操作者已經做了15個指令，則命令提示字元輸出如`student@localhost 15~`該如何設定PS1
    - `PS1=[\u@\h \# \W]\$`
    ![](https://i.imgur.com/z21F8gB.png)

### 7.1.4 區域/全域變數、父程序與子程序
- 變數是有使用範圍的，一般來說變數的使用範圍分為:
    - 區務變數: 變數只能在目前這個shell當中存在，不會被子程序所沿用
    - 全域變數: 變數會儲存在一個共用的記憶體空間，可以讓子程序繼承使用。
- 將變數提升為全域變數的方式為透過export
- 觀察可用env或export來觀察
### 例題:
- 1. 使用set或env或export觀察是否存在mypp這個變數?
    - set可以列出全部(區域全域)變數，env / export只能列出全域變數
    - `set |grep mypp`
    - `env |grep mypp`
    - `export |grep mypp`
- 2. 設定mypp的內容為`from_ppid`並且呼叫出來
    - `mypp="from_ppid"`
    - `echo $mypp`
    ![](https://i.imgur.com/YzD7yXg.png)

- 3. 使用set或env或export觀察是否存在mypp這個變數?
- 4. 執行`/bin/bash`進入下一個bash的子程序環境中
- 5. 使用set或env或export觀察是否存在mypp這個變數?同時說明為什麼?
    - 沒有，區域變數不會繼承
- 6. 設定mypp2的內容為from_cpid並且呼叫出來
    - `mypp2="from_cpid"`
    - `echo mypp2`
- 7. 使用exit離開子程序回到原本的父程序
- 8. 觀察是否存在mypp2這個變數?為什麼
    - 找不到，不同程序
- 9. 使用export mypp後，使用env或export觀察是否存在?
    - `export mypp`
    - `env |grep mypp`
- 10. 執行`/bin/bash`進入下一個bash的子程序環境中
- 11. 使用set或env或export觀察是否存在mypp這個變數?同時說明為什麼?
- 12. 回到原本的父程序中
- 由原本的bash衍生出來的程序都是該bash的子程序，而bash可以執行bash產生一隻bash的子程序，兩隻bash之間僅有全域變數(環境變數)會帶給子程序，而子程序的變數基本上是不會回傳給父程序的
### 7.1.5 使用kill管理程序
- kill並不是刪除程序，而是給予程序一個訊號來管理，預設的訊號為15號，該訊號的功能為正常關閉程序的意思
- 而想要強制關閉該程式，就得要使用`-9`這個號碼來處理了
#### 例題:
- 1. 將vim程序放進背景中暫停
    - `vim test.txt`
    - ctrl+z
- 2. 使用jobs -l進一步列出該程序的PID號碼
    - `jobs -l`
- 3. 使用kill PID號碼嘗試刪除該工作，是否能夠生效?
    - `kill PIDNUM`
- 4. 若無法刪除，請使用`kill -9 PID`的方式來刪除
    - `kill -9 8572`
### 7.1.6 login shell and non-login shell
- 取得bash的情況有很多種，但大致可分為兩大類
    - 需要輸入帳號密碼才能取得bash的行為: 從tty2登入，或者是輸入su -來取得某個帳號的使用權，這種情況被稱為是login shell的變數設定檔讀取方式
    - 使用者已經取得bash或是其他的互動介面，然後透過該次登入後執行bash，從圖形介面按下終端機、直接在文字界面輸入bash來取得bash子程序，輸入su來切換身分等等，稱為non-login shell
- 通常login shell讀取設定檔的流程是:
    - `/etc/profile:`這是系統整體的設定，你最好不要修改這個檔案
    - `~/.bash_profile`或`~/.bash_login`或`~/.profile`(只會讀一個，依據優先順序決定): 屬於使用者個人設定，你要改自己的資料，就寫入這裡
- 由於login shell已經讀取了`/etc/profile`因此已經設定了大部分的全域變數設定，所以no-login shell只需要少部分的設定即可。故non-login shell只會讀取一個個人設定檔，亦即是:
    - `~/.bashrc`
#### 例題:
- 1. 觀察一下`~/.bash_profile`的內容，說明該檔案設定了什麼項目
- 2. 觀察`~/.bashrc`的內容，說明該檔案設定了什麼項目
- 由於`~/.bash_profile`也是讀取`~/.bashrc`使用者只需要將設定放置於家目錄下的`.bashrc`就可以讓兩者讀取了。
#### 例題:
- 嘗試設定student的操作環境
- 1. 請在student的家目錄編輯`.bashrc`，增加底下的項目
    - 設定history可以輸出10000筆資料
        - `HISTSIZE=1000`
    - 設定cp時，其實會主動加入`cp -i`的選項
        - `alias cp="cp -i"
    - 設定執行rm時，其實會主動加入`rm -i`的選項
        - `alias rm="rm -i"`
    - 設定執行mv時，其實會主動加入`mv -i`的選項
        - `alias mv="mv -i"`
    - 增加PATH的搜尋目錄在`/home/student/cmd/`目錄
        - `PATH=$PATH:~/cmd`
    - 設定一個變數名稱為kver，其內容是目前的核心版本
        - `kver=$(uname -r)`
    - 強迫語系使用`zh_TW.utf8`這個項目，且必須要設定為全域變數
        - `export LANG=zh_TW.utf8`
    - 讓提示字元項目中，增加時間與操作指令次數的項目
        - `PS1="[\u@h\A\#\W]$"`
    - 使用wc指令分析`~/.bash_history`的行數，將該行數紀錄於`h_start`的變數中
        - `h_start=$( wc -l ~/.bash_history | awk '{print $1}' )`
- 2. 設定完畢後，如何在不登出的情況下，讓設定生效?
    - `source ~/.bashrc`
- 當使用者登出bash時，bash會依據家目錄下的`.bash_logout`來進行後續的動作，因此使用者有需要額外進行某些工作時，可以在此檔案中設定。
- 使用者應該要特別注意，`.bash_logout`僅會在login-shell的環境下登出才會執行。在non-login shell的環境下登出，這個檔案並不會被運作
#### 例題:
- 每次登出bash時都會:
    - 1. 使用date取得`YYYY/MM/DD HH:MM`的格式，並且轉存到家目錄的history.log檔案中
    - 2. 使用history加上管線命令與wc來分析結束時的history行數，將該數值設定為`h_end`，搭配之前設定的`h_start`開始的行數，計算出這次執行指令的行數號碼(應該是h_end - h_start +1)，設定為h_now，透過`history ${h_now}`，將最新的指令準存到history.log當中
- 嘗試使用su -student來登入student，在隨意進程數個指令，之後登出bash回到原本的bash當中，觀察`~/history.log`是否有資訊紀錄
## 7.2 系統救援
- 簡易的系統救援，直接以root的密碼與身分才進入救援的模式，然後處理好了檔案系統。
- 萬一root的shell被不小心修改了，導致無法使用root的密碼進入系統時，該如何處理?
### 7.2.1 透過正規的systemd方式救援
- 先讓系統被破壞:
- `vim /etc/fstab`
    - `/dev/mapper/centos-home1 /home xfs defaults 0 0`
- `usermod -s /sbin/nologin root`
- `su -`
- `reboot`
- -.-嚇死人
### 7.2.2 透過bash直接救援(optional)
- 在開機流程使用`init=/bin/bash`
