# Some Windows Useful Commands

## List Services
It is possible to obtain the list of services running on the host by executing the following commands:

### Command Prompt

1. List all services<br />
```cmd
net start
```

2. List all services<br />
```cmd
sc queryex type=service state=all
```

3. List service names only<br />
```cmd
sc queryex type=service state=all | find /i "SERVICE_NAME:"
```

4. Search for specific service<br />
```cmd
sc queryex type=service state=all | find /i "SERVICE_NAME: myService"
```

5. SGet the status of a specific service<br />
```cmd
sc query myService
```

### Powershell

1. List all services<br />
```powershell
Get-Service
```

2. Search for specific service<br />
```powershell
Get-Service | Where-Object {$_.Name -like "*myService*"}
```

3. Get the status of a specific service<br />
```powershell
Get-Service myService
```

4. Get a list of the running services<br />
```powershell
Get-Service | Where-Object {$_.Status -eq "Running"}
```

5. Get a list of the running services<br />
```powershell
Get-Service | Where-Object {$_.Status -eq "Stopped"}
```

<br />

## Scheduled Task

1. Get list of all scheduled task
```powershell
Get-ScheduledTask | ForEach-Object {
    [pscustomobject]@{
        Name = $_.TaskName
        Path = $_.TaskPath
        LastResult = $(($_ | Get-ScheduledTaskInfo).LastTaskResult)
        NextRun = $(($_ | Get-ScheduledTaskInfo).NextRunTime)
        Status = $_.State
        Command = $_.Actions.execute
        Arguments = $_.Actions.Arguments
    }
} | Out-File -FilePath C:\Users\$env:UserName\Desktop\result.txt
```
