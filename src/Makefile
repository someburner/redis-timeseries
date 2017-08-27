#####################
# redis-timeseries
#####################

SO_TARGET = tsdb-module.so
# SO_TARGET = redis-tsdb-module.so

SO_OUTPUT = tsdb.so

#####################
# Debug Options
#####################
DEBUG_FLAGS = -g -ggdb -O2


#####################
# Directory Setup
#####################
# Get absolute paths for src, project
SRC_DIR = $(shell pwd -P)
PROJ_ROOT = $(subst /src,,$(abspath $(SRC_DIR)))

# SDK location - change this if RedisModulesSDK is located elsewhere
RMSDK_DIR = $(PROJ_ROOT)/RedisModulesSDK
RMSDK_UTIL_DIR = $(RMSDK_DIR)/rmutil

# Set environment variable to the location of redismodule.h
# NOTE: Must be 'RMUTIL_LIBDIR' as it is checked for by RedisModulesSDK
RMUTIL_LIBDIR = $(RMSDK_UTIL_DIR)
RMUTIL_LIB = $(RMSDK_UTIL_DIR)/librmutil.a

#####################
# Build flags
#####################
# find the OS
uname_S := $(shell sh -c 'uname -s 2>/dev/null || echo not')
# Compile flags for linux / osx
ifeq ($(uname_S),Linux)
	SHOBJ_CFLAGS ?= -fno-common -g -ggdb
	SHOBJ_LDFLAGS ?= -shared -Bsymbolic -Bsymbolic-functions
else
	SHOBJ_CFLAGS ?= -dynamic -fno-common -g -ggdb
	SHOBJ_LDFLAGS ?= -bundle -undefined dynamic_lookup
endif

CFLAGS = -Wall $(DEBUG_FLAGS) -fPIC -std=gnu99 -D_GNU_SOURCE -DREDIS_MODULE_TARGET

# NOTE: export these to submake (SDK)
ifeq ($(MAKECMDGOALS),rmutil)
export CFLAGS
export DEBUG_FLAGS
endif

#####################
# Sources
#####################
# INCLUDE_DIRS = -I$(RMSDK_DIR) -I$(RMSDK_UTIL_DIR)
INCLUDE_DIRS = -I$(RMSDK_DIR)

# Find all .c files
C_SRCS = $(wildcard $(SRC_DIR)/*.c) $(wildcard $(SRC_DIR)/dep/*.c)
# And generate corresponding .o files
C_OBJS = $(patsubst $(SRC_DIR)/%.c, $(SRC_DIR)/%.o, $(C_SRCS))
# External libraries
LIBS = -lrmutil -lc -lm

# Compiler
CC=gcc
CFLAGS += $(INCLUDE_DIRS)

#####################
# Directives
#####################
.PHONY: lib libtsdb.a test docker

all: rmutil $(SO_TARGET) postbuild

rmutil: FORCE
	$(MAKE) -C $(RMSDK_DIR) all
.PHONY: rmutil

postbuild:
	cp -f $(SO_TARGET) $(SO_OUTPUT)
	$(MAKE) clean

$(SO_TARGET): rmutil $(C_OBJS)
	$(LD) -o $@ $(C_OBJS) $(SHOBJ_LDFLAGS) -L$(RMSDK_UTIL_DIR) $(LIBS)

libtsdb.a: rmutil $(C_OBJS)
	ar rcs $@ $(C_OBJS)

lib: libtsdb.a

cleansdk:
	$(MAKE) -C $(RMSDK_DIR) clean

clean: cleansdk
	rm -rvf *.xo $(SO_TARGET) *.o *.a

test: $(SO_TARGET)
	python2 -m pytest .

docker:
	cd .. && docker build -t redis-tsdb .

FORCE:
