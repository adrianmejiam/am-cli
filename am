#!/data/data/com.termux/files/usr/bin/bash

# 定义颜色常量
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[0;33m'
CYAN='\033[0;36m'
NC='\033[0m' # 恢复默认颜色

# 配置文件参数
CONFIG_FILE="$HOME/.am_downloader.conf"
REPO_DIR="apple-music-alac-atmos-downloader"
DEFAULT_ALAC_PATH="/storage/emulated/0/Music/AM/AM-DL"
DEFAULT_ATMOS_PATH="/storage/emulated/0/Music/AM/AM-DL-Atmos"

# 依赖配置
REQUIRED_PKGS=(git golang gpac ffmpeg yq)
declare -A PKG_DESC=(
    [git]="版本控制工具"
    [golang]="Go语言环境" 
    [gpac]="MP4多媒体处理"
    [ffmpeg]="音视频转码工具"
    [yq]="YAML配置处理器"
)

# 初始化配置
init_config() {
    if [ ! -f "$CONFIG_FILE" ]; then
        cat > "$CONFIG_FILE" <<EOF
# Apple Music 下载配置
ALAC_SAVE_FOLDER="$DEFAULT_ALAC_PATH"
ATMOS_SAVE_FOLDER="$DEFAULT_ATMOS_PATH"
EOF
    fi
    source "$CONFIG_FILE"
}

# 增强存储权限检查
check_storage() {
    while [ ! -d "$HOME/storage/shared" ]; do
        clear
        echo -e "${YELLOW}[!] 必须授予存储权限才能继续！${NC}"
        echo -e "${CYAN}请按以下步骤操作："
        echo -e "1. 在Termux中执行以下命令："
        echo -e "   ${YELLOW}termux-setup-storage${NC}"
        echo -e "2. 在系统弹窗中点击 [允许]"
        echo -e "3. 按回车键继续...${NC}"
        read -p ""
        
        termux-setup-storage
        sleep 3
        
        if [ ! -d "$HOME/storage/shared" ]; then
            echo -e "\n${RED}[!] 授权未完成，请检查："
            echo -e "1. 已正确执行 termux-setup-storage"
            echo -e "2. Termux已获得存储权限"
            echo -e "3. 重新启动Termux后重试${NC}"
            read -p "按回车键重试..."
        fi
    done
}

# 依赖安装检查
install_dependencies() {
    echo -e "\n${YELLOW}[+] 检查系统依赖...${NC}"
    for pkg in "${REQUIRED_PKGS[@]}"; do
        if ! dpkg -s $pkg &>/dev/null; then
            echo -e "${CYAN}正在安装 ${PKG_DESC[$pkg]}...${NC}"
            pkg install -y $pkg || {
                echo -e "${RED}[!] $pkg 安装失败! 请手动执行："
                echo -e "pkg install $pkg${NC}"
                exit 1
            }
        fi
    done
}

# 仓库管理
manage_repo() {
    REPO_URL="https://github.com/zhaarey/apple-music-alac-atmos-downloader.git"

    if [ ! -d "$REPO_DIR" ]; then
        echo -e "${YELLOW}[+] 克隆仓库...${NC}"
        git clone --depth=1 "$REPO_URL" "$REPO_DIR" || {
            echo -e "${RED}[!] 仓库克隆失败! 请检查网络或 Git 配置${NC}"
            exit 1
        }
    else
        echo -e "${GREEN}[√] 检查仓库更新...${NC}"
        pushd "$REPO_DIR" > /dev/null

        LOCAL_COMMIT=$(git rev-parse HEAD)
        REMOTE_COMMIT=$(git ls-remote origin -h refs/heads/$(git rev-parse --abbrev-ref HEAD) | awk '{print $1}')

        if [ "$LOCAL_COMMIT" != "$REMOTE_COMMIT" ]; then
            echo -e "${CYAN}发现新版本，正在更新（强制覆盖本地更改）...${NC}"
            git reset --hard origin/$(git rev-parse --abbrev-ref HEAD)
            git pull --ff-only
        else
            echo -e "${CYAN}当前已是最新版本${NC}"
        fi

        popd > /dev/null
    fi
}

# 路径有效性验证
validate_path() {
    if ! mkdir -p "$1" 2>/dev/null; then
        echo -e "${RED}[!] 无法创建目录：$1${NC}"
        return 1
    fi
    
    if ! touch "$1/.testwrite" 2>/dev/null; then
        echo -e "${RED}[!] 目录不可写：$1${NC}"
        return 1
    fi
    
    rm -f "$1/.testwrite"
    return 0
}

# URL有效性验证
validate_url() {
    local url="$1"
    if [[ "$url" != *"music.apple.com"* ]]; then
        echo -e "${RED}[!] 链接格式错误，请输入有效的Apple Music链接${NC}"
        return 1
    fi
    return 0
}

# 配置菜单
config_menu() {
    while :; do
        clear
        echo -e "${GREEN}==== 系统配置 ====${NC}"
        echo "1. 路径配置"
        echo "0. 返回主菜单"
        read -p "请选择: " choice

        case $choice in
        1)
            path_config_menu
            ;;
        0)
            break
            ;;
        *)
            echo -e "${RED}无效选项!${NC}"
            sleep 1
            ;;
        esac
    done
}

# 路径配置菜单
path_config_menu() {
    while :; do
        clear
        echo -e "${GREEN}==== 下载路径配置 ====${NC}"
        echo "1. 修改普通音频路径 (当前: $ALAC_SAVE_FOLDER)"
        echo "2. 修改杜比全景声路径 (当前: $ATMOS_SAVE_FOLDER)"
        echo "3. 测试当前路径有效性"
        echo "4. 恢复默认配置"
        echo "0. 返回上级菜单"
        read -p "请选择: " choice

        case $choice in
        1)
            read -p "请输入新路径: " new_path
            if validate_path "$new_path"; then
                ALAC_SAVE_FOLDER="$new_path"
                sed -i "s|ALAC_SAVE_FOLDER=.*|ALAC_SAVE_FOLDER=\"$new_path\"|" "$CONFIG_FILE"
                echo -e "${GREEN}路径更新成功！${NC}"
            else
                echo -e "${RED}路径设置失败，请检查权限！${NC}"
            fi
            sleep 2
            ;;
        2)
            read -p "请输入新路径: " new_path
            if validate_path "$new_path"; then
                ATMOS_SAVE_FOLDER="$new_path"
                sed -i "s|ATMOS_SAVE_FOLDER=.*|ATMOS_SAVE_FOLDER=\"$new_path\"|" "$CONFIG_FILE"
                echo -e "${GREEN}路径更新成功！${NC}"
            else
                echo -e "${RED}路径设置失败，请检查权限！${NC}"
            fi
            sleep 2
            ;;
        3)
            echo -e "\n${CYAN}路径有效性测试：${NC}"
            validate_path "$ALAC_SAVE_FOLDER" && \
            echo -e "普通音频路径: ${GREEN}有效${NC}" || \
            echo -e "普通音频路径: ${RED}无效${NC}"
            
            validate_path "$ATMOS_SAVE_FOLDER" && \
            echo -e "杜比全景声路径: ${GREEN}有效${NC}" || \
            echo -e "杜比全景声路径: ${RED}无效${NC}"
            read -p "按回车键继续..."
            ;;
        4)
            ALAC_SAVE_FOLDER="$DEFAULT_ALAC_PATH"
            ATMOS_SAVE_FOLDER="$DEFAULT_ATMOS_PATH"
            sed -i "s|ALAC_SAVE_FOLDER=.*|ALAC_SAVE_FOLDER=\"$DEFAULT_ALAC_PATH\"|" "$CONFIG_FILE"
            sed -i "s|ATMOS_SAVE_FOLDER=.*|ATMOS_SAVE_FOLDER=\"$DEFAULT_ATMOS_PATH\"|" "$CONFIG_FILE"
            echo -e "${GREEN}已恢复默认路径配置！${NC}"
            sleep 1
            ;;
        0)
            break
            ;;
        *)
            echo -e "${RED}无效选项!${NC}"
            sleep 1
            ;;
        esac
    done
}

# 使用帮助信息
show_help() {
    clear
    echo -e "${GREEN}==== 使用说明 ====${NC}"
    echo -e "\n${GREEN}★ 链接格式示例${NC}"
    echo -e "  - 专辑: https://music.apple.com/us/album/whenever-you-need-somebody-2022-remaster/1624945511"
    echo -e "  - 歌单: https://music.apple.com/us/playlist/taylor-swift-essentials/pl.3950454ced8c45a3b0cc693c2a7db97b9"
    echo -e "  - 杜比: https://music.apple.com/us/album/1989-taylors-version-deluxe/1713845538 (需带杜比标记)"
    
    echo -e "\n${GREEN}★ m3u8端口模式说明${NC}"
    echo -e "  - true: 满血端口"
    echo -e "  - false: 残血端口"

    echo -e "\n${RED}⚠ 常见问题${NC}"
    echo -e "  - 解密错误 127.0.0.1:10020 → 重启客户端"
    echo -e "  - 下载24 192出现 EOF错误 → 把get-m3u8-from-device: true（把true改成false）"
    echo -e "  - 下载完成后→ 改成true，不然后续下载是残血"
    echo -e "  - nano /data/data/com.termux/files/home/apple-music-alac-atmos-downloader/config.yaml"
    echo -e "\n${YELLOW}按任意键返回主菜单...${NC}"
    read -n1 -s
}

# 下载功能
run_download() {  
    local choice=$1
    cd "$REPO_DIR" || { echo -e "${RED}无法进入项目目录！${NC}"; return 1; }

    # 更新配置文件
    yq eval ".alac-save-folder = \"$ALAC_SAVE_FOLDER\"" -i config.yaml  
    yq eval ".atmos-save-folder = \"$ATMOS_SAVE_FOLDER\"" -i config.yaml

    while true; do
        # 获取有效链接
        while :; do
            read -p "请输入有效链接（或输入 0 返回主菜单）: " url
            if [[ "$url" == "0" ]]; then
                cd ..
                return
            fi
            if validate_url "$url"; then
                break
            fi
        done

        case $choice in  
        1)  
            echo -e "${CYAN}开始下载指定曲目...${NC}"  
            go run main.go --select "$url"
            ;;  
        2)  
            echo -e "${CYAN}开始下载专辑内容...${NC}"  
            go run main.go "$url"  
            ;;  
        3)  
            echo -e "${CYAN}开始下载杜比全景声...${NC}"  
            go run main.go --atmos "$url"  
            ;;  
        esac  

        # 显示保存路径  
        echo -e "\n${GREEN}文件保存位置："  
        [ $choice -ne 3 ] && echo -e "普通音频: ${CYAN}$ALAC_SAVE_FOLDER${NC}"  
        [ $choice -eq 3 ] && echo -e "杜比全景声: ${CYAN}$ATMOS_SAVE_FOLDER${NC}"  
    done

    cd ..
}

# 查看专辑质量
chakan_zhuanji() {  
    cd "$REPO_DIR" || { echo -e "${RED}无法进入项目目录！${NC}"; return 1; }

    while true; do
        # 获取有效链接
        while :; do
            read -p "请输入有效链接（或输入 0 返回主菜单）: " url
            if [[ "$url" == "0" ]]; then
                cd ..
                return
            fi
            if validate_url "$url"; then
                break
            fi
        done

        echo -e "${CYAN}正在获取专辑质量信息...${NC}"
        go run main.go --debug "$url"
    done
}

# 主程序流程
init_config
check_storage
install_dependencies
manage_repo

# 主菜单循环
while :; do
    clear
    echo -e "${GREEN}==== Apple Music 下载器 ====${NC}"
    echo "1. 下载指定曲目"
    echo "2. 下载专辑/歌单"
    echo "3. 下载杜比全景声"
    echo "4. 查看专辑质量"
    echo "5. 系统配置"
    echo "6. 使用说明"
    echo "0. 退出程序"
    read -p "请输入选项 [1-0]: " choice

    case $choice in
    1|2|3)  run_download $choice ;;
    4)      chakan_zhuanji ;;
    5)      config_menu ;;
    6)      show_help ;;
    0)      echo -e "${GREEN}感谢使用，再见！${NC}"; exit 0 ;;
    *)      echo -e "${RED}无效选项，请重新输入！${NC}"; sleep 1 ;;
    esac
done