# CANDOR-Bench 安装指南

本指南提供了多种安装方式，适用于不同的使用场景。

## 📋 系统要求

### 基础要求
- **操作系统**: Linux (推荐 Ubuntu 20.04+) 或 macOS
- **Python**: 3.8+
- **Git**: 2.25+
- **内存**: 最小 8GB，推荐 16GB+
- **磁盘空间**: 最小 10GB

### 编译工具链（如需编译C++算法）
- **CMake**: 3.15+
- **GCC/Clang**: 支持 C++17
- **Intel MKL** 或 **OpenBLAS** (用于 Faiss)

---

## 🚀 快速开始（仅Python算法）

如果只需要运行Python实现的算法或测试框架本身：

```bash
# 1. 克隆仓库（不包含子模块）
git clone --recursive https://github.com/DataSysResearch/CANDOR-Bench.git
cd CANDOR-Bench

# 2. 安装Python依赖
pip install -r requirements.txt

# 3. 运行测试
python -m pytest tests/ -v
```

**优点**: 快速启动，适合开发框架本身  
**缺点**: 无法使用C++算法实现

---

## 🔧 完整安装（包含所有算法）

### 方式1: 本地编译（推荐用于性能测试）

适用于需要测试cache性能、CPU性能等底层指标的场景。

#### 步骤1: 克隆仓库并初始化子模块

```bash
# 克隆主仓库
git clone --recursive https://github.com/DataSysResearch/CANDOR-Bench.git
cd CANDOR-Bench

# 初始化并更新所有子模块
git submodule update --init --recursive
```

#### 步骤2: 安装系统依赖

**Ubuntu/Debian:**
```bash
sudo apt-get update
sudo apt-get install -y \
    build-essential \
    cmake \
    git \
    python3-dev \
    libopenblas-dev \
    libomp-dev \
    swig
```

**macOS:**
```bash
brew install cmake libomp openblas swig
```

#### 步骤3: 安装Python依赖

```bash
# 创建虚拟环境（推荐）
python3 -m venv venv
source venv/bin/activate  # Linux/macOS
# Windows: venv\Scripts\activate

# 安装依赖
pip install --upgrade pip
pip install -r requirements.txt
```

#### 步骤4: 编译算法库

```bash
cd algorithms_impl

# 编译所有算法（需要较长时间）
./build.sh

# 或者单独编译某个算法
# ./build.sh faiss
# ./build.sh diskann
# ./build.sh gti
```

#### 步骤5: 验证安装

```bash
cd ..
python -c "import PyCANDYAlgo; print('✓ CANDY installed')"
python -c "import faiss; print('✓ Faiss installed')"
python -c "import diskannpy; print('✓ DiskANN installed')"
```

**优点**: 
- ✅ 完全访问硬件性能
- ✅ 可以测试cache miss、CPU性能等
- ✅ 最佳性能表现
- ✅ 支持所有算法

**缺点**: 
- ❌ 编译时间长（30-60分钟）
- ❌ 依赖系统环境
- ❌ 可能遇到编译问题

---

### 方式2: Docker（推荐用于功能测试和开发）

适用于快速部署和功能验证，但不适合精确的性能测试。

#### Docker限制说明

⚠️ **Docker容器中的性能测试限制**:
1. **CPU cache**: 容器可能共享宿主机cache，影响cache miss测试
2. **内存**: 容器内存限制可能影响大数据集测试
3. **I/O**: 容器文件系统可能影响磁盘I/O性能
4. **隔离**: 多容器运行会相互影响性能

**建议**: 
- Docker用于**功能测试**、**开发调试**、**CI/CD**
- 本地编译用于**性能基准测试**、**论文实验**

#### 使用Docker

```bash
# 构建镜像
docker build -t sage-db-bench .

# 运行容器
docker run -it --name sage-bench \
    -v $(pwd)/results:/app/results \
    sage-db-bench bash

# 在容器内运行测试
python tests/test_streaming.py
```

#### Docker Compose（多服务）

```yaml
# docker-compose.yml
version: '3.8'
services:
  sage-bench:
    build: .
    volumes:
      - ./results:/app/results
      - ./raw_data:/app/raw_data
    environment:
      - OMP_NUM_THREADS=4
    command: python -m bench.runner --config configs/test.yaml
```

运行:
```bash
docker-compose up
```

---

### 方式3: Conda环境（推荐跨平台）

适用于需要隔离环境但又想本地编译的场景。

```bash
# 创建conda环境
conda create -n sage-bench python=3.9
conda activate sage-bench

# 安装依赖
conda install numpy scipy pandas matplotlib pyyaml h5py
conda install -c conda-forge faiss-cpu  # 或 faiss-gpu
pip install gdown

# 克隆仓库
git clone --recursive https://github.com/DataSysResearch/CANDOR-Bench.git
cd CANDOR-Bench

# 编译其他算法
cd algorithms_impl
./build.sh
```

**优点**: 
- ✅ 环境隔离
- ✅ 跨平台支持好
- ✅ 可以使用预编译的faiss

---

## 📦 预编译二进制（未来支持）

计划提供预编译的Python wheels:

```bash
# 未来版本
pip install sage-db-bench[all]  # 包含所有算法
pip install sage-db-bench[faiss]  # 仅Faiss
pip install sage-db-bench[diskann]  # 仅DiskANN
```

---

## 🧪 验证安装

### 基础验证

```bash
# 检查Python包
python -c "import numpy, pandas, yaml, h5py; print('✓ Core packages OK')"

# 检查框架
python -c "from bench import BenchmarkRunner; print('✓ Framework OK')"

# 检查数据集
python -c "from datasets import DATASETS; print(f'✓ {len(DATASETS)} datasets available')"
```

### 算法验证

```bash
# 运行快速测试
python tests/test_algorithms.py

# 或手动测试某个算法
python << EOF
from bench.algorithms import get_algorithm
algo = get_algorithm('faiss_hnsw')
print(f'✓ Algorithm {algo.__class__.__name__} loaded')
EOF
```

### 完整测试套件

```bash
# 运行所有测试
pytest tests/ -v

# 运行特定测试
pytest tests/test_streaming.py -v
pytest tests/test_datasets.py -v
```

---

## 🔍 故障排除

### 子模块问题

```bash
# 如果子模块为空
git submodule update --init --recursive

# 如果子模块状态异常
git submodule foreach --recursive git reset --hard
git submodule update --remote
```

### 编译问题

```bash
# 清理构建缓存
cd algorithms_impl
rm -rf build/
./build.sh clean

# 查看详细编译日志
./build.sh 2>&1 | tee build.log
```

### Python导入问题

```bash
# 检查PYTHONPATH
export PYTHONPATH="${PYTHONPATH}:$(pwd)"

# 或使用开发模式安装
pip install -e .
```

### Faiss安装问题

```bash
# 如果pip安装faiss失败，使用conda
conda install -c conda-forge faiss-cpu

# 或使用特定numpy版本
pip install "numpy<2.0" faiss-cpu
```

---

## 📊 性能测试最佳实践

### 1. 系统配置

```bash
# 禁用CPU频率调节（需要root）
sudo cpupower frequency-set -g performance

# 禁用Turbo Boost（可选）
echo 1 | sudo tee /sys/devices/system/cpu/intel_pstate/no_turbo

# 设置CPU亲和性
taskset -c 0-7 python your_benchmark.py
```

### 2. 监控工具

```bash
# 安装性能监控工具
pip install psutil py-cpuinfo

# 使用perf监控cache miss（Linux）
perf stat -e cache-misses,cache-references python your_benchmark.py
```

### 3. 数据集准备

```bash
# 预下载数据集
python prepare_dataset.py --dataset sift
python prepare_dataset.py --dataset glove

# 计算ground truth
python compute_gt.py --dataset sift --runbook runbooks/baseline.yaml
```

---

## 🌐 在服务器上部署

### SSH部署

```bash
# 在本地
git bundle create sage-bench.bundle --all
scp sage-bench.bundle user@server:/tmp/

# 在服务器
cd /workspace
git clone /tmp/sage-bench.bundle sage-db-bench
cd sage-db-bench
git submodule update --init --recursive

# 按照本地编译步骤继续
```

### 使用screen/tmux进行长时间测试

```bash
# 启动screen会话
screen -S sage-bench

# 运行测试
python run_experiments.py

# 分离会话: Ctrl+A, D
# 重新连接: screen -r sage-bench
```

---

## 📚 下一步

安装完成后，请参考:
- [README.md](README.md) - 项目概述和快速开始
- [algorithms_impl/README.md](algorithms_impl/README.md) - 算法编译详细说明
- [USAGE.md](USAGE.md) - 使用指南和实验配置

---

## 🆘 获取帮助

- **Issues**: https://github.com/DataSysResearch/CANDOR-Bench/issues
- **Discussions**: https://github.com/DataSysResearch/CANDOR-Bench/discussions
- **Email**: [维护者邮箱]
