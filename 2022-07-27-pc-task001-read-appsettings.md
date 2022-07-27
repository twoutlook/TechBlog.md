# pc-task001
- pc 生管
- task001 訂單物料分析_C90 
- 生管的訂單物料分析,需要先上傳三個指定檔案,再讀取其內容並寫入DB
## 上傳檔案在 Blazor 不能獨力實現, 必需和 API 使用 POST 搭配才能實現
- POST API 使用　PostAsync 要指定 Server, 開發時和正式發部的 Server 不同, 在 appsettings.json 配置
## 上傳三個指定檔案
### Controllers
- FileUploadPcTask001Controller.cs
```
var path1 = config["FileUploadPcTask001:Path1"];
var path2 = config["FileUploadPcTask001:Path2"];
var path = Path.Combine(path1, path2, file.FileName);

await using FileStream fs = new(path, FileMode.Create);
await file.CopyToAsync(fs);
```
### Pages
- FileUploadPcTask001Controller.razor
  -  private async Task OnInputFilePc13Change(InputFileChangeEventArgs e)
  -  private async Task OnInputFilePc11Change(InputFileChangeEventArgs e)
  -  private async Task OnInputFilePc03Change(InputFileChangeEventArgs e) 
  -  目前做法是按  &lt; InputFile OnChange="@OnInputFilePc13Change" /> 檢查必需一致的檔案名稱, 才上傳到指的位置使用其檔名

