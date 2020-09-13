---
title: UWP经验 - 如何大幅提高UWP的文件遍历速度
lang: zh
type: post
date: 2020/9/13 13:00:00
categories:
- UWP
tags:
- uwp
- file
---

在UWP中，通常是使用`StorageFile`进行文件相关的操作，大多数情况下，这没有问题。但如果涉及到文件夹遍历，你就会发现建立在`Storage`基础上的文件查询太慢了。

以一个普通的VuePress前端项目为例，在加载完本地依赖后，整个项目文件数量在20,000个左右（包括node_modules）。使用`StorageFolder.GetItemsAsync()`，然后循环递归查询，耗时约4分半。

试想，如果你要做一个编辑器，展示文件树，打开应用后可能要等5分钟才能完全加载完，这简直是噩梦。

但现在，通过公开的Win32 API，我们可以直接把时间压缩在4秒左右，这种文件查询速度绝对是一个飞跃式的提升（尽管可能比不上真正的Win32应用查询速度）。

<!--More-->

## 简介

Win32的API通常是通过文件路径访问文件的，这在UWP里基本行不通（UWP对通过路径访问文件有着严格的限制）。在以前，涉及到Win32文件访问的API（比如`System.IO`命名空间下的一些API）只能在应用文件夹内进行操作，没有权限访问其它目录。

但在1803之后，UWP得到了新的文件API的加持。微软将`fileapifromapp.h`引入了UWP，添加了一些以`FromApp`作为后缀的API，扩大了文件访问API的范围。常见的，比如文档(Document)，下载(Download)等文件夹都可以访问了，开了`broadFileSystemAccess`权限之后，访问范围就更大了。

这次我们要用到的API就是新引入的API：[FindFirstFileExFromApp](https://docs.microsoft.com/en-us/previous-versions/windows/desktop/legacy/mt846625(v=vs.85))

## 方法说明

点进上面我提供的方法文档之后可以发现，方法是用C++定义的，毕竟Win32的历史比C#早得多。为了能够正常使用这个方法，我们需要进行一些结构体定义和类型转换：

### 定义结构与枚举

```csharp
using FileAttributes = System.IO.FileAttributes;

//...

public const int FIND_FIRST_EX_LARGE_FETCH = 2;

public enum FINDEX_INFO_LEVELS
{
    FindExInfoStandard = 0,
    FindExInfoBasic = 1
}

public enum FINDEX_SEARCH_OPS
{
    FindExSearchNameMatch = 0,
    FindExSearchLimitToDirectories = 1,
    FindExSearchLimitToDevices = 2
}

[StructLayout(LayoutKind.Sequential, CharSet = CharSet.Auto)]
public struct WIN32_FIND_DATA
{
    public uint dwFileAttributes;
    public System.Runtime.InteropServices.ComTypes.FILETIME ftCreationTime;
    public System.Runtime.InteropServices.ComTypes.FILETIME ftLastAccessTime;
    public System.Runtime.InteropServices.ComTypes.FILETIME ftLastWriteTime;
    public uint nFileSizeHigh;
    public uint nFileSizeLow;
    public uint dwReserved0;
    public uint dwReserved1;
    [MarshalAs(UnmanagedType.ByValTStr, SizeConst = 260)]
    public string cFileName;
    [MarshalAs(UnmanagedType.ByValTStr, SizeConst = 14)]
    public string cAlternateFileName;
}
```

:::tip
MarshalAs 这个特性用于在托管代码（C#）和非托管代码（C++）之间进行类型转换
:::

这里的一些枚举和结构体是`FindFirstFileExFromApp`所需要的，其中`WIN32_FIND_DATA`这个结构体是我们在后续方法中需要拿到的文件基础数据。

从这里你也能发现，在C++中引入个头文件就可以轻易调用的方法，在C#里则要麻烦的多，需要自己定义结构，还要搞一搞类型转换，不过相信我，这一切都是值得的。

### 定义方法

FindFirstFileExFromApp

```csharp
[DllImport("api-ms-win-core-file-fromapp-l1-1-0.dll", SetLastError = true, CharSet = CharSet.Unicode)]
public static extern IntPtr FindFirstFileExFromApp(
    string lpFileName,
    FINDEX_INFO_LEVELS fInfoLevelId,
    out WIN32_FIND_DATA lpFindFileData,
    FINDEX_SEARCH_OPS fSearchOp,
    IntPtr lpSearchFilter,
    int dwAdditionalFlags);
```

FindNextFile

```csharp
[DllImport("api-ms-win-core-file-l1-1-0.dll", CharSet = CharSet.Unicode)]
static extern bool FindNextFile(IntPtr hFindFile, out WIN32_FIND_DATA lpFindFileData);
```

FindClose

```csharp
[DllImport("api-ms-win-core-file-l1-1-0.dll")]
static extern bool FindClose(IntPtr hFindFile);
```

这里的方法定义就是按照文档里的来了，这里要注意，文件名、参数名、参数类型、返回类型都是要对应上的，因为这是你从DLL中引入的方法。（所以我说麻烦，你还要找C++对应的C#的类型）

不同的方法可能会从不同的DLL中引入，关于方法具体在哪个DLL，可以查看这篇文档：[APIs present on all Windows 10 devices](https://docs.microsoft.com/en-us/uwp/win32-and-com/win32-apis)

### 如何使用

定义完了所需结构与方法，接下来就要看看怎么用了，这里我们可以打开一个文件夹，然后把这个文件夹里面所有文件名罗列出来。

1. 拿到文件夹：

```csharp
var folderPicker = new Windows.Storage.Pickers.FolderPicker();
folderPicker.SuggestedStartLocation = Windows.Storage.Pickers.PickerLocationId.Desktop;
folderPicker.FileTypeFilter.Add("*");
StorageFolder folder = await folderPicker.PickSingleFolderAsync();

StorageApplicationPermissions.FutureAccessList.AddOrReplace(Guid.NewGuid().ToString("N"), folder);
```

在第一次的时候，我们还是需要通过`FolderPicker`来选取文件夹，而不是直接通过路径访问。始终记得，我们开发的是UWP应用，而且如非必须，不要开`broadFileSystemAccess`权限。

在获取到文件夹之后，我们可以将其加入`FutureAccessList`获得后续访问的权限。这样我们才能在接下来的步骤中通过路径访问其中的文件/文件夹。

2. 创建递归函数

```csharp
public async Task<int> FindFilesWithWin32(string folderPath, int count)
{
    WIN32_FIND_DATA findData;
    FINDEX_INFO_LEVELS findInfoLevel = FINDEX_INFO_LEVELS.FindExInfoBasic;
    int additionalFlags = FIND_FIRST_EX_LARGE_FETCH;

    IntPtr hFile = FindFirstFileExFromApp(folderPath + "\\*.*", 
                                          findInfoLevel,
                                          out findData, 
                                          FINDEX_SEARCH_OPS.FindExSearchNameMatch, 
                                          IntPtr.Zero,
                                          additionalFlags);
    if (hFile.ToInt32() != -1)
    {
        do
        {
            if (((FileAttributes)findData.dwFileAttributes & FileAttributes.Directory) != FileAttributes.Directory)
            {
                var fn = findData.cFileName;
                Debug.WriteLine(fn);
                ++count;
            }
            else
            {
                if (findData.cFileName != "." && findData.cFileName != "..")
                    count = await FindFilesWithWin32(folderPath + "\\" + findData.cFileName, count);
            }
        } while (FindNextFile(hFile, out findData));

        FindClose(hFile);
    }

    return count;
}
```

:::tip
方法里面的参数说明在方法文档里有，这里就不赘述了，按需修改即可
:::

拿到文件夹路径之后，我们通过`FindFirstFileExFromApp`拿文件（也可能是文件夹）指针，拿到之后，判断一下是否有效，有效则开始在当前目录进行遍历。

- `FindFirstFileExFromApp`用于进行文件夹非空的判断以及初始化索引。
- `FindNextFile`用于调整指针至下一个文件/文件夹，并改变findData的值
- `FindClose`用于结束遍历，释放指针

在方法内部，如果判断得到当前获取到的findData是文件夹，那么进行递归调用。获取到文件后，在控制台输出文件名。

需要注意的是，每个文件夹内部有两个特殊的`通用文件夹`，一个是`.`，一个是`..`。前者表示当前文件夹，后者表示上一级文件夹。这种相对路径的表示方法我想各位都不陌生。

3. 输出结果

```csharp
var watch = Stopwatch.StartNew();
count = await FindFilesWithWin32(path, count);
Debug.WriteLine("文件数量：" + count + ", 消耗时间：" + watch.ElapsedMilliseconds / 1000.0 + " s");
watch.Stop();
```

这里加了个计时器，用于测量递归遍历的耗时。

### 对照组（Storage）

作为对照，我们来看看在UWP中使用Storage相关的方法如何进行文件夹递归查询文件的：

```csharp
public async Task<int> FindFilesWithStorage(StorageFolder folder,int count)
{
    var items = await folder.GetItemsAsync();
    foreach (var item in items)
    {
        if (item is StorageFolder subFolder)
            count = await FindFilesWithStorage(subFolder, count);
        else if(item is StorageFile file)
        {
            ++count;
            Debug.WriteLine(file.Name);
        }
    }
    return count;
}
```

:::tip
真要查的时候可能还是会通过一些Query的方法，不过在Storage基础上的查询速度并没有提高多少
:::

说简单是真的很简单，不需要结构定义，不需要类型转换，就写个简单的递归调用即可。可是结果如何，我在文章开头也做过测试了，你可以自己尝试一下。

如果两者差距不大，我是真的喜欢后者这种简单的调用。可是差距实在太大，我没得选。

## 小结

坦白来说，在C#中使用C++定义的Win32 API确实挺麻烦的，但效果也确实是惊人。我非常希望微软这边能简化相关API的调用方法，或者用C#包一层，提供给开发者简单的调用方式。

本文通过调用Win32 API，极大地提高了UWP文件遍历的速度。但目前，这种方法只适用于Windows10 1803以上的桌面版。

至于说如何通过Win32 API读取写入文件，那就是下一篇博文的事情了。