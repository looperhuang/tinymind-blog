---
title: C# use curl.exe
date: 2024-11-07T03:05:57.433Z
---

## 下載curl.exe
https://curl.se/download.html#Win64

## CurlHelper.cs
```C#
public class CurlHelper
    {
        // curl.exe 路徑
        private static readonly string CurlPath = $"C:\\AP06BZ\\EXE\\CURL.EXE";

        /// <summary>
        /// Get 請求
        /// </summary>
        /// <param name="url"></param>
        /// <param name="headers"></param>
        /// <returns></returns>
        public static string GetRequest(string url, Dictionary<string, string> headers = null)
        {
            string arugments = $"-X GET \"{url}\"";
            if (headers != null)
            {
                foreach (var header in headers)
                {
                    arugments += $" -H \"{header.Key}: {header.Value}\"";
                }
            }
            return ExecuteCurlCommand(arugments);
        }

        /// <summary>
        /// POST 請求(JSON資料)
        /// </summary>
        /// <param name="url"></param>
        /// <param name="jsonData"></param>
        /// <param name="headers"></param>
        /// <returns></returns>
        public static string PostRequest(string url, string jsonData, Dictionary<string, string> headers = null)
        {
            string arguments = $"-X POST -H \"Content-Type: application/json\" -d \"{jsonData}\" \"{url}\"";
            if (headers != null)
            {
                foreach (var header in headers)
                {
                    arguments += $" -H \"{header.Key}: {header.Value}\"";
                }
            }
            return ExecuteCurlCommand(arguments);
        }

        /// <summary>
        /// POST 請求(表單資料)
        /// </summary>
        /// <param name="url"></param>
        /// <param name="formData"></param>
        /// <param name="headers"></param>
        /// <returns></returns>
        public static string PostFormRequest(string url, Dictionary<string, string> formData, Dictionary<string, string> headers = null)
        {
            string arguments = "-X POST";
            foreach (var item in formData)
            {
                arguments += $" -F \"{item.Key}={item.Value}\"";
            }
            arguments += $" \"{url}\"";
            if (headers != null)
            {
                foreach (var header in headers)
                {
                    arguments += $" -H \"{header.Key}: {header.Value}\"";
                }
            }
            return ExecuteCurlCommand(arguments);
        }


        /// <summary>
        /// 執行curl命令返回輸出
        /// </summary>
        /// <param name="arguments"></param>
        /// <returns></returns>
        private static string ExecuteCurlCommand(string arguments)
        {
            var startInfo = new ProcessStartInfo
            {
                FileName = CurlPath,
                Arguments = arguments,
                RedirectStandardOutput = true,
                UseShellExecute = false,
                CreateNoWindow = true,
            };
            using (var p = Process.Start(startInfo))
            {
                using (var reader = p.StandardOutput)
                {
                    return reader.ReadToEnd();
                }
            }
        }
    }
```

## Usage
```C#
 string url = "apiurl...";
 var headers = new Dictionary<string, string>
 {
       {"Authorization", "Bearer your_api_key"}
 };
 string response = CurlHelper.GetRequest(url, headers);
 Console.WriteLine("Get Response: " + response);
```