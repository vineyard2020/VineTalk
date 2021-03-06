CUDA_VERSION=`which nvcc | cut -f 4 -d '/'`
#`pkg-config --list-all | grep cuda | grep -v 'cudart'| cut -f 1 -d ' '`

CUDA_PATH = /usr/local/cuda-8.0#$(CUDA_VERSION)

HOST_COMPILER = g++
CC = gcc

NVCC := $(CUDA_PATH)/bin/nvcc -ccbin $(HOST_COMPILER) --default-stream per-thread

# internal flags
NVCCFLAGS = -std=c++11 -m64 -gencode arch=compute_30,code=sm_30 -gencode arch=compute_35,code=sm_35 \
-gencode arch=compute_37,code=sm_37 -gencode arch=compute_50,code=sm_50 \
-gencode arch=compute_52,code=sm_52 -gencode arch=compute_52,code=compute_52 \
-shared -Xcompiler -fPIC

CCFLAGS     := -std=c++11
LDFLAGS     :=


# Debug build flags
ifeq ($(dbg),1)
      NVCCFLAGS += -g -G
      BUILD_TYPE := debug
else
      BUILD_TYPE := release
endif

ALL_CCFLAGS :=
ALL_CCFLAGS += $(NVCCFLAGS)
ALL_CCFLAGS += $(EXTRA_NVCCFLAGS)
ALL_CCFLAGS += $(addprefix -Xcompiler ,$(CCFLAGS))
ALL_CCFLAGS += $(addprefix -Xcompiler ,$(EXTRA_CCFLAGS))

ALL_LDFLAGS :=
ALL_LDFLAGS += $(ALL_CCFLAGS)
ALL_LDFLAGS += $(addprefix -Xlinker ,$(LDFLAGS))
ALL_LDFLAGS += $(addprefix -Xlinker ,$(EXTRA_LDFLAGS))


################################################################################

SAMPLE_ENABLED := 1

# Gencode arguments
ifeq ($(TARGET_ARCH),armv7l)
SMS ?= 35 37 50 52
else
SMS ?= 35 37 50 52
endif

ifeq ($(SMS),)
$(info >>> WARNING - no SM architectures have been specified - waiving sample <<<)
SAMPLE_ENABLED := 0
endif

ifeq ($(GENCODE_FLAGS),)
# Generate SASS code for each SM architecture listed in $(SMS)
$(foreach sm,$(SMS),$(eval GENCODE_FLAGS += -gencode arch=compute_$(sm),code=sm_$(sm)))

# Generate PTX code from the highest SM architecture in $(SMS) to guarantee forward-compatibility
HIGHEST_SM := $(lastword $(sort $(SMS)))
ifneq ($(HIGHEST_SM),)
GENCODE_FLAGS += -gencode arch=compute_$(HIGHEST_SM),code=compute_$(HIGHEST_SM)
endif
endif

ALL_CCFLAGS += -po maxrregcount=16

ifeq ($(SAMPLE_ENABLED),0)
EXEC ?= @echo "[@]"
endif

################################################################################
CXX = $(HOST_COMPILER)

VINETALK  = ../../../vineyard/vine_talk
VINECONTR = ../../../vineyard/vine_controller
JOBGEN = ../../../vineyard/vine-applications/job_generator

INCLUDES  := -I../common/inc -I./include
INCLUDES += -I. -I$(CUDA_PATH)/include, -I$(CUDA_PATH)/samples/common/inc,  -I$(CUDA_PATH)/targets/x86_64-linux/include

INCLUDES += -I$(VINETALK)/build/include -I$(VINECONTR)/include -I$(JOBGEN)/include
#INCLUDES += -I$(VINETALK)/build/include -I$(VINECONTR)/include 
INCLUDES += -I$(CUDA_PATH)/include

LIBFLAGS = -shared -fPIC
NVCCLIB = --compiler-options '-fPIC' --shared

LCUDA = -L$(CUDA_PATH)/lib64 -lcuda -lcudart
#LFLVINE = -Wl,-rpath,$(VINETALK)/build -L$(VINETALK)/build -lvine -lrt
LFLVINE = -lvine -lrt
LDFLAGS += -lc -ldl -lm

all: clean BlackScholes.o vine_darkGray.o vine_convolutionSeparable.o vine_histogram.o vine_MonteCarloMultiGPU.o 

BlackScholes.o: src/BlackScholes.cpp
	$(NVCC) $(NVCCFLAGS) $(INCLUDES)  $< $(LFLVINE) -o objects/$@

vine_convolutionSeparable.o: src/vine_convolutionSeparable.cpp
	$(NVCC) $(NVCCFLAGS) $(INCLUDES)  $< $(LFLVINE) -o objects/$@

vine_histogram.o: src/vine_histogram.cpp
	$(NVCC) $(NVCCFLAGS) $(INCLUDES)  $< $(LFLVINE) -o objects/$@

vine_MonteCarloMultiGPU.o: src/vine_MonteCarloMultiGPU.cpp
	$(NVCC) $(NVCCFLAGS) $(INCLUDES)  $< $(LFLVINE) -o objects/$@

#c_darkGray.o: src/c_darkGray.cpp
#	$(NVCC) $(NVCCFLAGS) $(INCLUDES)  $< $(LFLVINE) -o objects/$@

#cu_darkGray.o: src/cu_darkGray.cu
#	$(NVCC) $(NVCCFLAGS) --ptxas-options=-v -DLIBRARY_BUILD  $(INCLUDES)  $< $(LFLVINE) -o objects/$@

vine_darkGray.o: src/vine_darkGray.cpp
	$(NVCC) $(NVCCFLAGS) -L/usr/X11R6/lib -lm -lpthread -lX11 $(INCLUDES)  $< $(LFLVINE) -o objects/$@

	


clean:
	rm -rf objects/*.o
