# nockchain

Nockchain 由@ZorpZK开发，非常抽象的一个团队 ， 
Delphi Digital 领投 500w美刀，做 ZKVM 工作量证明的 $Nock 

机制： 代币 100% 通过 ZKPOW挖矿产出，无任何团队、 VC 、基金会份额  $NOCK 
总供应量：约 42.9 亿  $NOCK 
作用：用于支付 Nockchain 上的区块空间


最低配置要求：

配置CPU：8核
内存：64 GB  
SSD：200 GB


安装步骤：

### 第1步：安装依赖项

1. **更新软件包列表并升级现有软件包**:
    
    ```bash
    sudo apt-get update && sudo apt-get upgrade -y
    ```
    
2. **安装必要的软件包**:Bash
    
    ```bash
    sudo apt install curl iptables build-essential git wget lz4 jq make gcc nano automake autoconf tmux htop nvme-cli libgbm1 pkg-config libssl-dev libleveldb-dev tar clang bsdmainutils ncdu unzip libleveldb-dev  -y
    ```
    
3. **安装 Rust**:BashBash
    
    ```bash
    curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
    ```
    
    安装过程中，当提示时，通常选择默认选项 (例如按 `1`)。
    安装完成后，使 Rust 生效：
    
    `source $HOME/.cargo/env`
    
4. **安装 Docker (主网可能会选择Docker部署方式)**:
    - **卸载旧版本 (如果存在)**:
        
        ```bash
        for pkg in docker.io docker-doc docker-compose podman-docker containerd runc; do sudo apt-get remove $pkg; done
        ```
        
    - **更新软件包并安装依赖**:
        
        ```bash
        sudo apt-get update
        sudo apt-get install ca-certificates curl gnupg
        ```
        
    - **添加Docker官方GPG密钥**:
        
        ```bash
        sudo install -m 0755 -d /etc/apt/keyrings
        curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
        sudo chmod a+r /etc/apt/keyrings/docker.gpg
        ```
        
    - **设置Docker仓库**:
        
        ```bash
        echo \
          "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
          "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
          sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
        ```
        
    - **安装Docker Engine**:
        
        ```bash
        sudo apt update -y && sudo apt upgrade -y
        sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
        ```
        
    - **测试Docker安装**:
    你应该会看到一条表示Docker安装成功的消息。
        
        ```bash
        sudo docker run hello-world
        ```
        
    - **设置Docker开机自启并立即启动**:
        
        ```bash
        sudo systemctl enable docker
        sudo systemctl restart docker
        ```
        
    

### 第2步：克隆 Nockchain 代码仓库

```bash
git clone https://github.com/zorp-corp/nockchain
cd nockchain
```

### 第3步：安装 Choo (Jock/Hoon 编译器)

```bash
make install-hoonc
```

这将编译 `hoonc`，它是用于运行 Jock 程序和 ZKVM 应用程序的基于 Nock 的编译器。

### 第4步：构建

构建过程可能需要超过15分钟。

1. **安装节点二进制文件**:
    
    ```bash
    make build
    ```
    
2. **安装钱包二进制文件**:
    
    ```bash
    make install-nockchain-wallet
    ```
    
3. **安装 Nockchain**:
    
    ```bash
    make install-nockchain
    ```
    

### 第5步：设置钱包

1. **确保你在 `nockchain` 目录下**:
    
    `# 如果你不在该目录下，请先 cd nockchain`
    
    ```bash
    cd nockchain
    ```
    
2. **设置 PATH 环境变量 (重要！)**:
    
    `export PATH="$PATH:$(pwd)/target/release"`
    
    **注意**: 每次重新启动终端会话后，在执行钱包命令之前，都需要**重新执行** `cd nockchain` (如果不在该目录) 和 `export PATH="$PATH:$(pwd)/target/release"` 这两个命令。这样可以避免 `"wallet: command not found"` 的错误。为了方便，你可以考虑将 `export PATH="$PATH:/path/to/your/nockchain/target/release"` (将 `/path/to/your/nockchain` 替换为实际路径)添加到你的 `~/.bashrc` 或 `~/.zshrc` 文件中，然后执行 `source ~/.bashrc` (或 `source ~/.zshrc`) 使其永久生效。
    
    ```bash
    nano ~/.bashrc
    ```
    
    ```bash
    export PATH="$HOME/nockchain/target/release:$PATH"
    ```
    
3. **创建钱包**:
    
    ```bash
    nockchain-wallet keygen
    ```
    
    **务必保存好你的 `memo` (助记词)、`private key` (私钥) 和 `public key` (公钥)。这是恢复和访问你钱包的唯一途径！**
    

### 第6步：配置节点

节点的配置在 `.env` 文件中。

1. **打开 `.env`**:
    
    ```bash
    nano .env
    ```
    
2. **找到 `MINING_PUBKEY`**: 将其值替换为你在上一步创建钱包时获得的**公钥**。
    
    ```bash
    MINING_PUBKEY=<public-key>
    ```
    
3. **端口 (Ports)**: 默认情况下，节点使用端口 `3005` 和 `3006`。如果这些端口在你的VPS上被其他程序占用，请在此配置文件中修改它们。

### 第7步：备份key

1. 备份私钥:
    
    ```
    nockchain-wallet export-keys
    ```
    
    私钥将以文件 `keys.export` 保存在nockchain文件夹下.
    
2. 导入私钥:
    
    ```
    nockchain-wallet import-keys --input keys.export
    ```
    

### 第8步：运行节点

运行矿工节点:

```
nockchain --mining-pubkey <your_pubkey> --mine
```

