# Reduce ION memory size for minimal multimedia use

## 修改说明

针对 SG2002 LicheeRV Nano SD 版本，将 ION（多媒体内存）从 105MB 缩减为 30MB，适用于不需要复杂多媒体功能（如编码、ISP 处理）的轻量级使用场景，可释放出约 75MB 给 Linux 系统使用。

### 修改的文件

**[build/boards/sg200x/sg2002_licheervnano_sd/memmap.py](file:///root/LicheeRV-Nano-Build-Less-ion-memory-/build/boards/sg200x/sg2002_licheervnano_sd/memmap.py#L43)**

- 第 43 行：`ION_SIZE = 105 * SIZE_1M` → `ION_SIZE = 30 * SIZE_1M`

### 内存布局对比

| 区域 | 修改前 (105MB) | 修改后 (30MB) |
|------|---------------|---------------|
| ION 起始地址 | 0x89500000 | 0x8e000000 |
| ION 大小 | 105 MB | 30 MB |
| ION 结束地址 | 0x8fdfffff | 0x8fdfffff |
| H26X 码流缓冲区 | 2 MB | 2 MB |
| ISP 内存 | 20 MB | 20 MB |
| FreeRTOS 保留 | 2 MB | 2 MB |
| 可给 Linux 使用内存 | ~120 MB | ~195 MB |

### 验证结果

- ✅ 内存布局无重叠，所有 assert 检查通过
- ✅ u-boot (FSBL + OpenSBI + U-Boot) 编译成功
- ✅ Linux 内核 5.10 编译成功
- ✅ 设备树编译成功
- ✅ OSDRV 内核模块编译成功

## 构建方法

### 1. 安装系统依赖

```bash
sudo apt-get update
sudo apt-get install -y build-essential gcc g++ make bison flex libssl-dev \
    libelf-dev device-tree-compiler bc python3 python3-pyelftools git wget
```

### 2. 获取源码（含子模块）

```bash
git clone <your-fork-url> LicheeRV-Nano-Build
cd LicheeRV-Nano-Build
# 如果已经 clone 了，初始化子模块
git submodule update --init --recursive
```

### 3. 配置并编译

```bash
# 加载构建环境
source build/cvisetup.sh

# 选择板级配置
defconfig sg2002_licheervnano_sd

# 全量编译
build_all
```

### 4. 常用编译命令

| 命令 | 说明 |
|------|------|
| `build_uboot` | 只编译 u-boot |
| `build_kernel` | 只编译内核 |
| `build_middleware` | 只编译中间件 |
| `build_buildroot` | 只编译 buildroot 根文件系统 |
| `clean_uboot` | 清理 u-boot |
| `clean_kernel` | 清理内核 |
| `clean_all` | 清理所有编译产物（耗时较长） |

## 注意事项

### 并行编译竞态条件

如果在 **clean 后第一次编译** 时遇到以下错误：
```
fatal error: linux/cvi_type.h: No such file or directory
fatal error: linux/vi_snsr.h: No such file or directory
```

这是并行编译的竞态条件问题（ISP sensor 驱动在内核头文件安装完成前就开始编译），**不是代码错误**。直接**重新运行一次 `build_all`** 即可解决。

### ION 大小调整适用场景

- **30MB（当前配置）**：仅使用基础摄像头预览、无视频编码/解码、无复杂 ISP 算法
- **105MB（默认）**：完整多媒体功能，支持 H.264/H.265 编码、ISP 处理等

如果你的应用需要：
- 视频编码（H.264/H.265）
- 多摄像头
- 复杂的 ISP 图像处理
- 视频解码播放

请适当增大 ION_SIZE。

### 输出文件位置

编译成功后，镜像文件在：
```
install/soc_sg2002_sg2002_licheervnano_sd/
```

## 参考

- 原始版本对比：https://github.com/sipeed/LicheeRV-Nano-Build/compare/20240813...20240816（从 75MB 调整到 105MB）
