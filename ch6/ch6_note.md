# ch6_note
###### tags: `Linux`
[TOC]
# ch6_磁碟基礎檔案系統管理
- CentOs7 linux檔案系統主要支援EXT2家族(最新版為EXT4)以及XFS大型檔案系統兩種
- XFS相當是和大容量磁碟
- 這章一堆名詞看的不是很懂
### 名詞
- EXT2/ EXT3/ EXT4
- XFS
- inode
    - 特性?
- block
    - 特性?
- 磁區、磁柱
- 磁碟分割表
    - MBR分割表
        - 主要分割、延伸分割、邏輯分割
    - GPT分割表
# 6.1 認識Linux檔案系統
- XFS相當適合大容量磁碟，格式化的效能非常快
- 無論哪種檔案系統，都必須要符合inode與block等檔案系統使用的特性
## 6.1.1 磁碟檔名與磁碟分割
- 磁碟內的圓型磁碟盤常見的物理特性:
    - 磁區(sector): 最小的物理儲存單位
        - 512bytes格式
        - 4K格式
    - 磁柱(Cylinder)
        - 將磁區組成一個圓
    - 分割的最小單位可能是磁柱也可能是磁區，與分割工具有關
    - 磁碟分割表主要有兩種格式: MBR與GPT
        - MBR: 第一個磁區最重要
            - 1. 主要開機區(Master boot record, MBR): 446bytes
            - 2. ==分割表(partition table): 64bytes==
        - GPT: 分割表除了分割數量擴充較多之外，支援的磁碟容量也可以超過2TB
- 整顆磁碟必須要經過分割之後，Linux作業系統才能夠==讀取分割槽內的檔案系統
- Linux分割主要有兩種分割
    - 早期的MBR
        - 有2TB容量的限制
    - 現今的GPT
- 磁碟檔名主要為
    - 實體磁碟的檔名`/dev/sd[a-p]`
    - 虛擬磁碟檔名`/dev/vd[a-p]`
- 由於虛擬機器的環境中，大部分磁碟的容量還是小於2TB條件，因此傳統的MBR還是有其存在的需求
- `ls /dev | grep sd`
### MBR磁碟分割的限制
- ==MBR的紀錄區塊僅有64bytes用在分割表，因此預設分割表僅能記錄四筆分割資訊==
- 所謂的分割資訊即是紀錄開始與結束的磁區
- 四筆紀錄主要為:
    - 主要分割槽(primary)
    - 延伸分割槽(extended)
        - ==不能被格式化應用，需要再從延伸分割當中割出邏輯分割之後才能夠應用==
- 相關性:
    - 主要分割與延伸分割最多可以有四筆(硬碟限制)
    - 延伸分割最多只能有一個(作業系統限制)
    - 邏輯分割是由延伸分割持續切割出來的分割槽
    - 能夠被格式化後，做為資料存取的分割槽為主要分割與邏輯分割。延伸分割無法格式化
    - 邏輯分割的數量依作業系統不同在Linux系統中SATA硬碟已經可以突破63個以上的分割限制
### GPT磁碟分割
- 常見的磁碟區有512bytes與4k容量，為了相容於所有的磁碟，因此在磁區的定義上面，大多會使用所謂的邏輯區塊位址(logical block address, LBA)
- GPT將磁碟所有區塊以此LBA(預設為512bytes)來規劃，而第一個LBA稱為LBA0(從0開始編號)
- 與MBR僅使用第一個512bytes區塊來記錄不同，GPT使用了34個LBA區塊來記錄分割資訊，同時與過去MBR僅有一的區塊的情況不同，GPT除了前面34個LBA之外，整個磁碟的最後33個LBA也拿來做為另一個備份
- LBA2~LBA33為實際紀錄分割表的所在，每個LBA紀錄4筆資料，所以共可記錄32 * 4 = 128筆以上的分割資訊
- 因為每個LBA為512bytes，因此每筆紀錄可占用512 / 4 = 128 bytes的紀錄
- 因為每筆紀錄只要記錄開始與結束兩個磁區的位置，因此紀錄的磁區位置最多可達64位元，若每個磁區容量為512bytes，則單一分割槽的最大容量就可以限制到8ZB，其中一ZB為2^30TB
- 此外，每筆GPT的分割紀錄，都是屬於primary的分割紀錄，可以直接拿來進行格式化應用的。
### 例題
- 1. 超過幾個TB以上的磁碟，通常預設會使用GPT的分割表?
    - 2T?
        - MBR的分割表僅有64bytes，其中分為4筆紀錄，所以每筆紀錄有16bytes
        - ==16bytes當中，紀錄磁區個數的==，又僅有4bytes，這4bytes約為4*8=32bits這麼多
        - 如果以數值表示，則為2^32，亦即為2^2 * 2^30，也就是共有4 * 2^30這麼多磁區可用
        - 由於MBR大多僅支援512bytes的磁區的裝置，也就是0.5K/每個磁區
        - 兩者相乘，亦即4*2^30磁區*0.5K/每個磁區 = 4*2^30 * 0.5k = 2*2^30K = 2 * 2^20M = 2*2^10G = 2TB
- 2. 某一磁碟的分割為使用MBR分割表，該系統當中共有5個==可以進行格式化的分割槽==，假設該磁碟含有2個主分割(primary)，請問該磁碟的分割槽的磁碟檔名應該是如何?(假設為實體磁碟的檔名，且該系統僅有一顆磁碟時)
    - 磁碟檔名: 第一個實體磁碟為`/dev/sda`，第一個虛擬磁碟為`/dev/vda`，我們假設以實體磁碟為例
    - 分割槽檔名，第一個分割槽檔名會是`/dev/sad1`第二個基本上就是`/dev/sda2`以此類推
    - 分割槽檔名前4筆已經保留給primary與extended了!所以logical由`/dev/sda5`開始編號
    - 假設檔名編號為連續，因此，兩個primary分割檔名:`/dev/sda1, /dev/sda2`
    - 一定要有一個延伸分割，所以會是`/dev/sda3`
    - 還需要3個logical，所以會是`/dev/sda5`, `/dev/sda6`, `/dev/sda7`
    - 最終可以被格式化使用的就是`/dev/sda1`, `/dev/sda2`, `/dev/sda5`, `/dev/sda6`, `/dev/sda7`
- 3. 某一個磁碟預設使用了MBR的分割表，目前僅有2個主分割，還留下1TB的容量。若管理員還有4個需要使用的分割槽，每個分割曹需要大約100GB，你認為應該如何進行分割較佳?
    - 注意上一題的結論，所有的剩餘容量盡量都保留給延伸分割才好，這樣才能進行完整的容量分配
    - 全部的1TB容量通通交給`/dev/sda3`這個延伸分割
    - 分割出4個100GB的邏輯分割，亦即如下:
        - `/dev/sda5` 100G
        - `/dev/sda6` 100G
        - `/dev/sda7` 100G
        - `/dev/sda8` 100G
## 6.1.2 Linux的EXT2檔案系統
- 新的作業系統在規劃檔案系統時，一般檔案會有屬性(如權限、時間、身分資料紀錄等)以及實際資料的紀錄，同時整個檔案系統會紀錄全部的資訊，因此通常檔案系統會有如下幾個部分
    - superblock: 記錄此filesystem的整體資訊，包括`inode/block`的總量、使用量、剩餘量，以及檔案系統的格式與相關資訊等
    - inode: 紀錄檔案的屬性，一個檔案占用一個inode，同時記錄此檔案的資料所在的block號碼。
    - block: 實際記錄檔案的內容，若檔案太大時，會占用多個block
### superblock(超級區塊)
- superblock為整個檔案系統的總結資訊處，要讀取檔案系統一定要從superblock讀起。superblock主要紀錄資料為:
    - block與inode的總量
    - 未使用與已使用的inode/block數量
    - block與inode的大小(block為1, 2, 4K，inode為128bytes或256bytes)
    - filesystem的掛載時間、最近一次寫入資料的時間、最近一次檢驗磁碟(fsck)的時間等檔案系統的相關資訊
    - 一個valid bit數值，若此檔案系統已被掛載，則valid bit為0，若未被掛載，則valid bit為1
### inode table(inode 表格)
- 每一個inode都有號碼，而inode的內容在紀錄檔案的屬性以及該檔案實際資料是放置在哪幾號block內。inode紀錄的檔案資料至少有底下這些:
    - 該檔案的存取模式(read/write/excute)
    - 該檔案的擁有者與群組(owner/group)
    - 該檔案的容量。
    - 該檔案建立或狀態改變的時間(ctime)
    - 最近一次的讀取時間(atime)
    - 最近修改的時間(mtime)
    - 定義檔案特性的旗標(flag)如SetUID
    - 該檔案真正內容的指向(pointer)
- 由於每個檔案固定會占用一個inode，而目前檔案所記載的屬性資料越來越多，因此inode有底下幾個特色:
    - 每個inode大小均固定為128bytes(新的ext4與xfs可設定到256bytes)
    - 每個檔案都僅會占用一個inode而已
    - 承上，因此檔案系統能夠建立的檔案數量與inode的數量有關
    - 系統讀取檔案時需要先找到inode，並分析inode所紀錄的權限與使用者是否符合，若符合才能夠開始實際讀取block的內容
### data block(資料區塊)
- 檔案實際的資料存放在data block上面，每個block也都會有號碼，提供給檔案來儲存實際資料，也讓inode可以紀錄資料放在哪個block號碼內
    - 原則上，block的大小與數量在格式化完就不能夠再改變了(除非重新格式化)
    - 每個block內最多只能夠放置一個檔案的資料
    - 如果檔案大於block的大小，則一個檔案會占用多個block數量
    - 如果檔案小於block，則該block的剩餘容量就不能夠再被使用了(磁碟空間浪費)
- 一般來說，檔案系統內的一個檔案被讀取時，流程是這樣的:
    - 1. 讀到檔案的inode號碼
    - 2. 由inode內的權限設定判定使用者能否存取此檔案
    - 3. 若能讀取則開始讀取inode內所記錄的資料放置於哪些block號碼內
    - 4. 讀出block號碼內的資料，組合起來成為一個檔案的實際內容
- 新建檔案的流程則是這樣的:
    - 1. 有寫入檔案的需求時，先到metadata區找到沒有使用中的inode號碼
    - 2. 到該inode號碼內，將所需要的權限與屬性相關資料寫入，然後在metadata區規範該inode為使用中，且更新superblock資訊
    - 3. 到metadata區找到沒有使用中的block號碼，將所需要的實際資料寫入block當中，若資料量太大，則繼續到metadata當中找到更多的未使用中的block號碼，持續寫入，直到寫完資料為止。
    - 4. 同步更新inode的紀錄與superblock的內容。
- 刪除檔案的流程:
    - 1. 將該檔案的inode號碼與找到所屬相關的block號碼內容抹除
    - 2. 將metadata區域的相對應的inode與block號碼規範為未使用
    - 3. 同步更新superblock資料
### 例題:
- 1. Linux的EXT2檔案系統家族中，格式化之後，除了metadata區塊之外，還有哪三個很重要的區塊?
    - superblock、inode table、data block area
- 2. 檔案的屬性、權限等資料主要放置於檔案系統的哪個區塊內?
    - inode中
- 3. 實際的檔案內容(程式碼或者是實際資料)放置在哪幾個區塊?
    - data block中
- 4. 每個檔案都會使用到幾個inode與block?
    - 一定用掉一個inode，但是block則看該檔案的容量而定，亦即block數量不固定。
- 5. Linux的EXT2檔案系統家族中，以CentOS7為例，inode與block的容量大致為多少byte?
    - `df -T`
    - `dumpe2fs /dev/vda2 | grep -i size`
### 6.1.3 目錄與檔名
- 當使用者在Linux下的檔案系統建立一個目錄時，檔案系統會分配一個inode與至少一塊block給該目錄。
    - inode紀錄該目錄的相關權限與屬性，並可記錄分配到的那塊block號碼；
    - block則是記錄在這個目錄下的檔名與該檔名占用的inode號碼資料
    - 也就是說目錄所佔用的block內容在紀錄如下的資訊:
- 讀取檔案資料時，最重要的就是讀到檔案的inode號碼，然而實際操作系統時，並不會理會inode號碼，而是透過檔名來讀寫資料的，因此，目錄的重要性就是記載檔名與該檔名對應的inode號碼
### 例題
- 1. 使用`ls -li /etc/hosts*`，觀察出現在最前面的數值，該數值即為inode號碼
    ![](https://i.imgur.com/vPJtnct.png)

- 2. 使用student的身分，建立`/tmp/inodecheck/`目錄，然後觀察`/tmp/inodecheck/`, `/tmp/inodecheck/`這兩個檔名的inode號碼
    - `cd /tmp`
    - `mkdir inodecheck`
    - `man ls`
        - ![](https://i.imgur.com/Q8P4SP1.png)
        - ![](https://i.imgur.com/l1Ff5vz.png)
    - `ls -i inodecheck inodecheck/.`
![](https://i.imgur.com/SiCdYbb.png)

- 3. 承上，使用`ll -d`觀察`/tmp/inodecheck`的第二個欄位，亦即連結欄位的數值為多少?嘗試說明為什麼?
    - inodecheck是空的資料夾，連結到它的檔案有1: 裡面的`.`，以及2: 外面(`/tmp/inodecheck`)的資料夾
![](https://i.imgur.com/AI1UhLn.png)
- 4. 建立`/tmp/inodecheck/check2/`目錄，同時觀察`/tmp/inodecheck/`, `/tmp/inodecheck/.`, `/tmp/inodecheck/check2/..`這三個檔名的inode號碼，然後觀察第二個欄位的數值變成什麼
    - `mkdir /tmp/inodecheck/check2`
    - `ls -li /tmp/inodecheck /tmp/inodecheck/. /tmp/inodecheck/check2/..`
    - ![](https://i.imgur.com/ooEwhnB.png)

### 6.1.4 ln連結檔的應用
- 目錄的預設連結數為2，這是因為每個目錄底下都有.這個檔名，而這個檔名代表目錄本身，因此目錄本身有兩個檔名連結到同一個inode號碼上
### 例題:
- 1. 前往`/dev/shm`建立名為check2的目錄，並更改工作目錄到`/dev/shm/check2`當中
    - `cd /dev/shm`
    - `mkdir chcek2`
    - `cd check2`
- 2. 將`/etc/hosts`複製到本目錄下，同時觀察檔名連結數。
    - `cp /etc/hosts .`
    - ll
    - ![](https://i.imgur.com/iQzKcgE.png)
- 3. 使用`ln hosts hosts.real`建立`hosts.real`實體連結檔，同時觀察這兩個檔案的inode號碼、屬性權限等，是否完全相同?為什麼?
    - man ln![](https://i.imgur.com/A0eimEJ.png)
    - ll -i
    ![](https://i.imgur.com/ThpaIV9.png)
    - 屬性權限相同，連接數+1
- 4. 使用`ln -s hosts hosts.symbo`建立`hosts.symbo`符號連結，同時觀察這兩個檔案的inode號碼、屬性權限等，是否相同?
    - man ln
    ![](https://i.imgur.com/Je3WKVj.png)
    - ![](https://i.imgur.com/rayTY3D.png)
    - 符號連結時，兩個檔名對應的inode不同
    - 注意容量，hosts有5個字元，因此占用5bytes
- 5. 使用`cat hosts; cat hosts.real; cat hosts.symbo`查閱檔案內容是否相同
    - man cat
    ![](https://i.imgur.com/9NwBLFK.png)
    - 相同
- 6. 請刪除hosts，然後觀察hosts.real, hosts.symbo的inode號碼、連結數檔案屬性等資料，發現什麼情況
    - `rm -r hosts`
    - `ll -i`
    ![](https://i.imgur.com/VRwCrpo.png)
- 7. 使用`cat hosts.real; cat hosts.symbo`發生什麼狀況?為什麼?
    ![](https://i.imgur.com/183QLS5.png)
- 8. 在`/dev/shm/check2`底下執行`ln /etc/hosts .`會發生什麼情況?分析原因為何
    - `ln /etc/hosts .`
    ![](https://i.imgur.com/k5F9p8r.png)
    - `/dev/shm`以及`/etc`就是`/`分屬不同的檔案系統，實體連結必須要指向同一個inode號碼，而且，必須要在同一個裝置上面，如果跨檔案系統，因為不同的檔案系統其inode號碼參照並不相同，所以不能跨檔案系統執行實體連結
### 6.1.5 檔案系統的掛載
- 就像隨身碟放入windows作業系統後，需要取得一個`H:\>`，或者是其他的磁碟名稱後才能被讀取一樣，Linux底下的目錄樹系統中，檔案系統裝置要能夠被讀取，就得要與目錄樹的某個目錄連結在一起，亦即進入該目錄即可看到該裝置的內容之意，該目錄就被稱為==掛載點==
- 觀察掛載點的方式最簡單為使用df (display filesystem)這個指令來觀察，而讀者也可以透過觀察inode的號碼來了解到掛載點的inode號碼
- man df
![](https://i.imgur.com/qzEG8UX.png)
### 例題
- 1. 檔案系統要透過掛載(mount)之後才能夠讓作業系統存取。那麼與檔案系統掛載的掛載點是一個目錄還是檔案
    - 掛載點就是進入點的目錄
- 2. 使用`df -T`指令觀察目前的系統中，屬於xfs檔案系統的掛載點有哪幾個?
    - `man df`
    - ![](https://i.imgur.com/zQFrTzZ.png)
    - ![](https://i.imgur.com/LwmwdHg.png)
- 3. 使用`ls -lid`觀察`/, /boot, /home, /etc, /root, /proc, /sys`等目錄的inode號碼
    - ![](https://i.imgur.com/E8ZpDfa.png)
    - ![](https://i.imgur.com/dorLH4E.png)
    - ![](https://i.imgur.com/YoOHQAx.png)
    - ![](https://i.imgur.com/p0zNBAi.png)
    - `ls -lid / /boot /home /etc /root /proc /sys`
    - ![](https://i.imgur.com/F0dcqKt.png)
- 4. 為什麼`/ /boot /home`的inode號碼會一樣?
## 6.2 檔案系統管理
- 一般來說，建立檔案系統需要的動作包括: ==分割、格式化與掛載==三個步驟。而分割又有MBR與GPT兩種方式，實作時需要特別留意
### 6.2.1 建立分割
- 建立分割之前，需要先判斷
    - 1. 目前系統內的磁碟檔名
    - 2. 磁碟目前的分割格式
### 例題
- 使用root身分完成如下的練習
- 1. 先用lsblk簡單的列出裝置檔名
    ![](https://i.imgur.com/2YoK6SV.png)

- 2. 使用man lsblk找出來
    - 1. 使用純文字(ASCII)顯示的選項
    ![](https://i.imgur.com/an8f0T1.png)

    - 2. 列出完整的裝置檔名的選項
    ![](https://i.imgur.com/KHoONGe.png)
    ![](https://i.imgur.com/U9Lst7i.png)

    - 3. 使用`parted < 完整裝置檔名 > print`指令，找出分割表的類別(MBR/GPT)
    ![](https://i.imgur.com/Ifm3PWH.png)
![](https://i.imgur.com/kpzmRfz.png)
- 注意上圖的Partition Table: gpt
- 如果是GPT的分割表，請使用`dgisk`指令來分割，若為`msdos(MBR)`分割表，需要使用`fdisk`來分割。
- `man gdisk`
![](https://i.imgur.com/xsogXE2.png)
- `gdisk /dev/sda`
![](https://i.imgur.com/tMpAolB.png)
![](https://i.imgur.com/ZeNDocH.png)
### 例題:
- 請使用root的身分完成底下的任務:
- 1. 使用gdisk/dev/vda進入gdisk的界面
- 2. 按下p取得目前的分割表，並且觀察目前是否還有其他剩餘的容量可以使用
![](https://i.imgur.com/6W3H26K.png)
- 3. 按下n進行新增的動作
    - 在partition number的地方直接按下enter使用預設值4
    - 在first sector的地方也可以直接按下enter使用預設值即可
![](https://i.imgur.com/kmBGlMn.png)
- 4. 按下p來查閱一下是否取得正確的容量
- 5. 上述動作觀察後，若沒有問題，按下w來儲存後離開
- 6. 使用`lsblk`是否有查閱到剛剛建立的分割槽檔名?
- 7. 使用partprobe之後，再次lsblk，此時是否出現了新的分割槽檔名
- 由於`/dev/vda`磁碟正在使用中，因此核心預設不會重新去探索分割表的變動，讀者需要使用`partprobe`強制核心更新目前使用中的磁碟的分割表，這樣才能夠找到正確的裝置檔名。
- 若需要列出核心偵測到的完整分割表，也能使用cat/proc/partitions來觀察
### 例題:
- 使用如上個例題的流程，再次建立如下兩個裝置:
    1. 大約1.5G(1500MB)的vfat分割槽(GUID應該是0700)但請自己找出來
        - 用1500M，不能用1.5G
    2. 大約1G的swap分割曹，同樣的請自己找出filesystem ID
- 請注意，分割完畢並且在gdisk介面按下w儲存後，務必使用lsblk觀察是否有出現剛剛建置的分割槽裝置檔名，若無該裝置檔名，則應該使用partprobe或是reboot強制核心重新抓取
### 6.2.2 建立檔案系統(磁碟格式化)
- 檔案系統的建立使用`mkfs`處理
- 記憶體置換應該要使用`mkswap`
### 例題:
- 使用`mkfs.xfs /dev/vda4`建立好XFS檔案系統
- 使用`mkfs.vfat /dev/vda5`建立好FAT檔案系統
- 使用`mkswap /dev/vda6`建立好swap記憶體置換空間
- 使用`blkid`查詢到每個裝置的相關檔案系統與UUID資訊
### 6.2.3 檔案系統的掛載/卸載
- 檔案系統要掛載時，請先注意到底下的需求:
    - 單一檔案系統不應該被重複掛載在不同的掛載點(目錄)中
    - 單一目錄不應該重複掛載多個檔案系統
    - 要做為掛載點的目錄，理論上應該都是空目錄才是。
- 常見的掛載方式如下:
    - ![](https://i.imgur.com/t6Ltcrk.png)
### 例題:
- 1 請使用umount以及swapoff的方式來將`/dev/vda4`, `/dev/vda5`, `/dev/vda6`卸載，並自行觀察是否卸載成功
### 6.2.4 開機自動掛載
- 開機自動掛載的參數設定檔寫入在`/etc/fstab`當中，不過在編譯這個檔案之前，管理員應該先知道系統掛載的限制:
    - 根目錄是必須掛載的，而且一定要先於其他mount point被掛載進來
    - 其他mount point必須為已建立的目錄，可任意指定，但一定要遵守必須的系統目錄架構原則(FHS)
    - 所有mount point在同一時間之內，只能掛載一次
    - 所有partition在同一時間之內，只能掛載一次
    - 如若進行卸載，您必須先將工作目錄移到`mount point`及其子目錄之外
# 6.3 開機過程檔案系統問題處理
## 6.3.1 檔案系統的卸載與移除
- 若需要將檔案系統卸載並回收，一般建議的流程如下:
    - 判斷檔案系統是否使用中，若使用中則需卸載
    - 查詢是否有寫入自動掛載設定檔，若有則需要將設定內容移除
    - 將檔案系統的`superblock`內容刪除
## 6.3.2 開機過程檔案系統出錯的救援
- 管理員如果修改過`/etc/fstab`卻忘記使用mount -a測試，則當設定錯誤，非常有可能會無法順利開機
- 如果是跟目錄設定出錯，問題會比較嚴重
- 如果是一般正規目錄設定錯誤，則依據該目錄的重要性，可能會進入單人維護模式，或者是依舊可以順利開機。
# 總結
1. 檔案分割的介紹與操作
2. mount
3. 開機自動mount
4. 簡易的除錯