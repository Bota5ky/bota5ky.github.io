[How do you set PowerShell's default directory?](https://stackoverflow.com/questions/32069265/how-do-you-set-powershells-default-directory)

- 删除文件
```powershell
Remove-Item -Path "*" -Include "*.txt" -Recurse -Force
```
- 获取文件个数
```powershell
Get-ChildItem <path> -Include *.dump -Recurse | select Name | Measure-Object
```
- 删除Sample开头的sqlserver数据库
```powershell
$Databases = Invoke-SQLcmd -ServerInstance $ServerInstance -Query ("SELECT * FROM sys.databases WHERE NAME LIKE 'Sample%'")
ForEach ($Database in $Databases){Invoke-SQLcmd -ServerInstance $ServerInstance -Query ("DROP DATABASE [" + $Database.Name + "]")
"$($Database.Name) is deleted."}
```
- dump文件转postgres
```powershell
psql -U postgres -d <databaseName> -f 'C:\xxxxxx.dump'
```
- 删除Sample开头的postgres数据库
```powershell
psql -U postgres -t -A -c "select datname from pg_database where datname ~ 'Sample\w*'" | ForEach-Object { dropdb --force --echo --username postgres "$_" }
```