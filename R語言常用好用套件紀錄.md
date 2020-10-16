# R語言常用好用套件紀錄
> [color=#40f1ef][name=LHB阿好伯, 2020/02/17][:earth_africa:](https://www.facebook.com/LHB0222/)
###### tags: `R`

[TOC]
# 資料科學整合套件-tidyverse![](https://i.imgur.com/LZKLrBU.png)
tidyverse整合數據科學所需的R套件
共享整理數據的基本設計原理，語法和數據結構
有望成為R的統一規範
改變整個R風格

 [:earth_asia:tidyverse](https://www.tidyverse.org/)

# 資料處理
## 時間序列lubridate![](https://i.imgur.com/s5oV2Tz.png)

:earth_asia: [lubridate](https://lubridate.tidyverse.org/)
快速方便地解析日期時間數據，提取和更新日期時間的組成部分（年，月，日，小時，分鐘和秒）
### 時間序列預測-forecast![](https://i.imgur.com/cgWBi3q.png)
用於顯示和分析單變量時間序列預測的方法和工具
通過狀態空間模型和自動ARIMA建模的指數平滑
:earth_asia:[forecast](https://pkg.robjhyndman.com/forecast/)
[:book: 線上參考書籍](https://otexts.com/fpp2/)
![](https://i.imgur.com/OPORftT.png)

### R與Python的預測推估套件-Prophet![](https://i.imgur.com/7aypMpF.png)
:earth_asia:[Prophet](https://facebook.github.io/prophet/)
![](https://i.imgur.com/lIh7OpU.png)

### 時間序列的工具包-timetk![](https://i.imgur.com/bEEcGGc.png)
便於可視化，整理和預處理時間序列數據以進行預測和機器學習預測。

![](https://i.imgur.com/BptnGMs.png)




## 數據表data.table![](https://i.imgur.com/T07i4cn.png)

提供具有語法和功能增強功能的Base R的高性能版本，data.frame以簡化易用性，便利性和編程速度。
:earth_asia: [data.table](https://rdatatable.gitlab.io/data.table/)
:earth_asia: [超高性能數據處理包data.table|張丹（Conan）](http://blog.fens.me/r-data-table/)

## 長寬表轉換-reshape2
僅使用兩個功能即可靈活地重組和聚合數據：melt和“ dcast”（或“ acast”）。

:earth_asia: [reshape2](https://github.com/hadley/reshape)

![](https://i.imgur.com/d0lVHte.png)

## 資料清洗-tidyr![](https://i.imgur.com/iObNquW.png)


:earth_asia: [tidyr](https://tidyr.tidyverse.org/)
:earth_asia: [tidyr演示動畫](https://github.com/gadenbuie/tidyexplain#relational-data)



## 優美的HTML表格-DT
R數據對象（矩陣或數據框架）可以顯示為HTML頁面上的表格
DataTables在表格中提供過濾，分頁，排序和許多其他功能
:earth_asia: [DT: An R interface to the DataTables library](https://rstudio.github.io/DT/)

## DT類似套件
基於React Table庫並使用reactR製作的R的交互式數據表
:earth_asia: [reactable](https://glin.github.io/reactable/index.html)

### 輕鬆生成信息豐富，出版用表格-gt![](https://i.imgur.com/zBzGnmG.png)
![](https://i.imgur.com/ALbYUxa.png)
![](https://i.imgur.com/4knt67l.png)
:earth_asia: [gt](https://github.com/rstudio/gt)

### 更改表格布局-flextable

flextable可以輕鬆地從創建報表data.frame。您可以合併單元格，添加頁眉行，添加頁腳行，更改任何格式並指定數據在單元格中的顯示方式。表格內容還可以包含內容，文本和圖像的混合類型
![](https://i.imgur.com/uPCfQx7.png)
:earth_asia: [flextable](https://davidgohel.github.io/flextable/articles/overview.html)

### 條件儲存格-formattable
應用於格式化數據表格
使數據表示更容易，更豐富，更靈活，傳達更多信息
![](https://i.imgur.com/qJsxJWY.png)

### 快速數據分析平台
![](https://i.imgur.com/p2xKyxt.png)
![](https://i.imgur.com/NeVoG5S.png)



## 運行SQL語法-sqldf
使用SQL處理R數據集
:earth_asia: [sqldf-github](https://github.com/ggrothendieck/sqldf) or [sqldf](https://cran.r-project.org/web/packages/sqldf/index.html)

## 字串處理-stringr![](https://i.imgur.com/H65eVlR.png) & stringi
快速，正確，一致，可移植以及方便的 字符串/文本處理
![](https://i.imgur.com/VK9z2YL.png)

:earth_asia:[stringr](https://stringr.tidyverse.org/)

:earth_asia:[stringi](http://www.gagolewski.com/software/stringi/)

## 矩陣處理-matricks ![](https://i.imgur.com/EbUqNGb.png)
主要函數 `m()` and `v()`
可以快速建立垂直向量與矩陣

一般矩陣的建立使用 `matrix()`
```R=
matrix(c(5, 6, 7,
         8, 0, 9,
         3, 7, 1), nrow = 3, byrow = TRUE)
```
:::success
         [,1] [,2] [,3]
     [1,]    5    6    7
     [2,]    8    0    9
     [3,]    3    7    1
:::
 
而使用 `m()` 則可以快速建立並且更簡單
```R=+
library(matricks)

m(5, 6, 7|
  8, 0, 9|
  3, 7, 1)
```
:::success
          [,1] [,2] [,3]
     [1,]    5    6    7
     [2,]    8    0    9
     [3,]    3    7    1
:::

```R=
v(1,2,3)
```
:::success
          [,1]
     [1,]    1
     [2,]    2
     [3,]    3
:::

:earth_asia:[matricks](https://krzjoa.github.io/matricks/)

## R&Python資料交換 - Feather
一種快速，輕量級的文件格式支援python和R語言,同時也可以被其他語言讀取
Feather當前支持以下列類型：
- 多種數值類型（int8，int16，int32，int64，uint8，uint16，uint32，uint64，float，double）。
- 邏輯/布爾值
- 日期，時間和時間戳
- 具有固定的可能值集的因子/類別變量
- UTF-8編碼的字符串
- 任意二進制數據

所有列類型均支持NA /空值

:::danger
read_feather(path, columns = NULL)
write_feather(x, path)
:::

:earth_asia:[Feather](https://github.com/wesm/feather)


# 資料可視化
## 必學會圖套件-ggplot2![](https://i.imgur.com/eOmgKN8.png)

:earth_asia:[ggplot2](https://ggplot2.tidyverse.org/)
![](https://i.imgur.com/uG8Kfi5.png)

### :earth_asia:[ggplot2擴增套件列表](https://exts.ggplot2.tidyverse.org/)
![](https://i.imgur.com/2QIym9k.png)

## ggplot2 GUI-esquisse![](https://i.imgur.com/EUnq2Ol.png)

以拖拉的方式快速生成ggplot圖形

:earth_asia:[esquisse](https://dreamrs.github.io/esquisse/index.html)

![](https://i.imgur.com/6X7cWyI.png)


## 圖片配置
## UpSetR
生成靜態UpSet圖。UpSet技術以矩陣佈局可視化集合相交
並基於分組和查詢引入集合。矩陣佈局可有效表示關聯數據
例如聚合和交集中元素的數量，以及從子集或元素屬性派生的其他摘要統計信息。
:earth_asia:[UpSetR](http://caleydo.org/tools/upset/)
```R=
upset(movies,attribute.plots=list(gridrows=60,plots=list(list(plot=scatter_plot, x="ReleaseDate", y="AvgRating"),
list(plot=scatter_plot, x="ReleaseDate", y="Watches"),list(plot=scatter_plot, x="Watches", y="AvgRating"),
list(plot=histogram, x="ReleaseDate")), ncols = 2))
```
![](https://i.imgur.com/KbE883J.png)


## ggplot2繪圖布置
### 快速進行ggplot2繪圖布置-patchwork![](https://i.imgur.com/znMpBuZ.png)

非常簡單且直觀的方式將單獨的ggplots組合到同一圖形中
:earth_asia:[patchwork](https://github.com/thomasp85/patchwork)

![](https://i.imgur.com/Phj4PzL.png)

### Cowplot
![](https://i.imgur.com/NQHX0MQ.png)

有助於創建具有出版物質量的圖形的各種功能，例如一組主題，將圖表對齊並將其排列成複雜的複合圖形的功能，以及使註釋註釋和/或將圖形與圖像混合變得容易的功能。
:earth_asia:[Cowplot](https://wilkelab.org/cowplot/index.html)

[![](https://i.imgur.com/am2548I.png)](https://www.r-bloggers.com/2020/10/how-to-automate-exploratory-analysis-plots/)


## 支援ggplot2交互式開源圖形庫-Plotly ![](https://i.imgur.com/5MVMjYG.png)
用於通過[plotly](https://plot.ly/)的JavaScript圖形庫創建交互式的基於Web的圖形
可轉換ggplot2的圖形為Plotly交互式圖形
並且也支援python


:earth_asia:[plotly](https://plot.ly/r/#basic-charts)


## 交互式網頁Shiny![](https://i.imgur.com/XpGnhGu.png)


交互式方法通過Shiny講述您的數據故事。讓用戶與您的數據和您的分析進行交互。並用R完成所有操作
 [:earth_asia: shiny](https://shiny.rstudio.com/reference/shiny/)

[:earth_asia:線上範例](https://shiny.rstudio.com/gallery/)
### 線上部署Shiny app

![](https://i.imgur.com/9h2yLzk.png)

:earth_asia: [Shinyapps](https://www.shinyapps.io/)


### 手機模板-shinyMobile![](https://i.imgur.com/GtmLI7T.png)


為“ iOS”，“ Android”，桌機以及漂亮的“shiny”小工具開發出色的“shiny”應用
![](https://i.imgur.com/OzXmGmV.png)

:earth_asia: [shinyMobile](https://rinterface.github.io/shinyMobile/)

### 創建Shiny拖放物件-ShinyDND
:earth_asia: [shinyDND](https://github.com/ayayron/shinydnd)

### Shiny物件提示與回饋-shinyFeedback
用於在Shiny的輸入項目旁邊顯示用戶提示與回饋

![](https://i.imgur.com/k8jUJhY.png)

:earth_asia: [shinyFeedback](https://merlinoa.github.io/shinyFeedback/index.html)



## 簡單創建儀表板-flexdashboard
使用R Markdown可以將一組相關的數據可視化製作成儀表板。

支持多種組件，包括htmlwidgets ; base, lattice, and grid graphics; tabular data; gauges and value boxes；和文字註釋。

靈活且易於指定基於行和列的佈局。可以智能地調整組件的大小以填充瀏覽器並適合在移動設備上顯示。

演示圖板佈局，用於呈現可視化序列和相關註釋。

（可選）使用Shiny動態驅動可視化。
![](https://i.imgur.com/YbnLhPV.png)
:earth_asia: [flexdashboard](https://rmarkdown.rstudio.com/flexdashboard/index.html)

# 打包Shiny成電腦軟體-electricShine
![](https://i.imgur.com/w2SecX3.png)
:earth_asia: [electricShine](https://github.com/chasemc/electricShine)

## 各式JavaScript可視化庫-htmlwidgets for R

就像繪圖一樣，在R控制台上使用JavaScript可視化庫
將小部件嵌入R Markdown文檔和Shiny Web應用程序中
使用無縫連接R和JavaScript的框架開發新的小部件

:earth_asia: [htmlwidgets](http://www.htmlwidgets.org/index.html)

## 開源交互式地圖-Leaflet for R

:earth_asia: [Leaflet](http://rstudio.github.io/leaflet/)

### 離線地圖圖資

:earth_asia: [GADM地圖和數據](https://gadm.org/index.html)

## 條件儲存格-formattable
格式化應用於向量和數據框
使數據表示更容易，更豐富，更靈活，傳達更多信息
![](https://i.imgur.com/fqUzO4S.png)

:earth_asia: [formattable](https://github.com/renkun-ken/formattable)

## [evoPalette](https://github.com/doehm/evoPalette)
evoPalette允許發現和生成R的新調色板
![](https://i.imgur.com/PyTzbyJ.png)
:earth_asia: [evoPalette](https://github.com/doehm/evoPalette)

## 各式調色盤集合-paletteer![](https://i.imgur.com/tRfGimr.png)
一個通用的接口將各調色板全面收集

:earth_asia: [paletteer](https://github.com/EmilHvitfeldt/paletteer)

| Name | Github | CRAN |
| :-- | :-- | :-- |
| awtools | [awhstin/awtools - 0.2.1](https://github.com/awhstin/awtools) | - |
| basetheme | [KKPMW/basetheme - 0.1.2](https://github.com/KKPMW/basetheme) | [0.1.2](https://cran.r-project.org/package=basetheme) |
| calecopal | [an-bui/calecopal - 0.1.0](https://github.com/an-bui/calecopal) | - |
| cartography | [riatelab/cartography - 2.3.1.0](https://github.com/riatelab/cartography) | [2.3.0](https://cran.r-project.org/package=cartography) |
| colorblindr | [clauswilke/colorblindr - 0.1.0](https://github.com/clauswilke/colorblindr) | - |
| colRoz | [jacintak/colRoz - 0.2.2](https://github.com/jacintak/colRoz) | - |
| dichromat | - | [2.0-0](https://cran.r-project.org/package=dichromat) |
| DresdenColor | [katiesaund/DresdenColor - 0.0.0.9000](https://github.com/katiesaund/DresdenColor) | - |
| dutchmasters | [EdwinTh/dutchmasters - 0.1.0](https://github.com/EdwinTh/dutchmasters) | - |
| fishualize | [nschiett/fishualize - 0.2.999](https://github.com/nschiett/fishualize) | [0.1.0](https://cran.r-project.org/package=fishualize) |
| futurevisions | [JoeyStanley/futurevisions - 0.1.0](https://github.com/JoeyStanley/futurevisions) | - |
| gameofthrones | [aljrico/gameofthrones - 1.0.2](https://github.com/aljrico/gameofthrones) | [1.0.2](https://cran.r-project.org/package=gameofthrones) |
| ggpomological | [gadenbuie/ggpomological - 0.1.2](https://github.com/gadenbuie/ggpomological) | - |
| ggsci | [road2stat/ggsci - 2.9](https://github.com/road2stat/ggsci) | [2.9](https://cran.r-project.org/package=ggsci) |
| ggthemes | [jrnold/ggthemes - 4.2.0](https://github.com/jrnold/ggthemes) | [4.2.0](https://cran.r-project.org/package=ggthemes) |
| ggthemr | [cttobin/ggthemr - 1.1.0](https://github.com/cttobin/ggthemr) | - |
| ghibli | [ewenme/ghibli - 0.3.1.9000](https://github.com/ewenme/ghibli) | [0.3.1](https://cran.r-project.org/package=ghibli) |
| grDevices | - | 2.0-14 |
| harrypotter | [aljrico/harrypotter - 2.1.1](https://github.com/aljrico/harrypotter) | [2.1.1](https://cran.r-project.org/package=harrypotter) |
| IslamicArt | [lambdamoses/IslamicArt - 0.1.0](https://github.com/lambdamoses/IslamicArt) | - |
| jcolors | [jaredhuling/jcolors - 0.0.4](https://github.com/jaredhuling/jcolors) | [0.0.4](https://cran.r-project.org/package=jcolors) |
| LaCroixColoR | [johannesbjork/LaCroixColoR - 0.1.0](https://github.com/johannesbjork/LaCroixColoR) | - |
| lisa | [tyluRp/lisa - 0.1.1.9000](https://github.com/tyluRp/lisa) | [0.1.1](https://cran.r-project.org/package=lisa) |
| MapPalettes | [disarm-platform/MapPalettes - 0.0.2](https://github.com/disarm-platform/MapPalettes) | - |
| miscpalettes | [EmilHvitfeldt/miscpalettes - 0.0.0.9000](https://github.com/EmilHvitfeldt/miscpalettes) | - |
| nationalparkcolors | [katiejolly/nationalparkcolors - 0.1.0](https://github.com/katiejolly/nationalparkcolors) | - |
| NineteenEightyR | [m-clark/NineteenEightyR - 0.1.0](https://github.com/m-clark/NineteenEightyR) | - |
| nord | [jkaupp/nord - 1.0.0](https://github.com/jkaupp/nord) | [1.0.0](https://cran.r-project.org/package=nord) |
| ochRe | [ropenscilabs/ochRe - 1.0.0](https://github.com/ropenscilabs/ochRe) | - |
| oompaBase | - | [3.2.9](https://cran.r-project.org/package=oompaBase) |
| palettesForR | [frareb/palettesForR - 0.1.2](https://github.com/frareb/palettesForR) | [0.1.2](https://cran.r-project.org/package=palettesForR) |
| palettetown | [timcdlucas/palettetown - 0.1.1.90000](https://github.com/timcdlucas/palettetown) | [0.1.1](https://cran.r-project.org/package=palettetown) |
| palr | [AustralianAntarcticDivision/palr - 0.2.0](https://github.com/AustralianAntarcticDivision/palr) | [0.2.0](https://cran.r-project.org/package=palr) |
| pals | [kwstat/pals - 1.6](https://github.com/kwstat/pals) | [1.6](https://cran.r-project.org/package=pals) |
| PNWColors | [jakelawlor/PNWColors - 0.1.0](https://github.com/jakelawlor/PNWColors) | - |
| Polychrome | - | [1.2.4](https://cran.r-project.org/package=Polychrome) |
| rcartocolor | [Nowosad/rcartocolor - 2.0.0](https://github.com/Nowosad/rcartocolor) | [2.0.0](https://cran.r-project.org/package=rcartocolor) |
| RColorBrewer | - | [1.1-2](https://cran.r-project.org/package=RColorBrewer) |
| Redmonder | - | [0.2.0](https://cran.r-project.org/package=Redmonder) |
| RSkittleBrewer | [alyssafrazee/RSkittleBrewer - 1.1](https://github.com/alyssafrazee/RSkittleBrewer) | - |
| scico | [thomasp85/scico - 1.1.0](https://github.com/thomasp85/scico) | [1.1.0](https://cran.r-project.org/package=scico) |
| tidyquant | [business-science/tidyquant - 1.0.0](https://github.com/business-science/tidyquant) | [1.0.0](https://cran.r-project.org/package=tidyquant) |
| trekcolors | [leonawicz/trekcolors - 0.1.2](https://github.com/leonawicz/trekcolors) | [0.1.1](https://cran.r-project.org/package=trekcolors) |
| tvthemes | [Ryo-N7/tvthemes - 1.1.0](https://github.com/Ryo-N7/tvthemes) | [1.1.0](https://cran.r-project.org/package=tvthemes) |
| unikn | [hneth/unikn - 0.2.0.9003](https://github.com/hneth/unikn) | [0.2.0](https://cran.r-project.org/package=unikn) |
| vapeplot | [seasmith/vapeplot - 0.1.0](https://github.com/seasmith/vapeplot) | - |
| vapoRwave | [moldach/vapoRwave - 0.0.0.9000](https://github.com/moldach/vapoRwave) | - |
| viridis | [sjmgarnier/viridis - 0.5.1](https://github.com/sjmgarnier/viridis) | [0.5.1](https://cran.r-project.org/package=viridis) |
| visibly | [m-clark/visibly - 0.2.7](https://github.com/m-clark/visibly) | - |
| werpals | [sciencificity/werpals - 0.1.0](https://github.com/sciencificity/werpals) | - |
| wesanderson | [karthik/wesanderson - 0.3.6.9000](https://github.com/karthik/wesanderson) | [0.3.6](https://cran.r-project.org/package=wesanderson) |
| yarrr | [ndphillips/yarrr - 0.1.6](https://github.com/ndphillips/yarrr) | [0.1.5](https://cran.r-project.org/package=yarrr) |

## 2D to 3D繪圖轉換-rayshader![](https://i.imgur.com/SLUD6X0.png)
rayshader是用於在R中生成2D和3D數據可視化效果的開源軟件包
rayshader使用基本R矩陣中的高程數據以及射線跟踪，球形紋理貼圖，疊加和環境光遮擋的組合來生成精美的2D和3D地形圖
除地圖外，rayshader還允許用戶將ggplot2對象轉換為精美的3D數據可視化效果

![](https://i.imgur.com/STIxqGp.png)


:earth_asia: [rayshader](https://github.com/tylermorganwall/rayshader)

# 網路資料
## 爬蟲-rvest![](https://i.imgur.com/djdmFrk.png)

rvest可幫助您從網頁中抓取信息。它被設計與[magrittr](https://github.com/smbache/magrittr)配合[使用](https://github.com/smbache/magrittr)，受python [Beautiful Soup](https://www.crummy.com/software/BeautifulSoup/)套件的啟發，輕鬆在常見的網頁抓取資料

### 好用Chrome套件
[XPath Helper](https://chrome.google.com/webstore/detail/xpath-helper/hgimnogjllphhhkhlmebbmlgjoejdpjl/related?hl=zh-TW)

[SelectorGadget](https://chrome.google.com/webstore/detail/selectorgadget/mhjhnkcfbdhnjickkkdbjoemdmbfginb?hl=zh-TW)

:earth_asia: [rvest](https://github.com/tidyverse/rvest)

## 讀取google硬碟資料![](https://i.imgur.com/oke5qFS.png)
允許您與R中的Google雲端硬盤中的文件進行交互
:earth_asia: [googledrive](https://googledrive.tidyverse.org/index.html)


# 實用
## 管道符-magritter `%>%`
![](https://i.imgur.com/7zBW70M.png)

減少開發時間並提高代碼的可讀性和可維護性


:earth_asia:[magritter](https://cran.r-project.org/web/packages/magrittr/vignettes/magrittr.html)


## 迴圈運算-purrr![](https://i.imgur.com/ggJvtJe.png)


通過提供用於處理函數和向量的完整且一致的工具集，增強了R的功能編程（FP）工具包。如果您以前從未聽說過FP，那麼最好的起點就是map()函數係列，它使您可以用更簡潔，更易於閱讀的代碼替換許多for循環。
:earth_asia: [purrr](https://purrr.tidyverse.org/)


## 創建MS office-officer R package![](https://i.imgur.com/IQOYyNl.png)

操作Word（.docx）和PowerPoint（*.pptx）文檔
可以將圖像，表格和文本利用R添加到的文檔中
:earth_asia: [officer R package](https://davidgohel.github.io/officer/)

## 創建與Excel-excelR

:earth_asia: [excelR](https://swechhya.github.io/excelR/)

## 線上電子表格-x-spreadsheet

![](https://i.imgur.com/N1oQIt8.png)

:earth_asia: [x-spreadsheet](https://github.com/myliang/x-spreadsheet)

##  renv 套件管理
https://rstudio.github.io/renv/articles/renv.html

## 自動執行R程式碼(Win) - taskscheduleR![](https://i.imgur.com/KnQIHL4.png)

允許在Windows設定特定時間點自動執行R程序
![](https://i.imgur.com/qWrGX8s.png)

## Excel中執行R_BERT
BERT是用於將Excel與統計語言R連接的工具
具體來說，它旨在支持從Excel電子表格單元格運行R函數
用Excel術語來說，是用R編寫用戶定義函數（UDF）
![](https://i.imgur.com/qslPaLD.png)
[:earth_asia:BERT](https://bert-toolkit.com/)


# 圖像處理
## R中的高級圖像處理 - magick![](https://i.imgur.com/UQjkOHU.png)
與ImageMagick的綁定：可用的最全面的開源圖像處理庫。支持許多常見格式（png，jpeg，tiff，pdf等）和操作（旋轉，縮放，裁切，修剪，翻轉，模糊等）
所有操作都是通過Magick ++ STL矢量化的，這意味著它們可以在單個幀或一系列幀上進行操作，以處理圖層，拼貼或動畫
在RStudio中，將圖像打印到控制台後會自動預覽，從而形成一個交互式編輯環境

:earth_asia: [magick](https://cran.r-project.org/web/packages/magick/vignettes/intro.html)

## 用於圖像處理的R包 - imager

:earth_asia: [imager](http://dahtah.github.io/imager/)

## 提取圖片顏色 - colorfindr
從各種圖像類型中提取顏色，繪製樹圖和顏色組成的3D散點圖，創建調色板。
:earth_asia: [colorfindr](https://github.com/zumbov2/colorfindr)

![](https://i.imgur.com/JAPpKpP.png)

# 其他實用工具

## 多線程提高運行性能-Microsoft R Open![](https://i.imgur.com/aOVbAuP.png)



:earth_asia: [Microsoft R Open: The Enhanced R Distribution](https://mran.revolutionanalytics.com/open)

## R與Python間快速交換數據-Feather
輕巧，最小的API：使數據盡可能快地寫入和寫出內存
與語言無關：無論是用Python還是R代碼編寫，Feather文件都是相同的。其他語言也可以讀取和寫入Feather文件

:earth_asia: [Feather](https://github.com/wesm/feather)
[參考範例](https://blog.rstudio.com/2016/03/29/feather/)


## 交互式學習-learnr

1. 敘述，圖形，插圖等資訊
2. 代碼練習（用戶可以直接編輯和執行的R代碼塊）。
3. 測驗問題
4. 視頻（受支持的服務包括YouTube和Vimeo）。
5. 交互式shiny
:earth_asia: [learnr](https://rstudio.github.io/learnr/index.html)


# [精選的R包，框架和軟件的精選列表](https://github.com/qinwf/awesome-R)

# tidymodels_機器學習
tidymodels框架是使用tidyverse原理進行建模和機器學習的軟件包的集合。
![](https://i.imgur.com/NalTYWN.png)

:earth_asia: [tidymodels](https://www.tidymodels.org/)


全文分享至
# [:page_with_curl: 全部文章列表](https://hackmd.io/@LHB-0222/AllWritings)


https://www.facebook.com/LHB0222/

有疑問想討論的都歡迎於下方留言

喜歡的幫我分享給所有的朋友 \o/

有所錯誤歡迎指教

![](https://i.imgur.com/47HlvGH.png)

