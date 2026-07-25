# 叢林學院電腦教室管理與維運規劃專案報告

專案名稱：電腦教室智慧化管理與資安維運系統規劃  

專案規模：50-100 台電腦（含 1台管理員專用機，1-3 台高權限開發專用機） 

專案性質：開源免費架構與原生機制整合方案

 ### 📋 目錄 (Table of Contents)

1. 專案概述 (Overview)

2. 系統架構與開源工具藍圖 (Architecture)

3. 核心功能實施細則 (Implementation Details)

4. 系統維運與防呆優化 (Maintenance & Optimization)

5. 標準作業流程 (Standard Operating Procedures)





### 🎯 專案概述 (Overview)

本專案旨在為規模 50 至 100 台之校園電腦教室建立一套兼顧「資安防護、自動化稽核、分級教學與高效維運」的綜合管理系統。透過結合 Windows 系統原生機制、開源軟體（Pi-hole、FOG、Veyon）與自動化腳本，在零商業軟體採購預算的前提下，實現全方位的教室智慧化管理。

### 🛠 系統架構與開源工具藍圖 (Architecture)


| 需求模組 | 技術方案與工具 | 核心功能說明 | POC測試結果 |
| :--- | :--- | :--- | :--- |
| **1. 登入稽核與日報表** | Windows GPO + PowerShell | 自動記錄學號、電腦編號、登入/登出時間，產出 CSV 匯總表。 | 需確認服務器系統與服務器登錄測試，須Windows Pro |
| **2. 登入自動彈窗** | GPO 指令碼 | 學生登入瞬間強制跳出置頂「教室守則」視窗，點擊同意後方可操作。 | 需確認服務器系統與服務器登錄測試 |
| **3. 分級權限控管** | 系統群組原則 (RBAC) | 普通機鎖定標準使用者與系統槽；開發機賦予 Administrator 與開發環境權限。 | 無需測試 |
| **4. 網站黑白名單** | **Pi-hole (開源)** | 網路層級統一過濾網域，阻斷違規娛樂網站，防止學生竄改 DNS。 | Pi-hole 儘支持Linux環境，考慮windows環境替代方案，如 [Adguardhome](https://github.com/AdguardTeam/Adguardhome) |
| **5. 軟體安裝申請** | Microsoft/Google Forms + 靜默安裝 | 數位化表單審核，配合批次指令更新至映像檔。 | 無需測試 |
| **6. 一鍵映像檔部署** | **FOG Project (開源)** | 透過網路開機 (PXE) 批次還原與更新全教室普通機。 | 須linux環境，或Windows Server上跑VM| 
| **7. 課堂即時監控** | **Veyon (開源)** | 教師端即時檢視學生畫面、螢幕廣播示範、一鍵鎖屏與遠端協助。 | 辦公室電腦測試ok | 
| **8. 維運與防呆優化** | 排程自動關機 + 熱點禁用 | 解決強制關機漏登出、防止學生開手機熱點或繞過防護。 | 待測試 |



### ⚙️ 核心功能實施細則 (Implementation Details)


**1. 身分稽核與出勤統計 (Audit & Reporting)**
學號追蹤：學生以個人學號帳號登入。登入/登出時自動觸發 PowerShell 腳本，將資料附加寫入共用日誌檔 (\\ServerIP\ComputerLogs\Login_Summary.csv)。

報表產出：管理者透過 Excel 樞紐分析表每日快速檢視各學生出勤與駐留總時數。

**2. 登入防呆彈窗 (Compliance Pop-up)**
強制提示：透過本機群組原則 (gpedit.msc) 派發置頂彈窗腳本。學生進入桌面後強制跳出「電腦教室守則」，必須點擊「我同意」按鈕後才能解除鎖定開始操作。

**3. 分級權限配置 (Role-Based Access Control)**
普通機 (45-95台)：設定為「標準使用者」，搭配硬體還原卡防護，禁止存取 C 槽及執行 CMD/PowerShell。

開發機 (3-5台)：獨立區域配置，賦予本地管理員權限，預先安裝 VS Code、Python、Git 等開發工具。

**4. 網路安全與黑白名單 (Network Security)**
Pi-hole 網管：所有學生電腦強制指定 DNS 指向區網內的 Pi-hole 伺服器，統一管理 Blocklist 與 Whitelist。

防破解加固：透過 GPO 鎖定網路介面卡屬性禁止學生竄改 IP/DNS，並透過指令禁用 Windows 內建「行動熱點」功能。


### 🔄 系統維運與防呆優化 (Maintenance & Optimization)


**1. 異常強迫關機補救機制**
問題：學生若直接按實體電源關機，將導致 Logoff 腳本無法執行，造成登出時間遺失。

對策：設定 Windows 工作排程器於每日閉館前（例如 21:50）自動執行結算腳本並觸發強制關機。

**2. 軟體安裝申請與部署工作流**
數位收單：透過 Microsoft Forms/Google Form 統一受理軟體申請（規範須於上課前 3 個工作天提出）。

靜默封裝：管理者審核通過後製作靜默安裝包 (.msi 或帶參數 .exe)。

FOG 批次派發：將調校完畢的普通機上傳至開源 FOG 伺服器 作為黃金映像檔。管理者可於後台一鍵勾選全體普通機，透過網路開機 (PXE) 在 15-20 分鐘內同步完成全教室系統還原與軟體更新。

**3. 課堂即時監控與教學互動 (Veyon Integration)**
畫面預覽：老師端透過 Veyon Master 即時檢視全班 50-100 台電腦縮圖。

課堂秩序掌控：提供「螢幕廣播」、「一鍵鎖屏」及「遠端協助」功能，有效維持上課秩序並即時解決學生操作問題。

<img width="1862" height="1788" alt="mermaid-diagram-2026-07-25-093942" src="https://github.com/user-attachments/assets/af0d0962-ac6b-4a13-99bb-7607f9d937b1" />

