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
    --database_path ./data/gerrard-hall/database.db \
    --image_path    ./data/gerrard-hall/images \
    --output_path   ./output/gerrard-hall/sparse
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
colmap gui --import_path ./output/south-building/sparse/0
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

## Support

Please, use GitHub Discussions at https://github.com/colmap/glomap/discussions
for questions and the GitHub issue tracker at https://github.com/colmap/glomap
for bug reports, feature requests/additions, etc.

## Contribution

Contributions (bug reports, bug fixes, improvements, etc.) are very welcome and
should be submitted in the form of new issues and/or pull requests on GitHub.

## License

```
Copyright (c) 2024, ETH Zurich.
All rights reserved.

Redistribution and use in source and binary forms, with or without
modification, are permitted provided that the following conditions are met:

    * Redistributions of source code must retain the above copyright
      notice, this list of conditions and the following disclaimer.

    * Redistributions in binary form must reproduce the above copyright
      notice, this list of conditions and the following disclaimer in the
      documentation and/or other materials provided with the distribution.

    * Neither the name of ETH Zurich nor the names of its contributors may
      be used to endorse or promote products derived from this software
      without specific prior written permission.

THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE
ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDERS OR CONTRIBUTORS BE
LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR
CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF
SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS
INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN
CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE)
ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE
POSSIBILITY OF SUCH DAMAGE.
```
