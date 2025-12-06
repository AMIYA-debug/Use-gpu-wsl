# 🚀 GPU Support in VS Code (WSL2 Guide)

It’s a really big issue that we can’t use GPU inside our beloved VS-Code and have to use Google Colab instead. It’s time to use your 4060s (or whatever) to full extent inside VS Code.

From my experience I did everything that should’ve worked (CUDA, cuDNN, Python 3.11, VS Code, conda env) and TensorFlow still runs on CPU. That exact situation is the common pain point: **TensorFlow pip wheels on native Windows (>= 2.11) are CPU-only**, so even a perfectly configured CUDA/cuDNN on Windows will not make `pip install tensorflow` use the GPU.

The official guidance is to run TensorFlow with GPU support inside WSL2 (Ubuntu), or to use older/native Windows stacks (TF 2.10) or special conda builds. So we’ll switch to WSL2. It’s a few steps but it’s reliable and future-proof. This is what most ML folks on Windows use -- GPU support works and the Linux pip wheels include GPU support.

---

## ⭐ What is WSL?
**WSL = Windows Subsystem for Linux**

It is a technology from Microsoft that lets you run a full Linux system (like Ubuntu) inside Windows, without dual-boot, without a separate PC, without VirtualBox, nothing heavy. It runs side-by-side with Windows, in a small isolated container-like environment.

Think of it like:
* 📱 Running an Android app on Windows
* 🖥️ Running Linux commands inside Windows
* 🚀 **But with direct GPU access for ML**

### ⭐ Is WSL safe?
**Yes.** 100% official Microsoft product. Used by millions of developers.

### ⭐ Will Ubuntu mess up my files?
**No.** It lives in a separate folder, isolated from everything. You can still access Windows files from Ubuntu and vice versa.

---

## Quick WSL2 steps (copy-paste in an Admin PowerShell)

### 1. Install WSL + Ubuntu
```powershell
wsl --install -d ubuntu-22.04
What will happen:

Windows downloads Ubuntu 22.04

Installs WSL2 backend

Prepares your Linux subsystem

If it asks you to restart → Restart your PC.

2. After restart, Ubuntu will automatically open
It will show something like: Installing, this may take a few minutes...

Then it will ask: Enter new UNIX username:

Just pick any username, like yourname.

Then enter a password (you won’t see characters; that’s normal).

3. Verify GPU Driver
In the Ubuntu terminal, copy–paste this:

Bash

nvidia-smi
You should see your RTX GPU and the NVIDIA driver table.

If nvidia-smi works → WSL GPU support is ready.

(If it says “command not found” or “No devices found” → we fix it, but usually it works instantly because modern Windows automatically installs the WSL-compatible NVIDIA driver).

4. Install Python + Create TensorFlow GPU Environment (inside Ubuntu)
Run everything inside your Ubuntu terminal:

Step 1: Update Ubuntu
Bash

sudo apt update && sudo apt upgrade -y
Step 2: Install Python, venv, pip
Bash

sudo apt install -y python3 python3-venv python3-pip build-essential
Step 3: Create your environment (name: ML-DL-lat)
Bash

python3 -m venv ML-DL-lat
Step 4: Activate the environment
Bash

source ML-DL-lat/bin/activate
Note: Every time you want to use this environment again, run: source ~/ML-DL-lat/bin/activate

Step 5: Install Full ML + DL Stack (GPU Enabled)
Upgrade pip:

Bash

pip install --upgrade pip setuptools wheel
Install TensorFlow GPU (Linux wheels include CUDA support automatically):

Bash

pip install "tensorflow[and-cuda]"
(This downloads TF + CUDA runtime + cuDNN for WSL)

Install PyTorch GPU (CUDA 12.1):

Bash

pip install torch torchvision torchaudio --index-url [https://download.pytorch.org/whl/cu121](https://download.pytorch.org/whl/cu121)
Core ML Libraries:

Bash

pip install numpy pandas scikit-learn scipy matplotlib seaborn tqdm
NLP / HuggingFace Stack:

Bash

pip install transformers datasets accelerate evaluate sentencepiece
CV Libraries:

Bash

pip install opencv-python pillow imageio
Jupyter + Environment Tools:

Bash

pip install jupyter jupyterlab ipywidgets
Verify Installation
Once you finish installing everything, run the GPU verification script inside Ubuntu:

Bash

python - << 'PY'
import tensorflow as tf
print("TensorFlow:", tf.__version__)
print("Built with CUDA:", tf.test.is_built_with_cuda())
print("GPUs:", tf.config.list_physical_devices("GPU"))
PY
Your output should be like:

TensorFlow: 2.20.0 built_with_cuda: True TF GPUs: [PhysicalDevice(name='/physical_device:GPU:0', device_type='GPU')] PyTorch: 2.5.1+cu121 cuda: True device: NVIDIA GeForce RTX 4050 Laptop GPU

YOU NOW HAVE:

TensorFlow GPU

PyTorch GPU

Full ML + DL stack

CUDA inside WSL working flawlessly

A clean, isolated environment

A setup recommended by NVIDIA, Microsoft, Google

NOW Setup VS-Code
Verify where VS Code is installed
Run this first and copy the one that prints True (or just note it):

PowerShell

Test-Path "$env:LOCALAPPDATA\Programs\Microsoft VS Code\Code.exe"
Test-Path "C:\Program Files\Microsoft VS Code\Code.exe"
Test-Path "C:\Program Files (x86)\Microsoft VS Code\Code.exe"
Let's say here it is: "$env:LOCALAPPDATA\Programs\Microsoft VS Code\Code.exe"

COPY/PASTE THIS INTO ADMIN POWERSHELL
PowerShell

# Path to VS Code (confirmed)
$codePath = "$env:LOCALAPPDATA\Programs\Microsoft VS Code\Code.exe"

# 1) Create helper script in C:\Windows
$scriptPath = "C:\Windows\open-in-wsl.ps1"

$scriptContent = @'
param([string]$winPath)

# If no folder passed, use current directory
if ([string]::IsNullOrEmpty($winPath)) {
    $winPath = (Get-Location).ProviderPath
}

$winPath = $winPath.TrimEnd('\')

# Convert "C:\Users\amiya\folder" → "/mnt/c/Users/amiya/folder"
$drive = $winPath.Substring(0,1).ToLower()
$rest  = $winPath.Substring(2).Replace("\","/")
$wslPath = "/mnt/$drive/$rest"

$codeExe = "$env:LOCALAPPDATA\Programs\Microsoft VS Code\Code.exe"

# Folder URI VS Code needs to attach to Ubuntu WSL
$folderUri = "vscode-remote://wsl+Ubuntu-22.04$wslPath"

Start-Process -FilePath $codeExe -ArgumentList "--folder-uri", $folderUri
'@

Set-Content -Path $scriptPath -Value $scriptContent -Encoding UTF8 -Force
Write-Host "Created script: $scriptPath"


# 2) Add Explorer Right-Click Entry: "Open in WSL (Ubuntu)"
reg add "HKCR\Directory\shell\OpenInWSL" /ve /d "Open in WSL (Ubuntu)" /f
reg add "HKCR\Directory\shell\OpenInWSL" /v Icon /d "$codePath" /f

# 3) Add command that executes the script
$cmd = 'powershell.exe -NoProfile -ExecutionPolicy Bypass -File "C:\Windows\open-in-wsl.ps1" "%V"'
reg add "HKCR\Directory\shell\OpenInWSL\command" /ve /d "$cmd" /f

Write-Host "`n*** Installed successfully! ***"
Write-Host "Right-click ANY folder → you'll see:  Open in WSL (Ubuntu)"
Force Windows 11 to always show classic context menu
If your system is Windows 11, it hides 3rd-party items behind “Show more options”. We can fix that.

Run this in Admin PowerShell:

PowerShell

reg add "HKCU\Software\Classes\CLSID\{86ca1aa0-34aa-4e8b-a509-50c905bae2a2}" /f
reg add "HKCU\Software\Classes\CLSID\{86ca1aa0-34aa-4e8b-a509-50c905bae2a2}\InprocServer32" /ve /f
Then restart Explorer again:

PowerShell

Stop-Process -Name explorer -Force
Then press WIN + R, run: explorer

Test it
Right-click any folder in Explorer → you should see Open in WSL (Ubuntu).

Click it — VS Code should open in your normal Windows UI but the window will be connected to WSL: Ubuntu-22.04 and open that folder path (mounted as /mnt/...).

NOW Finally
In a new terminal of VS Code (now it looks like a linux terminal).

Bash:

Bash

source ~/ML-DL-lat/bin/activate
Install Jupyter kernel for this env:

Bash

python -m ipykernel install --user --name ML-DL-lat --display-name "Python (ML-DL-lat)"
Go back to VS Code and click “Select Kernel”:

Click Select Kernel → you will see a menu.

Choose: Python (ML-DL-lat) [WSL] /home/amiya/anaconda3/envs/ML-DL-lat/bin/python

Test GPU in VS Code
After selecting kernel, run:

Python

import tensorflow as tf
print(tf.__version__)
print(tf.config.list_physical_devices("GPU"))
Expected output: ['/physical_device:GPU:0']