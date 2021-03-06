# Optimization of Shipping Schedulling of ACO 
## 背景
* 海運為國際貿易的主要方式: 
一艘船上面可以裝載上千甚至上萬個貨櫃，因此海運的貨運量是所有運具裡面最大的，許多大型企業都會將需要托運的貨物委託航運公司託運，故海運為國際貿易的主要方式。
* 船隻為重大的資本投資(單位:百萬美元): 
一艘船隻通常都伴隨著巨大的資本投資，隨著目前世界船舶大型化的趨勢，一艘船隻造價甚至可達上千萬美元。
* 適當的航線安排&調度對船隊績效有明顯影響
從上面可以看出船隻造價昂貴，購買船隻勢必會造成航運公司的財政負擔，然而船隻要是海運所必須的運具，故航運公司需要考慮如何安排適當的航線&調度船隻才可以為公司船隊帶來最大的利益。

綜合上面所述，我們想要使用蟻群演算來最佳化船公司的船隻調度，並嘗試探討本演算法是否真的會對航運公司的營運績效帶來正面的影響。
## 問題定義
首先我們會先定義何為船舶調度問題，在船舶調度問題的模擬中會有3個步驟:
* 需要找到幾條固定的航線
* 分析航線中的資訊，包含船隻數量、港口數量、航行時間、船隻數據、貨運量、油耗數據、總成本等等資訊，
* 將統整完的資訊輸入演算法中。

**老師特別提到港口容納船隻的工程問題**

**Answer : 由於我們可能會將不同運量船隻配入不同航線中，可能會有停靠港口無法容納過大船舶的問題，然而各航運公司在各個港口皆會購買/租貢屬於自己的港區，故停泊船隻的概念並不像停車格的概念而是像停車區域的概念，所以應該是不會有船隻過大無法停泊的問題，可能會因為船隻吃水深較深而有無法進港的疑慮；通常定期航線所停泊的港口內港水深都會到達一定深度才可以成為國際港口否則該港在目前國際船舶大型化的趨勢下是必會被淘汰。**

在以上資料及條件都完成後便可以進入參數計算及演算法模擬的部分。



## 相關參數
在這個模型中會有幾個主要的參數:
* 航線數據：我們從長榮海運公司的航線中找出3條亞洲-美洲的航線，分別為HTW、WSA4和WSA2，接著根據船期表依序計算出停泊港口數、停泊時間、航行時間、總時間

| 航線 | 停泊港口數 | 停泊時間 | 航行時間 | 總時間) |
| ---- | ---------- | -------- | -------- | ------- |
| HTW  | 6          | 7        | 23       | 30      |
| WSA4 | 11         | 11       | 20       | 31      |
| WSA2 | 9          | 18       | 21       | 39      |


* 船舶數據：同樣根據船期表所示，找出每條航線所使用的船舶運量、船舶數量還有船舶的淨噸位


| 航線 | 運量(TEU) | 淨噸位(T) | 船舶數量 | 船速(節) |
| ---- | --------- | --------- | -------- | -------- |
| HTW  | 12188     | 73121     | 6        | 23       |
| WSA4 | 12000     | 70268     | 8        | 22.3     |
| WSA2 | 6223      | 33168     | 4        | 19.5     |

* 船舶成本：根據不同運量、不同噸位的船舶計算其不同的油耗量、港口停泊費等數據後計算出一艘船航行一天之成本。  港口停泊費是由各國港口規定為準，一般是根據船之境噸位計算。  
油耗成本計算方式如下：  
依據標準，12188TEU的油耗為一個櫃0.011噸油 &rarr;  12188*0.011*0.04 = 129噸/天  
船用重油:0.94克/毫升、1噸=1000克 &rarr; 1000/0.94=1063公升  
129 x 1063公升 = 137127 公升/天 = 137.127 公秉/天  
最後，根據國內海運重柴油:25900元/公秉25900 x 137.127 &rarr; 3551589.3元/天


| 航線 | 燃油消耗 | 貨運量 |
| ---- | -------- | ------ |
| HTW  | 129      | 72000  |
| WSA4 | 113      | 104000 |
| WSA2 | 85       | 24892  |

| 航線 | 運量  | 單艘船型成本(萬元) |
| ---- | ----- | ------------------ |
| HTW  | 12188 | 10709.96           |
| WSA4 | 13000 | 9736.29            |
| WSA2 | 6223  | 9156.19            |


最後將一天的成本乘上總天數便可得到該航線的營運成本。

| 航線   | 停泊成本 | 航線總成本(萬元) |
| ------ | -------- | ----------------:|
| HTW    | 55.2     |         64259.76 |
| WSA4   | 81.03    |         77890.32 |
| WSA2   | 29.5423  |         36624.79 |
| 總成本 |          |        **178774.87** |




因為考慮到可能會將不同運量船舶配入不同航線中，所以我們另外將不同船隻在不同航線的成本計算出來

|     | HTW | WSA4 | WSA2 |
| --- | ---- | -------- | ---------------- |
|12188(TEU)    | 10709.96  | 11088.93     | 13880.64         |
|13000(TEU)     | 9388.44 | 9736.29    | 12162.67         |
| 6332(TEU)    | 7075.78 | 7333.61  | 9156.19         |



## 演算法
為了符合航運公司的實際經營情況，提出以下幾點假設：
* 在特定時刻中，航運公司沒有買賣船隻，也就是說船隻總數固定
* 同一種船隻的航速固定，也就是說航速不會因為貨物載重而影響

蟻群演算法(Ant Colony Optimization, ACO):
**蟻群演算法的優點在於避免了冗長的編程和籌劃，演算法本身是基於一定規則的隨機運行來尋找最佳配置，可以通過螞蟻經過留下的費洛蒙而不斷的去修正原來的路線，越短的路徑所留下的費洛蒙越多，從而使可行解逐步收斂於最優解**

以下為限制式條件
![](https://i.imgur.com/GSZkSnF.png)

因為目標函數是求極小值，所以費洛蒙的濃度應和目標函數的大小成反比，意即 **當目標函數越小、螞蟻留下的費洛蒙越大**  
以下為演算法流程圖

![](https://i.imgur.com/6zdAuA3.png)

經過蟻群演算法後，將各航線的船隻調配成以下：




| 航線 | 運量(TEU) | 船舶數量 | 成本(萬元) |
| ---- | --------- | -------- | ---------- |
| HTW  | 12188     | 3        |            |
|      | 12000     | 2        |            |
|      | 6223      | 2        |            |
|      |           |          | 65058.32   |
| WSA4 | 12188     | 3        |            |
|      | 12000     | 5        |            |
|      | 6223      | 0        |            |
|      |           |          | 81948.24   |
| WSA2 | 12188     | 0        |            |
|      | 12000     | 1        |            |
|      | 6223      | 2        | 23540      |
| 總成本|           |          | **170546.56**      |

## 結論
原本支出是 **178774.87萬元**，經過船隻在各航線的調配後，總成本為 **170546.56萬元**，下降 **5%**

但為何長榮並不一開始就將船隻混合使用而是使用單一運量的船舶我們認為原因為:
首先各船公司在進行航線配船一般會依循以下原則:
* 船舶技術及性能：考慮主要影響船舶正常運行的各種因素，例如尺度、船體強度、穩定性等等，這些因素直接關係到船舶在特定航線上是否合適的問題，畢竟有的航線風浪較大，而越大噸位的船隻意味著更大的穩定性，故在風浪大的航線使用大型船隻會較為安全。其他像是船舶載貨性能，目的則在於保證貨運質量。
* 船舶經濟性能：在各個條件皆符合的情況下，同一航線上船舶可以自由配置，但由於船舶噸位、油耗等有所不同，因此航線配船應該透過比較分析來配置最合理的船舶。

所以可以得知長榮在進行配船時一定也是依照上面的準則進行考量，後得到每條航線使用相同噸重船型的結果；然而我們的分析模型運用蟻群來得出最佳解，其中使用的參數也是極為貼近實際情況，除了航行路線海域狀況等無法考量外，本研究已經大概率接近現實的配船方式。






