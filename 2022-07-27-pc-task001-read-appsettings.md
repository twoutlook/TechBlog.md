# pc-task001
- pc 生管
- task001 訂單物料分析_C90 
- 生管的訂單物料分析,需要先上傳三個指定檔案,再讀取其內容並寫入DB

# 今天,07/27, 的問題是
- 可以用以下的方法(2) 搭配API 完成 pc-task001 , 三個檔案分別上傳
- 卻無法成功使用方法(2)上傳 2022效率看板.xlsm, 改用方法(1)是有成功!!!

## 官網
- https://docs.microsoft.com/en-us/aspnet/core/blazor/file-uploads?view=aspnetcore-6.0&pivots=server

## (1)Blazor 可以獨力完成 Pages/FileUpload1.razor

```
private async Task LoadFiles(InputFileChangeEventArgs e)
    {
        isLoading = true;
        loadedFiles.Clear();

        foreach (var file in e.GetMultipleFiles(maxAllowedFiles))
        {
            try
            {
                loadedFiles.Add(file);

                var trustedFileNameForFileStorage = Path.GetRandomFileName();
                var path = Path.Combine(Environment.ContentRootPath,
                        Environment.EnvironmentName, "unsafe_uploads",
                        trustedFileNameForFileStorage);

                await using FileStream fs = new(path, FileMode.Create);
                await file.OpenReadStream(maxFileSize).CopyToAsync(fs);
            }
            catch (Exception ex)
            {
                Logger.LogError("File: {Filename} Error: {Error}", 
                    file.Name, ex.Message);
            }
        }

        isLoading = false;
    }
 ```   

## (2)Blazor 和 API 使用 POST 搭配
- POST API 使用　PostAsync 要指定 Server, 開發時和正式發部的 Server 不同, 在 appsettings.json 配置

```

    private async Task OnInputFileChange(InputFileChangeEventArgs e)
    {
        shouldRender = false;
        long maxFileSize = 1024 * 15;
        var upload = false;

        using var content = new MultipartFormDataContent();

        foreach (var file in e.GetMultipleFiles(maxAllowedFiles))
        {
            if (uploadResults.SingleOrDefault(
                f => f.FileName == file.Name) is null)
            {
                try
                {
                    var fileContent = 
                        new StreamContent(file.OpenReadStream(maxFileSize));

                    fileContent.Headers.ContentType = 
                        new MediaTypeHeaderValue(file.ContentType);

                    files.Add(new() { Name = file.Name });

                    content.Add(
                        content: fileContent,
                        name: "\"files\"",
                        fileName: file.Name);

                    upload = true;
                }
                catch (Exception ex)
                {
                    Logger.LogInformation(
                        "{FileName} not uploaded (Err: 6): {Message}", 
                        file.Name, ex.Message);

                    uploadResults.Add(
                        new()
                        {
                            FileName = file.Name, 
                            ErrorCode = 6, 
                            Uploaded = false
                        });
                }
            }
        }

        if (upload)
        {
            var client = ClientFactory.CreateClient();

            var response = 
                await client.PostAsync("https://localhost:5001/Filesave", 
                content);

            if (response.IsSuccessStatusCode)
            {
                var options = new JsonSerializerOptions
                {
                    PropertyNameCaseInsensitive = true,
                };

                using var responseStream =
                    await response.Content.ReadAsStreamAsync();

                var newUploadResults = await JsonSerializer
                    .DeserializeAsync<IList<UploadResult>>(responseStream, options);

                if (newUploadResults is not null)
                {
                    uploadResults = uploadResults.Concat(newUploadResults).ToList();
                }
            }
        }

        shouldRender = true;
    }
    
```    



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

