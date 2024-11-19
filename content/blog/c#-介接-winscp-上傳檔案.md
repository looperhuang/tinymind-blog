---
title: C# 介接 WinScp 上傳檔案
date: 2024-10-04T07:19:50.946Z
---

## Install

```powershell
dotnet add package WinSCP
```

## Example
```C# 
// 連線設定
SessionOptions sessionOptions = new SessionOptions
{
    Protocol = Protocol.Sftp,
    HostName = "hostname",
    PortNumber = port,
    UserName = "xxxx",
    SshHostKeyFingerprint = "ssh-rsa 2048 xxxxxxxxxxxxxxxxxxxxxxxxxxx",
    SshPrivateKeyPath = "...." // 私鑰路徑自行調整(選用)
};

// 開啟會話
using var session = new Session();
session.Open(sessionOptions);

// 定義上傳的文件路徑和遠端目錄
string localFilePath = "文件路徑";
string remoteDirectory = "伺服器目錄"; // 遠端伺服器的目錄

// 上傳文件
TransferOperationResult transferResult;
transferResult = this.session.PutFiles(localFilePath, remoteDirectory);

// 檢查上傳是否成功
transferResult.Check();
```

## 參考資料
https://winscp.net/eng/docs/library#csharp
