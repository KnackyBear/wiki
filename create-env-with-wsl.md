# Create your development environment in Windows using WSL2

- [Create your development environment in Windows using WSL2](#create-your-development-environment-in-windows-using-wsl2)
  - [Resources](#resources)
  - [Install WSL 2](#install-wsl-2)
  - [OpenSSH Installation](#openssh-installation)
  - [Install GPG4Win](#install-gpg4win)
  - [Access your Yubikey in WSL2](#access-your-yubikey-in-wsl2)
    - [Prerequisites](#prerequisites)
    - [Sync socket](#sync-socket)
    - [Import GPG Key to WSL2](#import-gpg-key-to-wsl2)
    - [Using Yubikey over SSH](#using-yubikey-over-ssh)
  - [Configure git & github with your Yubikey](#configure-git--github-with-your-yubikey)
  - [Configure Visual Studio Code with WSL2](#configure-visual-studio-code-with-wsl2)
  - [Install docker for windows with WSL2 Backend](#install-docker-for-windows-with-wsl2-backend)
  - [Install python with pyenv](#install-python-with-pyenv)
  - [Install npm with nvm](#install-npm-with-nvm)

## Resources

  * [Official Microsoft's documentation to install WSL2](https://docs.microsoft.com/fr-fr/windows/wsl/install-win10)
  * [Official Microsoft's documentation to install OpenSSH Client & Server](https://docs.microsoft.com/en-us/windows-server/administration/openssh/openssh_install_firstuse)
  * [The ultimate guide to yubikey on WSL2 by Jaroslav Živný](https://dev.to/dzerycz/the-ultimate-guide-to-yubikey-on-wsl2-part-2-kli)
  * [WSL2-ssh-pageant github's repository by BlackReloaded](https://github.com/BlackReloaded/wsl2-ssh-pageant)
  * [Docker desktop WSL2 Backend](https://docs.docker.com/docker-for-windows/wsl/)
  * [Pyenv documentation](https://github.com/pyenv/pyenv)
  * [Nvm github's repository](https://github.com/nvm-sh/nvm)

## Install WSL 2

In a privileged powershell console :

Activate Windows Subsytem Linux
```
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
```

Activate Virtual Machine Platform
```
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
```
Then, reboot.

Download and install [Linux code upgrade](https://wslstorestorage.blob.core.windows.net/wslblob/wsl_update_x64.msi)

Define WSL2 as default version
```
wsl --set-default-version 2
```

Finally, install your Linux distribution using Microsoft Store.

## OpenSSH Installation

To install OpenSSH using PowerShell, run PowerShell as an Administrator. To make sure that OpenSSH is available, run the following cmdlet:
```
Get-WindowsCapability -Online | ? Name -like 'OpenSSH*'
```
This should return the following output if neither are already installed:
```
Name  : OpenSSH.Client~~~~0.0.1.0
State : NotPresent

Name  : OpenSSH.Server~~~~0.0.1.0
State : NotPresent
```
If ``OpenSSH.Client`` is already installed, go to the next chapter, else install the client components as needed:
```
Add-WindowsCapability -Online -Name OpenSSH.Client~~~~0.0.1.0
```

## Install GPG4Win

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

## Access your Yubikey in WSL2

### Prerequisites

Install socat and wsl2-ssh-pageant in WSL:
```
sudo apt install socat
mkdir ~/.ssh
wget https://github.com/BlackReloaded/wsl2-ssh-pageant/releases/download/v1.2.0/wsl2-ssh-pageant.exe -O ~/.ssh/wsl2-ssh-pageant.exe
chmod +x ~/.ssh/wsl2-ssh-pageant.exe
```

### Sync socket

Edit your ~/.bashrc (e.g. via nano or vim) and add following content:
```
# SSH Socket
# Removing Linux SSH socket and replacing it by link to wsl2-ssh-pageant socket
export SSH_AUTH_SOCK=$HOME/.ssh/agent.sock 
ss -a | grep -q $SSH_AUTH_SOCK 
if [ $? -ne 0 ]; then
  rm -f $SSH_AUTH_SOCK
  setsid nohup socat UNIX-LISTEN:$SSH_AUTH_SOCK,fork EXEC:$HOME/.ssh/wsl2-ssh-pageant.exe &>/dev/null &
fi
# GPG Socket
# Removing Linux GPG Agent socket and replacing it by link to wsl2-ssh-pageant GPG socket
export GPG_AGENT_SOCK=$HOME/.gnupg/S.gpg-agent 
ss -a | grep -q $GPG_AGENT_SOCK 
if [ $? -ne 0 ]; then
  rm -rf $GPG_AGENT_SOCK
  setsid nohup socat UNIX-LISTEN:$GPG_AGENT_SOCK,fork EXEC:"$HOME/.ssh/wsl2-ssh-pageant.exe --gpg S.gpg-agent" &>/dev/null &
fi
```

Restart WSL
```
wsl.exe --shutdown
```

### Import GPG Key to WSL2

If you check GPG keys available in WSL2 via ``gpg --list-keys`` or ``gpg --list-secret-keys`` and you get empty results, you have to first import them. It’s quite easy just run:
```
gpg --card-edit
```

Get your public key (url should be shown on gpg command output) via ``wget`` or ``curl``, then import it :
```
gpg --import PATH_TO_ASC_FILE
```

Now run ``gpg --list-keys`` you finally get your keys.

Now we are missing one small step. As you can see. The trustworthiness of our certificate is unknown (information next to the name). We can change it via running:
```
gpg --edit-key YOUR_KEY_ID # Last 8 chars of your pubkey's id
```

This opens gpg console insterface. Write:
```
trust # Change trust level
5     # Set trust level to ultimate
save  # Save the changes
```
If you list keys via ``gpg --list-keys`` now. You should be able to see ``[ultimate]`` next to your name.

### Using Yubikey over SSH

```
ssh-add -L 
```
You should see your auth key here. 

If something wrong, try to reload your gpg-agent :
```
# WSL2
$ exit
```

```
# Windows
wsl --shutdown
gpg-connect-agent killagent /bye
gpg-connect-agent /bye
wsl
```
Or check if ``%HOME%\.ssh\`` exists. If not, create it, then retry to ``ssh-add -L`` in WSL2.

## Configure git & github with your Yubikey

Create or update ``~\.gitconfig`` file with :
```
[commit]
	gpgsign = true
```

Link your GPG/SSH Keys to your github account :
[SSH Configuration](assets/git_gpg.png)

## Configure Visual Studio Code with WSL2

Download and install [Visual studio code](https://code.visualstudio.com/)
Download and install ``Remote Development`` module.

That's all, Enjoy :D

## Install docker for windows with WSL2 Backend

Download and install [Docker desktop for Windows](https://hub.docker.com/editions/community/docker-ce-desktop-windows/).

On configuration window, check ``Install required Windows components for WSL 2``

## Install python with pyenv

Download and install using ``pyenv-installer``
```
curl -L https://github.com/pyenv/pyenv-installer/raw/master/bin/pyenv-installer | bash
```

Add to your ``.bashrc``
```
export PYENV_ROOT="$HOME/.pyenv"
export PATH="$PYENV_ROOT/bin:$PATH"

# Pyenv initialization
if command -v pyenv 1>/dev/null 2>&1; then
  eval "$(pyenv init --path)"
fi
```

Install pyenv dependencies
```
sudo apt-get install --yes libssl-dev zlib1g-dev libbz2-dev libreadline-dev libsqlite3-dev llvm libncurses5-dev libncursesw5-dev xz-utils tk-dev libgdbm-dev lzma lzma-dev tcl-dev libxml2-dev libxmlsec1-dev libffi-dev liblzma-dev wget curl make build-essential python-openssl 
```

Reload your shell
```
# WSL2
exec $SHELL
```

You can now use pyenv to install multi version of python
```
# WSL2
# List versions
pyenv install -l

# Install version
pyenv install <version>
```

## Install npm with nvm

```
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.38.0/install.sh | bash
```

You can now use nvm to install mutl version of npm
```
# List version of npm (lts version)
nvm ls-remote --lts

# Install version of npm
nvm install lts/fermium

# Check current version of npm
nvm current

# To use a different version of npm (update PATH)
nvm use <version>
```