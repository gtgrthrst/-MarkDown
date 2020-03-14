R編程風格指南
===
> [color=#40f1ef][name=LHB阿好伯, Jan 26, 2019 10:05 PM][:earth_africa:](https://www.facebook.com/LHB0222/)

###### tags: `R`

R語言是一門主要用於統計計算和繪圖的高級編程語言

依循R語言編碼風格指南使我們的R代碼更**容易閱讀，分享和檢查**


[TOC]

>資料參考自[Google's R Style Guide](https://google.github.io/styleguide/Rguide.xml)

文檔名 : Test.R
---
副檔名為.R
文件名稱應具**有意義**可提示文檔內容

:+1: predict_ad_revenue.R 

:-1: foo.R

變數名稱(identifiers)
---
不使用下劃線（_）或連字符（-）
變量名稱應包含所有小寫字母和單詞以點（.）分隔
函數名稱有大寫字母且沒有點（CapWords）
- variable.name 
:+1: avg.clicks 
:-1:  avg_Clicks ，avgClicks

- FunctionName 
:+1: CalculateAvgClicks
:-1: calculate_avg_clicks ， calculateAvgClicks 

句法
---
- 最大行長度為80個字元
- 縮進代碼時，請使用兩個空格
- 運算符號前後需保持間距 （=， +，-，<-，等）
-- 在使用等號傳遞參數時，前後的空格是可選則的
-- 不要在逗號前放置空格，但始終在逗號後面放置一個空格


```R=
# Good
tabPrior <- table(df[df$daysFromOpt < 0, "campaignid"])
total <- sum(x[, 1])
total <- sum(x[1, ])

# Bad
tabPrior <- table(df[df$daysFromOpt < 0,"campaignid"])  # 逗號後需要一個空格
tabPrior<- table(df[df$daysFromOpt < 0, "campaignid"])  # <- 之前需要一個空格
tabPrior<-table(df[df$daysFromOpt < 0, "campaignid"])  # <-前後需空格 
total <- sum(x[,1])  # 逗號後需要一個空格
total <- sum(x[ ,1])  # 在逗號之後，而不是之前創建一個空格
```

---
- 在左括號前放置一個空格，函數調用除外 
```R=+
#Good
if (debug)

#Bad
if(debug)
```
- 不要在括號或方括號中的代碼周圍放置空格
```R=+
# Good
if (debug)
x[1, ]

# Bad
if ( debug )  #變數周圍沒有空格
x[1,]  # 逗號後需要一個空格
```
- 大括號不應該出現在單獨一行
```R=+
# Good
if (is.null(ylim)) {
  ylim <- c(0, 0.06)
}

# Bad
if (is.null(ylim)) {ylim <- c(0, 0.06)}

```

賦值
---
> 使用 ==<-== 而不是 **=**

```R=
# Good
x <- 5

# Bad
x = 5
```

也可以使用`assign("變數名稱",運算式)` 建立變數並將運算式值存入變數中

例如下面範例中可以讀取多個csv檔案位置並讀取資料儲存至變數中
```R=
dffiles <- choose.files()
for(i in 1:length(dffiles)){
  assign(dffiles[i],read.csv(dffiles[i],header = T,sep = "\t"))
}

```

備註
---
> 整個註釋行應以#和一個空格開頭 
```R=
# Create histogram of frequency of campaigns by pct budget spent.
hist(df$pctSpent,
     breaks = "scott",  # method for choosing number of buckets
     main   = "Histogram: fraction budget spent by campaignid",
     xlab   = "Fraction of budget spent",
     ylab   = "Frequency (count of campaignids)")
```

有些文件比較正式，會有所謂的文件註釋

就是要在文件代碼開始前需要聲明的一些內容

比如說什麼環境下運行本代碼

使用哪個版本的RStudio，約定編碼方式（通常是utf-8）

以及這個代碼文件主要是用來幹嘛的

通常文件註釋可以使用兩個＃

```R=+
## Version 1.1.463
## coding: utf-8 
## exploratory data analysis 
```

自訂函數
---
> 函數定義應首先列出沒有默認值的參數，然後列出具有默認值的參數。
```R=
# Good

PredictCTR <- function(query, property, num.days,
                       show.plot = TRUE)
# Bad
PredictCTR <- function(query, property, num.days, show.plot =
                       TRUE)
```


後記
===

部分指南我並未說明
原因是我並不了解原文意義
或未使用過相關功能
若是有興趣的也可以參考別人完整翻譯的版本
[来自Google的R语言编码风格指南](https://nanx.me/rstyle/)

fromatR_自動格式化R代碼
---

若是你已經寫了大量的代碼沒有多餘心力更改的話

好在謝益輝大神寫了fromatR幫助我們調整編碼風格[自動格式化R代碼](https://yihui.name/formatr/)

**轉換前**
```R=
# comments are retained;
# a comment block will be reflowed if it contains long comments;
#' roxygen comments will not be wrapped in any case
1+1

if(TRUE){
x=1  # inline comments
}else{
x=2;print('Oh no... ask the right bracket to go away!')}
1*3 # one space before this comment will become two!
2+2+2    # only 'single quotes' are allowed in comments

lm(y~x1+x2, data=data.frame(y=rnorm(100),x1=rnorm(100),x2=rnorm(100)))  ### a linear model
1+1+1+1+1+1+1+1+1+1+1+1+1+1+1+1+1+1+1+1+1+1+1  # comment after a long line
## here is a long long long long long long long long long long long long long comment which will be wrapped
```

**轉換效果如下**
```R=+
## comments are retained; a comment block will be
## reflowed if it contains long comments;
#' roxygen comments will not be wrapped in any case
1 + 1

if (TRUE) {
    x = 1  # inline comments
} else {
    x = 2
    print("Oh no... ask the right bracket to go away!")
}
1 * 3  # one space before this comment will become two!
2 + 2 + 2  # only 'single quotes' are allowed in comments

lm(y ~ x1 + x2, data = data.frame(y = rnorm(100), x1 = rnorm(100), 
    x2 = rnorm(100)))  ### a linear model
1 + 1 + 1 + 1 + 1 + 1 + 1 + 1 + 1 + 1 + 1 + 1 + 1 + 
    1 + 1 + 1 + 1 + 1 + 1 + 1 + 1 + 1 + 1  # comment after a long line
## here is a long long long long long long long long
## long long long long long comment which will be
## wrapped
```