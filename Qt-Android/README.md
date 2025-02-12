### ✅ **GitHub Repository Description (English & Persian)**

---

## **📌 English Description**  

### **Qt Android Build Environment (Docker-based)**
This repository provides a **fully automated and dynamic Docker environment** for **building Qt-based Android applications**.  
It eliminates the hassle of manually configuring the Android SDK, NDK, Qt, and other dependencies.  

🔹 **Key Features:**  
✔ **Dynamic project selection** – The project name can be specified at build time.  
✔ **Pre-installed Qt 6.5.2 for both desktop & Android** – No need to install manually.  
✔ **Android SDK & NDK included** – Fully set up and ready for Qt Android builds.  
✔ **Minimal setup required** – Just build and run the container!  
✔ **Automatic `.apk` generation** – Compiles and outputs a ready-to-install APK file.  

---

### **🚀 How to Use**
1️⃣ **Build the Docker Image**  
Replace `<your_project_name>` with the actual project name:  
```sh
docker build --build-arg PROJECT_NAME=<your_project_name> -t qt-android-builder .
```

2️⃣ **Run the Container to Build the APK**  
```sh
docker run --rm -v $(pwd)/output:/workspace/output qt-android-builder
```

3️⃣ **Install APK on a Device**  
```sh
adb install output/app-debug.apk
```

---

### **📂 Project Structure**
```
📦 qt-android-builder
 ┣ 📜 Dockerfile
 ┣ 📜 README.md
 ┗ 📂 <your_project_name>
    ┗ 📜 <your_project_name>.pro
```

---

### **💡 Notes**
- This setup **requires Docker** to be installed on your system.  
- The `PROJECT_NAME` argument allows multiple projects to be built dynamically.  
- The resulting APK will be saved in the `output` folder.  

🚀 **Now you can build Qt Android projects dynamically with ease!** 🎉

---
