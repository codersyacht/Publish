# Homebrew

### **Uninstallation (Optional)**

If you already have installed homebrew on your machine and wish to uninstall, then follow the steps below.

1. Be cautios about software packages installed via homebrew. Packages installed via homebrew can be found in the following directory, _/usr/local/Cellar_ (intel) and _/opt/homebrew/Cellar_ (ARM).
2. To uninstall the software packages installed via homebrew, execute the following commands:

Unscoped packages:

```CMD
brew uninstall -g <package_name>
```
example:

```CMD
brew uninstall -g node
```

Scoped packages:

```CMD
brew uninstall -g <@scope/package_name>
```

example:

```CMD
brew uninstall -g @angular/cli
```

3. To uninstall homebrew, execute the following command:
```CMD
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/uninstall.sh)"
```

4. You may delete the following directories manually, _/usr/local/Cellar_, _/usr/local/Caskroom_, _/usr/local/Homebrew_ (intel) and _opt/homebrew_ (ARM) . I will explain the significance of these folders later.

### **Prerequisites (Recommended)**

If you have Git on your machine,ensure that global remote.origin.url in unset. To check the configuration execute the following command:
```CMD
git config --list
```
A different remote.origin.url setting might throw the following error:

<img width="1120" alt="image" src="https://user-images.githubusercontent.com/128015499/225806964-82d97ff5-0d95-4a52-be4a-ed22662ec105.png">

You may either replace the remote.origin.url to _Homebrew/brew_ using the following command:

```CMD
git config --global --replace-all remote.origin.url "https://github.com/Homebrew/brew.git"
```
or unset remote.origin.url using the following command:

```CMD
git config --global --unset remote.origin.url
```

### **Installation of Homebrew**

1. Run the following command to install Homebrew:

```CMD
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

The installation process will prompt for sudo access. If installation is successful, the following message will be seen:

<img width="1120" alt="image" src="https://user-images.githubusercontent.com/128015499/225805404-18f2dcef-78b4-428b-b32f-a12774dc5397.png">

2. In ARM based system, the Homebrew's bin directory will not be in the classpath. The installation summary will provide addition steps to add the bin directory to path, so that brew command can be used from anywhere.

### **The FileSystems**

**Intel**
1. Homebrew will be installed in the _/usr/local/Homebrew_ directory. This is a local git repository of Homebrew. Navigate to the directory and execute _git config --list_. You will see the _remote.origin.url_ set to _https://github.com/Homebrew/brew_. You can find more information in _/usr/local/Homebrew/.git_.
2. A new directory named _/usr/local/Cellar_ will be created. This directory will contain all the software packages installed via Homebrew.
3. A new directory named _/usr/local/Caskroom_ will be created. This directory will contain GUI applications.
4. Few more supporting directories will be created under _/usr/local/_.
6. The _brew_ command can be found under _/usr/local/bin_. This is symbolic link to /_usr/local/Homebrew/bin/brew_.
7. Homebrew may also add symbolic links to packages that it installs to _/usr/local/bin_.

**ARM**

The entire Homebrew package including Cellar, Caskroom and other supporting directories will be create under /opt/homebrew.

### **Installing software packages**

```CMD
brew install telnet
```



**Mohamed Jawahar Hussain**
