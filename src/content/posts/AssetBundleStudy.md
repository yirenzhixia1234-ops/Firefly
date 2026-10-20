---
title: AB包学习笔记
published: 2026-09-12
pinned: false
description: AB包学习笔记
tags: [AssetBundle,学习]
author: boluobao
draft: false
---

# AssetBundle

## MD5码 

- MD5是MD5信息摘要算法的简称，它是一种广泛使用的密码散列函数，可以产生出一个128位（16字节）的散列值（hash value），用于确保信息传输完整一致。
- MD5码是通过对文件内容进行散列计算得到的，因此只要文件内容发生变化，MD5码就会发生变化。因此，MD5码常用于文件的完整性校验和数字签名等场景。

### 获得AB包的MD5码

```csharp
private string GetMD5Hash(string filePath)
    {
        using (FileStream file = new FileStream(filePath,FileMode.Open))
        {
            MD5 md5 = new MD5CryptoServiceProvider();
            byte[] retVal = md5.ComputeHash(file);
            file.Close();

            StringBuilder sb = new StringBuilder();
            for (int i = 0; i < retVal.Length; i++)
            {
                sb.Append(retVal[i].ToString("x2"));
            }
            return sb.ToString();
        }

    }
```