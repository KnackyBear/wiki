# Use Native OpenSSH over GPG (Smartcard) on Windows 10

Download and install [GPG4Win](https://www.gpg4win.org/).

Plug your smartcard, open Kleopatra then go to smartcard tab. You see your smartcard informations (eventually you have to refresh the page with F5).

[Kleopatra smartcard tab](assets/kleopatra_smartcard_tab.png)

Configuration > Configure Kleopatra > GnuPG System > Activate SSH Support and Putty Support :

[Kleopatra configuration](assets/kleopatra_smartcard_tab.png)

``%HOME%\AppData\Roaming\gnupg\gpg-agent.conf`` should be :
```

###+++--- GPGConf ---+++###
enable-ssh-support
enable-putty-support
###+++--- GPGConf ---+++### 28/07/2021 21:44:55 Paris, Madrid (heure d��t�)
# GPGConf edited this configuration file.
# It will disable options before this marked block, but it will
# never change anything below these lines.
```

Check in Windows Console that your key is here :
```
gpg --card-status
```

If something goes wrong, try to reload your gpg agent :
```
gpg-connect-agent killagent /bye
gpg-connect-agent /bye
```

Download [Pageant](https://github.com/benpye/wsl-ssh-pageant)

Put it your PATH environment variables.

In a powershell with Administrator right :
```
$job = Register-ScheduledJob -Name "GpgAgent" -ScriptBlock { 
		gpg-connect-agent.exe /bye 
		wsl-ssh-pageant.exe --systray --winssh ssh-pageant
	} -Trigger (New-JobTrigger -AtLogOn -User $([System.Security.Principal.WindowsIdentity]::GetCurrent().Name)) -ScheduledJobOption (New-ScheduledJobOption -StartIfOnBattery -ContinueIfGoingOnBattery) -RunNow
```

Then, change principal to run only on interactive logon instead of S4A.
```
$principal = New-ScheduledTaskPrincipal -LogonType Interactive -UserId $([System.Security.Principal.WindowsIdentity]::GetCurrent().Name)
Set-ScheduledTask -TaskPath \Microsoft\Windows\PowerShell\ScheduledJobs\ -TaskName $job.Name -Principal $principal
```

Check that your SSH agent is fine :
```
ssh-add -L
```

You should see your smartcard.

## Resources

  * [Windows 10 using gpg for ssh authentication](https://www.kaylyn.ink/journal/windows-10-using-gpg-for-ssh-authentication.-win32-openssh-edition./)
  * [WSL-SSH-Pageant github's project](https://github.com/benpye/wsl-ssh-pageant)