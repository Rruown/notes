# CMake项目搭建

一个好的项目层次化结构清晰、兼容性好（可配置）

## 组织结构

```
project
--cmake
----function.cmake
----system.cmake
----common.cmake
----...cmake
--Master
----Module1
------CMakeLists.txt
----Module2
------CMakeLists.txt
----...
--CMakeLists.txt
```

