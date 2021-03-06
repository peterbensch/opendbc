CC = clang
CXX = clang++

BASEDIR = ../..
PHONELIBS := ../../phonelibs

UNAME_S := $(shell uname -s)
UNAME_M := $(shell uname -m)

WARN_FLAGS = -Werror=implicit-function-declaration \
             -Werror=incompatible-pointer-types \
             -Werror=int-conversion \
             -Werror=return-type \
             -Werror=format-extra-args \
             -Wno-deprecated-declarations

CFLAGS = -std=gnu11 -g -fPIC -O2 $(WARN_FLAGS)
CXXFLAGS = -std=c++11 -g -fPIC -O2 $(WARN_FLAGS)
LDFLAGS =

ifeq ($(ARCH),aarch64)
CFLAGS += -mcpu=cortex-a57
CXXFLAGS += -mcpu=cortex-a57
endif

OBJDIR = obj

OPENDBC_PATH := $(shell python3 -c 'import opendbc; print(opendbc.DBC_PATH)')

DBC_SOURCES := $(sort $(wildcard $(OPENDBC_PATH)/*.dbc))
DBC_OBJS := $(patsubst $(OPENDBC_PATH)/%.dbc,$(OBJDIR)/%.o,$(DBC_SOURCES))
DBC_CCS := $(patsubst $(OPENDBC_PATH)/%.dbc,dbc_out/%.cc,$(DBC_SOURCES))
.SECONDARY: $(DBC_CCS)

LIBDBC_OBJS := $(OBJDIR)/dbc.o $(OBJDIR)/parser.o $(OBJDIR)/packer.o $(OBJDIR)/common.o

CWD := $(shell pwd)

.PHONY: all
all: $(OBJDIR) libdbc.so parser_pyx.so

include cereal.mk

# make sure cereal is built
libdbc.so:: ../../cereal/gen/cpp/log.capnp.h

../../cereal/gen/cpp/log.capnp.h:
	cd ../../cereal && make

libdbc.so:: $(LIBDBC_OBJS) $(DBC_OBJS)
	@echo "[ LINK ] $@"
	$(CXX) -fPIC -shared -o '$@' $^ \
		-I. -I../.. \
		$(CXXFLAGS) \
		$(LDFLAGS) \
		$(CEREAL_CXXFLAGS) \
		$(CEREAL_LIBS)

packer_impl.so: packer_impl.pyx packer_setup.py
	python3 packer_setup.py build_ext --inplace
	rm -rf build
	rm -f packer_impl.cpp

parser_pyx.so: libdbc.so parser_pyx_setup.py parser_pyx.pyx common.pxd
	python3 parser_pyx_setup.py build_ext --inplace
	rm -rf build
	rm -f parser_pyx.cpp

$(OBJDIR)/%.o: %.cc
	@echo "[ CXX ] $@"
	$(CXX) -fPIC -c -o '$@' $^ \
		-I. -I../.. \
		$(CXXFLAGS) \
		$(CEREAL_CXXFLAGS) \

$(OBJDIR)/%.o: dbc_out/%.cc
	@echo "[ CXX ] $@"
	$(CXX) -fPIC -c -o '$@' $^ \
		-I. -I../.. \
		$(CXXFLAGS) \
		$(CEREAL_CXXFLAGS) \

dbc_out/%.cc: process_dbc.py dbc_template.cc $(OPENDBC_PATH)/%.dbc
	@echo "[ DBC GEN ] $@"
	./process_dbc.py $(OPENDBC_PATH) '$@'

$(OBJDIR):
	mkdir -p $@

.PHONY: clean $(OBJDIR)
clean:
	rm -rf libdbc.so*
	rm -f dbc_out/*.cc
	rm -f dbcs.txt
	rm -f dbcs.csv
	rm -f *.so
	rm -rf $(OBJDIR)/*
