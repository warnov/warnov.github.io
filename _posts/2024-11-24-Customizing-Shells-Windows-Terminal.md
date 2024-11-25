---
title: Customize Your Command Line Experience with Oh My Posh and lsd
date: 2024-11-24 22:53:00 -0500
categories: [Windows, Developer]
tags: [windows, terminal, shell, ohmyposh, lsd, powershell, cmd, bash, azure cloud shell] 
image:
  path: /assets/img/posts/2024-11-24-Customizing-Shells-Windows-Terminal.png
---

# Customize Your Command Line Experience with Oh My Posh and lsd

If you've ever wanted to make your command line experience more vibrant, informative, and productive, this guide is for you. Today, we'll explore how to install and configure **Oh My Posh** and **lsd** to enhance different command line environments—all through **Windows Terminal**. Whether you're using **CMD**, **PowerShell**, **WSL Bash**, or **Azure Cloud Shell**, we'll help you achieve a cohesive, visually appealing terminal setup. Let's dive in!

## Why Customize Your Command Line?
A personalized command line can greatly improve your productivity. With the right tools, you can add color, icons, and useful information to your prompt, making navigation easier and giving you a sense of satisfaction every time you open the terminal. **Oh My Posh** offers a collection of stylish themes that bring your terminal to life, while **lsd** is an enhanced version of the `ls` command that adds icons and colors for easy file differentiation.

### Tools We Will Use
- **Oh My Posh**: A prompt theme engine that makes your terminal beautiful.
- **lsd**: A modern replacement for `ls`, with support for icons and colors.
- **Windows Terminal**: A versatile terminal application that allows us to use multiple shells in one interface.
- **Clink** (for CMD): A tool that enhances CMD to enable better prompts and integration with Oh My Posh.

## Setting Up CMD

**CMD** has always been functional but lacks the advanced capabilities you see in other shells. **Oh My Posh** and **Clink** bring CMD into the modern age with a colorful prompt and useful extras.

1. **Install Clink**: Start by installing Clink, which is required to enable Oh My Posh in CMD.
   - Visit the [Clink GitHub page](https://github.com/chrisant996/clink/releases) to download and install the latest release.
   - Clink significantly enhances CMD, allowing us to integrate advanced prompts.

2. **Add Oh My Posh with a Lua Script**:
   - Create a new file called `oh-my-posh.lua` in the Clink scripts directory. You can find this location by running `clink info` inside CMD.
   - Add the following line to the Lua file:
     ```lua
     load(io.popen('oh-my-posh init cmd'):read("*a"))()
     ```
   - This command integrates Oh My Posh into CMD and allows you to customize your prompt.

3. **Download the Oh My Posh Executable**:
   - Download Oh My Posh by running the following command in PowerShell or CMD:
     ```sh
     wget https://github.com/JanDeDobbeleer/oh-my-posh/releases/latest/download/posh-windows-amd64.exe -O %USERPROFILE%\oh-my-posh.exe
     ```
   - Place the executable somewhere accessible in your PATH.

4. **Restart CMD**: After creating the Lua script and adding the Oh My Posh executable, restart CMD to apply the new prompt.

5. **Install `lsd` for CMD**:
   - Download the `lsd` executable by using:
     ```sh
     wget https://github.com/Peltoche/lsd/releases/download/0.23.1/lsd_0.23.1_windows_amd64.zip -O %USERPROFILE%\lsd.zip
     ```
   - Extract the files and place them in a directory included in your PATH.
   - Create a batch file (`ls.bat`) that allows you to use `lsd` in CMD:
     ```cmd
     @echo off
     %USERPROFILE%\lsd\lsd.exe %*
     ```

## Setting Up PowerShell
**PowerShell** is already more advanced than CMD, but that doesn’t mean it can’t be improved further.

1. **Install Oh My Posh**:
   - Run the following command to install Oh My Posh:
     ```powershell
     Install-Module oh-my-posh -Scope CurrentUser -AllowPrerelease -Force
     ```
   - Add Oh My Posh to your PowerShell profile by editing the profile file:
     ```powershell
     notepad $PROFILE
     ```
   - Add the following to the profile:
     ```powershell
     Import-Module oh-my-posh
     Set-PoshPrompt -Theme powerlevel10k_classic
     ```

2. **Install `lsd` for PowerShell**:
   - Use **scoop** to install `lsd` conveniently:
     ```powershell
     iwr -useb get.scoop.sh | iex
     scoop install lsd
     ```
   - Add an alias for `ls` to use `lsd` instead:
     ```powershell
     Set-Alias ls lsd
     ```

## Setting Up WSL Bash
**WSL** (Windows Subsystem for Linux) brings the full power of a Linux environment to Windows. Here’s how to enhance WSL Bash with Oh My Posh and `lsd`:

1. **Install Oh My Posh** for Bash:
   - Download Oh My Posh:
     ```sh
     wget https://github.com/JanDeDobbeleer/oh-my-posh/releases/latest/download/posh-linux-amd64 -O ~/.local/bin/oh-my-posh
     chmod +x ~/.local/bin/oh-my-posh
     ```
   - Edit `.bashrc` to add Oh My Posh:
     ```sh
     nano ~/.bashrc
     ```
     Add the following:
     ```sh
     eval "$(oh-my-posh init bash --config ~/.poshthemes/powerlevel10k_classic.omp.json)"
     ```

2. **Install `lsd` for WSL**:
   - Download the `.deb` package for `lsd`:
     ```sh
     wget https://github.com/Peltoche/lsd/releases/download/0.23.1/lsd_0.23.1_amd64.deb
     ar x lsd_0.23.1_amd64.deb
     tar -xvf data.tar.zst
     mv usr/bin/lsd ~/.local/bin/
     ```
   - Update your `.bashrc` to add an alias for `ls` to use `lsd`.

## Setting Up Azure Cloud Shell
Azure Cloud Shell is a browser-based terminal that gives you access to a Linux environment. Here’s how to set up Oh My Posh and `lsd` there:

1. **Install Oh My Posh** for Azure Cloud Shell:
   - Download Oh My Posh:
     ```sh
     wget https://github.com/JanDeDobbeleer/oh-my-posh/releases/latest/download/posh-linux-amd64 -O ~/oh-my-posh
     chmod +x ~/oh-my-posh
     ```
   - Edit `.bashrc` to add Oh My Posh:
     ```sh
     nano ~/.bashrc
     ```
     Add the following line:
     ```sh
     eval "$(~/oh-my-posh init bash --config ~/powerlevel10k_classic.omp.json)"
     ```

2. **Install `lsd` in Azure Cloud Shell**:
   - Since you don’t have `sudo` in Cloud Shell, you’ll need to manually extract `lsd`:
     ```sh
     wget https://github.com/Peltoche/lsd/releases/download/0.23.1/lsd_0.23.1_amd64.deb
     ar x lsd_0.23.1_amd64.deb
     unzstd data.tar.zst
     tar -xvf data.tar
     mv usr/bin/lsd ~/.local/bin/
     ```

## Adding the Final Touch: Nerd Fonts
To fully enjoy **Oh My Posh** and **lsd**, you’ll need to install a **Nerd Font** that can display icons properly:

1. **Install Nerd Fonts**:
   - Visit [Nerd Fonts](https://www.nerdfonts.com/font-downloads) and download a font like **Cascadia Code PL** or **Fira Code**.
   - Set the **font face** in **Windows Terminal** to the Nerd Font you installed, ensuring icons display correctly in CMD, PowerShell, WSL, and Azure Cloud Shell.

## Conclusion
Customizing your command line can be transformative. With **Oh My Posh**, you get a sleek, informative prompt across all your shells. Meanwhile, **lsd** gives your file navigation a visual boost with colors and icons. Whether you’re using CMD, PowerShell, WSL Bash, or Azure Cloud Shell, these tools will help you build an attractive, functional terminal environment.

Try these customizations today, and make every terminal session a joy to work in!

