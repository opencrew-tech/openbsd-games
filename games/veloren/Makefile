# XXX openbsd support to patches
# gilrs-core
# rustls-platform-verifier src/tests/verification_real_world/mod.rs ?
# make a wraper to start with VOXYGEN_SCREENSHOT
# if ~/.config/user-dirs.dirs isn't configured properly ?

# XXX to improve/upstream
# cpal, sndio backend
# wgpu*
# rustix, tcp primitive
# shaderc-sys, shaderc_shared support (need SPIRV-Tools-shared)
# xkbcommon_dl, dlopen .so *with* version suffix

COMMENT =	multiplayer voxel RPG written in Rust

# game must stay in sync with weekly build to play online
# https://download.veloren.net/version/linux/x86_64/weekly
# run make weekly for automated process
.include "Makefile.inc"

DISTNAME =	veloren-${VELOREN_H}
PKGNAME =	veloren-0.18.0pl${VELOREN_D}
DIST_TUPLE =	gitlab veloren veloren ${VELOREN_H} .

# veloren git+ depends, NOTE:
# To avoid build error due to workspace in WRKSRC/Cargo.toml we have to build
# depends in WRKDIR instead, to handle patches we extract in WRKSRC then link
# it during post-extract. But we don't need patches for those atm ...

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
CPAL_ID =	4d14d64e325adefdc1c62012af027942c09b3eea
DIST_TUPLE+=	github jarmani cpal ${CPAL_ID} cpal	# Apache-2.0

CATEGORIES =	games

HOMEPAGE =	https://veloren.net/

MAINTAINER =	xxx

# GPL v3+
PERMIT_PACKAGE =	Yes

WANTLIB += ${MODCARGO_WANTLIB} m shaderc_shared sndio zstd

# xkbcommon_dl use dlopen, keep this extra for pkg tools
WANTLIB += xkbcommon

MODULES =		devel/cargo

# enable unstable features, see https://gitlab.com/veloren/veloren/-/issues/264
MODCARGO_ENV+=		RUSTC_BOOTSTRAP=1

# same as upstream build
MODCARGO_NO_DEFAULT_FEATURES = Yes
MODCARGO_FEATURES =	default-publish

# Veloren depends on rusqlite features only available with bundle version.
MODCARGO_CRATES_KEEP =	libsqlite3-sys

# XXX trigger a rebuild during install, why ?
MODCARGO_INSTALL_TARGET_PATHS = server-cli voxygen

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

# XXX, howto disable symbol without strip (without DEBUG)
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
_GEN_VAR?=	BUILD_USER=$$(whoami) WRKOBJDIR=$${t} PATCH_LIST='patch-*_toml'
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
	cd - >/dev/null && rm -rf ${_TMP}/veloren-versions

_lock:
# update kira & cpal to use patched dependencies to local path from DIST_TUPLE
# need online registry, can't use modcargo default
	@cd ${WRKSRC} && ${SETENV} ${MAKE_ENV} ${MODCARGO_CARGO_BIN} update \
		--manifest-path ${MODCARGO_CARGOTOML} \
		--package kira --package cpal && \
	cp ${WRKSRC}/Cargo.lock ${FILESDIR}/

lock:
	@t=${_GEN_DIR} && \
	echo "Veloren: extract in $${t} as $$(whoami)" && \
	make -D _GEN_LOCK -D _UPDATE ${_GEN_VAR} patch && \
	echo "Veloren: gen lock in $${t} as $$(whoami)" && \
	make ${_GEN_VAR} _lock && \
	echo "Veloren: gen crates in $${t} as $$(whoami)" && \
	make ${_GEN_VAR} modcargo-gen-crates > crates.inc && \
	echo "Veloren: rm $${t}..." && \
	[ -d "$${t}" ] && rm -rf $${t} && \
	echo "Veloren: rm $${t}, done." || (\
	echo "Veloren: FAIL, try again with _GEN_DIR=$${t} or rm it" ; \
	false )

weekly:
	make versions > Makefile.inc.new && \
		mv Makefile.inc{.new,}
	make makesum
	make lock
	make makesum
	make clean
	make configure
	make modcargo-gen-crates-licenses > crates.inc.lic && \
		mv crates.inc{.lic,}
	make clean

.include "crates.inc"

.include <bsd.port.mk>
