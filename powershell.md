# 常用命令

## 1.操作文件命令

### 创建文件夹

`New-Item -ItemType Directory -Name "NewFolder"（在当前路径下创建）```


`New-Item -ItemType Directory -Path "C:\Users\username\Documents\NewFolder"（指定全路径创建）`

### 删除文件夹

删除文件夹：使用 `Remove-Item` 命令，使用 `-Path` 参数指定要删除的文件夹的路径和名称，并使用 `-Recurse` 参数来指定删除整个目录树。例如：

`Remove-Item -Path C:\Temp\example_folder -Recurse`

删除文件夹：使用 `Remove-Item` 命令，同样使用通配符 `*` 指定文件夹名中的可变部分，并使用 `-Recurse` 参数来指定删除整个目录树，例如：

`Remove-Item -Path .\*example* -Recurse`

### 创建文件

`New-Item -ItemType File -Name "NewFile.txt" （在当前路径下创建）`

`New-Item -ItemType File -Path "C:\Users\username\Documents\NewFile.txt"  （指定全路径创建）`

### 删除文件

删除文件：使用 `Remove-Item` 命令，可以使用 `-Path` 参数指定要删除的文件的路径和文件名，例如：

`Remove-Item -Path C:\Temp\example.txt`

删除文件：使用 `Remove-Item` 命令，可以使用 `-Path` 参数指定要删除的文件的路径和文件名，例如：

`Remove-Item -Path .\*example*.txt`

```powershell
New-Item -ItemType File -Path "C:\Users\username\Documents\NewFile.txt
```