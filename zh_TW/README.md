# Docker Cheat Sheet

**想要一起來完善這份速查表嗎？請看[貢獻手冊](#貢獻手冊contributing)部分！**

## 目錄

* [為何使用 Docker](#為何使用-docker)
* [系統環境](#系統環境)
* [安裝](#安裝)
* [容器(Containers)](#容器container)
* [鏡像(Images)](#鏡像images)
* [網絡(Networks)](#網絡networks)
* [倉管中心和倉庫(Registry & Repository)](#倉管中心和倉庫registry--repository)
* [Dockerfile](#dockerfile)
* [層(Layers)](#層layers)
* [連結(Links)](#連結links)
* [卷標(Volumes)](#卷標volumes)
* [暴露連接埠(Exposing Ports)](#暴露連接埠exposing-ports)
* [最佳實踐](#最佳實踐)
* [安全](#安全security)
* [小貼士](#小貼士)
* [貢獻手冊(Contributing)](#貢獻手冊contributing)

## 為何使用 Docker

"通過 Docker, 開發者可以使用任何語言任何工具創建任何應用。“Dockerized” 的應用是完全可移植的，能在任何地方運行 - 不管是同事的 OS X 和 Windows 筆記本，或是在雲端運行的 Ubuntu QA 服務，還是在虛擬機運行的 Red Hat 產品數據中心。

 Docker Hub 上有 13,000+ 的應用，開發者可以從中選取一個進行快速擴展開發。Docker 跟蹤管理變更和依賴關係，讓系統管理員能更容易理解開發人員是如何讓應用運轉起來的。而開發者可以通過 Docker Hub 的共有/私有倉庫，構建他們的自動化編譯，與其他合作者共享成果。

Docker 幫助開發者更快地構建和發佈高質量的應用。" -- [什麼是 Docker](https://www.docker.com/what-docker/#copy1)

## 系統環境

我用的是 [Oh My Zsh](https://github.com/robbyrussell/oh-my-zsh) ，和 [Docker 插件](https://github.com/robbyrussell/oh-my-zsh/wiki/Plugins#docker) ，它可以自動補全 docker 的命令。YMMV。

### Linux

3.10.x 內核是能運行 Docker 的[最低要求](https://docs.docker.com/engine/installation/binaries/#check-kernel-dependencies)。

### MacOS

10.8 “Mountain Lion” 或者更新的版本。

## 安裝

### Linux

Docker 提供了快速安裝腳本：

```
curl -sSL https://get.docker.com/ | sh
```

如果你不想執行一個不明不白的 shell 腳本，那麼請看[安裝教程](https://docs.docker.com/engine/installation/linux/)，選擇你在用的發行版本。  

如果你是一個 Docker 超新手，那麼我建議你先去看看[系列教程](https://docs.docker.com/engine/getstarted/)。

### Mac OS X

下載和安裝 [Docker Toolbox](https://docs.docker.com/toolbox/overview/)。[Docker For Mac](https://docs.docker.com/docker-for-mac/) 很贊，但是它的安裝和 VirtualBox 不太一樣。詳情請查閲[比較](https://docs.docker.com/docker-for-mac/docker-toolbox/)。

> **注意** 如果你已經有安裝了 docker toolbox，那麼你可能會考慮通過 [Docker Machine](https://docs.docker.com/machine/install-machine/) 安裝包(不管是從 URL 或是 `docker-machine upgrade default`)升級，它確實會完成 docker-machine 的升級。但是它不會幫你升級 docker 版本 -- `docker-machine` 變成了 `1.10.3` 而 `docker` 還是原來的 `1.8.3` 或者你之前的什麼版本。
>
> 所以你最好是通過 Docker Toolbox DMG 檔案來升級，它會一次性的幫你處理好所有的升級。

安裝好 Docker Toolbox 之後，通過 VirtualBox provider 安裝帶 Docker Machine 的 VM:

```
docker-machine create --driver=virtualbox default
docker-machine ls
eval "$(docker-machine env default)"
```

然後啟動 container:

```
docker run hello-world
```

好了，你現在有了一個運行中的 Docker container 了。

如果你是一個 Docker 超新手，那麼我建議你先去看看[系列教程](https://docs.docker.com/engine/getstarted/)。

## 容器(Container)

[最基本的 Docker 進程](http://etherealmind.com/basics-docker-containers-hypervisors-coreos/)。容器(Container)之於虛擬機(Virtual Machine)就好比綫程之於進程。或者你可以把他們想成是 chroots on steroids。

### 生命周期

* [`docker create`](https://docs.docker.com/engine/reference/commandline/create) 創建一個容器但是不啟動。
* [`docker rename`](https://docs.docker.com/engine/reference/commandline/rename/) 允許重命名容器。
* [`docker run`](https://docs.docker.com/engine/reference/commandline/run) 在同一個操作中創建並啟動一個容器。
* [`docker rm`](https://docs.docker.com/engine/reference/commandline/rm) 刪除容器。
* [`docker update`](https://docs.docker.com/engine/reference/commandline/update/) 更新容器的資源限制。

如果你想要一個臨時容器，`docker run --rm` 會在容器停止之後刪除它。

如果你想映射宿主(host)的一個檔案夾到 docker 容器，`docker run -v $HOSTDIR:$DOCKERDIR`。參考 [Volumes](https://github.com/wsargent/docker-cheat-sheet/#volumes)。

如果你想同時刪除和容器關聯的 volumes ，那麼在刪除容器的時候必須包含 -v 選項，像這樣 `docker rm -v`。

在 docker 1.10 中還有一個 [logging driver](https://docs.docker.com/engine/admin/logging/overview/)，每個容器可以獨立使用。如果你想執行 docker 並帶上自定義日誌驅動，這樣 `docker run --log-driver=syslog`

### 啟動和停止

* [`docker start`](https://docs.docker.com/engine/reference/commandline/start) 啟動容器。
* [`docker stop`](https://docs.docker.com/engine/reference/commandline/stop) 停止運行中的容器。
* [`docker restart`](https://docs.docker.com/engine/reference/commandline/restart) 停止之後再啟動容器。
* [`docker pause`](https://docs.docker.com/engine/reference/commandline/pause/) 暫停運行中的容器，將其 "凍結" 在當前狀態。
* [`docker unpause`](https://docs.docker.com/engine/reference/commandline/unpause/) 結束容器暫停狀態。
* [`docker wait`](https://docs.docker.com/engine/reference/commandline/wait) 阻塞，到運行中的容器停止為止。
* [`docker kill`](https://docs.docker.com/engine/reference/commandline/kill) 向運行中容器發送 SIGKILL 指令。
* [`docker attach`](https://docs.docker.com/engine/reference/commandline/attach) 連結到運行中容器。

如果你想整合容器到[宿主進程管理(host process manager)](https://docs.docker.com/engine/admin/host_integration/)，那麼以 `-r=false` 啟動守護進程(daemon)然後使用 `docker start -a`。

如果你想通過宿主暴露容器的連接埠(ports)，請看[暴露連接埠](#exposing-ports)一節。

故障 docker 實例的重啟策略在[這裡](http://container42.com/2014/09/30/docker-restart-policies/)。

#### CPU 限制

你可以限制 CPU，包括使用所有 CPU 的百分比，或者使用特定內核數。

比如，你可以設置 [`cpu-shares`](https://docs.docker.com/engine/reference/run/#/cpu-share-constraint) 。這個設置看起來有點奇怪 -- 1024 的意思是 100% CPU，因此如果你希望容器使用全體 CPU 內核的 50%，應將其設置為 512。更多信息，請查閲 https://goldmann.pl/blog/2014/09/11/resource-management-in-docker/#_cpu :

```
docker run -ti --c 512 agileek/cpuset-test
```

你可以只對某些 CPU 內核使用 [`cpuset-cpus`](https://docs.docker.com/engine/reference/run/#/cpuset-constraint)]。請參閱 https://agileek.github.io/docker/2014/08/06/docker-cpuset/ 獲取更多細節以及一些不錯的視頻:

```
docker run -ti --cpuset-cpus=0,4,6 agileek/cpuset-test
```

注意，Docker 在容器內仍然可以**看到**所有的 CPU -- 雖然它只是用了其中一部分。請查閲 https://github.com/docker/docker/issues/20770 獲取更多細節。

#### 內存限制

你同樣可以在 Docker 設置[內存限制](https://docs.docker.com/engine/reference/run/#/user-memory-constraints) :

```
docker run -it -m 300M ubuntu:14.04 /bin/bash
```

#### 能力(Capabilities)

Linux 的 capability 可以通過使用 `cap-add` 和 `cap-drop` 設置。請參閱 https://docs.docker.com/engine/reference/run/#/runtime-privilege-and-linux-capabilities 獲取更多細節。這有助于提高安全性。

如需要掛載基于 FUSE 檔案系統，你需要同時結合 --cap-add 和 --device 使用:

```
docker run --rm -it --cap-add SYS_ADMIN --device /dev/fuse sshfs
```

授予對單個設備訪問權限:

```
docker run -it --device=/dev/ttyUSB0 debian bash
```

授予所有設備訪問權限:

```
docker run -it --privileged -v /dev/bus/usb:/dev/bus/usb debian bash
```

有關容器特權的更多詳情請參考[這裡](https://docs.docker.com/engine/reference/run/#/runtime-privilege-and-linux-capabilities)

### 信息

* [`docker ps`](https://docs.docker.com/engine/reference/commandline/ps) 查看運行中的所有容器。
* [`docker logs`](https://docs.docker.com/engine/reference/commandline/logs) 從容器中獲取日誌。(你也可以使用自定義日誌驅動，不過在 1.10 中，它只支持 `json-file` 和 `journald`)
* [`docker inspect`](https://docs.docker.com/engine/reference/commandline/inspect) 查看某個容器的所有信息(包括 IP 地址)。
* [`docker events`](https://docs.docker.com/engine/reference/commandline/events) 從容器中獲取事件(events)。
* [`docker port`](https://docs.docker.com/engine/reference/commandline/port) 查看容器的公開連接埠。
* [`docker top`](https://docs.docker.com/engine/reference/commandline/top) 查看容器中活動進程。
* [`docker stats`](https://docs.docker.com/engine/reference/commandline/stats) 查看容器的資源使用情況統計信息。
* [`docker diff`](https://docs.docker.com/engine/reference/commandline/diff) 查看容器的 FS 中有變化檔案信息。

`docker ps -a` 查看所有容器，包括正在運行的和已停止的。

`docker stats --all` 顯示正在運行的容器列表 

### 導入 / 導出

* [`docker cp`](https://docs.docker.com/engine/reference/commandline/cp) 在容器和本地檔案系統之間複製檔案或檔案夾。
* [`docker export`](https://docs.docker.com/engine/reference/commandline/export) 將容器的檔案系統切換為壓縮包(tarball archive stream)輸出到 STDOUT。

### 執行命令

* [`docker exec`](https://docs.docker.com/engine/reference/commandline/exec) 在容器中執行命令。

比如，進入正在運行的容器，在名為 foo 的容器中打開一個新的 shell 進程: `docker exec -it foo /bin/bash`.

## 鏡像(Images)

鏡像是[docker 容器的模板](https://docs.docker.com/engine/understanding-docker/#how-does-a-docker-image-work)。

### 生命周期

* [`docker images`](https://docs.docker.com/engine/reference/commandline/images) 查看所有鏡像。
* [`docker import`](https://docs.docker.com/engine/reference/commandline/import) 從壓縮檔案中創建鏡像。
* [`docker build`](https://docs.docker.com/engine/reference/commandline/build) 從 Dockerfile 創建鏡像。
* [`docker commit`](https://docs.docker.com/engine/reference/commandline/commit) 為容器創建鏡像，如果容器正在運行則會臨時暫停。
* [`docker rmi`](https://docs.docker.com/engine/reference/commandline/rmi) 刪除鏡像。
* [`docker load`](https://docs.docker.com/engine/reference/commandline/load) 通過 STDIN 從壓縮包加載鏡像，包括鏡像和標籤(images and tags) (0.7 起).
* [`docker save`](https://docs.docker.com/engine/reference/commandline/save) 通過 STDOUT 保存鏡像到壓縮包，包括所有的父層，標籤和版本(parent layers, tags & versions) (0.7 起).

### 信息

* [`docker history`](https://docs.docker.com/engine/reference/commandline/history) 查看鏡像歷史記錄。
* [`docker tag`](https://docs.docker.com/engine/reference/commandline/tag) 給鏡像命名打標(tags) (本地或者倉庫)。

### 清理

雖然你可以用 `docker rmi` 命令來刪除指定的鏡像，但是這裡有個稱為 [docker-gc](https://github.com/spotify/docker-gc) 的工具，它可以以一種安全的方式，清理掉那些不再被任何容器使用的鏡像。

### 加載/保存鏡像

從檔案中加載鏡像:
```
docker load < my_image.tar.gz
```
保存既有鏡像:
```
docker save my_image:my_tag | gzip > my_image.tar.gz
```

### 導入/導出容器

從檔案中將容器作為鏡像導入:
```
cat my_container.tar.gz | docker import - my_image:my_tag
```

導出既有容器:
```
docker export my_container | gzip > my_container.tar.gz
```

### 加載被保存的鏡像和導入作為鏡像導出的容器之間的不同

通過 `load` 命令來加載鏡像，會創建一個新的鏡像，並繼承原鏡像的所有歷史。
通過 `import` 將容器作為鏡像導入，也會創建一個新的鏡像，但並不包含原鏡像的歷史，因此生成的鏡像會比使用加載方式生成的鏡像要小。

## 網絡(Networks)

Docker 有[網絡(networks)](https://docs.docker.com/engine/userguide/networking/)功能。我並不是很瞭解它，所以這是一個擴展本文的好地方。這裡有篇筆記指出，這是一種可以不使用連接埠來達成 docker 容器間通信的好方法。詳情查閲[通過網絡來工作](https://docs.docker.com/engine/userguide/networking/work-with-networks/)。

### 生命周期

* [`docker network create`](https://docs.docker.com/engine/reference/commandline/network_create/)
* [`docker network rm`](https://docs.docker.com/engine/reference/commandline/network_rm/)

### 信息

* [`docker network ls`](https://docs.docker.com/engine/reference/commandline/network_ls/)
* [`docker network inspect`](https://docs.docker.com/engine/reference/commandline/network_inspect/)

### 連結

* [`docker network connect`](https://docs.docker.com/engine/reference/commandline/network_connect/)
* [`docker network disconnect`](https://docs.docker.com/engine/reference/commandline/network_disconnect/)

你可以為[容器指定 IP 地址](https://blog.jessfraz.com/post/ips-for-all-the-things/):

```
# 使用你自己的子網和網關創建一個橋接網絡
docker network create --subnet 203.0.113.0/24 --gateway 203.0.113.254 iptastic

# 基于以上創建的網絡，運行一個nginx容器並指定ip
$ docker run --rm -it --net iptastic --ip 203.0.113.2 nginx

# 在其他地方使用curl訪問這個ip（假設這是一個公網ip）
$ curl 203.0.113.2
```

## 倉管中心和倉庫(Registry & Repository)

倉庫(repository)是*被託管(hosted)*的已命名鏡像(tagged images)集合，這組鏡像用於構建容器檔案系統。

倉管中心(registry)是一個*託管服務(host)* -- 一個服務，用於存儲倉庫和提供 HTTP API，以便[管理上傳和下載倉庫](https://docs.docker.com/engine/tutorials/dockerrepos/)。

Docker.com 把它自己的[索引](https://hub.docker.com/)託管到了它的倉管中心，那裡有數量眾多的倉庫。不過話雖如此，這個倉管中心[並沒有很好的驗證鏡像](https://titanous.com/posts/docker-insecurity)，所以如果你很擔心安全問題的話，請儘量避免使用它。

* [`docker login`](https://docs.docker.com/engine/reference/commandline/login) 登入倉管中心。
* [`docker logout`](https://docs.docker.com/engine/reference/commandline/logout) 登出倉管中心。
* [`docker search`](https://docs.docker.com/engine/reference/commandline/search) 從倉管中心檢索鏡像。
* [`docker pull`](https://docs.docker.com/engine/reference/commandline/pull) 從倉管中心拉去鏡像到本地。
* [`docker push`](https://docs.docker.com/engine/reference/commandline/push) 從本地推送鏡像到倉管中心。

### 本地倉管中心

你可以創立一個本地的倉管中心，通過使用 [docker distribution](https://github.com/docker/distribution) 工程，細節請查看 [本地發佈(local deploy)](https://github.com/docker/docker.github.io/blob/master/registry/deploying.md) 介紹。  

也可以參考 [郵件列表](https://groups.google.com/a/dockerproject.org/forum/#!forum/distribution)。

## Dockerfile

[配置檔案](https://docs.docker.com/engine/reference/builder/)。當你執行 `docker build` 的時候會根據該配置檔案設置 Docker 容器。遠優於使用 `docker commit`。

下面是一些常用的編寫 Dockerfile 的編輯器和語法高亮模組︰
* 如果你使用 [jEdit](http://jedit.org)，我為 [Dockerfile](https://github.com/wsargent/jedit-docker-mode) 做了個語法高亮模組。
* [Sublime Text 2](https://packagecontrol.io/packages/Dockerfile%20Syntax%20Highlighting)
* [Atom](https://atom.io/packages/language-docker)
* [Vim](https://github.com/ekalinin/Dockerfile.vim)
* [Emacs](https://github.com/spotify/dockerfile-mode)
* [TextMate](https://github.com/docker/docker/tree/master/contrib/syntax/textmate)
* 如果要找更全面的關於編輯器或者 IDE 的內容，請看 [當 Docker 遇上 IDE](https://domeide.github.io/)

### 指令

* [.dockerignore](https://docs.docker.com/engine/reference/builder/#dockerignore-file)
* [FROM](https://docs.docker.com/engine/reference/builder/#from) 為其他指令設置基礎鏡像(Base Image)。
* [MAINTAINER](https://docs.docker.com/engine/reference/builder/#maintainer) 為生成的鏡像設置作者欄位。
* [RUN](https://docs.docker.com/engine/reference/builder/#run) 在當前鏡像的基礎上生成一個新層並執行命令。
* [CMD](https://docs.docker.com/engine/reference/builder/#cmd) 設置容器預設執行命令。
* [EXPOSE](https://docs.docker.com/engine/reference/builder/#expose) 告知 Docker 容器在運行時所要監聽的網絡連接埠。注意：並沒有實際上將連接埠設置為可訪問。
* [ENV](https://docs.docker.com/engine/reference/builder/#env) 設置環境變數。
* [ADD](https://docs.docker.com/engine/reference/builder/#add) 將檔案，檔案夾或者遠程檔案複製到容器中。緩存無效。儘量用 `COPY` 代替 `ADD`。
* [COPY](https://docs.docker.com/engine/reference/builder/#copy) 將檔案或檔案夾複製到容器中。
* [ENTRYPOINT](https://docs.docker.com/engine/reference/builder/#entrypoint) 將一個容器設置為可執行。
* [VOLUME](https://docs.docker.com/engine/reference/builder/#volume) 為外部掛載卷標或其他容器設置掛載點(mount point)。
* [USER](https://docs.docker.com/engine/reference/builder/#user) 設置執行 RUN / CMD / ENTRYPOINT 命令的用戶名。
* [WORKDIR](https://docs.docker.com/engine/reference/builder/#workdir) 設置工作目錄。
* [ARG](https://docs.docker.com/engine/reference/builder/#arg) 定義編譯時(build-time)變數。
* [ONBUILD](https://docs.docker.com/engine/reference/builder/#onbuild) 添加觸發指令，當該鏡像被作為其他鏡像的基礎鏡像時該指令會被觸發。
* [STOPSIGNAL](https://docs.docker.com/engine/reference/builder/#stopsignal) 設置通過系統向容器發出退出指令。
* [LABEL](https://docs.docker.com/engine/userguide/labels-custom-metadata/) 將鍵值對元數據(key/value metadata)應用到你的鏡像，容器，或者守護進程。 

### 教程

* [Flux7's Dockerfile Tutorial](http://flux7.com/blogs/docker/docker-tutorial-series-part-3-automation-is-the-word-using-dockerfile/)

### 例子

* [Examples](https://docs.docker.com/engine/reference/builder/#dockerfile-examples)
* [Best practices for writing Dockerfiles](https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/)
* [Michael Crosby](http://crosbymichael.com/) 還有更多的 [Dockerfiles best practices](http://crosbymichael.com/dockerfile-best-practices.html) / [take 2](http://crosbymichael.com/dockerfile-best-practices-take-2.html)
* [Building Good Docker Images](http://jonathan.bergknoff.com/journal/building-good-docker-images) / [Building Better Docker Images](http://jonathan.bergknoff.com/journal/building-better-docker-images)
* [Managing Container Configuration with Metadata](https://speakerdeck.com/garethr/managing-container-configuration-with-metadata)

## 層(Layers)

Docker 的版本化檔案系統是基于層的。就像[git的提交或檔案變更系統](https://docs.docker.com/engine/userguide/storagedriver/imagesandcontainers/)一樣。

注意: 如果你使用 [aufs](https://en.wikipedia.org/wiki/Aufs) 作為你的檔案系統，當刪除一個容器的時候，Docker 並不一定能成功刪除的檔案卷標！更多詳細信息請參閱 [PR 8484](https://github.com/docker/docker/pull/8484)。

## 連結(Links)

連結(Links)[通過 TCP/IP 連接埠](https://docs.docker.com/userguide/dockerlinks/)實現了 Docker 容器之間的通訊。[連結到 Redis](https://docs.docker.com/examples/running_redis_service/) 和 [Atlassian](https://blogs.atlassian.com/2013/11/docker-all-the-things-at-atlassian-automation-and-wiring/) 是兩個可用的例子。你還可以[通過 hostname 關聯連結](https://docs.docker.com/engine/userguide/networking/default_network/dockerlinks/#/updating-the-etchosts-file)。

注意: 如果你希望容器之間**只**通過連結進行通訊，在啟動 docker 守護進程的時候請添加參數 `-icc=false` 來禁用內部進程通訊。

如果你有一個名為 CONTAINER 的容器(通過 `docker run --name CONTAINER` 指定) 並且在 Dockerfile 中，它的連接埠暴露為:

```
EXPOSE 1337
```

然後，我們創建另外一個名為 LINKED 的容器:

```
docker run -d --link CONTAINER:ALIAS --name LINKED user/wordpress
```

然後 CONTAINER 的連接埠和別名將會以如下的環境變數出現在 LINKED 中:

```
$ALIAS_PORT_1337_TCP_PORT
$ALIAS_PORT_1337_TCP_ADDR
```

之後你就可以通過這種方式來連結它了。

要刪除連結，通過命令 `docker rm --link`。

通常，docker 服務之間的連結，是"服務發現"的一個子集，如果你打算在生產中大規模使用 Docker，這將是一個很大的問題。請參閱[The Docker Ecosystem: Service Discovery and Distributed Configuration Stores](https://www.digitalocean.com/community/tutorials/the-docker-ecosystem-service-discovery-and-distributed-configuration-stores)獲得更多細節。

## 卷標(Volumes)

Docker 的卷標(volumes)是一個[free-floating 檔案系統](https://docs.docker.com/engine/tutorials/dockervolumes/)。它們不應該連結到特定的容器上。好的做法是如果可能，應當把卷標掛載到[純數據容器(data-only containers)](https://medium.com/@ramangupta/why-docker-data-containers-are-good-589b3c6c749e)上。

### 生命周期

* [`docker volume create`](https://docs.docker.com/engine/reference/commandline/volume_create/)
* [`docker volume rm`](https://docs.docker.com/engine/reference/commandline/volume_rm/)

### 信息

* [`docker volume ls`](https://docs.docker.com/engine/reference/commandline/volume_ls/)
* [`docker volume inspect`](https://docs.docker.com/engine/reference/commandline/volume_inspect/)

卷標在不能使用連結(只有 TCP/IP )的情況下非常有用。例如，如果你有兩個 docker 實例需要通訊並在檔案系統上留下記錄。

你可以一次性將其掛載到多個 docker 容器上，通過 `docker run --volumes-from`。

因為卷標是獨立的檔案系統，它們通常被用於存儲各容器之間的瞬時狀態。也就是說，你可以配置一個無狀態臨時容器，關掉之後，當你有第二個這種臨時容器實例的時候，你可以從上一次保存的狀態繼續執行。

查看[卷標進階](http://crosbymichael.com/advanced-docker-volumes.html)來獲取更多細節。Container42 [非常有用](http://container42.com/2014/11/03/docker-indepth-volumes/)。

你可以[將宿主 MacOS 的檔案夾映射為 docker 卷標](https://docs.docker.com/engine/tutorials/dockervolumes/#mount-a-host-directory-as-a-data-volume):

```
docker run -v /Users/wsargent/myapp/src:/src
```

你也可以用遠程 NFS 卷標，如果你覺得你[有足夠勇氣](https://docs.docker.com/engine/tutorials/dockervolumes/#/mount-a-shared-storage-volume-as-a-data-volume)。

可還可以考慮運行一個純數據容器，像[這裡](http://container42.com/2013/12/16/persistent-volumes-with-docker-container-as-volume-pattern/)所說的那樣，提供可移植數據。

## 暴露連接埠(Exposing ports)

通過宿主容器暴露輸入連接埠是相當[繁瑣，但有效](https://docs.docker.com/engine/reference/run/#expose-incoming-ports)的。

這種方式可以將容器連接埠映射到宿主連接埠上(只使用本地主機(localhost)介面)，通過使用 `-p`:

```
docker run -p 127.0.0.1:$HOSTPORT:$CONTAINERPORT --name CONTAINER -t someimage
```

你可以告訴 Docker 容器在運行時監聽指定的網絡連接埠，通過使用 [EXPOSE](https://docs.docker.com/engine/reference/builder/#expose):

```
EXPOSE <CONTAINERPORT>
```

但是注意 EXPOSE 並不會暴露連接埠，你需要用參數 `-p` 。比如說你要在 localhost 上暴露容器的連接埠:

```
iptables -t nat -A DOCKER -p tcp --dport <LOCALHOSTPORT> -j DNAT --to-destination <CONTAINERIP>:<PORT>
```

如果你是在 Virtualbox 中運行 Docker，那麼你需要轉發連接埠(forward the port)，使用 [forwarded_port](https://docs.vagrantup.com/v2/networking/forwarded_ports.html)。它可以用於在 Vagrantfile 上配置暴露連接埠段，這樣你就可以動態的映射它們了:

```
Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  ...

  (49000..49900).each do |port|
    config.vm.network :forwarded_port, :host => port, :guest => port
  end

  ...
end
```

如果你忘記你將什麼連接埠映射到宿主容器上的話，使用 `docker port` 來查看它:

```
docker port CONTAINER $CONTAINERPORT
```

## 最佳實踐

這裡有一些最佳實踐的總結，以及一些討論:

* [The Rabbit Hole of Using Docker in Automated Tests](http://gregoryszorc.com/blog/2014/10/16/the-rabbit-hole-of-using-docker-in-automated-tests/)
* [Bridget Kromhout](https://twitter.com/bridgetkromhout) has a useful blog post on [running Docker in production](http://sysadvent.blogspot.co.uk/2014/12/day-1-docker-in-production-reality-not.html) at Dramafever.  
* There's also a best practices [blog post](http://developers.lyst.com/devops/2014/12/08/docker/) from Lyst.
* [A Docker Dev Environment in 24 Hours!](https://engineering.salesforceiq.com/2013/11/05/a-docker-dev-environment-in-24-hours-part-2-of-2.html)
* [Building a Development Environment With Docker](https://tersesystems.com/2013/11/20/building-a-development-environment-with-docker/)
* [Discourse in a Docker Container](https://samsaffron.com/archive/2013/11/07/discourse-in-a-docker-container)

## 安全(Security)

這節準備討論一些關於 Docker 安全性的問題。[安全](https://docs.docker.com/articles/security/)這章講述了更多細節。

首先第一件事: Docker 是有 root 權限的。如果你在 `docker` 組，那麼你就有[ root 權限](http://reventlov.com/advisories/using-the-docker-command-to-root-the-host)。如果你暴露了 docker unix socket 給容器，意味着你賦予了容器[宿主的 root 權限](https://www.lvh.io/posts/dont-expose-the-docker-socket-not-even-to-a-container.html)。Docker 不應該是你唯一的防禦措施。

### 安全提示

為了最大的安全性，你應該會考慮在虛擬機上運行 Docker 。這是直接從 Docker 安全團隊拿來的資料 -- [slides](http://www.slideshare.net/jpetazzo/linux-containers-lxc-docker-and-security) / [notes](http://www.projectatomic.io/blog/2014/08/is-it-safe-a-look-at-docker-and-security-from-linuxcon/)。然後，可以使用 AppArmor / seccomp / SELinux / grsec 之類的來[限制容器的權限](http://linux-audit.com/docker-security-best-practices-for-your-vessel-and-containers/)。更多細節，請查閲 [Docker 1.10 security features](https://blog.docker.com/2016/02/docker-engine-1-10-security/)。

Docker 鏡像 id 屬於[敏感信息](https://medium.com/@quayio/your-docker-image-ids-are-secrets-and-its-time-you-treated-them-that-way-f55e9f14c1a4) 所以它不應該向外界公開。你應該把他們當成密碼來對待。

參考 [Docker Security Cheat Sheet](https://github.com/konstruktoid/Docker/blob/master/Security/CheatSheet.adoc)中 - 作者是 [Thomas Sjögren](https://github.com/konstruktoid) - 關於如何提高容器安全的建議。

下載[docker 安全測試腳本](https://github.com/docker/docker-bench-security)，下載[白皮書](https://blog.docker.com/2015/05/understanding-docker-security-and-best-practices/) 以及訂閲[郵件列表](https://www.docker.com/docker-security) (不幸的是 Docker 並沒有獨立的郵件列表，只有 dev / user)。

你應該遠離那些使用編譯版本 grsecurity / pax 的不穩定內核，比如 [Alpine Linux](https://en.wikipedia.org/wiki/Alpine_Linux)。如果在產品中用了 grsecurity ，那麼你應該考慮使用有[商業支持](https://grsecurity.net/business_support.php)的[穩定版本](https://grsecurity.net/announce.php)，就像你對待 RedHat 那樣。它要 $200 每月，對於你的運維預算來說不值一提。

從 docker 1.11 開始，你可以輕鬆的限制在容器中可用的進程數，以防止 fork bombs。 這要求 linux 內核 >= 4.3 並且要在內核配置中打開 CGROUP_PIDS=y 。

```
docker run --pids-limit=64
```

同時，從 docker 1.11 開始，你也可以限制進程有再獲取新權限的能力了。該功能是 linux 內核從 version 3.5 開始就擁有的。你可以從[這篇博客](http://www.projectatomic.io/blog/2016/03/no-new-privs-docker/)中閲讀到更多關於這方面的內容。

```
docker run --security-opt=no-new-privileges
```

參考 [Docker Security Cheat Sheet](http://container-solutions.com/content/uploads/2015/06/15.06.15_DockerCheatSheet_A2.pdf) (它是個 PDF 版本，搞得非常難用，所以拷貝出來了) 的 [容器解決方案](http://container-solutions.com/is-docker-safe-for-production/):

關閉內部進程通訊:

```
docker -d --icc=false --iptables
```

設置容器為只讀:

```
docker run --read-only
```

通過 hashsum 來驗證卷標:

```
docker pull debian@sha256:a25306f3850e1bd44541976aa7b5fd0a29be
```

設置卷標為只讀:

```
docker run -v $(pwd)/secrets:/secrets:ro debian
```

在 Dockerfile 中定義並運行一個用戶，避免在容器中以 root 身份操作:

```
RUN groupadd -r user && useradd -r -g user user
USER user
```

### 用戶命名空間(User Namespaces)

還可以通過使用 [user namespaces](https://s3hh.wordpress.com/2013/07/19/creating-and-using-containers-without-privilege/) -- 這已經是 1.10 內建功能了，但預設情況下是不啟用的。

要在 Ubuntu 15.10 中啟用用戶命名空間 ("remap the userns")，請[跟着這篇博客的例子](https://raesene.github.io/blog/2016/02/04/Docker-User-Namespaces/)來做。

### 安全相關視頻

* [Using Docker Safely](https://youtu.be/04LOuMgNj9U)
* [Securing your applications using Docker](https://youtu.be/KmxOXmPhZbk)
* [Container security: Do containers actually contain?](https://youtu.be/a9lE9Urr6AQ)

### 安全路線圖

Docker 的路線圖提到關於[seccomp 的支持](https://github.com/docker/docker/blob/master/ROADMAP.md#11-security)。
這裡有個 AppArmor 策略生成器，叫做 [bane](https://github.com/jfrazelle/bane)，他們正在實現[安全配置檔案](https://github.com/docker/docker/issues/17142)。

## 小貼士

來源:

* [15 Docker Tips in 5 minutes](http://sssslide.com/speakerdeck.com/bmorearty/15-docker-tips-in-5-minutes)

### 最後的 Ids

```
alias dl='docker ps -l -q'
docker run ubuntu echo hello world
docker commit `dl` helloworld
```

### 帶命令行的提交 (需要 Dockerfile)

```
docker commit -run='{"Cmd":["postgres", "-too -many -opts"]}' `dl` postgres
```

### 獲取 IP 地址

```
docker inspect `dl` | grep IPAddress | cut -d '"' -f 4
```

或者安裝 [jq](https://stedolan.github.io/jq/):

```
docker inspect `dl` | jq -r '.[0].NetworkSettings.IPAddress'
```

或者用[go 模板](https://docs.docker.com/engine/reference/commandline/inspect)

```
docker inspect -f '{{ .NetworkSettings.IPAddress }}' <container_name>
```

### 獲取連接埠映射

```
docker inspect -f '{{range $p, $conf := .NetworkSettings.Ports}} {{$p}} -> {{(index $conf 0).HostPort}} {{end}}' <containername>
```

### 通過正則獲取容器

```
for i in $(docker ps -a | grep "REGEXP_PATTERN" | cut -f1 -d" "); do echo $i; done`
```

### 獲取環境設定

```
docker run --rm ubuntu env
```

### 強迫關閉正在運行的容器

```
docker kill $(docker ps -q)
```

### 刪除舊容器

```
docker ps -a | grep 'weeks ago' | awk '{print $1}' | xargs docker rm
```

### 刪除停止容器

```
docker rm -v `docker ps -a -q -f status=exited`
```

### 刪除 dangling 鏡像

```
docker rmi $(docker images -q -f dangling=true)
```

### 刪除所有鏡像

```
docker rmi $(docker images -q)
```

### 刪除 dangling 卷標

Docker 1.9 開始:

```
docker volume rm $(docker volume ls -q -f dangling=true)
```

1.9.0 中，過濾器 `dangling=false` 居然 _沒_ 用 - 它會被忽略然後列出所有的卷標。

### 查看鏡像依賴

```
docker images -viz | dot -Tpng -o docker.png
```

### Docker 容器瘦身  [Intercity 博客](http://bit.ly/1Wwo61N)

- 在當前運行層(RUN layer)清理 APT

這應當和其他 apt 命令在同一層中完成。
否則，前面的層將會保持原有信息，而你的鏡像則依舊臃腫。

```
RUN {apt commands} \
  && apt-get clean \  
  && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*
```

- 壓縮鏡像
```
ID=$(docker run -d image-name /bin/bash)
docker export $ID | docker import – flat-image-name
```

- 備份
```
ID=$(docker run -d image-name /bin/bash)
(docker export $ID | gzip -c > image.tgz)
gzip -dc image.tgz | docker import - flat-image-name
```

### 監視運行中容器的系統資源利用率

檢查某個單獨容器的 CPU, 內存, 和 網絡 i/o 使用情況，你可以:

```
docker stats <container>
```

按 id 列出所有的容器:

```
docker stats $(docker ps -q)
```

按名稱列出所有容器:

```
docker stats $(docker ps --format '{{.Names}}')
```

按指定鏡像名稱列出所有容器:

```
docker ps -a -f ancestor=ubuntu
```

## 貢獻手冊(Contributing)

這是關於如何為這份速查表做貢獻的說明。

### 打開 README.md

點擊 [README.md](https://github.com/wsargent/docker-cheat-sheet/blob/master/README.md) <-- 這個連結

![點擊它](../images/click.png)

### 編輯頁面

![選擇它](../images/edit.png)

### 更新和提交

![改變它](../images/change.png)

![提交](../images/commit.png)
