# MACE CPU / GPU for TinkerBoard Rockchip RK3288 Mali-T764 GPU

I compiled this on the TinkerBoard with chip rk3288 running TinkerOS 2.0.7.

The files / libs in this fork are from my compilation efforts for the RK3288.

## Issues
- CPU Demo crashes sometimes, could be due to build options:
```
CPPFLAGS := -mfloat-abi=hard -mfpu=neon
```

I installed the current env requirements listed here:
https://mace.readthedocs.io/en/latest/installation/env_requirement.html

No bazel, no update to cmake just the below packages.

```
sudo apt-get install python-setuptools
pip install -I jinja2==2.10 2.10
pip install -I pyyaml==3.12 3.12.0
pip install -I sh==1.12.14  1.12.14
pip install -I numpy==1.14.0
pip install -I six==1.11.0
```

Following the instructions from the original repo:
https://github.com/zhy520xp/mace-makefile-project

## Compile Protobuf 3.4.0
Download this:
https://cnbj1.fds.api.xiaomi.com/mace/third-party/protobuf/protobuf-3.4.0.zip

Unzip and follow as per instructions:
https://github.com/protocolbuffers/protobuf/tree/master/src

```
$ sudo apt-get install autoconf automake libtool curl make g++ unzip
$ unzip protobuf-3.4.0.zip
$ cd protobuf-3.4.0
$ ./autogen.sh
$ ./configure
$ make
```

Copy Protobuf library into mace path:
```
$ cp ./protobuf-3.4.0/src/.libs/libprotobuf-lite.a ./mace-makefile-project/library/protobuf3.4.0/libprotobuf-lite.a
```

## Compile Mace with GPU support
```
$ cd mace-makefile-project/mace
```

Change the Makefile to use the local TinkerBoard toolchain:
```
-CXX = aarch64-himix100-linux-g++
-CC = aarch64-himix100-linux-gcc
-AR = aarch64-himix100-linux-ar cqs
+CXX = g++
+CC = gcc
+AR = ar cqs
```

Clean and make:
```
$ make clean; make
```

Copy libmace.a to library gpu folder:
```
$ cp libmace.a ../library/mace_gpu
```

## Compile MACE with CPU

For CPU mode, change the Makefile setting to CPU mode and add the CFLAGS:
```
-PLATFORM = GPU
+PLATFORM = CPU

-CPPFLAGS := -O3 -std=c++11 -fpermissive -DMACE_ENABLE_NEON -DMACE_ENABLE_OPENMP -fopenmp
+CPPFLAGS := -O3 -std=c++11 -fpermissive -DMACE_ENABLE_NEON -DMACE_ENABLE_OPENMP -fopenmp -mfloat-abi=hard -mfpu=neon
```

Clean and make:
```
$ make clean; make
```

Copy libmace.a to library cpu folder
```
$ cp libmace.a ../library/mace_cpu
```

## OpenCL and Mali libraries

Copy system libOpenCL.so and libmali.so libs:
```
$ cp /usr/lib/arm-linux-gnueabihf/libOpenCL.so ./mace-makefile-project/library/opencl/libOpenCL.so
$ cp /usr/lib/arm-linux-gnueabihf/libOpenCL.so ./mace-makefile-project/unit_test_gpu/opencl_library/libOpenCL.so
$ cp /usr/lib/arm-linux-gnueabihf/libmali.so ./mace-makefile-project/library/opencl/libmali.so
$ cp /usr/lib/arm-linux-gnueabihf/libmali.so ./mace-makefile-project/unit_test_gpu/opencl_library/libmali.so
```

## Run GPU Test
Go into folder:
```
$ cd ./unit_test_gpu
```

Edit Makefile:
```
-CXX = aarch64-himix100-linux-g++
-CC = aarch64-himix100-linux-gcc
-AR = aarch64-himix100-linux-ar cqs
+CXX = g++
+CC = gcc
+AR = ar cqs
```

Clean and make:
```
$ make clean; make
```

Run Demo:
```
$ ./demo
```

You should see successful output like:
```
linaro@tinkerboard:~/mace-makefile-project/unit_test_gpu$ ./demo
W ../mace/core/runtime/opencl/opencl_runtime.cc:40] Set GPU configurations, gpu_perf_hint: 3, gpu_priority_hint: 3
=====>>>>>start to CreateMaceEngineFromProto
I ../mace/core/mace.cc:337] Create MaceEngine from model pb
I ../mace/core/mace.cc:135] Initializing MaceEngine
W ../mace/core/runtime/opencl/opencl_wrapper.cc:283] Loading OpenCL from ./opencl_library/libOpenCL.so
W ../mace/core/runtime/opencl/opencl_runtime.cc:356] Using device: Mali-T760
I ../mace/utils/tuner.h:129] There is no tuned parameters.
W ../mace/core/runtime/opencl/opencl_runtime.cc:428] There is no precompiled OpenCL binary in all OpenCL binary paths
=====>>>>>end to CreateMaceEngineFromProto
=====>>>>>Warm up Run Model spend time:1412 ms
=====>>>>>Normal Run Model spend time:167 ms
0.746094 ,0.128662 ,
I ../mace/core/mace.cc:194] Destroying MaceEngine
```


## Run CPU Test
Go into folder:
```
$ cd ./unit_test_cpu
```

Edit Makefile:
```
-CXX = aarch64-himix100-linux-g++
-CC = aarch64-himix100-linux-gcc
-AR = aarch64-himix100-linux-ar cqs
+CXX = g++
+CC = gcc
+AR = ar cqs
```

Clean and make:
```
$ make clean; make
```

Run Demo:
```
$ ./demo
```

You should see successful output like:
```
linaro@tinkerboard:/home/linaro/mace-makefile-project/unit_test_cpu# ./demo
=====>>>>>start to CreateMaceEngineFromProto
I ../mace/core/mace.cc:337] Create MaceEngine from model pb
I ../mace/core/mace.cc:135] Initializing MaceEngine
=====>>>>>end to CreateMaceEngineFromProto
=====>>>>>Warm up Run Model spend time:513 ms
=====>>>>>Normal Run Model spend time:171 ms
0.742202 ,0.134289 ,
I ../mace/core/mace.cc:194] Destroying MaceEngine
```
