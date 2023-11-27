# Windows Terminal 美化

对于没有安装 Windows Terminal 终端的小伙伴，可以在 Microsoft Store 中进行下载安装。<br />![Windows Terminal.gif](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202308211514971.gif)

[Home | Oh My Posh](https://ohmyposh.dev/)

1. 使用**管理员身份**打开 Windows Terminal 终端，选择运行 PowerShell；
2. 使用 `winget install JanDeDobbeleer.OhMyPosh -s winget` 命令安装 `oh-my-posh`；<br />![image-20230821153758290](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202308211537335.png)
3. 安装 [Nerd Fonts](https://www.nerdfonts.com/) 字体，官方推荐安装 [Meslo LGM NF](https://github.com/ryanoasis/nerd-fonts/releases/download/v3.0.2/Meslo.zip) 字体，不过可以根据自己的喜爱选择其他的字体，如 [DejaVuSansMono](https://github.com/ryanoasis/nerd-fonts/releases/download/v3.0.2/DejaVuSansMono.zip) 字体！！！将下载好的压缩包进行解压缩，然后全部选中进行安装即可！<br />![image-20230821155225568](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202308211552638.png)
4. 配置 Windows Terminal 中的 PowerShell 使用刚才安装的 `DejaVuSansMono` 字体；

   ```json
   "profiles": 
   {
       "defaults": 
       {
           "backgroundImage": "C:\\Users\\liulei\\Pictures\\Camera Roll\\1309265.jpg",
           "backgroundImageOpacity": 0.2,
           "experimental.retroTerminalEffect": false,
           "opacity": 80,
           "useAcrylic": true
       },
       "list": 
       [
           {
               "altGrAliasing": true,
               "antialiasingMode": "grayscale",
               "closeOnExit": "automatic",
               "colorScheme": "Campbell",
               "commandline": "%SystemRoot%\\System32\\WindowsPowerShell\\v1.0\\powershell.exe",
               "cursorShape": "bar",
               "font": 
               {
                   "face": "DejaVuSansMono Nerd Font Mono",
                   "size": 12.0
               },
               "guid": "{a54ddbc3-c7e4-4062-8121-e1442466fc31}",
               "hidden": false,
               "historySize": 9001,
               "icon": "ms-appx:///ProfileIcons/{61c54bbd-c2c6-5271-96e7-009a87ff44bf}.png",
               "name": "Windows PowerShell",
               "padding": "8, 8, 8, 8",
               "snapOnInput": true,
               "startingDirectory": "%USERPROFILE%"
           },
           {
               "colorScheme": "One Half Dark",
               "commandline": "%SystemRoot%\\System32\\cmd.exe",
               "elevate": true,
               "experimental.retroTerminalEffect": false,
               "guid": "{0caa0dad-35be-5f56-a8ff-afceeeaa6101}",
               "hidden": false,
               "name": "\u547d\u4ee4\u63d0\u793a\u7b26"
           },
           {
               "experimental.retroTerminalEffect": false,
               "guid": "{b453ae62-4e3d-5e58-b989-0a998ec441b8}",
               "hidden": false,
               "name": "Azure Cloud Shell",
               "source": "Windows.Terminal.Azure"
           }
       ]
   },
   ```

   ![image-20230821155741781](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202308211557844.png)

5. 配置 Windows Terminal 中的 PowerShell 应用 `oh-my-posh`；

   如果你不知道自己目前使用的是哪个 shell，可以使用 `oh-my-posh get shell` 命令进行查看，如下所示： <br />![image-20230821160214521](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202308211602613.png)

   使用 `notepad $PROFILE` 命令编辑 PowerShell 配置文件脚本

   > [!ATTENTION]
   >
   > 当上面的命令出现错误时，请确保首先使用 `New-Item -Path $PROFILE -Type File -Force` 命令创建配置文件；
   >
   > 在这种情况下，PowerShell 也可能阻止运行本地脚本。要解决此问题，请将 PowerShell 设置为仅要求使用 `set-ExecutionPolicy-RemoteSigned` 命令对远程脚本进行签名，或对配置文件进行签名。

   然后在配置文件中添加以下内容：`oh-my-posh init pwsh --config "$env:POSH_THEMES_PATH/amro.omp.json" | Invoke-Expression`，其中的 `amro` 为选择的主题，可以查看 [Themes | Oh My Posh](https://ohmyposh.dev/docs/themes) 总共有哪些主题，根据自己的喜爱进行更换，最后使用 `. $PROFILE` 命令使配置生效！<br />![image-20230821162335758](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202308211623928.png)

至此，Windows Terminal 终端使用 oh-my-posh 美化就圆满完成啦！🎉🎉🎉
