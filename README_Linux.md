# Portable Python on USB Drive for Linux/Raspberry Pi. (Not fully tested yet)

A complete guide to create a portable Python environment on a USB drive that can run on Linux systems and Raspberry Pi without requiring Python installation.

## Table of Contents
- [English Version](#english-version)
- [日本語版](#日本語版)

---

# English Version

## Overview

This guide will help you set up Python 3.11.9 on a USB drive, making it completely portable and usable on any Linux system or Raspberry Pi without root privileges.

## Requirements

- USB drive (minimum 2GB recommended)
- Linux computer or Raspberry Pi
- Internet connection for downloading Python and packages
- Basic command line knowledge

## Important Notes

### Architecture Considerations

Linux systems use different CPU architectures:
- **x86_64 (AMD64)**: Standard desktop/laptop Linux
- **ARM (armv7l)**: Raspberry Pi 3, Pi 4 (32-bit OS)
- **ARM64 (aarch64)**: Raspberry Pi 3, Pi 4, Pi 5 (64-bit OS)

**You must download the Python version matching your target system's architecture.**

## Method 1: Using Pre-built Python (Recommended for Beginners)

### Step 1: Download Python Standalone Build

The easiest way is to use pre-built portable Python from the Python Standalone Builds project.

1. Visit: https://github.com/indygreg/python-build-standalone/releases
2. Find the latest release for Python 3.11.9
3. Download the appropriate version:

**For standard Linux (x86_64):**
```
cpython-3.11.9+20240107-x86_64-unknown-linux-gnu-install_only.tar.gz
```

**For Raspberry Pi 32-bit (armv7):**
```
cpython-3.11.9+20240107-armv7-unknown-linux-gnueabihf-install_only.tar.gz
```

**For Raspberry Pi 64-bit (aarch64):**
```
cpython-3.11.9+20240107-aarch64-unknown-linux-gnu-install_only.tar.gz
```

![Download Standalone Build](images/linux_01_download_standalone.png)
*Screenshot showing the Python Standalone Builds release page*

### Step 2: Check Your System Architecture

Before downloading, check your system architecture:

```bash
uname -m
```

Output meanings:
- `x86_64` → Download x86_64 version
- `armv7l` → Download armv7 version (32-bit)
- `aarch64` → Download aarch64 version (64-bit)

![Check Architecture](images/linux_02_check_arch.png)
*Screenshot showing the uname command output*

### Step 3: Extract Python to USB

1. Insert your USB drive and mount it (usually auto-mounted as `/media/username/USB_NAME`)
2. Create a directory for Python:

```bash
cd /media/$USER/YOUR_USB_NAME
mkdir PortablePython
cd PortablePython
```

3. Extract the downloaded archive:

```bash
tar -xzf ~/Downloads/cpython-3.11.9+*-install_only.tar.gz
```

Your folder structure should look like:
```
/media/user/USB/PortablePython/
└── python/
    ├── bin/
    │   ├── python3
    │   ├── python3.11
    │   └── pip3
    ├── lib/
    └── include/
```

![Extracted Files Linux](images/linux_03_extracted_files.png)
*Screenshot showing the extracted Python files on USB*

### Step 4: Test Python Installation

```bash
cd /media/$USER/YOUR_USB_NAME/PortablePython/python/bin
./python3 --version
```

You should see: `Python 3.11.9`

![Python Version Check](images/linux_04_python_version.png)
*Screenshot showing successful Python version check*

### Step 5: Verify pip Installation

```bash
./python3 -m pip --version
```

Pip should already be included in the standalone build.

![Pip Version Linux](images/linux_05_pip_version.png)
*Screenshot showing pip version*

### Step 6: Install Additional Libraries

```bash
# Navigate to Python binary directory
cd /media/$USER/YOUR_USB_NAME/PortablePython/python/bin

# Install packages
./python3 -m pip install requests numpy pandas --user
```

**Note:** The `--user` flag installs packages in the portable Python's site-packages directory.

![Install Packages Linux](images/linux_06_install_packages.png)
*Screenshot showing package installation*

### Step 7: Running Python Scripts

#### Method 1: Using Bash Script

Create `run_script.sh` in the USB root:

```bash
#!/bin/bash

# Get the directory where this script is located
SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"

# Set Python path
PYTHON_BIN="$SCRIPT_DIR/PortablePython/python/bin/python3"

# Run your Python script
"$PYTHON_BIN" "$SCRIPT_DIR/my_project/main.py" "$@"

# Optional: wait for user input
read -p "Press Enter to exit..."
```

Make it executable:
```bash
chmod +x run_script.sh
```

Run it:
```bash
./run_script.sh
```

![Bash Script Linux](images/linux_07_bash_script.png)
*Screenshot showing the bash script content*

![Running Bash Linux](images/linux_08_running_bash.png)
*Screenshot showing script execution*

#### Method 2: Using Shebang in Python Scripts

Add a relative shebang to your Python script:

**my_script.py:**
```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

print("Hello from Portable Python on Linux!")
```

Create a wrapper script `run_my_script.sh`:
```bash
#!/bin/bash
SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
export PATH="$SCRIPT_DIR/PortablePython/python/bin:$PATH"
python3 "$SCRIPT_DIR/my_script.py" "$@"
```

#### Method 3: Creating an Alias (Temporary)

For the current session only:
```bash
alias portablepython='/media/$USER/YOUR_USB_NAME/PortablePython/python/bin/python3'
portablepython my_script.py
```

### Step 8: Example Project Structure

```
/media/user/USB/
├── PortablePython/
│   └── python/
│       ├── bin/
│       │   ├── python3
│       │   └── pip3
│       └── lib/
├── my_project/
│   ├── main.py
│   ├── requirements.txt
│   └── data/
├── run_script.sh
└── README.md
```

![Project Structure Linux](images/linux_09_project_structure.png)
*Screenshot showing the complete folder structure*

## Method 2: Building Python from Source (Advanced)

This method gives you full control but requires more time.

### Prerequisites

Install build dependencies (requires sudo):
```bash
sudo apt-get update
sudo apt-get install build-essential zlib1g-dev libncurses5-dev \
    libgdbm-dev libnss3-dev libssl-dev libreadline-dev \
    libffi-dev libsqlite3-dev wget libbz2-dev
```

### Build Steps

1. Download Python source:
```bash
cd /tmp
wget https://www.python.org/ftp/python/3.11.9/Python-3.11.9.tgz
tar -xzf Python-3.11.9.tgz
cd Python-3.11.9
```

2. Configure for portable installation:
```bash
./configure --prefix=/media/$USER/YOUR_USB_NAME/PortablePython/python \
    --enable-optimizations
```

3. Build and install:
```bash
make -j$(nproc)
make install
```

This will take 10-30 minutes depending on your system.

![Building Python](images/linux_10_building_python.png)
*Screenshot showing Python compilation process*

## Cross-Platform Considerations

### File Permissions

Linux requires execute permissions for scripts:
```bash
chmod +x run_script.sh
chmod +x PortablePython/python/bin/python3
```

### Mount Points

USB mount points vary by system:
- Ubuntu/Debian: `/media/$USER/USB_NAME`
- Raspberry Pi: `/media/pi/USB_NAME` or `/mnt/usb`
- Other distros: Check `/mnt` or `/media`

Use relative paths in your scripts to avoid mount point issues.

### Symbolic Links

Avoid using absolute symbolic links. Use relative paths instead:
```bash
# Bad (absolute)
ln -s /media/user/USB/PortablePython/python/bin/python3 python

# Good (relative)
cd /media/user/USB
ln -s PortablePython/python/bin/python3 python
```

## Troubleshooting

### Issue: "Permission denied" when running scripts
**Solution:**
```bash
chmod +x your_script.sh
chmod +x PortablePython/python/bin/python3
```

### Issue: "No such file or directory" for python3
**Solution:** Check the exact path:
```bash
find /media/$USER -name python3 -type f
```

### Issue: Libraries not found
**Solution:** Check Python's library path:
```bash
./python3 -c "import sys; print('\n'.join(sys.path))"
```

### Issue: Different architectures
**Solution:** You need separate USB drives or folders for each architecture:
```
USB/
├── PortablePython_x86_64/
├── PortablePython_armv7/
└── PortablePython_aarch64/
```

Create a detection script:
```bash
#!/bin/bash
ARCH=$(uname -m)
SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"

case $ARCH in
    x86_64)
        PYTHON="$SCRIPT_DIR/PortablePython_x86_64/python/bin/python3"
        ;;
    armv7l)
        PYTHON="$SCRIPT_DIR/PortablePython_armv7/python/bin/python3"
        ;;
    aarch64)
        PYTHON="$SCRIPT_DIR/PortablePython_aarch64/python/bin/python3"
        ;;
    *)
        echo "Unsupported architecture: $ARCH"
        exit 1
        ;;
esac

"$PYTHON" "$@"
```

### Issue: USB not mounting at same location
**Solution:** Use UUID or label-based mounting, or use relative paths exclusively.

## Benefits

- ✅ No installation required on target systems
- ✅ No root/sudo privileges needed (after initial setup)
- ✅ Consistent Python environment across different machines
- ✅ Easy to backup and share
- ✅ Works on Raspberry Pi and standard Linux distributions
- ✅ Isolated from system Python installations

## Performance Notes

- **USB 2.0**: Slower but works fine for most scripts
- **USB 3.0+**: Recommended for better performance
- **SD Card** (Raspberry Pi): Consider installing on the SD card instead if the Pi is dedicated

## Tips for Raspberry Pi

1. **Check your OS bit version:**
```bash
getconf LONG_BIT
```
Output `32` or `64` tells you which Python build to download.

2. **Power considerations:** USB 3.0 drives may need powered USB hub on older Raspberry Pi models.

3. **Auto-mount USB:** Add to `/etc/fstab` for consistent mounting (requires sudo).

---

# 日本語版

## 概要

このガイドでは、USBドライブにPython 3.11.9をセットアップし、root権限なしで任意のLinuxシステムやRaspberry Piで使用できる完全にポータブルな環境を作成する方法を説明します。

## 必要なもの

- USBドライブ（最小2GB推奨）
- LinuxコンピューターまたはRaspberry Pi
- Pythonとパッケージをダウンロードするためのインターネット接続
- 基本的なコマンドライン知識

## 重要な注意事項

### アーキテクチャの考慮事項

Linuxシステムは異なるCPUアーキテクチャを使用します：
- **x86_64 (AMD64)**: 標準的なデスクトップ/ノートパソコンLinux
- **ARM (armv7l)**: Raspberry Pi 3、Pi 4（32ビットOS）
- **ARM64 (aarch64)**: Raspberry Pi 3、Pi 4、Pi 5（64ビットOS）

**ターゲットシステムのアーキテクチャに一致するPythonバージョンをダウンロードする必要があります。**

## 方法1：ビルド済みPythonの使用（初心者に推奨）

### ステップ1：Python Standalone Buildのダウンロード

最も簡単な方法は、Python Standalone Buildsプロジェクトのビルド済みポータブルPythonを使用することです。

1. 訪問: https://github.com/indygreg/python-build-standalone/releases
2. Python 3.11.9の最新リリースを見つける
3. 適切なバージョンをダウンロード：

**標準Linux（x86_64）の場合：**
```
cpython-3.11.9+20240107-x86_64-unknown-linux-gnu-install_only.tar.gz
```

**Raspberry Pi 32ビット（armv7）の場合：**
```
cpython-3.11.9+20240107-armv7-unknown-linux-gnueabihf-install_only.tar.gz
```

**Raspberry Pi 64ビット（aarch64）の場合：**
```
cpython-3.11.9+20240107-aarch64-unknown-linux-gnu-install_only.tar.gz
```

![Standalone Buildのダウンロード](images/linux_01_download_standalone.png)
*Python Standalone Buildsリリースページを示すスクリーンショット*

### ステップ2：システムアーキテクチャの確認

ダウンロードする前に、システムアーキテクチャを確認します：

```bash
uname -m
```

出力の意味：
- `x86_64` → x86_64バージョンをダウンロード
- `armv7l` → armv7バージョンをダウンロード（32ビット）
- `aarch64` → aarch64バージョンをダウンロード（64ビット）

![アーキテクチャの確認](images/linux_02_check_arch.png)
*unameコマンドの出力を示すスクリーンショット*

### ステップ3：PythonをUSBに展開

1. USBドライブを挿入してマウント（通常は`/media/username/USB_NAME`として自動マウント）
2. Python用のディレクトリを作成：

```bash
cd /media/$USER/YOUR_USB_NAME
mkdir PortablePython
cd PortablePython
```

3. ダウンロードしたアーカイブを展開：

```bash
tar -xzf ~/Downloads/cpython-3.11.9+*-install_only.tar.gz
```

フォルダー構造は以下のようになります：
```
/media/user/USB/PortablePython/
└── python/
    ├── bin/
    │   ├── python3
    │   ├── python3.11
    │   └── pip3
    ├── lib/
    └── include/
```

![展開されたファイルLinux](images/linux_03_extracted_files.png)
*USB上の展開されたPythonファイルを示すスクリーンショット*

### ステップ4：Pythonインストールのテスト

```bash
cd /media/$USER/YOUR_USB_NAME/PortablePython/python/bin
./python3 --version
```

次のように表示されるはずです：`Python 3.11.9`

![Pythonバージョン確認](images/linux_04_python_version.png)
*Pythonバージョン確認の成功を示すスクリーンショット*

### ステップ5：pipインストールの確認

```bash
./python3 -m pip --version
```

pipはスタンドアロンビルドにすでに含まれているはずです。

![pipバージョンLinux](images/linux_05_pip_version.png)
*pipバージョンを示すスクリーンショット*

### ステップ6：追加ライブラリのインストール

```bash
# Pythonバイナリディレクトリに移動
cd /media/$USER/YOUR_USB_NAME/PortablePython/python/bin

# パッケージをインストール
./python3 -m pip install requests numpy pandas --user
```

**注：** `--user`フラグは、ポータブルPythonのsite-packagesディレクトリにパッケージをインストールします。

![パッケージインストールLinux](images/linux_06_install_packages.png)
*パッケージインストールを示すスクリーンショット*

### ステップ7：Pythonスクリプトの実行

#### 方法1：Bashスクリプトの使用

USBルートに`run_script.sh`を作成：

```bash
#!/bin/bash

# このスクリプトが配置されているディレクトリを取得
SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"

# Pythonパスを設定
PYTHON_BIN="$SCRIPT_DIR/PortablePython/python/bin/python3"

# Pythonスクリプトを実行
"$PYTHON_BIN" "$SCRIPT_DIR/my_project/main.py" "$@"

# オプション：ユーザー入力を待つ
read -p "Enterキーを押して終了..."
```

実行可能にする：
```bash
chmod +x run_script.sh
```

実行：
```bash
./run_script.sh
```

![BashスクリプトLinux](images/linux_07_bash_script.png)
*bashスクリプトの内容を示すスクリーンショット*

![Bash実行Linux](images/linux_08_running_bash.png)
*スクリプト実行を示すスクリーンショット*

#### 方法2：PythonスクリプトでShebangを使用

Pythonスクリプトに相対的なshebangを追加：

**my_script.py:**
```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

print("Linux上のポータブルPythonからこんにちは！")
```

ラッパースクリプト`run_my_script.sh`を作成：
```bash
#!/bin/bash
SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
export PATH="$SCRIPT_DIR/PortablePython/python/bin:$PATH"
python3 "$SCRIPT_DIR/my_script.py" "$@"
```

#### 方法3：エイリアスの作成（一時的）

現在のセッションのみ：
```bash
alias portablepython='/media/$USER/YOUR_USB_NAME/PortablePython/python/bin/python3'
portablepython my_script.py
```

### ステップ8：プロジェクト構造の例

```
/media/user/USB/
├── PortablePython/
│   └── python/
│       ├── bin/
│       │   ├── python3
│       │   └── pip3
│       └── lib/
├── my_project/
│   ├── main.py
│   ├── requirements.txt
│   └── data/
├── run_script.sh
└── README.md
```

![プロジェクト構造Linux](images/linux_09_project_structure.png)
*完全なフォルダー構造を示すスクリーンショット*

## 方法2：ソースからPythonをビルド（上級者向け）

この方法は完全な制御を提供しますが、より多くの時間が必要です。

### 前提条件

ビルド依存関係をインストール（sudoが必要）：
```bash
sudo apt-get update
sudo apt-get install build-essential zlib1g-dev libncurses5-dev \
    libgdbm-dev libnss3-dev libssl-dev libreadline-dev \
    libffi-dev libsqlite3-dev wget libbz2-dev
```

### ビルド手順

1. Pythonソースをダウンロード：
```bash
cd /tmp
wget https://www.python.org/ftp/python/3.11.9/Python-3.11.9.tgz
tar -xzf Python-3.11.9.tgz
cd Python-3.11.9
```

2. ポータブルインストール用に設定：
```bash
./configure --prefix=/media/$USER/YOUR_USB_NAME/PortablePython/python \
    --enable-optimizations
```

3. ビルドとインストール：
```bash
make -j$(nproc)
make install
```

システムによっては10〜30分かかります。

![Pythonのビルド](images/linux_10_building_python.png)
*Pythonコンパイルプロセスを示すスクリーンショット*

## クロスプラットフォームの考慮事項

### ファイルパーミッション

Linuxはスクリプトに実行権限が必要です：
```bash
chmod +x run_script.sh
chmod +x PortablePython/python/bin/python3
```

### マウントポイント

USBマウントポイントはシステムによって異なります：
- Ubuntu/Debian: `/media/$USER/USB_NAME`
- Raspberry Pi: `/media/pi/USB_NAME`または`/mnt/usb`
- その他のディストリビューション: `/mnt`または`/media`を確認

マウントポイントの問題を避けるため、スクリプトでは相対パスを使用してください。

### シンボリックリンク

絶対シンボリックリンクの
