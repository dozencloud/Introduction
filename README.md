# DozenCloud 伊達雲

## Intro

### 起源

目前市面上並沒有太盛行 ARM VPS 相關的服務，唯法國公司 Scaleway 提供 Bare-Metal ARM Module Server 給一般人租用。但是因為台灣地理位置距離法國太遠，網路延遲相當嚴重，所以開啟 DozenCloud 計畫，希望能在台灣建立原生 ARM 雲端環境，供 ARM 開發者可以方便開發與測試。更進一步因為 ARM 的低功耗、低成本優勢，更可在 DozenCloud 上架設平衡附載伺服器、資料庫中心。

### 使用
* Openstack
* Docker

## 架構

![](images/DozenCloud-design.png)

使用 X86 電腦作為 Openstack Controller ，以及 Docker Container 的儲存空間，使用 ARM 開發板作為 Openstack 計算節點，可以享受到 Openstack 在 X86 上的強大支援，亦可提供原生的 ARM 開發環境。

### Openstack

使用 Ubuntu 14.04 及 Openstack Kilo ，作為整體資源管理的核心，管理 ARM 開發板的運算資源以及 Container 中的網路控制。透過 Neutron 可以更方便的將 Container 劃分在不同子網路中，也可以做網路隔離以及防火牆的設置。

### Docker

Openstack 預設使用 KVM 作為虛擬化技術，但是因為 ARM 虛擬化還在發展當中，並非成熟技術，所以改用 Docker Container 技術取代效能消耗相對大的虛擬化技術，將原本 IaaS 改為 PaaS 發展，提高開發環境的使用者體驗，並可提供相同的 ARM 環境。並使用 BTRFS 作為 Docker 的儲存方式，透過 BTRFS 的 Copy-on-Write 功能，可以加快建立 Container 速度，並節省儲存空間的使用。

### ARM Develop Board

目前使用 Banana Pi M2 搭載 Ubuntu 15.04 以及 Linux Kernel 4.1 作為 Openstack 的計算節點，提供原生 ARM 環境以供開發者開發以及測試。

### NAS

因 Banana Pi M2 的儲存空間使用 microSD 卡，因效能不足以負荷大量 I/O ，所以使用 NAS 作為 Docker Container 存放空間。使用 FreeNAS 搭配 ZFS RaidZ 增強資料容錯功能，並使用 ZFS 提供之 zvol 功能，將硬碟空間劃分為虛擬硬碟透過 iSCSI 提供給 Banana Pi M2 作為硬碟使用，並在上建立具有 Copy-on-Write 功能的 BTRFS 檔案系統。

## 研究方向

### 記憶體限制

要將計算節點的 CGroup 相關功能開啟，必須要在開機載入 Kernel 時加上 `cgroup_enable=memory swapaccount=1` 作為參數，正在研究更改 `boot.scr` 的方式。

### 磁碟限制

因為效能需求選擇 BTRFS ，而非 Device Mapper ，但因此喪失 Device Mapper 提供的磁碟限制功能，必須尋找 BTRFS 的磁碟限制的替代方案

### Linux Kernel Config

修改 Kernel Config 並重新編譯 ARM Linux Kernel ，目的取得更佳的效能以及使用體驗。

### Nova-Docker

Openstack 連接 Docker 的 driver ，有實作 Heat 相關功能，尚待研究 Heat 功能。

### ZFS zvol

使用 ZFS zvol 雖然可以得到讓計算節點有原生的檔案系統支援，但是沒有辦法像 NFS 系統一樣可以快速針對節點損壞狀況進行快速 Container 移轉，必須要再研究如何進行 Docker Container 的移轉模式。