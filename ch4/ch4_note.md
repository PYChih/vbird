# ch4_note
###### tags: `Linux`
[TOC]
# 4.1 Linux傳統權限
- Linux權限的目的是在保護某些人的檔案資料
- 權限最終都是應用在某個/某群帳號上
- 權限都是設定在檔案/目錄上
## 4.1.1 使用者、群組與其他人
- Linux的檔案權限在設定上，主要依據三種身分
    - user/owner
    - group
    - others: 不是user也沒有加入group
### 檔案權限的觀察
- `ls -l`或`ll`
![](https://i.imgur.com/QCpDVxL.png)
- 第一個字代表檔案類型(`-`或`d`)
    - `-`: 一般檔案
    - `d`: 目錄檔
    - `l`: 連結檔
    - `b`: 裝置檔(區塊裝置(媒體))
    - `c`: 週邊裝置檔(滑鼠鍵盤
- 接續的9個字為rwx與-號
    - r: 可讀
    - w: 可寫
    - x: 可執行
- 三個字三個字一組指出
    - root權限、group權限、其他人權限
- rwx的位置不會改變，有該權限就會顯示字元，沒有該權限就會變成減號(-)
- 能看懂輸入`ls -l`顯示的整行資訊，依序是:
    - 檔案類型與權限
    - 檔案連結數
    - 該檔案的擁有者
    - 該檔案所屬群組
    - 檔案容量
    - 最後一次修改時間
    - 檔名
### 觀察帳號與權限的相關指令
- 若想知道帳號所屬的群組，可以使用id這個指令來觀察即可理解
![](https://i.imgur.com/zdXlKeN.png)
## 4.1.2 檔案屬性與權限的修改方式
- 對照圖:
![](https://i.imgur.com/2JAYMt4.png)
- 一般帳號僅能修改自己檔案的檔名、時間與權限，無法隨意切換使用者與群組，因此應使用root身分處理
### 使用chown修改檔案擁有者
- 查詢系統中是否有名為daemon的帳號
    - `id daemon`
- 將`[filename]`的使用者改為`[id]`
    - `chown [id] [filename]`
### 使用chgrp修改檔案擁有的群組
- 系統的群組都記錄在`/etc/group`檔案內
- 想了解系統是否存在某個群組，可以使用`grep`關鍵字擷取指令來查詢
    - `grep bin /etc/group`
- 使用chgrp改變檔案的群組
    - `chgrp bin checking`
### 使用chmod搭配數字法修改權限
- 用三個數字相加的結果修改
    - r=4
    - w=2
    - x=1
- 可讀可寫可執行=7
- 唯獨=4
### 使用chmod搭配符號法修改權限
- u: user, g: group, o: other, a: all
- `chmod u=rwx, g=rw, o=r [filename]`
### 其他屬性的修改
- 時間: 可以用`touch -t`
- 檔名: `mv`
# 4.2 基礎帳號管理
## 4.2.1 簡易帳號管理
- 最簡單的帳號管理就是建立帳號與給予密碼的任務
- `useradd [帳號名稱]`
- `PASSWD [PASSWD]`
- 使用userdel刪除帳號
## 4.2.2 帳號與群組關聯性管理
- 若需要建立帳號時，給予帳號一個次要的群組支援，就需要先情建置群組
- 有三個帳號prouser1, prouser2, prouser3加入共有的群組progroup
    - 先建立群組`groupadd progroup`
    - 透過useradd --help找到次要群組支援的選項為-G的項目，即可建立好群組、帳號與密碼
    - 管理員可以透過passwd --help找到--stdin的選項來操作密碼的給予
    ```
    groupadd progroup
    grep progroup /etc/group
    useradd -G progroup prouser1
    useradd -G progroup prouser2
    useradd -G progroup prouser3
    id prouser1
    echo mypassword | passwd --stdin prouser1
    ```
- 透過usermod修改群組資源
# 4.3 帳號與權限用途
- 使用者能使用系統上面的資源與權限有關，因此簡易的帳號管理之後，就需要與權限搭配設計
## 4.3.1 單一用戶所有權
- 一般用戶只能修改屬於自己的檔案的rwx權限，因此若root要協助複製資料給一般用戶時，需要特別注意該資料的權限
- 常用的命令執行檔在/bin裡，比如可以從bin複製ls出來，改名後以./filename執行
### 4.3.2 群組共用功能
- 可以將某個資料夾設定成770讓群組共享
    - 但770並非最好的處理方式，下一章將會學到SGID功能

# 總結:
- 看懂ls -l回傳的意思
- 修改檔案類型與權限、檔案連結數、檔案擁有者、檔案所屬群組、檔案修訂時間、檔名
- 創建/刪除/查看使用者與group
- 設定密碼