# SPDX-License-Identifier: GPL-3.0-only
#
# Copyright (C) 2021-2023 ImmortalWrt.org

include $(TOPDIR)/rules.mk

PKG_NAME:=v2rayA
# Snapshot of Bambyyno/v2rayA after v2.5.10. The fourth component is the
# commit date so apk sees this as newer than 2.5.10.20261027.
PKG_VERSION:=2.5.10.20261123
PKG_RELEASE:=1

# https://github.com/Bambyyno/v2rayA/commit/116895ef31c56e8ef6c5e2a6078c836709c127ab
# pull/2 head (cursor/auto-select-version-probe-415c). Not on main yet.
PKG_SOURCE_VERSION:=116895ef31c56e8ef6c5e2a6078c836709c127ab
PKG_SOURCE:=$(PKG_NAME)-$(PKG_SOURCE_VERSION).tar.gz
PKG_SOURCE_URL:=https://codeload.github.com/Bambyyno/v2rayA/tar.gz/$(PKG_SOURCE_VERSION)?
PKG_HASH:=ebad489b73c06b6354c3b5061280e255e3cadc7e6b6fa6d19d5bcb9467e1a1aa
PKG_BUILD_DIR:=$(BUILD_DIR)/$(PKG_NAME)-$(PKG_VERSION)/service

PKG_LICENSE:=AGPL-3.0-only
PKG_LICENSE_FILES:=LICENSE
PKG_MAINTAINER:=Tianling Shen <cnsztl@immortalwrt.org>

PKG_BUILD_DEPENDS:=golang/host
PKG_BUILD_PARALLEL:=1
PKG_BUILD_FLAGS:=no-mips16

GO_PKG:=github.com/v2rayA/v2rayA
GO_PKG_LDFLAGS_X:= \
	$(GO_PKG)/conf.Version=$(PKG_VERSION) \
	$(GO_PKG)/core/iptables.TproxyNotSkipBr=true
# go:embed only sees files copied into the Go build tree. html/js/css are
# not copied unless they match GO_PKG_INSTALL_EXTRA.
GO_PKG_INSTALL_EXTRA:=server/router/web

include $(INCLUDE_DIR)/package.mk
include $(TOPDIR)/feeds/packages/lang/golang/golang-package.mk

TAR_OPTIONS+= --strip-components 1
TAR_CMD=$(HOST_TAR) -C $(1)/.. $(TAR_OPTIONS)

define Package/v2raya
  TITLE:=A Linux web GUI client of Project V
  SECTION:=net
  CATEGORY:=Network
  SUBMENU:=Web Servers/Proxies
  DEPENDS:=$(GO_ARCH_DEPENDS) +ca-bundle
  URL:=https://github.com/Bambyyno/v2rayA
endef

define Package/v2raya/description
  v2rayA is a V2Ray Linux client supporting global transparent proxy,
  compatible with SS, SSR, Trojan(trojan-go), PingTunnel protocols.

  This package is built from github.com/Bambyyno/v2rayA commit
  116895ef31c56e8ef6c5e2a6078c836709c127ab.
endef

define Package/v2raya/conffiles
/etc/v2raya/
/etc/config/v2raya
endef

# Web UI stays the c78739c7 tarball. This service fix does not change the GUI.
# Rebuild only if that checkout's gui output must be replaced:
#   yarn --cwd gui --frozen-lockfile && yarn --cwd gui build
#   tar -C <checkout> -czf web-c78739c7.tar.gz web
WEB_FILE:=web-c78739c7.tar.gz
WEB_HASH:=4c131fe32f84f162c452d37bc4bfa02051ea0eafee05e90e77ed65640fb58904

# mips* and loongarch64 have no pure-Go sqlite driver in this tree
# (service/db/sqlite_driver_mipsle_nocgo.go). Those targets need cgo and
# the external linker so github.com/mattn/go-sqlite3 is built in.
# ARCH is values such as mipsel_24kc, not the bare GOARCH.
ifeq ($(findstring mips,$(ARCH))$(findstring loong,$(ARCH)),)
GO_PKG_TARGET_VARS:=$(filter-out CGO_ENABLED=%,$(GO_PKG_TARGET_VARS)) CGO_ENABLED=0
GO_PKG_DEFAULT_LDFLAGS:=$(filter-out -linkmode external,$(GO_PKG_DEFAULT_LDFLAGS))
endif

define Build/Prepare
	$(call Build/Prepare/Default)

	echo "$(WEB_HASH)  $(CURDIR)/$(WEB_FILE)" | sha256sum -c -
	mkdir -p $(PKG_BUILD_DIR)/server/router/web
	gzip -dc $(CURDIR)/$(WEB_FILE) | $(HOST_TAR) -C $(PKG_BUILD_DIR)/server/router/web $(TAR_OPTIONS)
endef

define Package/v2raya/install
	$(call GoPackage/Package/Install/Bin,$(PKG_INSTALL_DIR))
	$(INSTALL_DIR) $(1)/usr/bin/
	$(INSTALL_BIN) $(PKG_INSTALL_DIR)/usr/bin/v2rayA $(1)/usr/bin/v2raya

	$(INSTALL_DIR) $(1)/etc/v2raya/
	$(INSTALL_DIR) $(1)/etc/config/
	$(INSTALL_CONF) $(CURDIR)/files/v2raya.config $(1)/etc/config/v2raya
	$(INSTALL_DIR) $(1)/etc/init.d/
	$(INSTALL_BIN) $(CURDIR)/files/v2raya.init $(1)/etc/init.d/v2raya
endef

$(eval $(call GoBinPackage,v2raya))
$(eval $(call BuildPackage,v2raya))
