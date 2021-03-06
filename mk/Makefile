#
# Copyright (c) Citrix Systems, Inc.
# All rights reserved.
# 
# Redistribution and use in source and binary forms, with or without
# modification, are permitted provided that the following conditions
# are met:
# 
#   1) Redistributions of source code must retain the above copyright
#      notice, this list of conditions and the following disclaimer.
# 
#   2) Redistributions in binary form must reproduce the above
#      copyright notice, this list of conditions and the following
#      disclaimer in the documentation and/or other materials
#      provided with the distribution.
# 
# THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS
# "AS IS" AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT
# LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS
# FOR A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL THE
# COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE FOR ANY DIRECT,
# INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES
# (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
# SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION)
# HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT,
# STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE)
# ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED
# OF THE POSSIBILITY OF SUCH DAMAGE.
#

USE_BRANDING := yes
USE_BUILD_NUMBER := yes
IMPORT_BRANDING := yes
IMPORT_VERSIONS := yes
include $(B_BASE)/common.mk
include $(B_BASE)/winbuild.mk

INSTALL=install -m 644 -o root -g root

REPONAME=xen-api-sdk
REPO=$(call git_loc,$(REPONAME))
BINDINGS_TMP=$(MY_OBJ_DIR)/tmp
PS_TAR_TMP=$(BINDINGS_TMP)/powershell_tar
INSTALLER_TMP=$(BINDINGS_TMP)/installer_tar
WIX_TMP=$(BINDINGS_TMP)/wix_src
AUTOMATION_DLL_V2=$(CARBON_DISTFILES)/powershell-v2/System.Management.Automation.dll
CSHARP_SRC_TMP=$(BINDINGS_TMP)/XenServer.NET-src/XenServer.NET
CSHARP_BIN_TMP=$(BINDINGS_TMP)/XenServer.NET-bin/XenServer.NET
CSHARP_XMLRPC_TMP=$(BINDINGS_TMP)/csharp_xmlrpc

XMLRPC_DLL=CookComputing.XmlRpcV2.dll
XMLRPCLICENSE = $(CARBON_DISTFILES)/csharp/XML-RPC.NET/LICENSE

INSTALLER_DEFAULT_GUID=3285AFCC-B1FA-4D78-8F94-AF1F4C2C8027
INSTALLER_DEFAULT_VERSION=1\.0\.0

PRODUCT_GUID := $(shell uuidgen | tr a-z A-Z)

PATCHES=$(REPO)/mk/patches

OMAKE=DATAMODEL_DIR=$(PROJECT_OUTPUTDIR)/api/datamodel omake

# sources
XMLRPC_DIST=$(CARBON_DISTFILES)/xmlrpc
XMLRPC_COMMON_DIST=$(XMLRPC_DIST)/xmlrpc-common-3.1.jar
XMLRPC_CLIENT_DIST=$(XMLRPC_DIST)/xmlrpc-client-3.1.jar
WS_COMMONS_DIST=$(XMLRPC_DIST)/ws-commons-util-1.0.2.jar

API_SRC=$(MY_OBJ_DIR)/api-src
API_PYTHON=$(API_SRC)/xen-api-src/scripts/examples/python
PYTHON_FILES=XenAPI.py provision.py
PYTHON_SAMPLES=install.py powercycle.py permute.py vm_start_async.py watch-all-events.py fixpbds.py shell.py license.py

SDK_ZIP=XenServer-SDK-$(PRODUCT_VERSION).zip
SDK_ZIP_LN=XenServer-SDK.zip
SDK_FOLDER=$(MY_OUTPUT_DIR)/XenServer-SDK
SDK_NET_FOLDER=$(SDK_FOLDER)/XenServer.NET
SDK_PS_FOLDER=$(SDK_FOLDER)/XenServerPSSnapIn
SDK_PY_FOLDER=$(SDK_FOLDER)/XenServerPython
SDK_PY_FOLDER_SAMPLES=$(SDK_PY_FOLDER)/samples

export PRODUCT_VERSION
export PROJECT_OUTPUTDIR
export MY_OUTPUT_DIR
export BINDINGS_TMP
export SDK_FOLDER
export SDK_NET_FOLDER
export XMLRPC_DIST
export SDK_PS_FOLDER
export XMLRPCLICENSE

.PHONY: omake-phase
omake-phase:
	cd $(REPO) && $(OMAKE) dist

$(MY_OBJ_DIR)/$(XMLRPC_DLL):
	cp $(PROJECT_OUTPUTDIR)/dotnet-packages-ref/$(XMLRPC_DLL) $@

$(MY_OBJ_DIR)/csharp_xmlrpc.tar.gz: $(REPO)/mk/sign.bat omake-phase $(MY_OBJ_DIR)/$(XMLRPC_DLL)
	mkdir -p $(CSHARP_XMLRPC_TMP)
	cp -r $(CSHARP_SRC_TMP) $(CSHARP_XMLRPC_TMP)
	sed -i '/SignAssembly/ i <AssemblyOriginatorKeyFile>XenServer.NET.snk</AssemblyOriginatorKeyFile>' $(CSHARP_XMLRPC_TMP)/XenServer.NET/XenServer.csproj
	cp $< $(CSHARP_XMLRPC_TMP)
	cp $(MY_OBJ_DIR)/$(XMLRPC_DLL) $(CSHARP_XMLRPC_TMP)
	cp $(REPO)/csharp/XenServer.NET.snk $(CSHARP_XMLRPC_TMP)/XenServer.NET
	tar -C $(CSHARP_XMLRPC_TMP) -zcf $@ .
   
$(MY_OBJ_DIR)/XenServer.dll: $(MY_OBJ_DIR)/csharp_xmlrpc.tar.gz $(REPO)/mk/csharp.sh
ifeq ($(SKIP_WINDOWS_BUILD), yes)
	touch $@
else
	$(call win-build,$<,$@,$(REPO)/mk/csharp.sh)
endif

.PHONY: csharp-bin
csharp-bin: $(MY_OBJ_DIR)/$(XMLRPC_DLL) $(MY_OBJ_DIR)/XenServer.dll
	$(INSTALL) $^ $(CSHARP_BIN_TMP)
	mkdir -p $(SDK_NET_FOLDER)/bin && cp -r $(CSHARP_BIN_TMP)/* $(SDK_NET_FOLDER)/bin

$(MY_OBJ_DIR)/powershell.tar.gz: $(MY_OBJ_DIR)/XenServer.dll $(MY_OBJ_DIR)/$(XMLRPC_DLL) $(REPO)/mk/sign.bat
	mkdir -p $(PS_TAR_TMP)
	cp -r $(BINDINGS_TMP)/XenServerPSSnapIn $(PS_TAR_TMP)
	cp $^ $(PS_TAR_TMP)
	cp $(AUTOMATION_DLL_V2) $(PS_TAR_TMP)
	tar -C $(PS_TAR_TMP) -zcf $@ .

$(MY_OBJ_DIR)/XenServerPSSnapIn.dll: $(MY_OBJ_DIR)/powershell.tar.gz $(REPO)/mk/powershell.sh
ifeq ($(SKIP_WINDOWS_BUILD), yes)
	touch $@
else
	$(call win-build,$<,$@,$(REPO)/mk/powershell.sh)
endif

$(WIX_TMP)/src/ui/wixui:
	mkdir -p $(WIX_TMP)
	unzip -q -o $(CARBON_DISTFILES)/windows-build/wix-2.0.5805.0-sources.zip -d $(WIX_TMP)
	cd $(WIX_TMP)/src/ui/wixui && patch -p1 --binary < $(PATCHES)/wix_src_patch

$(MY_OBJ_DIR)/installer.tar.gz: $(WIX_TMP)/src/ui/wixui $(MY_OBJ_DIR)/XenServerPSSnapIn.dll \
                                $(MY_OBJ_DIR)/XenServer.dll $(MY_OBJ_DIR)/$(XMLRPC_DLL) \
                                $(REPO)/mk/sign.bat $(REPO)/mk/sign-ps.bat $(REPO)/powershell/*.ps1 \
                                $(REPO)/powershell/*.ps1xml $(REPO)/powershell/samples/*.ps1
	mkdir -p $(INSTALLER_TMP)
	unzip -q -o $(CARBON_DISTFILES)/windows-build/wix-2.0.5805.0-binaries.zip -d $(INSTALLER_TMP)
	rm $(INSTALLER_TMP)/Bitmaps/{bannrbmp,dlgbmp}.bmp
	cp -r $(REPO)/powershell/installer/* $(INSTALLER_TMP)
	cp $(REPO)/Bitmaps/* $(INSTALLER_TMP)/Bitmaps
	cp -r $^ $(INSTALLER_TMP)
	cp $(SDK_PS_FOLDER)/src/*.txt $(INSTALLER_TMP)
	$(INSTALL) $(XMLRPCLICENSE) $(INSTALLER_TMP)/LICENSE.CookComputing.XmlRpcV2
	chmod -R a=rwx $(INSTALLER_TMP)
	sed -e 's/$(INSTALLER_DEFAULT_GUID)/$(PRODUCT_GUID)/g' \
	    -e 's/$(INSTALLER_DEFAULT_VERSION)/$(PRODUCT_VERSION)/g' \
	    $(REPO)/powershell/installer/XenServerPSSnapIn.wxs > $(INSTALLER_TMP)/XenServerPSSnapIn.wxs
	tar -C $(INSTALLER_TMP) -zcf $@ .

$(MY_OBJ_DIR)/XenServerPSSnapIn.msi: $(MY_OBJ_DIR)/installer.tar.gz $(REPO)/mk/installer.sh
ifeq ($(SKIP_WINDOWS_BUILD), yes)
	touch $@
else
	$(call win-build,$<,$@,$(REPO)/mk/installer.sh)
endif

.PHONY: bins
bins: omake-phase $(MY_OBJ_DIR)/XenServerPSSnapIn.msi csharp-bin
	mkdir -p $(MY_OUTPUT_DIR)
	cp $(BINDINGS_TMP)/XenCenterBindings.zip $(MY_OUTPUT_DIR)/XenCenterBindings-$(BUILD_NUMBER).zip
	cp $(MY_OBJ_DIR)/FriendlyErrorNames.Designer.cs $(SDK_NET_FOLDER)/src
	cp $(MY_OBJ_DIR)/XenServerPSSnapIn.msi $(SDK_PS_FOLDER)/XenServerPSSnapIn-$(PRODUCT_VERSION).msi

.PHONY: python-module
python-module:
	mkdir -p $(API_SRC) && tar -xf $(PROJECT_OUTPUTDIR)/api/SOURCES/xen-api-src.tar.bz2  -C $(API_SRC) xen-api-src/scripts/examples/python
	mkdir -p $(SDK_PY_FOLDER) && cp $(addprefix $(API_PYTHON)/, $(PYTHON_FILES)) $(SDK_PY_FOLDER)
	mkdir -p $(SDK_PY_FOLDER_SAMPLES) && cp $(addprefix $(API_PYTHON)/, $(PYTHON_SAMPLES) README) $(SDK_PY_FOLDER_SAMPLES)

$(MY_OUTPUT_DIR)/$(SDK_ZIP): bins python-module
	cd $(MY_OUTPUT_DIR) && zip -q -r9 $(SDK_ZIP) XenServer-SDK
	ln -sf $(SDK_ZIP) $(MY_OUTPUT_DIR)/$(SDK_ZIP_LN)
	rm -Rf $(SDK_FOLDER)

.PHONY: build
build: clean $(MY_OUTPUT_DIR)/$(SDK_ZIP) $(MY_SOURCES)/MANIFEST

$(MY_SOURCES)/MANIFEST: $(MY_SOURCES_DIRSTAMP)
	echo $(REPONAME) apache2 file $(XMLRPC_COMMON_DIST) > $@
	echo $(REPONAME) apache2 file $(XMLRPC_CLIENT_DIST) >> $@
	echo $(REPONAME) apache2 file $(WS_COMMONS_DIST) >> $@

.PHONY: clean
clean:
	cd $(REPO) && $(OMAKE) clean
	rm -Rf $(BINDINGS_TMP)
	rm -Rf $(API_SRC)
	rm -f $(MY_OBJ_DIR)/{*.gz,*.zip,*.msi,*.dll}
