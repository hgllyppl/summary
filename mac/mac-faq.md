# FAQ
0. 命令别名

        // 用户目录下建立".bash_profile"，输入以下内容
        alias ll='ls -lhF’
        alias la='ls -lhAF’
        source .bash_profile
0. 删除隔离保护
        
        // 查看和删除隔离保护
        xattr -l xxx.sh
        xattr -d com.apple.quarantine xxx.sh
0. 识别NTFS

        NTFS -> extFat
0. missing xcrun

        xcode-select --install
        sudo xcode-select -switch /
0. Rootless

        // 重启并按住Command+R，执行以下命令
        csrutil enable/disable
0. ping 不通本机
<br>系统偏好设置 -> 安全性与隐私 -> 防火墙 -> 防火墙选项 -> 关闭隐身模式