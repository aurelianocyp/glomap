# GLOMAP: Global Structure-from-Motion Revisited

[Project page](https://lpanaf.github.io/eccv24_glomap/) | [Paper](https://arxiv.org/pdf/2407.20219)
---
bug：
- Failed to find Ceres - Found Eigen dependency, but the version of Eigen found (3.4.0) does not exactly match the version of Eigen Ceres was compiled with (3.3.4)
    - https://blog.csdn.net/weixin_44003484/article/details/123460836

## Getting Started
首先配置colmap：git clone https://github.com/colmap/colmap

colmap中需要一个 libgmock-dev库。安装方式：https://www.cnblogs.com/dinghou/p/15350690.html
```shell
sudo apt-get install \
    git \
    cmake \
    ninja-build \
    build-essential \
    libboost-program-options-dev \
    libboost-filesystem-dev \
    libboost-graph-dev \
    libboost-system-dev \
    libeigen3-dev \
    libflann-dev \
    libfreeimage-dev \
    libmetis-dev \
    libgoogle-glog-dev \
    libgtest-dev \
    libsqlite3-dev \
    libglew-dev \
    qtbase5-dev \
    libqt5opengl5-dev \
    libcgal-dev \
    libceres-dev
```
源码安装LZ4库：https://github.com/lz4/lz4
```shell
mkdir build
cd build
cmake .. -GNinja
ninja
sudo ninja install
```
Run COLMAP:
```shell
colmap -h
colmap gui
```
install glomap：

更换eigen从3.3.4到3.4版本：https://blog.csdn.net/CC977/article/details/122972719

卸载ceres更换3.4eigen编译的ceres2.0： sudo apt-get install  libceres-dev

https://github.com/siyandong/ceres-solver

更换完毕后还需要做一些小修改：https://blog.csdn.net/weixin_44003484/article/details/123460836（注意eigen位置）
```shell
mkdir build
cd build
cmake .. -GNinja
ninja && ninja install
```
After installation, one can run GLOMAP by (starting from a database)
```shell
glomap mapper --database_path DATABASE_PATH --output_path OUTPUT_PATH --image_path IMAGE_PATH
```
For more details on the command line interface, one can type `glomap -h` or `glomap mapper -h` for help.

We also provide a guide on improving the obtained reconstruction, which can be found [here](docs/getting_started.md)

Note:
- GLOMAP depends on two external libraries - [COLMAP](https://github.com/colmap/colmap) and [PoseLib](https://github.com/PoseLib/PoseLib).
  With the default setting, the library is built automatically by GLOMAP via `FetchContent`.
  However, if a self-installed version is preferred, one can also disable the `FETCH_COLMAP` and `FETCH_POSELIB` CMake options.
- To use `FetchContent`, the minimum required version of `cmake` is 3.28. If a self-installed version is used, `cmake` can be downgraded to 3.10.
- If your system does not provide a recent enough CMake version, you can install it as:
  ```shell
  wget https://github.com/Kitware/CMake/releases/download/v3.30.1/cmake-3.30.1.tar.gz
  tar xfvz cmake-3.30.1.tar.gz && cd cmake-3.30.1
  ./bootstrap && make -j$(nproc) && sudo make install
  ```

## End-to-End Example

In this section, we will use datasets from [this link](https://demuc.de/colmap/datasets) as examples.
Download the datasets and put them under `data` folder.

### From database

If a COLMAP database already exists, GLOMAP can directly use it to perform mapping:
```shell
glomap mapper \
    --database_path ./data/database.db \
    --image_path    ./data/images \
    --output_path   ./output/sparse
```

### From images

To obtain a reconstruction from images, the database needs to be established
first. Here, we utilize the functions from COLMAP:
```shell
colmap feature_extractor \
    --image_path    ./data/south-building/images \
    --database_path ./data/south-building/database.db
colmap exhaustive_matcher \
    --database_path ./data/south-building/database.db 
glomap mapper \
    --database_path ./data/south-building/database.db \
    --image_path    ./data/south-building/images \
    --output_path   ./output/south-building/sparse
```

### Visualize and use the results

The results are written out in the COLMAP sparse reconstruction format. Please
refer to [COLMAP](https://colmap.github.io/format.html#sparse-reconstruction)
for more details.

The reconstruction can be visualized using the COLMAP GUI, for example:
```shell
colmap gui --import_path ./output/sparse/0 --database_path ./data/database.db --image_path    ./data/images
```
Alternatives like [rerun.io](https://rerun.io/examples/3d-reconstruction/glomap)
also enable visualization of COLMAP and GLOMAP outputs.

If you want to inspect the reconstruction programmatically, you can use
`pycolmap` in Python or link against COLMAP's C++ library interface.

### Notes

- For larger scale datasets, it is recommended to use `sequential_matcher` or
  `vocab_tree_matcher` from `COLMAP`.
```shell
colmap sequential_matcher --database_path DATABASE_PATH
colmap vocab_tree_matcher --database_path DATABASE_PATH --VocabTreeMatching.vocab_tree_path VOCAB_TREE_PATH
```
- Alternatively, one can use
  [hloc](https://github.com/cvg/Hierarchical-Localization/) for image retrieval
  and matching with learning-based descriptors.



## Acknowledgement

We are highly inspired by COLMAP, PoseLib, Theia. Please consider also citing
them, if using GLOMAP in your work.

## notes
Windows使用指南。下载后进入glomap.exe所在的位置 .\glomap.exe mapper --database_path D:\AAA\files\code\python_code\a6000\workspace\nerfstudio-webui\data\workspace\ys1_1920_1080\2025_1_2_14229\ys1_1920_1080_377\colmap\database.db --output_path C:/Users/15061/Desktop/glomap-x64-windows/output --image_path D:AAA/files/code/python_code/a6000/workspace/nerfstudio-webui/data/workspace/验收1_1920_1080/2025_1_2_14229/验收1_1920_1080/images。目前只能用mapper，且Windows只能用命令行。
