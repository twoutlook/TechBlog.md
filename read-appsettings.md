# 生管的訂單物料分析,需要先上傳三個指定檔案,再讀取其內容並寫入DB
## 上傳檔案在 Blazor 不能獨力實現, 必需和 API 使用 POST 搭配才能實現
- POST API 使用　PostAsync 要指定 Server, 開發時和正式發部的 Server 不同, 在 appsettings.json 配置
