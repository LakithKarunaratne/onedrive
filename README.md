# OneDrive Free Client
###### A complete tool to interact with OneDrive on Linux. Built following the UNIX philosophy.

### Features:
* State caching
* Real-Time file monitoring with Inotify
* Resumable uploads
* Support OneDrive for Business (part of Office 365)
* Shared folders (not Business)

### What's missing:
* While local changes are uploaded right away, remote changes are delayed
* No GUI

## Setup

### Dependencies
* [libcurl](http://curl.haxx.se/libcurl/)
* [SQLite 3](https://www.sqlite.org/)
* [Digital Mars D Compiler (DMD)](http://dlang.org/download.html)

### Dependencies: Ubuntu/Debian
```sh
sudo apt install libcurl4-openssl-dev
sudo apt install libsqlite3-dev

# Ubuntu 18
sudo snap install --classic dmd && sudo snap install --classic dub

# Ubuntu 17
sudo wget http://master.dl.sourceforge.net/project/d-apt/files/d-apt.list -O /etc/apt/sources.list.d/d-apt.list
sudo apt-get update && sudo apt-get -y --allow-unauthenticated install --reinstall d-apt-keyring
sudo apt-get update && sudo apt-get install dmd-compiler dub
```

### Dependencies: Fedora/CentOS
```sh
sudo yum install libcurl-devel
sudo yum install sqlite-devel
curl -fsS https://dlang.org/install.sh | bash -s dmd
```

### Dependencies: Arch Linux
```sh
sudo pacman -S curl sqlite dlang
```

### Installation
```sh
git clone https://github.com/skilion/onedrive.git
cd onedrive
make
sudo make install
```

Using a different compiler (for example [LDC](https://wiki.dlang.org/LDC)):
```sh
make DC=ldmd2
```

### First run :zap:
After installing the application you must run it at least once from the terminal to authorize it.

You will be asked to open a specific link using your web browser where you will have to login into your Microsoft Account and give the application the permission to access your files. After giving the permission, you will be redirected to a blank page. Copy the URI of the blank page into the application.

### Uninstall
```sh
sudo make uninstall
# delete the application state
rm -rf .config/onedrive
```

## Configuration
Configuration is optional. By default all files are downloaded in `~/OneDrive` and only hidden files are skipped.
If you want to change the defaults, you can copy and edit the included config file into your `~/.config/onedrive` directory:
```sh
mkdir -p ~/.config/onedrive
cp ./config ~/.config/onedrive/config
nano ~/.config/onedrive/config
```

Available options:
* `sync_dir`: directory where the files will be synced
* `skip_file`: any files or directories that match this pattern will be skipped during sync.

Patterns are case insensitive. `*` and `?` [wildcards characters](https://technet.microsoft.com/en-us/library/bb490639.aspx) are supported. Use `|` to separate multiple patterns.

Note: after changing `skip_file`, you must perform a full synchronization by executing `onedrive --resync`

### Selective sync
Selective sync allows you to sync only specific files and directories.
To enable selective sync create a file named `sync_list` in `~/.config/onedrive`.
Each line of the file represents a relative path from your `sync_dir`. All files and directories not matching any line of the file will be skipped during all operations.
Here is an example of `sync_list`:
```text
Backup
Documents/latest_report.docx
Work/ProjectX
notes.txt
```
Note: after changing the sync list, you must perform a full synchronization by executing `onedrive --resync`

### Shared folders
Folders shared with you can be synced by adding them to your OneDrive. To do that open your Onedrive, go to the Shared files list, right click on the folder you want to sync and then click on "Add to my OneDrive".

### OneDrive service
If you want to sync your files automatically, enable and start the systemd service:
```sh
systemctl --user enable onedrive
systemctl --user start onedrive
```

To see the logs run:
```sh
journalctl --user-unit onedrive -f
```

Note: systemd is supported on Ubuntu only starting from version 15.04

## Using multiple accounts
First follow the basic installation steps, do not run. 

You can run multiple instances of the application by specifying a different config directory in  `~/.config` inorder to handle multiple OneDrive accounts. You need to create multiple folders in `~/.config` with the respective drive names prefereably in camelCase.

You need to ensure config file is placed within the folder

For exaple `~/.config/{onedriveAccountName}` can be setup as

```
# Directory where the files will be synced
sync_dir = "~/{oneDriveAccountName}"

# Skip files and directories that match this pattern
skip_file = ".*|~*"
```

### Create System Services

Go to path `/usr/lib/systemd/user` create .service file for each account

For example `{onedriveAccountName}.service` should contain

```
[Unit]
Description=OneDrive Free Client
Documentation=https://github.com/skilion/onedrive

[Service]
ExecStart=/usr/local/bin/onedrive -m --confdir="~/.config/{onedriveAccountName}"
Restart=no

[Install]
WantedBy=default.target
```
Now run the below command first to configure the client, once few files are downloaded hit `Ctrl+C` to exit. 
```
onedrive --monitor --confdir="~/.config/{onedriveAccountName}"
```
If you have multiple accounts that needs to be auto synced run this each time in the terminal to setup the client
```
onedrive --monitor --confdir="~/.config/{onedriveAccountName_1}"
```
#### Help: 
`--confdir` parameter sets the path to the configuration.

`--monitor` keeps the application running and monitoring for changes

If you'd like you can put `&` at the end of the command so it leaves the application in background and returns to the terminal interactive.

Enable OneDrive Service for the account to sync your files automatically, enable and start the systemd service:
```sh
systemctl --user enable {onedriveAccountName}
systemctl --user start {onedriveAccountName}
```
Use HTOP to check if the task is running in the background

Use `systemctl --user status {onedriveAccountName}` to check any errors if the program isn't running. 

## Extra

### All available commands:
```text
Usage: onedrive [OPTION]...

no option        Sync and exit
       --confdir Set the directory used to store the configuration files
-d    --download Only download remote changes
        --logout Logout the current user
-m     --monitor Keep monitoring for local and remote changes
   --print-token Print the access token, useful for debugging
        --resync Forget the last saved state, perform a full sync
       --syncdir Set the directory used to sync the files that are synced
-v     --verbose Print more details, useful for debugging
       --version Print the version and exit
-h        --help This help information.
```

### File naming
The files and directories in the synchronization directory must follow the [Windows naming conventions](https://msdn.microsoft.com/en-us/library/aa365247).
The application will crash for example if you have two files with the same name but different case. This is expected behavior and won't be fixed.

### Multi Account Sync Issues
If the sync isn't starting 
- Check file names
- Path Names
- Reference Paths
- To avoid errors do not use SPACES between words, use camelCase for naming configs and service files.
