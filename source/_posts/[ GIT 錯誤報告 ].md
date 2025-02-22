---
title:       "[ GIT 錯誤報告 ]"
subtitle:    ""
excerpt: "主要紀錄使用 git 時的相關錯誤，此外 git 相關的產品使用問題也會一併紀錄於此，不定時更新"
date:        2022-09-28
tags:        ["git"]
categories:  ["DeveOps", "Version Control"]
cover: /gallery/covers/git.webp
toc: true
---
## 錯誤紀錄

### 遠端儲存庫沒有對應的分支

```bash
error: src refspec main does not match any 
error: failed to push some refs to [repo_url]
```

**解決方法**  
將本地儲存庫待上傳的分支更名

```bash
git branch -m [old branch] [new branch] 
```

---

### 部分內容未被合併，該版本不是遠端儲存庫最新版本

```bash
! [rejected]        main -> main (non-fast-forward)  
error: failed to push some refs to [repo_url]
```

**解決方法**    
先拉取遠端儲存庫分支版本，並將其允許無相關的歷史紀錄與現在分支合併接著重新提交

```bash
git fetch origin main
git merge --allow-unrelated-histories origin/main  
git add .  
git commit
```

---

### .gitignore 沒有效果

原因：因為 `git` 通常會去讀取緩存的那份 `.gitignore`，而這份通常指的是該項目在遠端儲存庫當前的版本，所以才會無論怎麼對 `.gitignore` 操作都無法改變。  

**解決方法**  
先清除掉 local 端的 `git` 緩存，重新創建一份 `.gitignore` 後，再將此版本提交上主線程

```bash
git rm -r --cached .
git add .
git commit -m 'update .gitignore'
# 記得不要使用 git commit -a -m 或是 git commit . -m 的指令樣式，不然會將之前的內容也新增回來
```

這樣處理完後，如果有其他人 clone 此專案，只需執行前兩步即可

### 個人建立的私有資料庫無法上傳問題

原因：私有資料庫只允許擁有者和被邀請加入的開發者才能上傳、下拉使用，所以需要表示上傳者的身分才能成功上傳

**解決方法**  
總共有兩種方式解決，第一種較適合還沒 clone repo，第二種則適合已經 clone 但不想重新 clone 的開發者

1. 將 repo 以 ssh 方式 clone
2. 創建 personal access token 並以這樣的形式`（https://[your-username]:[your-private-access-token-here]@github.com/[your-repo]）`加入 remote url

### git clone Error："RPC failed"、"POST of 'XXX' failed"
使用 git clone 出現以下問題
```bash=
error: RPC failed; curl 92 HTTP/2 stream 5 was not closed cleanly: CANCEL (err 8)
error: 5492 bytes of body are still expected
fetch-pack: unexpected disconnect while reading sideband packet
fatal: early EOF
fatal: fetch-pack: invalid index-pack output
```
這是因為要 clone 的 repo 太大，導致無罰成功拉取

**解決方法**

第一種方法是減少拉取提交的次數，例如使用 `depth` 參數指定只拉取近幾次的提交。

```bash
git clone https://github.com/XXX/XXX.git  --depth 1
```
題外話，如果要拉取所有的歷史版本，可以用 `unshallow` 參數
```bash
git fetch --unshallow
```

第二種方法就是變更設定的緩存空間
```bash
git config --global http.postBuffer <大小>
# 如果要更改為 5 MB，就是設成 5M
```
如上面的問題顯示，http.postBuffer 的預設值是 1MB，因此 5492 bytes 無法拉取，所以我們要把 buffer 改超過 5MB 才可以順利拉取這份資料。

此外這個參數設定只影響透過 http 協議拉取 repo，如果用 SSH、或其他協議則不受此影響。

## 參考資料

1. [Fatal: refusing to merge unrelated histories](https://developer.aliyun.com/article/614459)  
2. [Rejected problem](https://blog.csdn.net/qq_27249535/article/details/121906285)
3. [git-repository-not-found-error-for-private-repository-on-github](https://stackoverflow.com/questions/56269686/git-repository-not-found-error-for-private-repository-on-github)
4. [git-clone-error](https://www.cnblogs.com/quenwaz/p/18115058)