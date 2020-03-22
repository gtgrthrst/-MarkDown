# ggplot2入門筆記-1
> [color=#40f1ef][name=LHB阿好伯, 2020/03/22][:earth_africa:](https://www.facebook.com/LHB0222/)
###### tags: `R`

[TOC]

ggplot2 是R中有名的數據可視化套件
功能強大、靈活便捷
其語法靈感源自Leland Wilkison的《圖形的語法》一書
使用ggplot2 可以輕鬆實現
- 高質量圖形的繪製，自動化添加圖例
- 疊加來自不同數據源的多個圖層（點、線、地圖、箱型圖等）
- 利用R 強大的建模功能添加平滑曲線，如loess 、線性模型與回歸 
- 保存任意ggplot2 圖形，方便修改或重複使用 
- 製作主題，滿足內部定製或雜誌風格的需求，便捷地應用到多幅圖形上 
- 從視覺角度上審視你的圖形，斟酌每一部分數據如何呈現在最終圖形上 

總結來說有以下幾點 
- ggplot2的核心理念是將繪圖與數據處理分離
- ggplot2是按圖層作圖，一層一層疊加
- ggplot2保有命令式作圖的調整函數，使其更具靈活性 
- ggplot2將常見的統計變換融入到了繪圖中

在ggplot2函數大概可以分為三個部分：
- 數據圖層
- 幾何圖層
- 美學圖層

細分可以分為
# 數據（Data）和映射（Mapping）
    -  將數據中的變量映射到圖形屬性。映射控制了二者之間的關係

映射：aes()函數是ggplot2中的映射函數
所謂的映射即為數據集中的數據關聯到相應的圖形屬性過程中一種對應關係
圖形的顏色，形狀，分組等都可以通過通過數據集中的變量映射

以下以ggplot2內建資料集diamonds展示
diamonds中包含近54,000顆鑽石的價格和其他屬性的數據集
變量如下

price 
美元價格($ 326~$ 18,823)

carat
鑽石的重量（0.2~5.01）

cut
切割的質量（Fair, Good, Very Good, Premium, Ideal）

color
鑽石顏色，從D（最佳）到J（最差）

clarity
鑽石淨度的度量（I1（最差），SI2，SI1，VS2，VS1，VVS2，VVS1，IF（最佳））

X
長度以毫米（0--10.74）

y
寬度（毫米）（0--58.9）

z
深度以毫米為單位（0--31.8）

```R=
library(ggplot2)
#載入ggplot2內建資料
data(diamonds)
#取出前六筆資料
head(diamonds)
```
:::success


|carat| cut | color | clarity | depth | table | price | x | y | z |
|------|------|------|------|------|------|------|------|------|------|
| 0.23 | Ideal | E | SI2 | 61.5 | 55 | 326 | 3.95 | 3.98 | 2.43 |
| 0.21 | Premium | E | SI1 | 59.8 | 61 | 326 | 3.89 | 3.84 | 2.31 |
| 0.23 | Good | E | VS1 | 56.9 | 65 | 327 | 4.05 | 4.07 | 2.31 |
| 0.29 | Premium | I | VS2 | 62.4 | 58 | 334 | 4.2 | 4.23 | 2.63 |
| 0.31 | Good | J | SI2 | 63.3 | 58 | 335 | 4.34 | 4.35 | 2.75 |
| 0.24 | Very Good | J | VVS2 | 62.8 | 57 | 336 | 3.94 | 3.96 | 2.48 |
:::
## 基本圖形
```R=+
p <- ggplot(data = diamonds,mapping = aes(x = price, y = carat)) +  #設定資料集與映射資料
  geom_point() #添加點圖層繪製散佈圖
```

![](https://i.imgur.com/kjgVZ7l.png)

稍微修改一下映射資料與添加的圖層就可以畫出盒鬚圖
```R=+
p <- ggplot(data = diamonds,mapping = aes(x = color, y = carat)) +  #設定資料集與映射資料
  geom_boxplot()  

p
```
![](https://i.imgur.com/qKGuwJC.png)

## 視覺通道映射
在R語言中有幾個作為通道映射的參數
用於修改圖形參數
| 映射參數 | 屬性 | 補充 |
|------|------|------|
| colour/col | 修改點、線與填充區域輪廓顏色 | 
| fill | 修改填充區域顏色 | 
| alpha | 設定顏色透明度範圍0~1(不透明) | 
| size | 設定點大小或是線的寬度 | 
| angle | 控制文字的擺放角度 | 
| vjust, hjust | 控制位置擺放,vjust為垂直位置,hjust為水平調整 | 
| linetype | 線條類型![](https://i.imgur.com/MdVzGZ9.png)| <http://sape.inf.usi.ch/quick-reference/ggplot2/linetype> |
| shape | 點形狀![](https://i.imgur.com/1vVBMlp.png) | <https://ggplot2.tidyverse.org/reference/scale_shape.html> |
|group|資料分組處理||
|family|[字體*](http://www.cookbook-r.com/Graphs/Fonts/)![](https://i.imgur.com/TIbtRFN.png)|plain常規, bold粗體, italic斜體,bold.italic粗斜體 |

### 顏色-colour

而添加一小段程式碼`colour=clarity`在映射中就可以將鑽石淨度作為顏色依據將散佈圖上的點依照鑽石淨度不同更換為不同顏色
由此可以發現在越高價的鑽石中淨度越低其克拉數需要越高才能達到一樣的售價
```R=
p <- ggplot(data = diamonds, mapping = aes(x = price, 
    y = carat, colour = clarity)) +  #設定資料集與映射資料
  geom_point()  

p
```

![](https://i.imgur.com/aLf0deT.png)

若是將`colour = clarity`修改成為`colour = color` 已可以看到類似的結果
![](https://i.imgur.com/4iyZmlr.png)

顏色也可以自訂喜歡的配色
 - R有657種內置的命名顏色，可以用列出colours()。
 - 也可以使用RGB規格，如以下形式的字符串"#RRGGBB"，
 - 您可以選擇添加顏色透明度"#RRGGBBAA"。

### 點符號樣式-shape

在ggplot2中也可以修改其點形狀作為分類的功能
只需要加上`shape = cut`即可
```R=+
p <- ggplot(data = diamonds, mapping = aes(x = price, 
    y = carat, colour = clarity, shape = cut)) +  #設定資料集與映射資料
  geom_point()  

p
```
![](https://i.imgur.com/rE6wSYx.png)

上述在映射中會添加許多微調的資訊
但這些資訊也可放置於繪製圖層中
在觀看程式碼時會更好理解
```R=
p <- ggplot(data = diamonds,
    mapping = aes(x = price, y = carat)) +  #設定資料集與映射資料
geom_point(aes(colour = color))
p
```
![](https://i.imgur.com/Kj42cIX.png)

經常在資料非常密集的情況下我們也會使用`alpha = 0.2`來調整透明度可以讓圖片整體看起來更好
``` R=
p <- ggplot(data = diamonds,
    mapping = aes(x = price, y = carat)) +  #設定資料集與映射資料
geom_point(aes(colour = color), alpha = 0.2) 
```
![](https://i.imgur.com/ysRncE5.png)


學習到這邊大家應該對於視覺通道映射的使用有一定的了解了
後續我就用幾個簡單的例子說明各視覺通道映射的使用
嘗試看看是否能理解程式碼的差別與作用

### 尺吋-size

```R=
df <- data.frame(x1 = c(1,2,3,4,5), y1 = c(1,2,3,4,5), z1 = c(1,2,3,4,5))
p <- ggplot(data = df, aes(x = x1, y = y1)) +  
  geom_point(aes(size = z1))
p
```

![](https://i.imgur.com/EJgKl8g.png)

### 標籤文字角度-angle

``` R=+
p <- ggplot(data = df, aes(x = x1, y = y1)) +  
  geom_text(aes(x = x1, y = y1, label = 360/z1, angle = 360/z1))
p
```
![](https://i.imgur.com/zmDm7QH.png)

### 分組-group
```R=
p <- ggplot(data = diamonds,mapping = aes(x = color)) +  #設定資料集與映射資料
  geom_bar() 
p
```
![](https://i.imgur.com/evOiF05.png)

```R=+
p <- ggplot(data = diamonds,mapping = aes(x = color)) +  #設定資料集與映射資料
  geom_bar(aes(group = cut,colour = cut)) 
p
```
![](https://i.imgur.com/ejWiitm.png)

### 填充區域顏色-fill
```R=+
p <- ggplot(data = diamonds,mapping = aes(x = color)) +  #設定資料集與映射資料
  geom_bar(aes(group = cut,fill = cut)) 
p
```
![](https://i.imgur.com/g8gQcvW.png)



全文分享至

https://www.facebook.com/LHB0222/

有疑問想討論的都歡迎於下方留言

喜歡的幫我分享給所有的朋友 \o/

有所錯誤歡迎指教

![](https://i.imgur.com/47HlvGH.png)

