# 🚀 Use Your GPU Inside VS Code with WSL2 (Complete Guide)

It’s a really big issue that we can’t use GPU inside our beloved VS-Code and have to use Google Collab instead.  
It’s time to use your **RTX 4060s (or whatever GPU you have)** to full extent inside VS Code.

From my experience I did everything that should’ve worked (CUDA, cuDNN, Python 3.11, VS Code, conda env) and **TensorFlow still ran on CPU**.  
This is a very common pain point:

 **TensorFlow pip wheels on native Windows (>= 2.11) are CPU-only**  
 Even a perfect CUDA/cuDNN install will NOT make Windows TensorFlow use GPU.

So the official + recommended way:

1. Run TensorFlow with GPU inside **WSL2 Ubuntu**  
2. Linux wheels include CUDA + cuDNN automatically  
3. Most ML/AI engineers on Windows use this setup  

---

# ⭐ What is WSL?

WSL = **Windows Subsystem for Linux**, an official Microsoft technology.

It allows you to run a **full Linux system inside Windows** without dual boot.

Think of it like:

-  Running an Android app on Windows  
-  Running Linux commands inside Windows  
-  With **direct GPU access** for deep learning  

SAFE? →  Yes, used by millions.  
Mess files? →  No, Ubuntu is isolated.

---

# ⚡ Quick WSL2 Setup (Admin PowerShell)

## 1️ Install WSL + Ubuntu

```powershell
wsl --install -d ubuntu-22.04
```

What happens:

- Downloads Ubuntu 22.04  
- Installs WSL2 backend  
- Prepares Linux subsystem  

If asked → Restart PC.

---

## 2️ First Launch of Ubuntu

You’ll see:

```
Installing, this may take a few minutes...
```

Then it will ask:

```
Enter new UNIX username:
```

Pick any username & password.

---

## 3️ Check GPU inside Ubuntu

```bash
nvidia-smi
```

If you see your RTX GPU → GPU works.  
If not → driver issue.

---

# 🐍 Install Python + TensorFlow GPU Environment (Ubuntu)

Run all commands inside Ubuntu terminal.

---

##  Step 1 -- Update Ubuntu
```bash
sudo apt update && sudo apt upgrade -y
```

---

##  Step 2 -- Install Python, venv, pip
```bash
sudo apt install -y python3 python3-venv python3-pip build-essential
```

---

##  Step 3 -- Create Environment
```bash
python3 -m venv ML-DL-lat
```

---

##  Step 4 -- Activate Environment
```bash
source ML-DL-lat/bin/activate
```

To reactivate later:
```bash
source ~/ML-DL-lat/bin/activate
```

---

#  Install Full ML + DL Stack (GPU Enabled)

```bash
pip install --upgrade pip setuptools wheel
```

---

##  TensorFlow GPU  
Linux wheels automatically include CUDA + cuDNN.

```bash
pip install "tensorflow[and-cuda]"
```

---

##  PyTorch GPU (CUDA 12.1)
```bash
pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu121
```

---

##  Core ML Libraries
```bash
pip install numpy pandas scikit-learn scipy matplotlib seaborn tqdm
```

---

##  NLP / HuggingFace
```bash
pip install transformers datasets accelerate evaluate sentencepiece
```

---

##  Computer Vision Libraries
```bash
pip install opencv-python pillow imageio
```

---

##  Jupyter + Tools
```bash
pip install jupyter jupyterlab ipywidgets
```

---

#  Verify GPU Support

```bash
python - << 'PY'
import tensorflow as tf
print("TensorFlow:", tf.__version__)
print("Built with CUDA:", tf.test.is_built_with_cuda())
print("GPUs:", tf.config.list_physical_devices("GPU"))
PY
```

Expected:

```
TensorFlow: 2.20.0
built_with_cuda: True
GPUs: [PhysicalDevice('/physical_device:GPU:0')]
PyTorch: 2.5.1+cu121
cuda: True
Device: NVIDIA GeForce RTX 4050 Laptop GPU
```

---

# 🎉 You Now Have:

-  TensorFlow GPU  
-  PyTorch GPU  
-  Full ML + DL stack  
-  CUDA working inside WSL  
-  Clean isolated environment  
-  Setup recommended by NVIDIA, Microsoft, Google  

---

#  Setup VS Code Integration

Check VS Code path:

```powershell
Test-Path "$env:LOCALAPPDATA\Programs\Microsoft VS Code\Code.exe"
Test-Path "C:\Program Files\Microsoft VS Code\Code.exe"
Test-Path "C:\Program Files (x86)\Microsoft VS Code\Code.exe"
```

Lets say correct path is:
```
$env:LOCALAPPDATA\Programs\Microsoft VS Code\Code.exe
```

---

#  Add “Open in WSL (Ubuntu)” to Right-Click Menu

Paste ALL of this in **ADMIN PowerShell**:

```powershell
# Path to VS Code (confirmed)
$codePath = "$env:LOCALAPPDATA\Programs\Microsoft VS Code\Code.exe"

# 1) Create helper script in C:\Windows
$scriptPath = "C:\Windows\open-in-wsl.ps1"

$scriptContent = @'
param([string]$winPath)

if ([string]::IsNullOrEmpty($winPath)) {
    $winPath = (Get-Location).ProviderPath
}

$winPath = $winPath.TrimEnd('\')

$drive = $winPath.Substring(0,1).ToLower()
$rest  = $winPath.Substring(2).Replace("\","/")
$wslPath = "/mnt/$drive/$rest"

$codeExe = "$env:LOCALAPPDATA\Programs\Microsoft VS Code\Code.exe"
$folderUri = "vscode-remote://wsl+Ubuntu-22.04$wslPath"

Start-Process -FilePath $codeExe -ArgumentList "--folder-uri", $folderUri
'@

Set-Content -Path $scriptPath -Value $scriptContent -Encoding UTF8 -Force
Write-Host "Created script: $scriptPath"

reg add "HKCR\Directory\shell\OpenInWSL" /ve /d "Open in WSL (Ubuntu)" /f
reg add "HKCR\Directory\shell\OpenInWSL" /v Icon /d "$codePath" /f

$cmd = 'powershell.exe -NoProfile -ExecutionPolicy Bypass -File "C:\Windows\open-in-wsl.ps1" "%V"'
reg add "HKCR\Directory\shell\OpenInWSL\command" /ve /d "$cmd" /f

Write-Host "*** Installed successfully! ***"
```

---

#  Fix Windows 11 “Show More Options”

```powershell
reg add "HKCU\Software\Classes\CLSID\{86ca1aa0-34aa-4e8b-a509-50c905bae2a2}" /f
reg add "HKCU\Software\Classes\CLSID\{86ca1aa0-34aa-4e8b-a509-50c905bae2a2}\InprocServer32" /ve /f
```

Restart Explorer:

```powershell
Stop-Process -Name explorer -Force
```

Then run:
```
explorer
```

---

#  Final Setup in VS Code 

Right click on the folder you want to open and select "open in WSL ubuntu"
and open a new terminal in VS code

Activate environment:

```bash
source ~/ML-DL-lat/bin/activate
```

Install Jupyter kernel:

```bash
python -m ipykernel install --user --name ML-DL-lat --display-name "Python (ML-DL-lat)"
```

Select kernel:

- Python (ML-DL-lat)  
---

# 👨‍💻 Test GPU Inside VS Code

```python
import tensorflow as tf
print(tf.__version__)
print(tf.config.list_physical_devices("GPU"))
```

Expected:

```
['/physical_device:GPU:0']
```

