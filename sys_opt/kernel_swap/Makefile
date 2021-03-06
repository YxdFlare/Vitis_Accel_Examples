.PHONY: help

help::
	$(ECHO) "Makefile Usage:"
	$(ECHO) "  make all TARGET=<sw_emu/hw_emu/hw> DEVICE=<FPGA platform> HOST_ARCH=<aarch32/aarch64/x86> EDGE_COMMON_SW=<rootfs and kernel image path>"
	$(ECHO) "      Command to generate the design for specified Target and Shell."
	$(ECHO) "      By default, HOST_ARCH=x86. HOST_ARCH and EDGE_COMMON_SW is required for SoC shells"
	$(ECHO) ""
	$(ECHO) "  make clean "
	$(ECHO) "      Command to remove the generated non-hardware files."
	$(ECHO) ""
	$(ECHO) "  make cleanall"
	$(ECHO) "      Command to remove all the generated files."
	$(ECHO) ""
	$(ECHO)  "  make test DEVICE=<FPGA platform>"
	$(ECHO)  "     Command to run the application. This is same as 'check' target but does not have any makefile dependency."
	$(ECHO)  ""
	$(ECHO) "  make sd_card TARGET=<sw_emu/hw_emu/hw> DEVICE=<FPGA platform> HOST_ARCH=<aarch32/aarch64/x86> EDGE_COMMON_SW=<rootfs and kernel image path>"
	$(ECHO) "      Command to prepare sd_card files."
	$(ECHO) "      By default, HOST_ARCH=x86. HOST_ARCH and EDGE_COMMON_SW is required for SoC shells"
	$(ECHO) ""
	$(ECHO) "  make check TARGET=<sw_emu/hw_emu/hw> DEVICE=<FPGA platform> HOST_ARCH=<aarch32/aarch64/x86> EDGE_COMMON_SW=<rootfs and kernel image path>"
	$(ECHO) "      Command to run application in emulation."
	$(ECHO) "      By default, HOST_ARCH=x86. HOST_ARCH and EDGE_COMMON_SW is required for SoC shells"
	$(ECHO) ""
	$(ECHO) "  make build TARGET=<sw_emu/hw_emu/hw> DEVICE=<FPGA platform> HOST_ARCH=<aarch32/aarch64/x86> EDGE_COMMON_SW=<rootfs and kernel image path>"
	$(ECHO) "      Command to build xclbin application."
	$(ECHO) "      By default, HOST_ARCH=x86. HOST_ARCH and EDGE_COMMON_SW is required for SoC shells"
	$(ECHO) ""

# Points to top directory of Git repository
COMMON_REPO = ../../
PWD = $(shell readlink -f .)
ABS_COMMON_REPO = $(shell readlink -f $(COMMON_REPO))

TARGET := hw
HOST_ARCH := x86
SYSROOT := 

include ./utils.mk

XSA := $(call device2xsa, $(DEVICE))
TEMP_DIR := ./_x.$(TARGET).$(XSA)
BUILD_DIR := ./build_dir.$(TARGET).$(XSA)

# SoC variables
RUN_APP_SCRIPT = run_app.sh
PACKAGE_OUT = package.$(TARGET)

LAUNCH_EMULATOR = $(PACKAGE_OUT)/launch_$(TARGET).sh
RESULT_STRING = TEST PASSED

VPP := v++
SDCARD := sd_card

#Include Libraries
include $(ABS_COMMON_REPO)/common/includes/opencl/opencl.mk
include $(ABS_COMMON_REPO)/common/includes/xcl2/xcl2.mk
CXXFLAGS += $(xcl2_CXXFLAGS)
LDFLAGS += $(xcl2_LDFLAGS)
HOST_SRCS += $(xcl2_SRCS)
CXXFLAGS += $(opencl_CXXFLAGS) -Wall -O0 -g -std=c++11
LDFLAGS += $(opencl_LDFLAGS)

HOST_SRCS += src/host.cpp

# Host compiler global settings
CXXFLAGS += -fmessage-length=0
LDFLAGS += -lrt -lstdc++ 

ifneq ($(HOST_ARCH), x86)
	LDFLAGS += --sysroot=$(SYSROOT)
endif

# Kernel compiler global settings
CLFLAGS += -t $(TARGET) --platform $(DEVICE) --save-temps 
ifneq ($(TARGET), hw)
	CLFLAGS += -g
endif

EXECUTABLE = vmulvadd
CMD_ARGS = $(BUILD_DIR)/krnl_vmul.xclbin $(BUILD_DIR)/krnl_vadd.xclbin
EMCONFIG_DIR = $(TEMP_DIR)
EMU_DIR = $(SDCARD)/data/emulation

BINARY_CONTAINERS += $(BUILD_DIR)/krnl_vmul.xclbin
BINARY_CONTAINER_krnl_vmul_LINK += $(BUILD_DIR)/krnl_vmul.link.xclbin
BINARY_CONTAINER_krnl_vmul_OBJS += $(TEMP_DIR)/krnl_vmul.xo
BINARY_CONTAINERS += $(BUILD_DIR)/krnl_vadd.xclbin
BINARY_CONTAINER_krnl_vadd_LINK += $(BUILD_DIR)/krnl_vadd.link.xclbin
BINARY_CONTAINER_krnl_vadd_OBJS += $(TEMP_DIR)/krnl_vadd.xo

CP = cp -rf

.PHONY: all clean cleanall docs emconfig
all: check-devices $(EXECUTABLE) $(BINARY_CONTAINERS) emconfig sd_card

.PHONY: exe
exe: $(EXECUTABLE)

.PHONY: build
build: check-vitis $(BINARY_CONTAINERS)

# Building kernel
$(TEMP_DIR)/krnl_vmul.xo: src/krnl_vmul.cpp
	mkdir -p $(TEMP_DIR)
	$(VPP) $(CLFLAGS) --temp_dir $(TEMP_DIR) -c -k krnl_vmul -I'$(<D)' -o'$@' '$<'
$(TEMP_DIR)/krnl_vadd.xo: src/krnl_vadd.cpp
	mkdir -p $(TEMP_DIR)
	$(VPP) $(CLFLAGS) --temp_dir $(TEMP_DIR) -c -k krnl_vadd -I'$(<D)' -o'$@' '$<'
$(BUILD_DIR)/krnl_vmul.xclbin: $(BINARY_CONTAINER_krnl_vmul_OBJS)
	mkdir -p $(BUILD_DIR)
ifeq ($(HOST_ARCH), x86)	
	$(VPP) $(CLFLAGS) --temp_dir $(BUILD_DIR) -l $(LDCLFLAGS) -o'$(BUILD_DIR)/krnl_vmul.link.xclbin' $(+)
	$(VPP) -t $(TARGET) --platform $(DEVICE) -p $(BUILD_DIR)/krnl_vmul.link.xclbin --package.out_dir $(PACKAGE_OUT) -o $(BUILD_DIR)/krnl_vmul.xclbin
else
	$(VPP) $(CLFLAGS) --temp_dir $(BUILD_DIR) -l $(LDCLFLAGS) -o'$(BUILD_DIR)/krnl_vmul.xclbin' $(+)
endif

$(BUILD_DIR)/krnl_vadd.xclbin: $(BINARY_CONTAINER_krnl_vadd_OBJS)
	mkdir -p $(BUILD_DIR)
ifeq ($(HOST_ARCH), x86)	
	$(VPP) $(CLFLAGS) --temp_dir $(BUILD_DIR) -l $(LDCLFLAGS) -o'$(BUILD_DIR)/krnl_vadd.link.xclbin' $(+)
	$(VPP) -t $(TARGET) --platform $(DEVICE) -p $(BUILD_DIR)/krnl_vadd.link.xclbin --package.out_dir $(PACKAGE_OUT) -o $(BUILD_DIR)/krnl_vadd.xclbin
else
	$(VPP) $(CLFLAGS) --temp_dir $(BUILD_DIR) -l $(LDCLFLAGS) -o'$(BUILD_DIR)/krnl_vadd.xclbin' $(+)
endif

$(BUILD_DIR)/$(PACKAGE_OUT)/krnl_vmul.xclbin: $(BUILD_DIR)/krnl_vmul.xclbin $(BUILD_DIR)/krnl_vadd.xclbin $(EXECUTABLE)
ifneq ($(HOST_ARCH), x86)
	$(VPP) -t $(TARGET) --platform $(DEVICE) -p $(BUILD_DIR)/krnl_vmul.xclbin --package.sd_file $(BUILD_DIR)/krnl_vadd.xclbin --package.out_dir $(PACKAGE_OUT) --package.rootfs $(EDGE_COMMON_SW)/rootfs.ext4 --package.sd_file $(SD_IMAGE_FILE) --package.sd_file xrt.ini --package.sd_file $(RUN_APP_SCRIPT) --package.sd_file $(EXECUTABLE) -o $(BUILD_DIR)/$(PACKAGE_OUT)/krnl_vmul.xclbin
else
	$(error package command not needed for x86 platform)
endif

sd_card : $(BINARY_CONTAINERS) $(EXECUTABLE) gen_run_app
ifneq ($(HOST_ARCH), x86)
	make $(BUILD_DIR)/$(PACKAGE_OUT)/krnl_vmul.xclbin
endif

# Building Host
$(EXECUTABLE): check-xrt $(HOST_SRCS) $(HOST_HDRS)
	$(CXX) $(CXXFLAGS) $(HOST_SRCS) $(HOST_HDRS) -o '$@' $(LDFLAGS)

emconfig:$(EMCONFIG_DIR)/emconfig.json
$(EMCONFIG_DIR)/emconfig.json:
	emconfigutil --platform $(DEVICE) --od $(EMCONFIG_DIR)

check: all
ifeq ($(findstring zcu102_base_202010, $(DEVICE)), zcu102_base_202010)
$(error This example is not supported for $(DEVICE))
endif
ifeq ($(findstring zcu104_base_202010, $(DEVICE)), zcu104_base_202010)
$(error This example is not supported for $(DEVICE))
endif
ifeq ($(findstring vck, $(DEVICE)), vck)
$(error This example is not supported for $(DEVICE))
endif
ifeq ($(findstring zc7, $(DEVICE)), zc7)
$(error This example is not supported for $(DEVICE))
endif
ifeq ($(TARGET),$(filter $(TARGET),sw_emu hw_emu))
ifeq ($(HOST_ARCH), x86)
	$(CP) $(EMCONFIG_DIR)/emconfig.json .
	XCL_EMULATION_MODE=$(TARGET) ./$(EXECUTABLE) $(BUILD_DIR)/krnl_vmul.xclbin $(BUILD_DIR)/krnl_vadd.xclbin
else
	$(ABS_COMMON_REPO)/common/utility/run_emulation.pl "./${LAUNCH_EMULATOR} | tee run_app.log" "./${RUN_APP_SCRIPT} $(TARGET)" "${RESULT_STRING}" "7"
endif
else
ifeq ($(HOST_ARCH), x86)
	./$(EXECUTABLE) $(BUILD_DIR)/krnl_vmul.xclbin $(BUILD_DIR)/krnl_vadd.xclbin
endif
endif

.PHONY: test
test: $(EXECUTABLE)
ifeq ($(TARGET),$(filter $(TARGET),sw_emu hw_emu))
ifeq ($(HOST_ARCH), x86)
	XCL_EMULATION_MODE=$(TARGET) ./$(EXECUTABLE) $(BUILD_DIR)/krnl_vmul.xclbin $(BUILD_DIR)/krnl_vadd.xclbin
else
	$(ABS_COMMON_REPO)/common/utility/run_emulation.pl "./${LAUNCH_EMULATOR} | tee embedded_run.log" "./${RUN_APP_SCRIPT} $(TARGET)" "${RESULT_STRING}" "7"
endif
else
ifeq ($(HOST_ARCH), x86)
	./$(EXECUTABLE) $(BUILD_DIR)/krnl_vmul.xclbin $(BUILD_DIR)/krnl_vadd.xclbin
else
	$(ECHO) "Please copy the content of sd_card folder and data to an SD Card and run on the board"
endif
endif


# Cleaning stuff
clean:
	-$(RMDIR) $(EXECUTABLE) $(XCLBIN)/{*sw_emu*,*hw_emu*} 
	-$(RMDIR) profile_* TempConfig system_estimate.xtxt *.rpt *.csv 
	-$(RMDIR) src/*.ll *v++* .Xil emconfig.json dltmp* xmltmp* *.log *.jou *.wcfg *.wdb

cleanall: clean
	-$(RMDIR) build_dir* sd_card*
	-$(RMDIR) package.*
	-$(RMDIR) _x* *xclbin.run_summary qemu-memory-_* emulation/ _vimage/ pl* start_simulation.sh *.xclbin

