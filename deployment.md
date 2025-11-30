Step1: 准备离线包
    
    1.拉取所需镜像
    docker pull outlinewiki/outline:0.83.0
    docker pull postgres:15
    docker pull redis:7

    2.打包所有镜像为一个tar文件
    docker save outlinewiki/outline:0.83.0 postgres:15 redis:7 -o "D:\工作&实习\中芯工作\项目\智能值守\outline\outline-offline-images.tar"
    （大小：1.46G）

    3.下载配置文件模板
    下载网址：https://github.com/outline/outline
    下载文件：docker-compose.yml、env.sample（重命名为.env）

    4.修改配置文件
    4.1 docker-compose.yml文件新增内容：
        outline:
            image: outlinewiki/outline:0.83.0

    4.2 .env文件修改内容（当前未修改）：
        # 必填：服务公网/内网地址（离线环境填本机IP）
        URL=http://192.168.1.100:3000

        # 必填：强随机密钥（至少32字符）
        SECRET_KEY=your_strong_random_string_here_1234567890
        UTILS_SECRET=another_strong_random_string_here_0987654321

        # 数据库连接（与 docker-compose.yml 中一致）
        DATABASE_URL=postgresql://outline:mysecretpassword@db:5432/outline

        # Redis
        REDIS_URL=redis://redis:6379

        # 禁用外部服务（离线必需）
        GOOGLE_CLIENT_ID=
        SLACK_APP_ID=
        AWS_ACCESS_KEY_ID=
        COLLABORATION_SERVER_SECRET=



Step2: 在离线电脑上部署Outline
    情况 A：离线机是 Windows
        1.在WSL2 Ubuntu中进行如下操作
        '''
        # 安装 Docker（即使无网络，也可提前下载 .deb 包，但此处假设你已预装或使用 Docker Desktop 共享）
        # 导入镜像
        docker load -i /mnt/d/outline-offline-images.tar  # 假设U盘挂载在 /mnt/d

        # 复制配置文件到当前目录
        cp /mnt/d/docker-compose.yml .
        cp /mnt/d/.env .

        # 编辑 .env，设置关键参数
        nano .env
        '''
        
        在 .env 中至少修改：
        URL=http://<离线机IP>:3000        # 如 http://192.168.1.100:3000
        SECRET_KEY=your_strong_random_string_32_chars+
        UTILS_SECRET=another_strong_random_string
        DATABASE_URL=postgresql://outline:mysecretpassword@db:5432/outline

        2. 启动服务
        docker-compose up -d

        3.访问outline
        在离线 Windows 浏览器中访问：http://localhost:3000 或 http://<本机IP>:3000

    情况 B：离线机是 Linux
        docker load -i outline-offline-images.tar
        cp docker-compose.yml .env ./
        # 编辑 .env（同上）
        docker-compose up -d



注：
附件存储：默认存于容器内，重启会丢失！务必在 docker-compose.yml 中添加 volume 挂载：
services:
  outline:
    volumes:
      - ./data/uploads:/var/lib/outline/uploads

