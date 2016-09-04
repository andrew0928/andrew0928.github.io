---
layout: post
title: "我常被問的 Windows Container 問題"
categories:
- "Docker"
- "Windows Container"
- "作業系統"
- "技術隨筆"
published: true
comments: true
---

原本，Windows Container 單純就我自己研究而已，因為工作上用到的都是 ASP.NET Web Form 居多 (有部分早已轉移
到 MVC，但是主要還是 WebForm)，加上商用軟體，用到一堆外面的第三方套件，想要藉由升級到 .NET Core, 然後直接
享用 Docker 帶來的一堆好處，那真的是想都不用想了...

不過，有幸參加了 8/27 Community Open Camp 活動，擔任一場 session speaker 後發現，其實還不少人對 Windows Containeru
有興趣的，但是因為用的人還不多，而且到現在也還只有 Tech Preview 5 .. 使用起來還不少問題，現在切入是得花點
時間克服障礙的

既然我都花時間搞懂了，我就把官方 FAQ 沒告訴你的 FAQ 紀錄一下吧~

<!--more-->

## 參考資料
首先，幾個適合已經懂 Docker on Linux, 卻還不熟 Windows Container 的人可以看的參考資料:

* 關於 Windows 容器 - [常見問題集](https://msdn.microsoft.com/zh-tw/virtualization/windowscontainers/about/faq)
* 容器主機部署 - [Windows Server](https://msdn.microsoft.com/zh-tw/virtualization/windowscontainers/deployment/deployment)

另外，有些有趣且深入的文章，可以先找看看有沒有你要的資訊:
* [细说Windows与Docker之间的趣事](http://www.infoq.com/cn/articles/windows-and-docker)

## Q1. Windows Container 跟 Docker for Windows 是一樣的東西嗎?
> **不一樣**。Windows Container 是 Microsoft 初次在 Windows Server 2016 提供的功能 (目前還尚未 release)。
> Microsoft 比照 Docker 的架構，開發了 Windows 版的 container engine. container 共用的是 windows kernel,
> 不是 linux kernel. **你在 Windows Container 內能執行的，是貨真價實的 Windows Application, Not LINUX Application**.
>
> Docker 官方提供的 Docker for Windows, 只是 Windows 版的 (Linux) Docker 執行套件。他是在 Windows
> 背後用 Hyper-V 跑一個 AplineLinux VM, 但是幫你把管理工具在前端 Windows 都設定好，方便你用 Windows 使用
> Linux VM 裡面的 Docker Engine. **你在 Docker for Windows 內能執行的，是 Linux Application, Not Windows Application**.
>
> 基本上，你可以把 Windows Container 想像成，Microsoft 推出的一套 Container Engine, 除了 Kernel 改用 Windows Kernel
> 之外，其他一切都沿用 Docker Eco system 的產品。包含 Docker API, Docker Client, Docker Management Protocol,
> Image format, Registry, Cluster ... etc 等等通通都共用的產品。

## Q2. 怎麼樣才能使用 Windows Container ?
> 支援的列表，可參考官網的 [說明](https://msdn.microsoft.com/zh-tw/virtualization/windowscontainers/deployment/system_requirements)
>
> 目前唯一支援的，只有 Windows Server 2016. 最新公開的版本，是 2016/04 釋出的 TP5 (tech preview 5), 按照 Microsoft
> 官方說法，2016/10 會 release. 啟用 Windows Container 有些步驟要執行，請參考:
>
> Windows Container 支援兩種 isolation type, 一種就是標準的 process / namespace 隔離，Windows Container 額外
> 支援了 Hyper-V 的隔離層級。Windows 10 Pro / Ent 版本在 2016/08 RS 更新後，就能夠使用 Hyper-V 層級的 Windows Container.
> 安裝步驟可以參考:

## Q3. Windows Container 能執行現有的 Docker Container Image 嗎?
> 不行。Windows Container 只能執行 Windows Application, 你必須準備 for windows 的 container image.
> container image 取得的來源跟 docker 一樣，可以自己從 dockerfile build, 或是去 hub.docker.com or 其他 registry
> 拉回來 (pull) 使用

## Q4. 我能用 Docker Client 管理 Windows Container 嗎?
> 可以。Docker Client 是可以共用的，你甚至可以用 Linux 版的 Docker Client 來管理 Windows Server 上的 Container Engine

## Q5. Windows Container 的 Image 可以從哪裡取得?
> 完全跟 Docker Registry 一樣。[hub.docker.com](http://hub.docker.com) 上面也可以找到 for windows 的 container image. 建議可以從 microsoft
> 開始找，例如 microsoft/windowsservercore, microsoft/nanoserver, 或是常用的 microsoft/iis 等等都是不錯的起點。

## Q6. 目前 Windows Server 2016 Tech Preview 5 有那些問題?
> 除了有些狀況下，會跑出來的零星 Error 之外，其實大部分狀況都能運作良好。如果你要評估可行性，其實現在就可以開始用了。
> 不過，windows container engine 比較頭痛的是，部分網路相關功能還未包含在 TP5 之內。(像我)想部屬微服務架構時，會被這
> 限制搞得很煩..
>
> 為了協助 developer 能更容易的 deploy microservice architecture applicaion, docker 其實在這方面下了很大的功夫，包含
> container link (在兩個 container 之間建立專屬連線, 你就不用開 port 限制 ip 還搞一堆授權的問題)，還有內建 dns resolation 等等
> 方便的功能。很不幸的是你想在 windows container 用這些功能，大概得等 release. 我轉貼[官網的說明](https://msdn.microsoft.com/en-us/virtualization/windowscontainers/management/container_networking):
>
>> **Unsupported features**
>> 
>> The following networking features are not supported today through Docker CLI
>> 
>> container linking (e.g. --link)
>> name/service-based IP resolution for containers. This will be supported soon with a servicing update
>> default overlay network driver
>> The following network options are not supported on Windows Docker at this time:
>> 
>> * --add-host
>> * --dns
>> * --dns-opt
>> * --dns-search
>> * -h, --hostname
>> * --net-alias
>> * --aux-address
>> * --internal
>> * --ip-range
>
