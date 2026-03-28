---
title: "WSL2 하나면 윈도우에서 리눅스 AI 환경이 된다 — 설치부터 GPU 연결까지"
date: 2026-03-28
draft: false
tags: ["WSL2", "리눅스", "AI", "개발환경", "Windows", "GPU", "CUDA"]
categories: ["AI 활용법"]
description: "윈도우 사용자가 WSL2를 설치하고 GPU 패스스루까지 설정해서 리눅스 기반 AI 개발환경을 완성하는 과정을 처음부터 끝까지 다룹니다."
ShowToc: true
---

예전에 [AI 하려면 리눅스를 써야 하냐는 글](/posts/2026-02-18-linux-for-ai/)을 쓴 적이 있는데, 그때 결론이 "본격적으로 하면 리눅스가 편해지는 시점이 온다"였습니다. 그 시점이 오면 어떻게 해야 하느냐. 컴퓨터를 포맷하고 우분투를 깔 필요까지는 없습니다. 윈도우 안에서 리눅스를 돌릴 수 있는 WSL2라는 물건이 있기 때문입니다.

이 글은 "WSL2가 뭔지는 대충 들어봤는데 실제로 세팅하면 뭘 얻는 건지, 어떻게 하는 건지 모르겠다"는 사람을 위한 글입니다. 그냥 따라 치면 끝나도록 정리했습니다.

## WSL2가 해결해주는 문제

윈도우에서 AI 작업을 하다 보면 부딪히는 벽이 몇 개 있습니다.

PyTorch 설치할 때 conda vs pip 충돌. CUDA 버전 맞추느라 삽질. [Docker](/posts/2026-02-22-docker-beginner/)가 네이티브가 아니라 Docker Desktop이라는 무거운 앱을 깔아야 하는 것. 파이썬 패키지 중에 윈도우에서 빌드가 안 되는 것들. 이런 것들이 쌓이면 "그냥 리눅스 깔까" 싶은 순간이 옵니다.

WSL2는 이 문제를 기막히게 풀어줍니다. 윈도우 커널 위에서 진짜 리눅스 커널이 돌아갑니다. 예전의 WSL1은 리눅스 시스템 콜을 윈도우가 번역하는 방식이라 호환성이 떨어졌는데, WSL2는 경량 Hyper-V 가상 머신 위에서 실제 리눅스 커널(현재 6.6 기반)을 실행합니다. 그래서 거의 네이티브 속도가 나오고, NVIDIA GPU까지 직접 물려서 쓸 수 있습니다. 윈도우의 파일 탐색기, VS Code, 브라우저 같은 GUI 도구는 그대로 쓰면서 터미널 작업만 리눅스에서 하는 구조라서, 양쪽의 장점만 취하는 셈입니다.

## 1단계: WSL2 설치

Windows 10 (빌드 19041 이상) 또는 Windows 11이면 됩니다. 관리자 권한으로 PowerShell을 열고 한 줄만 치면 됩니다.

```powershell
wsl --install
```

이러면 WSL2 + Ubuntu 24.04가 기본으로 깔립니다. 재부팅하라고 하면 재부팅하고, 우분투 창이 뜨면 사용자 이름과 비밀번호를 설정합니다. 끝.

혹시 이미 WSL1이 깔려 있다면 버전을 올려야 합니다.

```powershell
wsl --set-version Ubuntu 2
```

설치가 됐는지 확인:

```powershell
wsl -l -v
```

NAME에 Ubuntu, VERSION에 2가 나오면 정상입니다.

## 2단계: 우분투 초기 세팅

우분투 터미널이 열렸으면 바로 패키지 업데이트부터 합니다.

```bash
sudo apt update && sudo apt upgrade -y
```

그다음 AI 작업에 거의 항상 필요한 기본 도구들을 한꺼번에 설치합니다.

```bash
sudo apt install -y build-essential git curl wget unzip python3-pip python3-venv
```

이 패키지들이 뭔지 하나하나 설명하면 글이 길어지니까 간단히만: build-essential은 C/C++ 컴파일러(일부 파이썬 패키지가 빌드할 때 필요), git은 버전 관리, 나머지는 다운로드와 파이썬 환경 도구입니다.

파이썬 가상환경을 하나 만들어둡니다. 시스템 파이썬에 직접 패키지를 깔면 나중에 꼬이기 때문입니다.

```bash
python3 -m venv ~/ai-env
echo 'source ~/ai-env/bin/activate' >> ~/.bashrc
source ~/.bashrc
```

이제 터미널을 열 때마다 자동으로 가상환경이 활성화됩니다.

## 3단계: NVIDIA GPU 연결 — 여기가 핵심

WSL2에서 GPU를 쓰려면 **윈도우 쪽에** 최신 NVIDIA 드라이버가 깔려 있어야 합니다. 주의할 점은 WSL2 안의 우분투에는 드라이버를 설치하지 않는다는 겁니다. 윈도우 드라이버가 자동으로 WSL2에 GPU를 넘겨줍니다.

윈도우에서 NVIDIA 드라이버를 최신으로 업데이트하세요. GeForce Experience 앱이 있으면 거기서 해도 되고, nvidia.com에서 직접 받아도 됩니다. 드라이버 버전이 525.60 이상이면 대부분 문제 없습니다.

드라이버 설치 후, 우분투 터미널에서 확인합니다.

```bash
nvidia-smi
```

GPU 이름, 드라이버 버전, CUDA 버전이 표시되면 성공입니다. RTX 3060 12GB 기준으로 이런 식의 출력이 나옵니다:

```
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 560.35.03    Driver Version: 560.94       CUDA Version: 12.6     |
| GPU Name        ... GeForce RTX 3060                                        |
| Memory          ... 12288MiB                                                |
+-----------------------------------------------------------------------------+
```

만약 nvidia-smi가 안 된다면 십중팔구 윈도우 드라이버가 오래된 겁니다. 드라이버를 업데이트하고 WSL2를 재시작(`wsl --shutdown` 후 다시 열기)하면 해결됩니다.

참고로 이 GPU 패스스루는 RTX 20 시리즈 이상에서 지원됩니다. GTX 1080 Ti 같은 구세대 카드도 기본적인 CUDA 연산은 가능하지만, 최신 드라이버의 WSL2 지원이 점점 축소되고 있어서 RTX 30 시리즈 이상을 권장합니다. AMD GPU는 현재 WSL2에서 공식 지원하지 않습니다. ROCm이 리눅스 네이티브에서는 되지만 WSL2에서는 아직 안 됩니다.

## 4단계: PyTorch + CUDA 설치

GPU가 잡혔으면 PyTorch를 CUDA 버전에 맞춰 설치합니다. 2026년 3월 기준으로 PyTorch 2.5가 최신이고, CUDA 12.4를 권장합니다.

```bash
pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu124
```

설치 후 GPU가 정상적으로 인식되는지 파이썬으로 확인합니다.

```python
python3 -c "import torch; print(torch.cuda.is_available()); print(torch.cuda.get_device_name(0))"
```

`True`와 GPU 이름이 출력되면 완벽합니다. 이 두 줄이 나오면 GPU 세팅은 끝입니다.

## 5단계: 실제로 뭘 할 수 있는지

환경이 갖춰졌으니 실제로 해볼 수 있는 것들을 짚겠습니다.

**Ollama로 로컬 LLM 돌리기.** [내 PC에서 AI 모델 돌리기](/posts/2026-03-10-local-llm/) 글에서 다뤘던 Ollama가 WSL2에서 바로 설치됩니다.

```bash
curl -fsSL https://ollama.com/install.sh | sh
ollama run llama3.1:8b
```

직접 테스트해보면 윈도우 네이티브 Ollama보다 WSL2 버전이 리눅스 최적화 덕에 토큰 생성 속도가 체감될 정도로 빠릅니다. VRAM 8GB 이상이면 8B 모델은 무리 없이 돌아갑니다.

**Stable Diffusion WebUI 실행.** AUTOMATIC1111이나 ComfyUI 같은 이미지 생성 도구도 WSL2에서 훨씬 안정적으로 돌아갑니다. 윈도우에서 설치할 때 흔히 겪는 xformers 빌드 에러가 리눅스에서는 거의 발생하지 않습니다.

**Hugging Face 모델 학습/파인튜닝.** transformers, accelerate, bitsandbytes 같은 라이브러리가 리눅스에서만 정상 동작하는 경우가 있습니다. 특히 bitsandbytes는 윈도우 지원이 불안정한데, WSL2에서는 그냥 `pip install`로 끝납니다.

**Docker도 네이티브처럼 돌아갑니다.** Docker Desktop 없이도 WSL2 안에서 Docker Engine을 직접 설치해서 쓸 수 있습니다. 컨테이너 기반 AI 파이프라인을 구성하거나, 남이 만든 Docker 이미지를 그대로 실행하는 것도 리눅스와 동일한 방식으로 가능합니다. Docker Desktop은 윈도우 시작 시 메모리를 1~2GB씩 잡아먹는데, WSL2 내부 Docker는 컨테이너를 띄울 때만 리소스를 씁니다.

```bash
# WSL2 우분투 안에서 Docker 설치
curl -fsSL https://get.docker.com | sh
sudo usermod -aG docker $USER
```

설치 후 터미널을 재시작하면 `docker run hello-world`로 정상 동작을 확인할 수 있습니다. Docker 컨테이너 안에서도 GPU를 쓰려면 NVIDIA Container Toolkit을 추가로 설치해야 합니다.

```bash
# NVIDIA Container Toolkit 설치
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg
curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
  sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
  sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list
sudo apt update && sudo apt install -y nvidia-container-toolkit
sudo nvidia-ctk runtime configure --runtime=docker
sudo systemctl restart docker
```

설치 후 `docker run --gpus all nvidia/cuda:12.4.0-base-ubuntu22.04 nvidia-smi`를 실행해서 컨테이너 안에서 GPU가 잡히면 성공입니다.

## 메모리 설정을 반드시 건드려야 하는 이유

WSL2의 기본 메모리 할당은 전체 RAM의 50%입니다. 32GB RAM이면 WSL2가 16GB만 쓰는 겁니다. AI 작업에서는 이게 부족할 수 있습니다.

윈도우 사용자 폴더(보통 `C:\Users\사용자이름\`)에 `.wslconfig` 파일을 만들어서 조정합니다. PowerShell에서 `notepad "$env:USERPROFILE\.wslconfig"`를 실행하면 바로 편집할 수 있습니다.

```ini
[wsl2]
memory=24GB
swap=8GB
processors=8
```

32GB RAM 기준으로 WSL2에 24GB를 주면 윈도우 기본 동작에는 8GB면 충분합니다(브라우저를 20개씩 띄우지 않는다면). 수치는 본인 환경에 맞게 조정하면 됩니다. 파일 저장 후 `wsl --shutdown`으로 재시작하면 적용됩니다.

## VS Code 연동

VS Code를 쓴다면 `Remote - WSL` 확장을 설치하세요. 그러면 VS Code가 윈도우에서 열리지만 내부적으로는 WSL2의 리눅스 파일 시스템에서 작업하게 됩니다. 터미널도 자동으로 우분투 셸이 열리고, 파이썬 인터프리터도 WSL2 안의 것을 잡아줍니다.

우분투 터미널에서 프로젝트 폴더로 이동한 뒤 `code .`을 치면 윈도우의 VS Code가 해당 폴더를 WSL2 모드로 열어줍니다. 이 워크플로우에 익숙해지면 윈도우인지 리눅스인지 의식하지 않게 됩니다.

한 가지 주의: 프로젝트 파일은 WSL2 내부 파일 시스템(`/home/사용자/` 아래)에 두세요. `/mnt/c/` 같은 윈도우 마운트 경로에 두면 파일 I/O가 5~10배 느립니다. 이건 WSL2의 아키텍처적 제약이라서 피할 수 없습니다.

## 트러블슈팅 3가지

세팅하다 보면 높은 확률로 부딪히는 문제를 미리 정리해둡니다.

**1. nvidia-smi는 되는데 CUDA가 안 잡힘.** PyTorch를 CPU 버전으로 설치했을 가능성이 높습니다. `pip uninstall torch`한 뒤 위의 `--index-url` 옵션을 붙여서 재설치하면 됩니다.

**2. 디스크 공간 부족.** WSL2의 가상 디스크는 기본적으로 C 드라이브에 생깁니다. AI 모델 파일이 수 GB~수십 GB씩 하기 때문에 금방 차는데, D 드라이브 등으로 옮기고 싶다면 `wsl --export`로 백업 후 `wsl --import`로 다른 경로에 복원하면 됩니다.

**3. 네트워크 문제.** WSL2가 NAT 모드로 동작해서 간혹 포트포워딩이 필요할 수 있습니다. Jupyter Notebook이나 Gradio 앱을 WSL2에서 실행한 뒤 윈도우 브라우저에서 접속할 때 `localhost`로 안 되면 WSL2의 IP를 직접 쓰거나, `.wslconfig`에 `networkingMode=mirrored`를 추가하세요. 미러 모드는 Windows 11 22H2 이상에서 지원됩니다.

## 결국 이게 가장 현실적인 구성이다

듀얼부팅은 매번 재부팅해야 하니 번거롭고, 순수 리눅스 데스크톱은 한글 입력이나 카카오톡 같은 일상 앱에서 불편합니다. 가상 머신은 GPU 패스스루가 까다롭습니다. WSL2는 이 세 가지의 단점을 전부 피하면서 리눅스 개발환경을 줍니다.

실제로 많은 AI 개발자들이 "회사에서는 WSL2, 서버에서는 네이티브 리눅스"라는 조합으로 일하고 있습니다. 세팅에 걸리는 시간은 처음이라도 30분이면 충분합니다.

정리하면 이런 순서입니다: WSL2 설치(5분) - 우분투 패키지 업데이트(5분) - NVIDIA 드라이버 확인(5분) - PyTorch 설치(10분) - .wslconfig 설정(2분) - VS Code 연동(3분). 30분이면 끝나니까 한번 해보세요. `wsl --install` 한 줄이면 시작입니다.
