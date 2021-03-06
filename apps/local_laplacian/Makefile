CXX ?= g++

ifdef BUILD_PREFIX
LIB_HALIDE = bin/$(BUILD_PREFIX)/libHalide.a
else
LIB_HALIDE = bin/libHalide.a
endif

LIBPNG_LIBS_DEFAULT = $(shell libpng-config --ldflags)
LIBPNG_CXX_FLAGS ?= $(shell libpng-config --cflags)
# Workaround for libpng-config pointing to 64-bit versions on linux even when we're building for 32-bit
ifneq (,$(findstring -m32,$(CXX)))
ifneq (,$(findstring x86_64,$(LIBPNG_LIBS_DEFAULT)))
LIBPNG_LIBS ?= -lpng
endif
endif
LIBPNG_LIBS ?= $(LIBPNG_LIBS_DEFAULT)
PNGFLAGS=$(LIBPNG_LIBS) $(LIBPNG_CXX_FLAGS)
UNAME = $(shell uname)

ifneq (,$(findstring ptx,$(HL_TARGET)))
  CUDA_LDFLAGS=-L/usr/lib/nvidia-current -lcuda
endif
ifneq (,$(findstring cuda,$(HL_TARGET)))
  CUDA_LDFLAGS=-L/usr/lib/nvidia-current -lcuda
endif
ifeq ($(UNAME), Darwin)
ifneq (, $(findstring opencl,$(HL_TARGET)))
  OPENCL_LDFLAGS=-framework OpenCL
endif
ifneq (, $(findstring spir,$(HL_TARGET)))
  OPENCL_LDFLAGS=-framework OpenCL
endif
ifneq (, $(findstring spir64,$(HL_TARGET)))
  OPENCL_LDFLAGS=-framework OpenCL
endif
else
ifneq (, $(findstring opencl,$(HL_TARGET)))
  OPENCL_LDFLAGS=-lOpenCL
endif
ifneq (, $(findstring spir,$(HL_TARGET)))
  OPENCL_LDFLAGS=-lOpenCL
endif
ifneq (, $(findstring spir64,$(HL_TARGET)))
  OPENCL_LDFLAGS=-lOpenCL
endif
endif

all: process

local_laplacian: ../../ local_laplacian.cpp
	$(MAKE) -C ../../ $(LIB_HALIDE)
	$(CXX) local_laplacian.cpp -I ../../include ../../$(LIB_HALIDE) -o local_laplacian -lpthread -ldl -O3

local_laplacian.o: local_laplacian
	./local_laplacian

process: process.cpp local_laplacian.o
	$(CXX) -I../support -Wall -O3 process.cpp local_laplacian.o -o process -lpthread -ldl $(PNGFLAGS) $(CUDA_LDFLAGS) $(OPENCL_LDFLAGS)

out.png: process
	./process ../images/rgb.png 8 1 1 out.png

clean:
	rm -f process local_laplacian.o local_laplacian
