###### tags: 'OS Lab111 Meeting'
# 0912 Meeting




## 一言以蔽之，Android系統是真多工，iOS是假多工
  - iOS：把主要的記憶體都讓出來給目前前景的APP使用
  - Android：若亂清記憶體容易造成其他APP產生page fault
</br>

## Android為節省記憶體採用的方法
  - 引入 ZRAM/ZSWAP
  
  - 缺點：
    - 判斷要壓縮哪部分可能不夠準確
	- 壓縮這種事通常很需要CPU運算
	- 既然都叫CPU多算一些東西了當然也有可能會多耗一些電，對行動裝置不利
	
  - GOOGLE打了新的patch到linux kernel：MQ/2Q演算法（這部份我沒記得很清楚）
</br>

## Android的置換機制（SWAP）
  - 將近期較少用的page SWAP OUT（類似LRU的概念）
  - 過去設計是只置換 clean page（因為這樣比較快）
  - 現今設計是也置換 dirty page（這點不確定 待查）
</br>

## 實驗室的研究
  - 單位時間內的存取次數通常都分佈在前跟後（圖表成中間凹下的形狀）

  - malloc分配出來的空間分冷熱兩種
    - HOT：比較常存取的
    - COLD：比較少存取的

  - 因COLD較少用，需要SWAP OUT的時候就從這塊先（符合前面說的置換機制）
  - 可以有個profile的機制告訴OS kernel哪些區塊是比較常存取的（這部份最好由公正第三方做評測，APP開發者都不老實😡）
    系統在做malloc的時候可以可以根據此資訊做分配
    修改函式庫的目的也就是為了成達到這功能
</br>

## Reference
- <https://ithelp.ithome.com.tw/articles/10243650>
</br>





</br>
</br>

--------


### 記憶體
* keywords: DRAM, MVM
* 不知道結果 --> 正在做 ing
* Android 手機閃退的問題
    * 不像 ios 系統可以將某個程式的prio 調到最大
    * 但 android 不行，會發生 page fault 然後被系統接管
    * 記憶體不足可能會閃退
    * 因為 android 手機是真正的多工手機

* 現在(Linux)分割 2/3 的記憶體出來當虛擬硬碟 --> swap space --> block device --> 約 1.5 倍
* 為甚麼以前沒有發生?
    * 記憶體貴 > $cpu
    * 後來記憶體變便宜
    * 後來不隨意增加硬體 --> 手機的大小、耗電量
* Google 針對此問題提出
    * 分割記憶體
    * 記憶體壓縮
    * 將沒在用的記憶體丟到 ZRAM/Zswap 去做壓縮
    * 但現在判斷準確率不夠 --> 來不及壓縮 
    * Out Of Memory killer
    * 2 queue --> multi-queue algorithm

* 實驗室方法:
    * google 分成好壞陣營
    * 實驗室讓好的更好、壞得更壞
    * 將常用的記憶體搬到同一個 page
        * 同一個 malloc 出來的行為非常相似
        * 觀察 malloc 出來的記憶體分配到哪些地方
        * 觀察 cpu 存取的記憶體在哪裡
        * 就可以知道 malloc 出來的記憶體物件是屬於常用還是不常用
        * 引導 C 函數庫 malloc caller 
        * Android (DVM)
 
