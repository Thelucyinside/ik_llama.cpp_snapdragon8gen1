
# Android

## Build on Android using Termux
[Termux](https://github.com/termux/termux-app#installation) is a powerful terminal emulator and Linux environment for Android that allows you to build and run `llama.cpp` directly on your device without needing root access.

**1. Install Termux and Required Packages:**

   First, install Termux from F-Droid or GitHub. After installation, open Termux and run the following commands to update packages and install necessary build tools:
   ```bash
   pkg update && pkg upgrade -y
   pkg install git cmake clang make
   ```

**2. Get the Code:**

   Clone the `llama.cpp` repository:
   ```bash
   git clone https://github.com/ggerganov/llama.cpp.git
   cd llama.cpp
   ```

**3. Build `llama.cpp`:**

   Create a build directory and compile the project using CMake:
   ```bash
   mkdir build
   cd build
   cmake ..
   make -j$(nproc)
   ```
   This will build the necessary binaries (like `main`, `server`, etc.) in the `build/bin` directory. The build process should automatically detect and enable optimizations for your device's CPU, including Snapdragon 8 Gen 1 and similar processors with DOTPROD support.

**4. Download a Model:**

   You'll need a GGUF-formatted model file. You can download one from Hugging Face or other sources. For example, to download a small test model:
   ```bash
   # From the llama.cpp/build directory
   cd ../
   # Example: Download a small Phi-3 model (replace with your desired model)
   curl -L -o models/phi-3-mini-4k-instruct.Q4_K_M.gguf https://huggingface.co/microsoft/Phi-3-mini-4k-instruct-gguf/resolve/main/Phi-3-mini-4k-instruct.Q4_K_M.gguf?download=true
   # Ensure the models directory exists
   mkdir -p models
   # If you downloaded it elsewhere, move it to the models directory or your preferred location.
   ```
   It's often recommended to store models in Termux's home directory (`~/`) or a subdirectory for easy access.

**5. Run `llama.cpp`:**

   Navigate to the directory containing the executables and run them. For example, to run the `main` example for inference:
   ```bash
   cd build/bin
   ./main -m ../../models/phi-3-mini-4k-instruct.Q4_K_M.gguf -n 256 --color -i -ins
   ```
   Replace `phi-3-mini-4k-instruct.Q4_K_M.gguf` with the path to your downloaded model.

   To run the server:
   ```bash
   ./server -m ../../models/phi-3-mini-4k-instruct.Q4_K_M.gguf -c 4096
   ```

**Troubleshooting:**

*   **Storage Access:** If you need to access files on your shared storage (e.g., SD card or Downloads folder), run `termux-setup-storage` in Termux and grant the necessary permissions.
*   **Performance:** Building directly in Termux provides good performance. Ensure your device has sufficient RAM and storage. Background processes on Android can sometimes interfere; closing unnecessary apps might help.
*   **CPU Features:** The build process is designed to automatically detect CPU features. If you encounter "Illegal instruction" errors with pre-built binaries or suspect an issue, rebuilding from source within Termux is the best way to ensure compatibility.

## Building the Project using Android NDK
Obtain the [Android NDK](https://developer.android.com/ndk) and then build with CMake.

Execute the following commands on your computer to avoid downloading the NDK to your mobile. Alternatively, you can also do this in Termux:
```
$ mkdir build-android
$ cd build-android
$ export NDK=<your_ndk_directory>
$ cmake -DCMAKE_TOOLCHAIN_FILE=$NDK/build/cmake/android.toolchain.cmake -DANDROID_ABI=arm64-v8a -DANDROID_PLATFORM=android-23 -DCMAKE_C_FLAGS=-march=armv8.4a+dotprod ..
$ make
```

Install [termux](https://github.com/termux/termux-app#installation) on your device and run `termux-setup-storage` to get access to your SD card (if Android 11+ then run the command twice).

Finally, copy these built `llama` binaries and the model file to your device storage. Because the file permissions in the Android sdcard cannot be changed, you can copy the executable files to the `/data/data/com.termux/files/home/bin` path, and then execute the following commands in Termux to add executable permission:

(Assumed that you have pushed the built executable files to the /sdcard/llama.cpp/bin path using `adb push`)
```
$cp -r /sdcard/llama.cpp/bin /data/data/com.termux/files/home/
$cd /data/data/com.termux/files/home/bin
$chmod +x ./*
```

Download model [llama-2-7b-chat.Q4_K_M.gguf](https://huggingface.co/TheBloke/Llama-2-7B-Chat-GGUF/blob/main/llama-2-7b-chat.Q4_K_M.gguf), and push it to `/sdcard/llama.cpp/`, then move it to `/data/data/com.termux/files/home/model/`

```
$mv /sdcard/llama.cpp/llama-2-7b-chat.Q4_K_M.gguf /data/data/com.termux/files/home/model/
```

Now, you can start chatting:
```
$cd /data/data/com.termux/files/home/bin
$./llama-cli -m ../model/llama-2-7b-chat.Q4_K_M.gguf -n 128 -cml
```

Here's a demo of an interactive session running on Pixel 5 phone:

https://user-images.githubusercontent.com/271616/225014776-1d567049-ad71-4ef2-b050-55b0b3b9274c.mp4
