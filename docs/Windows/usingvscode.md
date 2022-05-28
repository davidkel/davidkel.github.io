# Tips

## remote SSH

1. use git's ssh rather than inbuilt openssh as trying to get the file permissions right and directory permissions right is a nightmare. For the config file is better to store it outside of the windows homedir .ssh dir, eg use c:\.ssh

2. use wsl2 to copy/paste the key rather than vscode (it will add 0D0A rather than 0A to the key which means it can't be loaded.