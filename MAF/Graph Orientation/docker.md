https://dev.to/felipecrs/simply-run-docker-on-wsl2-3o8

Then, configure your Docker CLI to use it by assuring that the following is present in your `~/.docker/config.json`:  

```
   {
     "credsStore": "wincred.exe"
   }
```


for vscode install extensions:
- Docker
- Dev Containers
- Docker DX

for gpu:
- curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg
- curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \ sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \ sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list
- sudo apt-get update && sudo apt-get install -y nvidia-container-toolkit
- sudo systemctl restart docker
-  tested : docker run --rm --gpus all nvidia/cuda:12.1.0-base-ubuntu22.04 nvidia-smi
- in powershell:
	- wsl --shutdown 
	- wsl


Usage:
- launch: docker compose up -d
	- no "--wait-for-client" if you don't use debug
- sudo apt install x11-xserver-utils
- xhost +local:docker
- status docker ps

Attach debugger:
- Open VS Code instance in container:
	- ctrl + maj + p : DevContainer : Attach to running container
	- open folder : /app
- F5 => python file => Remote Attach => localhost=> 5678

Install Teli linux:
- tar -xzf TeliCamSDK_Linux_v4.0.5.1.tar.gz
- sudo ./TeliCamSDK_Linux_v4.0.5.1/install.sh
- Viewer:
	- sudo apt update 
	- sudo apt install libqt5widgets5 libqt5gui5 libqt5core5a libqt5xml5
	-  QT_QPA_PLATFORM=wayland /opt/TeliCamSDK/bin/TeliViewer

Install mqtt:
- sudo snap install mosquitto

**Important** connect vs code wsl 

install uv windows:
- powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
- uv init
- move uv cache : 
	- Move-Item "C:\Users\you\AppData\Local\uv\cache" "D:\uv-cache"
	- [System.Environment]::SetEnvironmentVariable("UV_CACHE_DIR", "D:\uv-cache", "User")
	- restart shell
	- check with : uv cache dir
- Create env : uv venv
- Activate with: .venv\Scripts\activate
- Then sync : uv sync


deamon :
- Error response from daemon: failed to connect to the backend: timed out dialing Hyper-V socket: driver not connecting
- Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V -All
- Get-Service vmms | Restart-Service

context:
- docker context ls : u want something like `rancher-desktop`, not `default`.
- docker context use rancher-desktop
	- if : context "rancher-desktop": context not found
	- docker context create rancher-desktop --docker "host=npipe:////./pipe/rancher-desktop-backend"
	- Get-Process | Where-Object { $_.Name -like "*rancher*" -or $_.Name -like "*wsl*" -or $_.Name -like "*dockerd*" } | Stop-Process -Force

services:
  app:
    build: .
    ports:
      - "5678:5678"
    volumes:
      - ./src:/app/src
    environment:
      - PYTHONPATH=/app

  app-debug:
    build: .
    ports:
      - "5678:5678"
    volumes:
      - ./src:/app/src
    environment:
      - PYTHONPATH=/app
    command: python -m debugpy --listen 0.0.0.0:5678 --wait-for-client src/main.py