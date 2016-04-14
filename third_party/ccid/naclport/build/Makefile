# Copyright 2016 Google Inc.
#
# This library is free software; you can redistribute it and/or
# modify it under the terms of the GNU Lesser General Public
# License as published by the Free Software Foundation; either
# version 2.1 of the License, or (at your option) any later version.
#
# This library is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the GNU
# Lesser General Public License for more details.
#
# You should have received a copy of the GNU Lesser General Public License
# along with this library; if not, write to the Free Software Foundation,
# Inc., 51 Franklin Street, Fifth Floor, Boston, MA 02110-1301 USA.

# This makefile builds the port of CCID library under the Native Client.
#
# Upon successful build, the resulting static library is installed into the NaCl
# SDK libraries directory.

TARGET := google_smart_card_ccid

include ../../../../common/make/common.mk

include $(COMMON_DIR_PATH)/make/nacl_module_building_common.mk

include $(COMMON_DIR_PATH)/cpp/include.mk

include $(THIRD_PARTY_DIR_PATH)/libusb/naclport/include.mk

include $(THIRD_PARTY_DIR_PATH)/pcsc-lite/naclport/common/include.mk

include ../include.mk


CCID_SOURCES_PATH := ../../src/src

CCID_NACL_SOURCES_PATH := ../src


# Rules for invoking the source files compilation into object files

# * HAVE_PTHREAD definition enables the support of working with multiple readers
#   simultaneously;
# * log_msg and log_xxd are redefined in order to not collide with the symbols
#   from the PC/SC-Lite server-side libraries;
# * PCSCLITE_HP_DROPDIR constant points to the directory containing the configs
#   for all PC/SC-Lite server drivers;
COMMON_CPPFLAGS := \
	-DHAVE_PTHREAD=1 \
	-Dlog_msg=ccid_log_msg \
	-Dlog_xxd=ccid_log_xxd \
	-DPCSCLITE_HP_DROPDIR='"/crx/pcsc/drivers"' \
	-I$(CCID_SOURCES_PATH) \
	-I$(LIBUSB_INCLUDE_PATH) \
	-Wall \

CCID_SOURCES := \
	$(CCID_SOURCES_PATH)/ccid.c \
	$(CCID_SOURCES_PATH)/ccid_usb.c \
	$(CCID_SOURCES_PATH)/commands.c \
	$(CCID_SOURCES_PATH)/debug.c \
	$(CCID_SOURCES_PATH)/ifdhandler.c \
	$(CCID_SOURCES_PATH)/strlcpy.c \
	$(CCID_SOURCES_PATH)/tokenparser.c \
	$(CCID_SOURCES_PATH)/utils.c \

# * BUNDLE constant contains the name of subdirectory under the PC/SC-Lite
#   server driver configs directory (see also the PCSCLITE_HP_DROPDIR constant);
# * USE_SYSLOG enables using syslog functions for logging, which in this NaCl
#   port are redefined in order to log both to stderr and to JavaScript console
#   (see the /common/cpp/ library);
# * VERSION constant contains the CCID version;
# * __linux definition allows to use some header files that are usually provided
#   under Linux (but are actually provided by the NaCl SDK too);
# * The "macro-redefined" warning diagnostic is disabled because of some
#   non-clean code;
CCID_CPPFLAGS := \
	$(COMMON_CPPFLAGS) \
	-DBUNDLE='"ifd-ccid.bundle"' \
	-DUSE_SYSLOG \
	-DVERSION='"$(CCID_VERSION)"' \
	-D__linux \
	-I$(CCID_NACL_SOURCES_PATH) \
	-I$(COMMON_SYSLOG_INCLUDE_PATH) \
	-I$(PCSC_LITE_ORIGINAL_HEADERS_DIR_PATH) \
	-Wno-macro-redefined \

$(foreach src,$(CCID_SOURCES),$(eval $(call COMPILE_RULE,$(src),$(CCID_CPPFLAGS))))

CCID_OPENCT_SOURCES := \
	$(CCID_SOURCES_PATH)/openct/buffer.c \
	$(CCID_SOURCES_PATH)/openct/checksum.c \
	$(CCID_SOURCES_PATH)/openct/proto-t1.c \

CCID_OPENCT_CPPFLAGS := \
	$(COMMON_CPPFLAGS) \
	-I$(CCID_NACL_SOURCES_PATH) \
	-I$(PCSC_LITE_ORIGINAL_HEADERS_DIR_PATH) \

$(foreach src,$(CCID_OPENCT_SOURCES),$(eval $(call COMPILE_RULE,$(src),$(CCID_OPENCT_CPPFLAGS))))

CCID_SIMCLIST_SOURCES := \
	$(CCID_SOURCES_PATH)/simclist.c \

# * log_msg and log_xxd are redefined in order to not collide with the symbols
#   from the PC/SC-Lite server-side libraries;
# * SIMCLIST_NO_DUMPRESTORE is defined in order to disable some features of the
#   simclist library which are not compiling under the NaCl SDK environment;
# * The "macro-redefined" warning diagnostic is disabled because of some
#   non-clean code;
CCID_SIMCLIST_CPPFLAGS := \
	$(COMMON_CPPFLAGS) \
	-Dlog_msg=ccid_log_msg \
	-Dlog_xxd=ccid_log_xxd \
	-DSIMCLIST_NO_DUMPRESTORE \
	-Wno-macro-redefined \

$(foreach src,$(CCID_SIMCLIST_SOURCES),$(eval $(call COMPILE_RULE,$(src),$(CCID_SIMCLIST_CPPFLAGS))))

CCID_TOWITOKO_SOURCES := \
	$(CCID_SOURCES_PATH)/towitoko/atr.c \
	$(CCID_SOURCES_PATH)/towitoko/pps.c \

CCID_TOWITOKO_CPPFLAGS := \
	$(COMMON_CPPFLAGS) \
	-I$(CCID_NACL_SOURCES_PATH) \
	-I$(PCSC_LITE_ORIGINAL_HEADERS_DIR_PATH) \

$(foreach src,$(CCID_TOWITOKO_SOURCES),$(eval $(call COMPILE_RULE,$(src),$(CCID_TOWITOKO_CPPFLAGS))))


# Variable containing the list of all source files whose compiled object files
# will have to be linked together

SOURCES := \
	$(CCID_SOURCES) \
	$(CCID_OPENCT_SOURCES) \
	$(CCID_SIMCLIST_SOURCES) \
	$(CCID_TOWITOKO_SOURCES) \


# Rules for adding a dependency on the installation of the separate libraries'
# headers

$(foreach src,$(SOURCES),$(eval $(call DEPEND_COMPILE_ON_NACL_LIBRARY_HEADERS,$(src),$(CPP_COMMON_HEADERS_INSTALLATION_STAMP_FILE))))
$(foreach src,$(SOURCES),$(eval $(call DEPEND_COMPILE_ON_NACL_LIBRARY_HEADERS,$(src),$(LIBUSB_HEADERS_INSTALLATION_STAMP_FILE))))


# Rules for building the driver config file and putting it into the resulting
# out directory under the PC/SC-Lite server driver configs directory

Info.plist: $(CCID_SOURCES_PATH)/create_Info_plist.pl $(CCID_SOURCES_PATH)/../readers/supported_readers.txt $(CCID_SOURCES_PATH)/Info.plist.src
	@rm -f Info.plist
	$(CCID_SOURCES_PATH)/create_Info_plist.pl \
		$(CCID_SOURCES_PATH)/../readers/supported_readers.txt \
		$(CCID_SOURCES_PATH)/Info.plist.src \
		--target=libccid.so \
		--version=$(CCID_VERSION) > Info.plist.build
	@mv Info.plist.build Info.plist

$(eval $(call CLEAN_RULE,Info.plist.build))
$(eval $(call CLEAN_RULE,Info.plist))

$(eval $(call COPY_TO_OUT_DIR_RULE,Info.plist,pcsc/drivers/ifd-ccid.bundle/Contents))


# Rules for building the JSON file containing the list of the supported USB
# readers in the format ready for being pasted into App's manifest file
# (see <https://developer.chrome.com/apps/app_usb#manifest>)

app_manifest_usb_devices.json: create_app_manifest_usb_devices.py $(CCID_SOURCES_PATH)/../readers/supported_readers.txt
	@rm -f app_manifest_usb_devices.json
	./create_app_manifest_usb_devices.py \
		< $(CCID_SOURCES_PATH)/../readers/supported_readers.txt \
		> app_manifest_usb_devices.json.build
	@mv app_manifest_usb_devices.json.build app_manifest_usb_devices.json

all: app_manifest_usb_devices.json

$(eval $(call CLEAN_RULE,app_manifest_usb_devices.json))


# Rules for linking of the compiled object files and installing the resulting
# static library into the NaCl SDK libraries directory

$(eval $(call NACL_LIBRARY_BUILD_RULE,$(SOURCES)))