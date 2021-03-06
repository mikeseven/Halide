PNGFLAGS=$(shell libpng-config --ldflags) $(shell libpng-config --cflags)
ifeq ($(HL_TARGET),ptx)
  SCHEDULE=100
  CUDA_LFLAGS=-L/usr/local/cuda/lib -lcuda
else
  SCHEDULE=0
endif

ifeq ($(HL_TARGET),opencl)
  SCHEDULE=100
  OPENCL_LDFLAGS=-framework opencl
else
  SCHEDULE=0
endif

ifdef BUILD_PREFIX
LIB_HALIDE = bin/$(BUILD_PREFIX)/libHalide.a
else
LIB_HALIDE = bin/libHalide.a
endif

all: process

camera_pipe: ../../ camera_pipe.cpp
	$(MAKE) -C ../../ $(LIB_HALIDE)
	$(CXX) camera_pipe.cpp -I ../../include/ ../../$(LIB_HALIDE) -o camera_pipe -ldl -lpthread

curved.o: camera_pipe
	./camera_pipe 8 0 # 8-bit output,

fcam/Demosaic.o: fcam/Demosaic.cpp fcam/Demosaic.h
	$(CXX) -c -I../support -Wall -fopenmp -O3 $< -o $@

fcam/Demosaic_ARM.o: fcam/Demosaic_ARM.cpp fcam/Demosaic_ARM.h
#	$(CXX) -c -mfpu=neon -I../support -Wall -fopenmp -O3 $< -o $@
	$(CXX) -c -I../support -Wall -fopenmp -O3 $< -o $@

process: process.cpp curved.o fcam/Demosaic.o fcam/Demosaic_ARM.o
	$(CXX) -I../support -Wall -O3 $^ -o $@ -lpthread -ldl -fopenmp $(CUDA_LFLAGS) $(OPENCL_LDFLAGS) $(PNGFLAGS)

out.png: process 
	./process ../images/bayer_raw.png 3700 2.0 50 out.png

clean:
	rm -f out.png process curved.o camera_pipe
