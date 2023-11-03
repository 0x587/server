https://www.jetbrains.com/help/datalore/install-datalore-enterprise-using-docker.html

### 克隆配置仓库

```bash
git clone https://github.com/JetBrains/datalore-configs.git
```

### 预先拉取镜像
```bash
sudo docker pull jetbrains/datalore-server:2023.5.1
sudo docker pull jetbrains/datalore-postgres:2023.5.1
sudo docker pull jetbrains/datalore-agent:2023.5.1
```

### 配置docker volume位置
```bash
cd /data
sudo mkdir datalore
```

### 初始化启动datalore

docker compose
```yaml
version: "3.9"
services:
  datalore:
    image: jetbrains/datalore-server:2023.5.1
    ports:
      - "8080:8080"
    expose: [ "8081", "5050", "4060" ]
    networks:
    # your data storage location
      - datalore-agents-network
      - datalore-backend-network
    volumes:
      - "/data/datalore:/opt/data"
      - "/var/run/docker.sock:/var/run/docker.sock"
    environment:
      #DATALORE_PUBLIC_URL: "https://datalore.example.com"
      DB_PASSWORD: "changeme"
  postgresql:
    image: jetbrains/datalore-postgres:2023.5.1
    expose: [ "5432" ]
    networks:
      - datalore-backend-network
    volumes:
      - "postgresql-data:/var/lib/postgresql/data"
    environment:
      POSTGRES_PASSWORD: "changeme"
volumes:
  postgresql-data: { }
networks:
  datalore-agents-network:
    name: datalore-agents-network
  datalore-backend-network:
    name: datalore-backend-network
```

登录创建管理员账户并输入License激活

查看License https://account.jetbrains.com/licenses

### 配置GPU

#### nvidia配置
```bash
# 驱动安装
sudo apt install nvidia-utils-535-server
nvidia-smi

distribution=$(. /etc/os-release;echo $ID$VERSION_ID) \
   && curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add - \
   && curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | sudo tee /etc/apt/sources.list.d/nvidia-docker.list
   
sudo apt-get update

sudo apt-get install -y nvidia-docker2

sudo systemctl restart docker
```

#### 构建自定义镜像
``` Dockerfile
FROM jetbrains/datalore-agent:2023.5.1

RUN /opt/python/envs/minimal/bin/python -m pip install torch torchvision torchaudio -i https://pypi.tuna.tsinghua.edu.cn/simple

RUN /opt/python/envs/minimal/bin/python -m pip install h5py tqdm pynrrd -i https://pypi.tuna.tsinghua.edu.cn/simple

RUN /opt/datalore/build_code_insight_data.sh /opt/python/envs/minimal
```

``` bash
sudo docker build -t agent-torch .
```

#### 配置GPU配合和自定义镜像

关闭datalore

agents-config.yaml
``` yaml
docker:
  network: datalore-agents-network
  dataloreHost: datalore
  instances:
    - id: basic-agent
      default: true
      label: "docker-base"
      description: "docker-base"
      image: docker.io/jetbrains/datalore-agent:2023.5.1

    - id: torch-agent
      default: false
      label: "torch-agent"
      description: "torch-agent"
      image: agent_torch
      deviceRequests:
        - capabilities: [ [ "gpu" ] ]
```

修改compose映射agents-config.yaml

```yaml
volumes:
      - "/data/datalore:/opt/data"
      - "/var/run/docker.sock:/var/run/docker.sock"
      - "/home/shawn/datalore-configs/agents-config.yaml:/opt/datalore/configs/agents-config.yaml"
```

重启datalore
