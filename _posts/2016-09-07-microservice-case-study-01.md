---
layout: post
title: "微服務架構 #1, WHY and HOW?"
categories:
published: false
comments: true
logo: 
---

八月底，接受了 Microsoft 的邀請，參加了 Community Open Camp 研討會，講了這場 "微服務架構 導入經驗分享 - .NET + Windows Container"。
其實這個主題涵蓋範圍還蠻大的，不過我一直認為，container 技術單獨介紹的話，那他就是個技術而已，若從他存在的目的來介紹，那他就是能解決
問題的好東西。因此我特地訂了這個主題: container + microservices.

<!--more-->

一切要從 SOA (service oriented architecture) 開始說起。2000+ internet 紅了之後，很多大廠就開始主推 SOA 架構。不過在當時是叫好不叫座，
主要是因為開發的技術跟佈署的技術還跟不上，導致這種架構是只有大型團隊才玩得起。現在的 container, 大幅改善了佈署的門檻，也大幅降低了小規模
佈署的成本 (以前最小單位是 server, 後來進步到 vm, 現在則是縮小到 container), container + microservices 突然之間就變成顯學。

佈署方便且快速的特性，也影響到開發團隊的流程，逐漸的促成了 devops .. 這些都是 container 會快速普及的原因。其實我很想整個交代一遍，不過
這次 session 時間只有 40 分鐘，相當有限... 最後當然是略過了很多細節，我決定把重點擺在 microservices, 而實際的案例則用 container 來展示。
我用的是 asp.net (mvc) project, 因此也特地照顧了較少接觸 open source 資源的 windows developer 族群，demo 採用 windows container。

我準被的內容，包含 demo, 若是要講個 4hr 也是夠講的... XD，裡面其實太多細節跟經驗值得分享的。既然當天的場子講不完，我就整理一下用文字的
方式在這邊分享吧。我會分成這幾個章節，預計會分成三篇文章來說明:

1. 微服務架構
2. 容器化的佈署方式 (demo: windows container)
3. 實際案例說明 (大型人資系統, ASP.NET WebForm application)

前言結束，本文正式開始!


------
# 微服務(microservices) vs 單體式架構(monolithic)

軟體就是不斷的 reuse code, 不斷的堆疊, 累積成能解決問題的 application. 累積這些 code 的方法很多，過去的軟體開發方式，最常用的就是
library / component 的方式，把 code / binary 組裝成一個大型的 application。先來看張圖:

![單體式架構(monolithic) vs 微服務(microservices)](/wp-content/uploads/2016/09/microservice-slides-06.PNG)


## 單體式架構

這兩種架構，應該從巨觀來看。不論你的軟體是用什麼方式開發的，最終你的 application 是 "單一" 一個 application, 而他累積功能的作法
是透過在 application 內不斷 reuse library or component 的方式，那就稱做單體式架構 (monolithic)。這種思維下，通常用的是同一種
平台，框架，或是語言所打造的大型應用程式。執行或佈署時，對於作業系統而言，就是個 process，只是他的規模大了一點。

作業系統對應用程式的管理，都是以 process 為單位的，也就是說一旦這個 application 出了問題，就是整個 process restart。若是效能不足
需要擴充，就是整個 prcess scale out。規模過大的 application, 往往難以微調，充分運用整台 server 的硬體資源。而 OS 的保護機制也是
以 process 為最小單位，若你的 application 某部分出了問題，也無法進行隔離，可能會導致整個 process fail, 需要 restart.






這張圖應該這樣看，上面的 application 代表的就是你開發的應用程式。裡面的小方塊，我們可以把他看成你的系統內幾個主要的模組。
單體式架構的 application 主要都是用 "元件" (component) 或是 "函示庫" (library) 的方式組合起來的。這種方式，最終會產出
一個規模龐大的 application, 就是這張圖左方的 Monolithic application approach 的例子。

軟體靠重複使用現有模組 (reuse) 來建構大系統，單體式架構的思維下，reuse 的手法主要就是在 code base 的前提下來組合重複使用
既有的模組。舉例來說，.NET 有很多 Microsoft 提供的 BCL (Base Class library), 我們也很容易從 Nuget 取得很多別人開發好的
Package (DLL, assemblies). 這些套件最終被組合到一個大型的 application 內。我們 reuse 的是他的 code, 在我們自己的 OS 內
運行。對於 OS 來說，我們的 application 就是個很龐大的 process 而已, 除了大小之外，跟別的 process 並無不同。OS 都是以 process
為單位，做 "啟動"、"執行"、"終止" 等等操作。

Code base 過大，在開發及測試的過程中也會帶來很多困難。系統的複雜度，是隨著你的功能的增加，而呈指數遞增的。有十倍功能的系統，
操作跟測試的複雜度可能會暴增為 100 ~ 1000 倍，規模越大會越難維護，我想開發過系統的人都有這種體認。然而規模變大，帶來的問題
不只這個，系統執行的維運管理也會是個大問題。舉例來說，當一個模組沒有做好記憶體管理，導致 Out Of Memory Exception, 毀掉的
是整個 process, 而不是單一這個模組。當一個模組 fail, 導致整個 process 掛掉的時候，對，就是整個應用程式掛掉了。

單一一個大型 application, 在 OS 的管理角度來看是無法再分割的，執行時無法再切割為更小的佈署單位。效能不足時，只能對整個
application 進行 scale out，無論你產生瓶頸的是哪一個模組... OS 無法只針對特定模組做 scale out. 這也代表你無法針對特定模組
的資源使用最佳化...














# 前言

其實一開始本來有埋了一些梗，後來都拿掉了.. 很多新技術的導入 (尤其是企業內有時程壓力的狀況下)，往往碰到最大的問題都是人，而不是技術本身。
我常跟別人聊到，最頭痛的狀況就是有些專家，他拿他的專業能力來潑你冷水，而不是拿來解決問題。怎麼說? 因為這些專家夠專業，所以能夠馬上舉出
十個新技術的缺點，來說明導入會有什麼困難。就因為這些人是專家，有時你還真沒辦法反駁他。如果這些專家，在意的不適專案成不成功，而是在意
自己的工作會不會加重，那大概都不會有好下場了。但是很多事情不下定決心改善，是不會成功的。這時如何說服這些專家，就是推廣新技術能否成功的主要因素了。

隨便舉個例子，下列的對話是真正發生過的 case:
![被刪掉的投影片 - 常會不自覺潑冷水的專家](/wp-content/uploads/2016/09/microservice-slides-03.PNG)

當然，最後我們並沒有被這些冷水澆熄改變架構的念頭 (不然就沒這個分享的 session 了)，第一階段的調整也順利上線，接下來更大規模的佈署則要
看看 windows server 2016 上市之後再來評估... 不過對話內點到兩個問題，就是我們改用微服務架構碰到的困難:
1. 我們是用 .NET solution, 但是理想架構下，同時會需要 .NET + Open Source solutions
2. Open Source solutions 大都是在 Linux 上面的表現最好，大規模佈署的話會有混搭 (windows + linux) 的需要
3. 微服務架構若沒有搭配 container, 佈署過程會變的很辛苦, 可是當紅的 Docker 是依賴 Linux Kernel 的 container engine ...

這些問題，在後面的介紹都會提到如何因應的，因此這邊我就不多說。先從基礎來說明，為何我們需要用微服務架構。

正式開始這主題之前，先給大家參考一下當天的投影片..
<iframe src="http://www.slideshare.net/chickenwu/slideshelf" width="760px" height="570px" frameborder="0" marginwidth="0" marginheight="0" scrolling="no" style="border:none;" allowfullscreen webkitallowfullscreen mozallowfullscreen></iframe>

# 微服務(microservices) vs 單體式架構(monolithic)



## 單體式架構

這張圖應該這樣看，上面的 application 代表的就是你開發的應用程式。裡面的小方塊，我們可以把他看成你的系統內幾個主要的模組。
單體式架構的 application 主要都是用 "元件" (component) 或是 "函示庫" (library) 的方式組合起來的。這種方式，最終會產出
一個規模龐大的 application, 就是這張圖左方的 Monolithic application approach 的例子。

軟體靠重複使用現有模組 (reuse) 來建構大系統，單體式架構的思維下，reuse 的手法主要就是在 code base 的前提下來組合重複使用
既有的模組。舉例來說，.NET 有很多 Microsoft 提供的 BCL (Base Class library), 我們也很容易從 Nuget 取得很多別人開發好的
Package (DLL, assemblies). 這些套件最終被組合到一個大型的 application 內。我們 reuse 的是他的 code, 在我們自己的 OS 內
運行。對於 OS 來說，我們的 application 就是個很龐大的 process 而已, 除了大小之外，跟別的 process 並無不同。OS 都是以 process
為單位，做 "啟動"、"執行"、"終止" 等等操作。

Code base 過大，在開發及測試的過程中也會帶來很多困難。系統的複雜度，是隨著你的功能的增加，而呈指數遞增的。有十倍功能的系統，
操作跟測試的複雜度可能會暴增為 100 ~ 1000 倍，規模越大會越難維護，我想開發過系統的人都有這種體認。然而規模變大，帶來的問題
不只這個，系統執行的維運管理也會是個大問題。舉例來說，當一個模組沒有做好記憶體管理，導致 Out Of Memory Exception, 毀掉的
是整個 process, 而不是單一這個模組。當一個模組 fail, 導致整個 process 掛掉的時候，對，就是整個應用程式掛掉了。

單一一個大型 application, 在 OS 的管理角度來看是無法再分割的，執行時無法再切割為更小的佈署單位。效能不足時，只能對整個
application 進行 scale out，無論你產生瓶頸的是哪一個模組... OS 無法只針對特定模組做 scale out. 這也代表你無法針對特定模組
的資源使用最佳化...



## 微服務架構

![為何要採用 "微服務架構" ?](/wp-content/uploads/2016/09/microservice-slides-09.png)

微服務的架構，採取另一個不同的思考方式來解決單體式架構的問題。不知各位是否還記得幾年前很流行的 "服務導向架構" (SOA, Service Oriented Architecture) ?
其實背後的觀念有些是雷同的。就是把單體式的 application, **拆解**為幾個獨立的 application. 只要你拆解的適當，拆解的夠徹底，每個服務
的大小及規模都會受到良好的控制。你會更容易維護，佈署及管理。當單一 application 的規模受到控制之後，單體式架構碰到的問題就再也不存在。

我簡單的用算式做個比喻。假設有 10000 行 code 的系統，維護的困難度是 100。那麼當 code 變成 10 倍，維護的困難度可能會是 100^2 = 10000.
但是若將他拆成 10 套獨立的系統，那麼維護的困難度就是 100 x 10 = 1000, 這個幅度隨著系統的規模越大會越明顯。簡單化張圖表示這個概念:

![複雜度 v.s. 規模大小](/wp-content/uploads/2016/09/microservice-slides-09-0.png)

除了開發的複雜度之外，微服務架構也能在其他領域展現優勢，例如: 錯誤隔離的能力也比較好。當你的系統切割為數個獨立的服務，單一服務掛掉時，
並不會直接影響到其他服務，其他服務仍然能正常運作，影響到的只是會呼叫到這個服務的相關功能。因此微服務的架構下，能有較佳的失敗隔離表現。
另外服務的功能較單一，使用的資源特性也較明確。例如某些服務是 CPU bound, 某些則是 IO bound, 做適當的安排，就能充分運用硬體的每一分資源，
提高資源使用率，節省營運成本。微服務架構也因此能有較佳的延展性，從很小的規模 (1 ~ 3 台 server)，到很龐大的規模 (1000 台 server) 都能
應付。

從這角度來看，採用微服務的優點再明顯不過了。用多個小服務代替單一大服務，避開單一大型服務開發及維護的困難。
看起來很棒，只是... 要做到微服務架構，也是有他的困難要克服的... 

第一個碰到的，就是開發的挑戰了。採用元件或是涵式庫，使用上是相當容易的，通常只要加入參考就
可以用。因為這些規格通常是同一種語言，編譯器，平台或是框架下的產物，使用上完全沒有阻礙。但是
服務跟服務之間的挑戰就困難的多，服務之間要互相溝通，一定要透過事先定義好的介面來進行，這通常
會是跟平台無關的API (例如 Http API, or RESTful API)。API 的開發並不難，難在你定義
的 AP**I** 是否夠收斂札實? 是否在能達到同樣目的下有最小的介面? 另一個挑戰是 API 必須有
很好的向前相容，因為將來每個服務可能都會個別升級，你會面臨同時有新舊 client 都在使用你的服務，
相容性沒做好，服務組成的 application 就會垮台。服務之間的介面設計，是架構師面臨的**第一個挑戰**。

再往上一層，同樣是開發層面的挑戰是: "到底要將整個系統切割成哪些服務???" 這是微服務架構的設計
核心。每個服務內的 code, 都是高內聚力的的程式碼，每一段都息息相關，因此適合封裝成一個獨立的服務。
而服務與服務之間，則是低偶合的，適合切割出來，只透過預先定義好的 API 或是 protocol 來溝通及傳遞
訊息。如何分析與找出整套系統內的邊界，找到適合分割的地方切下去，這是架構師面臨的**第二個挑戰**。

在單體式架構下，佈署系統時，原本只要維護一套服務就可以了，但是微服務化之後，就變成要維護很多服務了。
管理及維護的複雜度就出現了。你可以想像安裝好一套系統，得要裝 10 個database, 30 個 web site, 防火牆
及安全性分別顧好這些 application 之間的存取原則做設定就要有 100 條以上的 rules ... 將來還要做好
這些服務的備份及還原... 更別提這些 application 每個都可以個別 scale out... 佈署問題是微服務架構
面臨的**第三個挑戰**。


# 如何將系統改為微服務架構?

