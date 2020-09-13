---
title: UWP经验 - 通过Win32 API读写文件
lang: zh
type: post
date: 2020/9/13 17:00:00
categories:
- UWP
tags:
- uwp
- file
---

按照《[向文件进行写入的最佳做法](https://docs.microsoft.com/zh-cn/windows/uwp/files/best-practices-for-writing-to-files)》这篇文档所述，UWP中推荐的做法是使用`FileIO`或者`PathIO`进行文件读写。这没问题，但这俩工具类都各有缺陷：

- `FileIO`建立在`StorageFile`基础上，想读写，先拿到`StorageFile`
- `PathIO`可以通过路径，但默认仅限于应用文件夹，其它目录没权限（需要一些特殊操作）。

那么有没有既能通过路径获取文件，又没有严格的权限限制的方法呢？当然有了！

<!--More-->

## 简介

正如前文《[UWP经验 - 如何大幅提高UWP的文件遍历速度](./uwp_file_1)》所述，1803之后，UWP获得了一些Win32 API的加持，本文正建立在此基础之上。

***所以没看前文的同学可以先看看***

我们这回要做的事情比遍历文件夹简单，也不用进行复杂的结构定义，当然，方法还是要引入的。

## 方法说明

### 定义参数及方法

```csharp
internal const int GENERIC_READ = unchecked((int)0x80000000);
internal const int GENERIC_ALL = unchecked((int)0x10000000);
internal const int OPEN_EXISTING = 3;
internal const int FILE_ATTRIBUTE_NORMAL = 0x80;

[DllImport("api-ms-win-core-file-fromapp-l1-1-0.dll", SetLastError = true, CharSet = CharSet.Unicode)]
internal static extern SafeFileHandle CreateFileFromApp(string lpFileName,
            int dwDesiredAccess,
            int dwShareMode,
            IntPtr lpSecurityAttributes,
            uint dwCreationDisposition,
            uint dwFlagsAndAttributes,
            SafeFileHandle hTemplateFile);
```

这次要引入的是`CreateFile`方法的变体：[CreateFileFromApp](https://docs.microsoft.com/en-us/previous-versions/windows/desktop/legacy/mt846585(v=vs.85))。

这次我没有定义特别多的枚举，严格来说，一些参量要定义枚举的，这边我就省略了，直接写的固定值。

返回值写的`SafeFileHandle`，这个在UWP里面也是能用的（我以前还一直觉得不能用）。这个类型就是一个文件句柄了，后面我们可以通过这个句柄对文件进行读写。

### 读文件

:::warning
这里直接用文件路径是因为该文件的上层文件夹已经被我加进`FutureAccessList`里面了，详情可以查看上一篇文章中选择文件夹那一块。不然的话通过路径访问还是会被禁止的，开了`broadFileSystemAccess`另说。
:::

```csharp
public async Task<string> ReadFileFromPath(string path)
{
    var h = CreateFileFromApp(path, GENERIC_READ, 0, IntPtr.Zero, OPEN_EXISTING, FILE_ATTRIBUTE_NORMAL, new SafeFileHandle(IntPtr.Zero, true));
    if (!h.IsInvalid)
    {
        using (h)
        using (FileStream stream = new FileStream(h, FileAccess.Read))
        using (StreamReader reader = new StreamReader(stream))
        {
            string text = await reader.ReadToEndAsync();
            return text;
        }
    }
    throw new FileNotFoundException("File not found");
}
```

在获取到句柄之后我们就可以通过句柄创建`FileStream`，然后用`StreamReader`读就完事了。

### 写文件

```csharp
PathIO.WriteTextAsync(path, content);
```

你可能觉得奇怪，为什么这里直接上`PathIO`了，也没啥，简单呗。

好吧，不开玩笑，这个应该是内部的问题，照理来说，我们只需要把读文件的函数稍加修改即可。但实际上，即便我们把文件夹丢进`FutureAccessList`里面也不管用，依然会有权限问题（但是用`PathIO`不会）。

很神奇，我也只当这是一个坑了，反正怎么写不是写，能写就行。

当然了，再说一次，PathIO能写入，那是我把文件的上级文件夹加进`FutureAccessList`的原因，其它情况会有权限问题。

## 小结

用Win32 API对文件进行读写（实际上只是读）倒没有遍历看上去那么复杂。在第一次选取文件夹之后，将文件夹加进`FutureAccessList`之中，则可以避免权限问题。这样看来，第一次请求就相当于是用户进行的一次授权了。

`PathIO`能读也能写，就调用来说，比直接上Win32 API要方便，速度也差不多（可能就慢一点点），所以我个人还是推荐用PathIO的，适用性会更好一些。

说到底，UWP对文件访问进行限制还是为了数据安全。用Win32 API也只是在已授权的范围内进行活动，不会再像过去那样通过各种手段绕开权限限制读写文件，实在没必要。

获取文件基本信息、读/写文件，我们对文件的常用操作不就是这些吗？至于修改文件信息、移动/删除文件，也有相应的API提供，通过这两篇文章，我想大家都有能力自己处理。

---

快速文件遍历以及配套的文件读写，这是我在做Markdown编辑器过程中遇到的问题，特此记录。