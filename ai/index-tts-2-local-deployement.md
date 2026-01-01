# Index-TTS2

## 本地部署

### 1. 下载代码
```
git clone https://github.com/index-tts/index-tts.git
cd index-tts
```
官方说明需要使用LFS获取大文件：
```
git lfs install
git lfs pull
```
由于LFS额度限制，很可能无法拉取，可执行`git lfs ls-files`查询哪些大文件还没有下载，然后手动操作。通常是`examples`目录下的语音示例文件：
```
wget -P examples https://github.com/index-tts/index-tts/raw/refs/heads/main/examples/emo_hate.wav
wget -P examples https://github.com/index-tts/index-tts/raw/refs/heads/main/examples/emo_sad.wav
wget -P examples https://github.com/index-tts/index-tts/raw/refs/heads/main/examples/voice_01.wav
...
wget -P examples https://github.com/index-tts/index-tts/raw/refs/heads/main/examples/voice_12.wav
```

### 2. 安装uv
```
apt install python3-pip
pip install -U uv
```
如果安装uv出现以下错误：
```
note: If you believe this is a mistake, please contact your Python installation or OS distribution provider. You can override this, at the risk of breaking your Python installation or OS, by passing --break-system-packages.
```
可以先设置临时环境变量再执行：
```
export PIP_BREAK_SYSTEM_PACKAGES=1
```

### 3. 安装依赖
使用uv安装依赖时，会创建虚拟环境，将所有依赖安装到.venv目录：
```
uv sync --all-extras
```
如果执行失败，再执行一遍。如果中国大陆地区用户下载缓慢，可选用国内镜像。

```
uv sync --all-extras --default-index "https://mirrors.aliyun.com/pypi/simple"
uv sync --all-extras --default-index "https://mirrors.tuna.tsinghua.edu.cn/pypi/web/simple"
```

### 4. 下载模型
```
uv tool install "modelscope"
modelscope download --model IndexTeam/IndexTTS-2 --local_dir checkpoints
```

### 5. GPU检测
```
uv run tools/gpu_check.py
```

### 6. 启动Web
```
uv run webui.py
```
项目首次运行还会自动下载部分小模型，如网络访问HuggingFace较慢，可设置HF_ENDPOINT后再运行：
```
export HF_ENDPOINT="https://hf-mirror.com"
```

## 构建镜像

### CNB同名制品
```
docker build -t docker.cnb.cool/arraywork/index-tts2:latest .
docker push docker.cnb.cool/arraywork/index-tts2:latest
```
