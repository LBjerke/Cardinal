#!/usr/bin/make -f
# Makefile for DISTRHO Plugins #
# ---------------------------- #
# Created by falkTX
#

# --------------------------------------------------------------
# Build config

PREFIX  ?= /usr/local
DESTDIR ?=
SYSDEPS ?= false

# --------------------------------------------------------------
# Import base definitions

USE_NANOVG_FBO = true
include ../dpf/Makefile.base.mk

# --------------------------------------------------------------
# Build setup

BUILD_DIR = ../build/rack

ifeq ($(MACOS),true)
BASE_FLAGS += -DARCH_MAC
else ifeq ($(WINDOWS),true)
BASE_FLAGS += -DARCH_WIN
else
BASE_FLAGS += -DARCH_LIN
endif

BASE_FLAGS += -DPRIVATE=

BASE_FLAGS += -I../dpf/dgl/src/nanovg
BASE_FLAGS += -I../dpf/distrho
BASE_FLAGS += -I../include
BASE_FLAGS += -I../include/neon-compat
BASE_FLAGS += -IRack/include
ifeq ($(SYSDEPS),true)
BASE_FLAGS += -DCARDINAL_SYSDEPS
BASE_FLAGS += $(shell pkg-config --cflags jansson libarchive samplerate speexdsp)
else
BASE_FLAGS += -DZSTDLIB_VISIBILITY=
BASE_FLAGS += -IRack/dep/include
endif
BASE_FLAGS += -IRack/dep/filesystem/include
BASE_FLAGS += -IRack/dep/fuzzysearchdatabase/src
BASE_FLAGS += -IRack/dep/glfw/include
BASE_FLAGS += -IRack/dep/nanosvg/src
BASE_FLAGS += -IRack/dep/osdialog
BASE_FLAGS += -IRack/dep/oui-blendish
BASE_FLAGS += -IRack/dep/pffft

ifeq ($(DEBUG),true)
BASE_FLAGS += -UDEBUG
endif

ifeq ($(HAVE_LIBLO),true)
BASE_FLAGS += -DHAVE_LIBLO $(LIBLO_FLAGS)
endif

ifeq ($(HEADLESS),true)
BASE_FLAGS += -DHEADLESS
ifeq ($(WITH_LTO),true)
BASE_FLAGS += -ffat-lto-objects
endif
endif

ifeq ($(WASM),true)
BASE_FLAGS += -DNANOVG_GLES2=1
BASE_FLAGS += -msse -msse2 -msse3 -msimd128
# FIXME
BASE_FLAGS += -DRTLD_DEEPBIND=0
else
BASE_FLAGS += -pthread
endif

ifeq ($(WINDOWS),true)
BASE_FLAGS += -D_USE_MATH_DEFINES
BASE_FLAGS += -DWIN32_LEAN_AND_MEAN
BASE_FLAGS += -I../include/mingw-compat
BASE_FLAGS += -I../include/mingw-std-threads
endif

BUILD_C_FLAGS += -std=gnu11
BUILD_C_FLAGS += -fno-finite-math-only
BUILD_CXX_FLAGS += -fno-finite-math-only

# use our custom function to filter out VCV trademarks
BUILD_CXX_FLAGS += -DnsvgParseFromFile=nsvgParseFromFileCardinal

# --------------------------------------------------------------
# Rack files to build

RACK_FILES += AsyncDialog.cpp
RACK_FILES += CardinalModuleWidget.cpp
RACK_FILES += custom/asset.cpp
RACK_FILES += custom/dep.cpp
RACK_FILES += custom/library.cpp
RACK_FILES += custom/network.cpp
RACK_FILES += custom/osdialog.cpp
RACK_FILES += override/blendish.c
RACK_FILES += override/context.cpp
RACK_FILES += override/plugin.cpp
RACK_FILES += override/Engine.cpp
RACK_FILES += override/MenuBar.cpp
RACK_FILES += override/Model.cpp
RACK_FILES += override/Scene.cpp

RACK_FILES += Rack/dep/pffft/pffft.c
RACK_FILES += Rack/dep/pffft/fftpack.c
RACK_FILES += Rack/dep/osdialog/osdialog.c

IGNORED_FILES  = Rack/src/asset.cpp
IGNORED_FILES += Rack/src/audio.cpp
IGNORED_FILES += Rack/src/common.cpp
IGNORED_FILES += Rack/src/context.cpp
IGNORED_FILES += Rack/src/dep.cpp
IGNORED_FILES += Rack/src/discord.cpp
IGNORED_FILES += Rack/src/gamepad.cpp
IGNORED_FILES += Rack/src/keyboard.cpp
IGNORED_FILES += Rack/src/library.cpp
IGNORED_FILES += Rack/src/midi.cpp
IGNORED_FILES += Rack/src/network.cpp
IGNORED_FILES += Rack/src/plugin.cpp
IGNORED_FILES += Rack/src/rtaudio.cpp
IGNORED_FILES += Rack/src/rtmidi.cpp
IGNORED_FILES += Rack/src/app/AudioDisplay.cpp
IGNORED_FILES += Rack/src/app/MenuBar.cpp
IGNORED_FILES += Rack/src/app/MidiDisplay.cpp
IGNORED_FILES += Rack/src/app/Scene.cpp
IGNORED_FILES += Rack/src/app/TipWindow.cpp
IGNORED_FILES += Rack/src/engine/Engine.cpp
IGNORED_FILES += Rack/src/plugin/Model.cpp
IGNORED_FILES += Rack/src/window/Window.cpp
IGNORED_FILES += $(wildcard Rack/src/core/*.cpp)

RACK_FILES += $(wildcard Rack/src/*.c)
RACK_FILES += $(wildcard Rack/src/*/*.c)
RACK_FILES += $(filter-out $(IGNORED_FILES),$(wildcard Rack/src/*.cpp))
RACK_FILES += $(filter-out $(IGNORED_FILES), $(wildcard Rack/src/*/*.cpp))

# --------------------------------------------------------------
# lots of warnings from VCV side

BASE_FLAGS += -Wno-unused-but-set-variable
BASE_FLAGS += -Wno-unused-parameter
BASE_FLAGS += -Wno-unused-result
BASE_FLAGS += -Wno-unused-variable

ifeq ($(MACOS),true)
BASE_FLAGS += -Wno-unknown-warning-option
endif

# --------------------------------------------------------------
# Build targets

TARGET = rack.a

all: $(TARGET)
ifeq ($(MOD_BUILD),true)
	$(MAKE) -C CardinalFX lv2 resources
else
	$(MAKE) -C CardinalFX
	$(MAKE) -C CardinalSynth
	$(MAKE) -C Cardinal
endif

resources:
	$(MAKE) resources -C Cardinal
	$(MAKE) resources -C CardinalFX
	$(MAKE) resources -C CardinalSynth

clean:
	rm -f $(TARGET)
	rm -rf $(BUILD_DIR)
	$(MAKE) clean -C Cardinal
	$(MAKE) clean -C CardinalFX
	$(MAKE) clean -C CardinalSynth

# --------------------------------------------------------------
# Build commands

RACK_OBJS = $(RACK_FILES:%=$(BUILD_DIR)/%.o)

$(TARGET): $(RACK_OBJS)
	@echo "Creating $@"
	$(SILENT)rm -f $@
	$(SILENT)$(AR) crs $@ $^

$(BUILD_DIR)/%.c.o: %.c
	-@mkdir -p "$(shell dirname $(BUILD_DIR)/$<)"
	@echo "Compiling $<"
	$(SILENT)$(CC) $< $(BUILD_C_FLAGS) -c -o $@

$(BUILD_DIR)/override/MIDI_CV.cpp.o: override/MIDI_CV.cpp
	-@mkdir -p "$(shell dirname $(BUILD_DIR)/$<)"
	@echo "Compiling $<"
	$(SILENT)$(CXX) $< $(BUILD_CXX_FLAGS) -UPRIVATE -c -o $@

$(BUILD_DIR)/%.cpp.o: %.cpp
	-@mkdir -p "$(shell dirname $(BUILD_DIR)/$<)"
	@echo "Compiling $<"
	$(SILENT)$(CXX) $< $(BUILD_CXX_FLAGS) -c -o $@

# --------------------------------------------------------------

-include $(RACK_OBJS:%.o=%.d)

# --------------------------------------------------------------
