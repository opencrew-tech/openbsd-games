# XXX openbsd support to patches
# gilrs-core
# rustls-platform-verifier src/tests/verification_real_world/mod.rs ?

# XXX to improve/upstream
# cpal, sndio backend, output only
# wgpu*
# rustix, tcp primitive
# shaderc-sys, shaderc_shared support (need SPIRV-Tools-shared)
# xkbcommon_dl, dlopen .so *with* version suffix

COMMENT =	multiplayer voxel RPG written in Rust

# game must stay in sync with weekly build to play online
# https://download.veloren.net/version/linux/x86_64/weekly

# make versions
VELOREN_V =	weekly/2cc17122/1769648660
VELOREN_H =	2cc1712251e89ef600a489c04eb8b7c6f127b46b
VELOREN_D =	20260129
SPECS_ID =	4e2da1df29ee840baa9b936593c45592b7c9ae27
AUTHC_ID =	ae0e16783a9f9041951296885f082308e155db79
PORTPICKER_ID =	df6b37872f3586ac3b21d08b56c8ec7cd92fb172
#CONROD_TAG =	copypasta_0.7 # prefer H instead
CONROD_ID =	c74446362371dc837b8b4b6d6ddcd7693b48d9b6
ICED_ID =	veloren-winit-0.28

DISTNAME =	veloren-${VELOREN_H}
PKGNAME =	veloren-0.18.0pl${VELOREN_D}
DIST_TUPLE =	gitlab veloren veloren ${VELOREN_H} .

# veloren git+ depends, NOTE:
# To avoid build error due to workspace in WRKSRC/Cargo.toml we have to build
# depends in WRKDIR instead, to handle patches we extract in WRKSRC then link
# it during post-extract. But we don't need patches atm ...

# specs, Entity-Component System written in Rust, Apache-2.0
# authc, auth server and command-line client for Veloren, GPL v3+
# portpicker-rs, pick a free unused port, public domain
# conrod, easy-to-use, 2D GUI library in Rust, MIT / Apache 2.0
# iced, cross-platform GUI library for Rust, inspired by Elm, MIT
DIST_TUPLE+=	github amethyst specs ${SPECS_ID} specs
DIST_TUPLE+=	gitlab veloren auth ${AUTHC_ID} authc
DIST_TUPLE+=	github xMAC94x portpicker-rs ${PORTPICKER_ID} portpicker
DIST_TUPLE+=	gitlab veloren conrod ${CONROD_ID} conrod
DIST_TUPLE+=	github Imberflur iced ${ICED_ID} iced

# add sndio-sys backend to cpal ...
CPAL_ID =	a022193cc3881bc104b42ad843b179db9c4b14d2
DIST_TUPLE+=	github jarmani cpal ${CPAL_ID} cpal	# Apache-2.0
# update kira to new cpal version (0.16 -> 0.17)
KIRA_ID =	849ff39b4191c7538557515b9a33b1729ff684a5
DIST_TUPLE+=	github tesselode kira ${KIRA_ID} kira	# MIT

CATEGORIES =	games

HOMEPAGE =	https://veloren.net/

MAINTAINER =	xxx

# GPL v3+
PERMIT_PACKAGE =	Yes

WANTLIB += ${MODCARGO_WANTLIB} m shaderc_shared zstd

# xkbcommon_dl use dlopen, keep this extra for pkg tools
WANTLIB += xkbcommon

MODULES =		devel/cargo

# enable unstable features, see https://gitlab.com/veloren/veloren/-/issues/264
MODCARGO_ENV+=		RUSTC_BOOTSTRAP=1

# sndio-sys use bindgen which need libclang.so
MODCARGO_ENV+=		LIBCLANG_PATH=${LOCALBASE}/llvm19/lib/

# same as upstream build
MODCARGO_NO_DEFAULT_FEATURES = Yes
MODCARGO_FEATURES =	default-publish

# Veloren depends on rusqlite features only available with bundle version.
MODCARGO_CRATES_KEEP =	libsqlite3-sys

# XXX trigger a rebuild during install, why ?
MODCARGO_INSTALL_TARGET_PATHS = server-cli voxygen

# sndio-sys use bindgen which need libclang.so, only during build
BUILD_DEPENDS+=		devel/llvm/19
# shaderc build using shaderc_shared need sprivtools symbol but doesn't link
# anythink to SPIRV-Tools-shared (from port-lib-depends-check)
BUILD_DEPENDS+=		graphics/spirv-tools

RUN_DEPENDS =		devel/desktop-file-utils

# XXX wip gilrs : sysutils/libudev-openbsd wip gilrs
LIB_DEPENDS =		archivers/zstd \
			graphics/shaderc \
			sysutils/libudev-openbsd \
			x11/xkbcommon

# Veloren specific: in-game version & store data under user's home
MAKE_ENV+=		VELOREN_GIT_VERSION=${VELOREN_V}
MAKE_ENV+=		VELOREN_USERDATA_STRATEGY="system"

CONFIGURE_STYLE =	cargo

# XXX
DEBUG_PACKAGES =	${BUILD_PACKAGES}

SUBST_VARS+=		WRKDIR

post-extract:
# link depends in WRKDIR, see note about veloren git+ depends at top of Makefile
	ln -fs ${WRKSRC}/specs ${WRKDIR}/specs
	ln -fs ${WRKSRC}/authc ${WRKDIR}/authc
	ln -fs ${WRKSRC}/portpicker ${WRKDIR}/portpicker
	ln -fs ${WRKSRC}/conrod ${WRKDIR}/conrod
	ln -fs ${WRKSRC}/iced ${WRKDIR}/iced
	ln -fs ${WRKSRC}/cpal ${WRKDIR}/cpal
	ln -fs ${WRKSRC}/kira ${WRKDIR}/kira
.if empty(_GEN_LOCK)
	cp ${WRKSRC}/Cargo.lock{,.orig}
	cp ${FILESDIR}/Cargo.lock ${WRKSRC}/
.endif

post-patch:
# note patch."source" doesn't work from config neither root .toml, hardcode path
	${SUBST_CMD} ${WRKSRC}/Cargo.toml
	${SUBST_CMD} ${WRKSRC}/client/Cargo.toml
	${SUBST_CMD} ${WRKSRC}/client/i18n/Cargo.toml
	${SUBST_CMD} ${WRKSRC}/server/Cargo.toml
	${SUBST_CMD} ${WRKSRC}/voxygen/Cargo.toml

post-install:
	# assets
	${INSTALL_DATA_DIR} ${PREFIX}/share/veloren
	cp -pr ${WRKSRC}/assets ${PREFIX}/share/veloren/
	# menu
	${INSTALL_DATA_DIR} ${PREFIX}/share/applications
	${INSTALL_DATA} \
		${WRKSRC}/assets/voxygen/net.veloren.veloren.desktop \
		${PREFIX}/share/applications/
	${INSTALL_DATA_DIR} ${PREFIX}/share/pixmaps
	${INSTALL_DATA} \
		${WRKSRC}/assets/voxygen/net.veloren.veloren.png \
		${PREFIX}/share/pixmaps/
	${INSTALL_DATA_DIR} ${PREFIX}/share/metainfo
	${INSTALL_DATA} \
		${WRKSRC}/assets/voxygen/net.veloren.veloren.metainfo.xml \
		${PREFIX}/share/metainfo/

### targets for port maintainer(s)

_TMP?=		./
_GEN_VAR?=	BUILD_USER=$$(whoami) WRKOBJDIR=$${t} PATCH_LIST=patch-*_toml
_GEN_DIR?=	$$(realpath `mktemp -d ${_TMP}/veloren.XXXXXXX`)

# Note, to check all git depends are correctly handled, search "git = "
# from https://gitlab.com/search?
# search=git%20%3D%20&project_id=10174980&repository_ref=${VELOREN_H}
_VELOREN_GIT=	https://gitlab.com/veloren/veloren.git/
_CONROD_GIT=	https://gitlab.com/veloren/conrod.git
versions:
	# make versions
	@mkdir -p ${_TMP}/veloren-versions ;				\
	cd ${_TMP}/veloren-versions ;					\
	export HOME=${_TMP}/veloren-versions ;				\
	git init > git.log 2>&1 ;					\
	git fetch ${_VELOREN_GIT} tag weekly --depth 1 >> git.log 2>&1 ;\
	git checkout FETCH_HEAD >> git.log 2>&1 ;			\
	_hash_time=$$(git log -n 1 --pretty=format:%h/%ct --abbrev=8) ; \
	_tag=$$(git describe --exact-match --tags HEAD) ;		\
	printf "VELOREN_V =\t%s/%s\n" $$_tag $$_hash_time ;		\
	printf "VELOREN_H =\t%s\n" $$(git log -n 1 --pretty=format:%H) ;\
	printf "VELOREN_D =\t%s\n" $$(git log -n 1 --pretty=format:%ad	\
		--date=format:'%Y%m%d') ;				\
	printf "SPECS_ID =\t%s\n" $$(awk -F '"'				\
		'/amethyst\/specs/{print $$4}' Cargo.toml) ;		\
	printf "AUTHC_ID =\t%s\n" $$(awk -F '"'				\
		'/veloren\/auth/{print $$4}' server/Cargo.toml) ;	\
	printf "PORTPICKER_ID =\t%s\n" $$(awk -F '"'			\
		'/xMAC94x\/portpicker/{print $$4}' server/Cargo.toml) ;	\
	_conrod_tag=$$(awk -F '"'					\
		'/veloren\/conrod.git/{print $$4;exit;}' voxygen/Cargo.toml) ; \
	printf "#CONROD_TAG =\t%s # prefer H instead\n" $$_conrod_tag ;	\
	printf "CONROD_ID =\t%s\n" $$(					\
		git ls-remote --heads ${_CONROD_GIT} |			\
		awk "/$${_conrod_tag}/{print \$$1;exit;}") ;		\
	printf "ICED_ID =\t%s\n" $$(awk -F '"'				\
		'/Imberflur\/iced/{print $$6}' voxygen/Cargo.toml) ;	\
	rm -rf ${_TMP}/veloren-versions

# update kira & cpal to use patched dependencies to local path from DIST_TUPLE
_lock:
	@cd ${WRKSRC} && ${SETENV} ${MAKE_ENV} ${MODCARGO_CARGO_BIN} update \
		--manifest-path ${MODCARGO_CARGOTOML} \
		--package kira --package cpal && \
	cp ${WRKSRC}/Cargo.lock ${FILESDIR}/

lock:
	@t=${_GEN_DIR} && \
	echo "Veloren: extract in $${t} as $$(whoami)" && \
	make -D _GEN_LOCK ${_GEN_VAR} patch && \
	echo "Veloren: gen-modules in $${t} as $$(whoami)" && \
	make -D _GEN_LOCK ${_GEN_VAR} _lock && \
	echo "Veloren: rm $${t}..." && \
	[ -d "$${t}" ] && rm -rf $${t} && \
	echo "Veloren: rm $${t}, done." || (\
	echo "Veloren: FAIL, try again with _GEN_DIR=$${t} or rm it" ; \
	false )

.include "crates.inc"

.include <bsd.port.mk>
