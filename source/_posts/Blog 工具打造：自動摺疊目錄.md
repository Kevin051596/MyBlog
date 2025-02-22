---
title:       "Blog 工具打造：自動摺疊目錄"
subtitle:    "快速連結的動畫效果編程紀錄"
excerpt: "覺得快速連結的清單太冗長嗎？那你可能跟我一樣需要一個能動態顯示的快速連結清單，能隨著我閱讀的進度自動調整清單內容，才不會顯得到處都充滿文字，這篇是關於我的動態清單編程紀錄，我覺得整體程式是還有很多點可以優化啦，但至少效果還不錯"
date:        2023-08-27
tags:        ["front-end", "javascript", "hugo"]
categories:  ["Develop", "Web"]
cover:       /gallery/covers/default.jpg
toc: true
---

前言：這篇文章是我記錄改編 Blog 的過程。我以 Hugo 上 [Clean White Blog](https://themes.gohugo.io/themes/hugo-theme-cleanwhite/) 的風格作為樣板， 在此之上增加更多有趣的互動功能，同時增進自己在前端開發的能力。

### 生成快速連結
1. 首先需要定位我們要尋找的範圍
    ```javascript
    var P = $(文本範圍的 selector)
    ```
2. 接著在這個範圍找出我們要的 `tag` ，也就是 `h1~h6`
    ```javascript
    a = P.find('h1,h2,h3,h4,h5,h6');
    ```
3. 將這些 `tag` 元素的資訊重組成快速連結並加入清單中
    ```javascript
    a.each(function () {
        n = $(this).prop('tagName').toLowerCase();
        i = "#" + $(this).prop('id');
        t = $(this).text();
        c = $(`<a aria-label="${t}" href="${i}" rel="nofollow">${t}</a>`);
        l = $(`<li class="${n}_nav"></li>`).append(c);
        $(selector).append(l);
    })
    ```
完整的生成函式
```javascript
function generateCatalog(selector) {

    _containerSelector = 'div.post-container' // 文本範圍

    // init
    var P = $(_containerSelector), a, n, t, l, i, c;
    a = P.find('h1,h2,h3,h4,h5,h6');

    // clean
    $(selector).html('') // 預防裡面有其他元素

    // appending
    a.each(function () {
        n = $(this).prop('tagName').toLowerCase();
        i = "#" + $(this).prop('id');
        t = $(this).text();
        c = $(`<a aria-label="${t}" href="${i}" rel="nofollow">${t}</a>`);
        l = $(`<li class="${n}_nav"></li>`).append(c);
        $(selector).append(l);
    })
    return a;
}
```

### 整理所有連結的資訊

#### 要整理的資訊
我們總共需要 **六** 種資訊：
- 快速連結中的標題元素
- 標題層級下的所有子標題元素
- 第一個子標題元素
- 子標題數量
- 文章中標題位置
- 文章中的 `tag` 樣式

在這邊我是將上面的 a 抓來當作 sideElements，因為 a 裡面每個元素的 `id` 可以用來定位文本範圍內所要的標題元素，而 `innerText` 可以用來定位快速連結上的元素。  
也就是說，我用 `id` 來定位文本內的標題位置，用 `innerText` 來定位快速連結的位置。  

在這邊子標題數量先預設為 0 ，等等會提到如何計算該層級下的子標題數量。  
還有對於「標題層級下的所有子標題元素」的作法我是用範圍搜索，也就是查詢直到跟我同層級的標題出現，但這樣會有一個問題，就是差很多層級的子標題也會納入，而這個問題等一下也會說明

```javascript
sideElements.each((index, element) => {
    const name = element.id
    const innerTagsCount = 0 // 子標題數量預設為 0
    const sideElement = $(`[aria-label="${element.innerText}"]`) // 快速連結中的標題元素
    const top =  $(`[id="${name}"]`).offset().top // 文章中標題位置
    const htag = sideElement.parent()
    const innerTags = htag.nextUntil(`[class="${htag[0].className}"]`) // 標題層級下的所有子標題元素

    //第一個子標題元素
    let innerFirstTag = null, i = 0 
    while(!innerFirstTag && i < innerTags.length){
        if(innerTags[i].className > htag[0].className) innerFirstTag = innerTags[i].className
        i++
    }

    const tagName = $(`[id="${name}"]`).prop("tagName") // 文章中的 `tag` 樣式

    all_htags.push({ sideElement, innerTags, innerFirstTag, innerTagsCount, top, tagName })
})
```

#### 隱藏所有子標題
我先宣告 `firstTag` 記錄整篇文章第一個吃到的標題當作最上層級，而其他標題的層級如果與最上層級不同就隱藏。但這麼做的前提是最主要的標題前面沒有任何子標題，不然子標題會變成你的最上層級而導致結構矛盾(我是覺得這樣架構長的蠻奇怪的啦~
```javascript
if(!firstTag) firstTag = tagName
if(tagName !== firstTag) sideElement.hide()
```

### 紀錄每個層級的數量

接著宣告一個陣列用來儲存 `h1~h6` 的數量，但是這個紀錄實際上是記錄標題本身加上其子標題數量，而非真的每個層級的個別數量。  
在這裡我打算從後面算回來，也比較好去計算子標題的數量

```javascript
let count = [0, 0, 0, 0, 0]
for(let i = all_htags.length - 1; i >= 0; i--){
    const index = parseInt(all_htags[i].tagName.slice(-1))
    all_htags[i].innerTagsCount = 1 // 先加上自己
    count[index - 1] += 1 // 將對應的層級加入
    for(let j = index; j < 5; j++){ // 將其所有子層級總和
        all_htags[i].innerTagsCount += count[j]
        count[index - 1] += count[j]
        count[j] = 0
    }
}
```

以下面例子為例，右邊的數字為其所得出的子標題數量
```
h1 6
- h2 4
-- h3 1
-- h3 1
-- h3 1
- h2 1
- h2 1
```

### 監聽滾動觸發效果
我們總共需要兩種效果
- 換色：滾動到哪，哪個快速連結就顯示青色，反之就是灰色
- 摺疊：滾動到哪，就自動打開那層級下的子標題，反之，將其闔上

#### 換色

`sideElement` 代表的是快速連結的元素

```javascript
sideElement.attr("style", "color: #337ab7") // 顯示青色
sideElement.removeAttr("style", "color") // 移除顏色
```
#### 摺疊

`innerTags` 代表的是子標題群組，但由於我們是透過範圍抓取的，所以會將更為次等的子標題也抓進來，所以我們要篩選出差一個層級的子標題，而不是差兩個層級的子標題或差更多層級的子標題

```javascript
innerTags.filter((index, tag) => tag.className === innerFirstTag).slideDown() // 向下展開
innerTags.filter((index, tag) => tag.className === innerFirstTag).slideDown() // 向上摺疊
```

#### 掛上滾動監聽
我們還需要當前滾動的位置與是不是已經滾到底部，不然如果已經滾到底部了，但快速連結的狀態卻不是最後一個就會很奇怪，通常這會出現在最後內容太短的情況

```javascript
let curPos = window.pageYOffset // 當前位置
let isBottom = parseInt(curPos + document.documentElement.clientHeight) === document.documentElement.scrollHeight
// 是否到達底部
```

最後條件判斷一下，是不是有子標題，有就加上摺疊效果和著色效果，沒有就只上著色效果。並且特判最後一個標題，避免內容太短的狀況
滾動監聽完整程式碼
```javascript
$(window).scroll(() => {
    let curPos = window.pageYOffset
    let isBottom = parseInt(curPos + document.documentElement.clientHeight) === document.documentElement.scrollHeight
    all_htags.forEach((item, index) => {
        const {sideElement, innerTags, innerFirstTag, innerTagsCount, top} = item
        if(index < all_htags.length - 1){ // 是否為最後一個標題
            if(innerTagsCount > 1){ // 是否有子標題
                if(curPos >= top && curPos < all_htags[index + innerTagsCount].top && !isBottom){
                    sideElement.attr("style", "color: #337ab7") // 上色
                    innerTags.filter((index, tag) => tag.className === innerFirstTag).slideDown() // 展開
                }
                else{
                    sideElement.removeAttr("style", "color") // 除色
                    innerTags.filter((index, tag) => tag.className === innerFirstTag).slideUp() // 摺疊
                }
            }else{
                if(curPos >= top && curPos < all_htags[index + innerTagsCount].top && !isBottom) sideElement.attr("style", "color: #337ab7") // 上色
                else sideElement.removeAttr("style", "color") // 除色
            }
        }else{ // 最後一個標題
            if(curPos >= top || isBottom) sideElement.attr("style", "color: #337ab7") // 上色
            else sideElement.removeAttr("style", "color") // 除色
        }
    })
})
```


### 完整程式碼

```javascript
function generateCatalog(selector) {

    _containerSelector = 'div.post-container'

    // init
    var P = $(_containerSelector), a, n, t, l, i, c;
    a = P.find('h1,h2,h3,h4,h5,h6');

    // clean
    $(selector).html('')

    // appending
    a.each(function () {
        n = $(this).prop('tagName').toLowerCase();
        i = "#" + $(this).prop('id');
        t = $(this).text();
        c = $(`<a aria-label="${t}" href="${i}" rel="nofollow">${t}</a>`);
        l = $(`<li class="${n}_nav"></li>`).append(c);
        $(selector).append(l);
    })
    return a;
}

const sideElements = generateCatalog(".catalog-body"), all_htags = []
let firstTag = null
sideElements.each((index, element) => {
    const name = element.id
    const innerTagsCount = 0
    const sideElement = $(`[aria-label="${element.innerText}"]`)
    const top =  $(`[id="${name}"]`).offset().top
    const htag = sideElement.parent()
    const innerTags = htag.nextUntil(`[class="${htag[0].className}"]`)

    let innerFirstTag = null, i = 0
    while(!innerFirstTag && i < innerTags.length){
        if(innerTags[i].className > htag[0].className) innerFirstTag = innerTags[i].className
        i++
    }

    const tagName = $(`[id="${name}"]`).prop("tagName")
    if(!firstTag) firstTag = tagName
    if(tagName !== firstTag) sideElement.hide()

    all_htags.push({ sideElement, innerTags, innerFirstTag, innerTagsCount, top, tagName })
})

let count = [0, 0, 0, 0, 0]
for(let i = all_htags.length - 1; i >= 0; i--){
    const index = parseInt(all_htags[i].tagName.slice(-1))
    all_htags[i].innerTagsCount = 1
    count[index - 1] += 1
    if(i !== all_htags.length - 1 && index !== 5){
        for(let j = index; j < 5; j++){
            all_htags[i].innerTagsCount += count[j]
            count[index - 1] += count[j]
            count[j] = 0
        }
    }
}

$(window).scroll(() => {
    let curPos = window.pageYOffset
    let isBottom = parseInt(curPos + document.documentElement.clientHeight) === document.documentElement.scrollHeight
    all_htags.forEach((item, index) => {
        const {sideElement, innerTags, innerFirstTag, innerTagsCount, top} = item
        if(index < all_htags.length - 1){
            if(innerTagsCount > 1){
                if(curPos >= top && curPos < all_htags[index + innerTagsCount].top && !isBottom){
                    sideElement.attr("style", "color: #337ab7")
                    innerTags.filter((index, tag) => tag.className === innerFirstTag).slideDown()
                }
                else{
                    sideElement.removeAttr("style", "color")
                    innerTags.filter((index, tag) => tag.className === innerFirstTag).slideUp()
                }
            }else{
                if(curPos >= top && curPos < all_htags[index + innerTagsCount].top && !isBottom) sideElement.attr("style", "color: #337ab7")
                else sideElement.removeAttr("style", "color")
            }
        }else{
            if(curPos >= top || isBottom) sideElement.attr("style", "color: #337ab7")
            else sideElement.removeAttr("style", "color")
        }
    })
})
```