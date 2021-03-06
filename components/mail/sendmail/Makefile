#
# CDDL HEADER START
#
# The contents of this file are subject to the terms of the
# Common Development and Distribution License (the "License").
# You may not use this file except in compliance with the License.
#
# You can obtain a copy of the license at usr/src/OPENSOLARIS.LICENSE
# or http://www.opensolaris.org/os/licensing.
# See the License for the specific language governing permissions
# and limitations under the License.
#
# When distributing Covered Code, include this CDDL HEADER in each
# file and include the License file at usr/src/OPENSOLARIS.LICENSE.
# If applicable, add the following below this CDDL HEADER, with the
# fields enclosed by brackets "[]" replaced with your own identifying
# information: Portions Copyright [yyyy] [name of copyright owner]
#
# CDDL HEADER END
#

#
# Copyright (c) 2012, 2016, Oracle and/or its affiliates. All rights reserved.
#

include ../../../make-rules/shared-macros.mk

COMPONENT_NAME=		sendmail
COMPONENT_VERSION=	8.15.2
COMPONENT_REVISION=	3
COMPONENT_SRC=		$(COMPONENT_NAME)-$(COMPONENT_VERSION)
COMPONENT_PROJECT_URL=	ftp://ftp.sendmail.org/pub/sendmail/
COMPONENT_ARCHIVE=	$(COMPONENT_NAME).$(COMPONENT_VERSION).tar.gz
COMPONENT_ARCHIVE_HASH=	\
    sha256:24f94b5fd76705f15897a78932a5f2439a32b1a2fdc35769bb1a5f5d9b4db439
COMPONENT_ARCHIVE_URL=	$(COMPONENT_PROJECT_URL)$(COMPONENT_ARCHIVE)

TPNO=			23958

include $(WS_MAKE_RULES)/prep.mk
include $(WS_MAKE_RULES)/justmake.mk
include $(WS_MAKE_RULES)/ips.mk

# Although we build 32_and_64, everything except libmilter just wants 32,
# which is why the macros just below only have values for 32.  libmilter
# is dealt with further below.
PKG_PROTO_DIRS += $(BUILD_DIR_32)
PKG_PROTO_DIRS += $(BUILD_DIR_32)/obj.SunOS.$(OS_VERSION).$(ARCH)
PKG_PROTO_DIRS += $(COMPONENT_DIR)/files
PKG_PROTO_DIRS += $(COMPONENT_DIR)/files/man

# We set ARCH to match sendmail's configure-like script: on x86, it simply
# uses 'uname -m', but on sparc it shortens "sun4*" into "sun4".
ARCH	:=	$(shell uname -m)
ifeq ($(findstring sun4,$(ARCH)),sun4)
ARCH	=	sun4
endif
FILE_DIR=$(COMPONENT_DIR)/files
M4_ARG=-D_NO_MAKEINFO_
SM_BUILD_SUB_DIR=obj.SunOS.$(OS_VERSION).$(ARCH)
SM_BUILD_DIR=$(@D)/$(SM_BUILD_SUB_DIR)

# libmilter is built 32- and 64-bit, but its Makefile installs into a common
# location, so we use the macro below in its manifest to distinguish between
# the different built objects.
PKG_MACROS += LIBMILTER_BUILD_DIR=$(SM_BUILD_SUB_DIR)

CLEAN_PATHS += $(FILE_DIR)/aux/mailq

# This appends "+Sun" to the version string, which we do for historical
# reasons.  If we did this via patch, then we would have to revise the patch
# with every new release, since the version string changes every time.
COMPONENT_PREP_ACTION =	($(GSED) -i -e 's/\(Version\[\] = "8\.[^"]*\)/\1+Sun/' \
	$(COMPONENT_SRC)/sendmail/version.c &&\
	cp $(COMPONENT_SRC)/devtools/M4/depend/X11.m4 \
		$(COMPONENT_SRC)/devtools/M4/depend/Solaris.m4)

# libmilter and sendmail must come before include, so appending to the default
# value of LINT_FLAGS will not work; we must redefine it here.
SM_DEFS=	-DSOLARIS=$(shell echo $(SOLARIS_VERSION) | \
    $(GSED) -e 's/\.//' -e 's/$$/00/')
SM_INCLUDES=	-I$(SM_BUILD_DIR)/libmilter -I$(SOURCE_DIR)/sendmail \
    -I$(SOURCE_DIR)/include -I$(SOURCE_DIR) -I.
LINT_FLAGS =	-nsvx $(SM_DEFS) $(SM_INCLUDES)
CCOPTS.32 =	-D_FILE_OFFSET_BITS=64
CCOPTS =	-erroff=E_STATEMENT_NOT_REACHED $(CC_BITS) $(CCOPTS.$(BITS))

COMPONENT_BUILD_ARGS += CC="$(CC)"
COMPONENT_BUILD_ARGS += CCOPTS="$(CCOPTS)"
COMPONENT_BUILD_ARGS += CCLINK="$(CC)"
COMPONENT_BUILD_ARGS += CC_PIC="$(CC_PIC)"
COMPONENT_BUILD_ARGS += LDOPTS="$(CC_BITS)"

$(SOURCE_DIR)/.prep:	$(SOURCE_DIR)/devtools/Site/site.config.m4

# Sendmail looks for its build configuration in various per-OS config files
# in its devtools/Site sub-dir, with site.config.m4 being the fallback.  So
# create that file as part of the 'prep' phase.
#

define SITE_CONFIG_M4
APPENDDEF(`conf_sendmail_ENVDEF', `-DSUN_EXTENSIONS')
APPENDDEF(`conf_sendmail_ENVDEF', `-DVENDOR_DEFAULT=VENDOR_SUN')
APPENDDEF(`conf_sendmail_ENVDEF', `-DSUN_INIT_DOMAIN')
APPENDDEF(`conf_sendmail_ENVDEF', `-DSUN_SIMPLIFIED_LDAP')
APPENDDEF(`conf_sendmail_ENVDEF', `-DTCPWRAPPERS')
APPENDDEF(`conf_sendmail_ENVDEF', `-DSASL')
APPENDDEF(`conf_sendmail_ENVDEF', `-DSTARTTLS')
APPENDDEF(`conf_sendmail_ENVDEF', `-D_FFR_LOCAL_DAEMON')
APPENDDEF(`conf_sendmail_ENVDEF', `-D_FFR_MAIL_MACRO')
APPENDDEF(`conf_sendmail_ENVDEF', `-DIPV6_FULL=0')
APPENDDEF(`conf_sendmail_LIBS', `-lsldap')
APPENDDEF(`conf_sendmail_LIBS', `-lwrap')
APPENDDEF(`conf_sendmail_LIBS', `-lumem')
APPENDDEF(`conf_sendmail_LIBS', `-lssl')
APPENDDEF(`conf_sendmail_LIBS', `-lcrypto')
APPENDDEF(`confMAPDEF', `-DLDAPMAP')
APPENDDEF(`confLIBS', `-lldap_r-2.4 -llber-2.4 -lsasl -lc')
define(`confDEPEND_TYPE', `Solaris')
define(`confINCDIRS', `-I/usr/include/openldap -I/usr/include/sasl')
define(`confMKDIR', `/usr/bin/mkdir')
define(`confMTCFLAGS', `$(CC_PIC)')
define(`sendmailTARGET_LINKS', `')
endef
#define(\`confOPTIMIZE', \`-g')

export SITE_CONFIG_M4

$(SOURCE_DIR)/devtools/Site/site.config.m4:	unpack
	$(file > $@,$(SITE_CONFIG_M4))

# mailq's Makefile doesn't grok the 32- and 64-bit build. We just want 64.
# So we clean before we build, and the last build (64) wins, which is what
# we want.
COMPONENT_POST_BUILD_ACTION = cd $(FILE_DIR)/aux ; \
    $(GMAKE) BITS=$(BITS) clean build

COMPONENT_POST_INSTALL_ACTION += cd $(BUILD_DIR_$(BITS))/obj.SunOS.5.11.$(ARCH)/mail.local \
	&& $(ENV) $(COMPONENT_INSTALL_ENV) $(GMAKE) force-install DESTDIR=$(PROTO_DIR);
COMPONENT_POST_INSTALL_ACTION += cd $(FILE_DIR)/aux ; $(GMAKE) install ;
COMPONENT_POST_INSTALL_ACTION += \
    $(CP) $(FILE_DIR)/cf/domain/solaris-generic.m4 $(@D)/cf/domain/ ;
COMPONENT_POST_INSTALL_ACTION += \
    $(CP) $(FILE_DIR)/cf/cf/sendmail.mc $(@D)/cf/cf/ ;
# Without the two $(CP)s below we would pick up the version of submit.cf
# from under $(COMPONENT_SRC), which is not what we want because it contains
# information (user, hostname, date, build path) from the upstream machine
# where the distro was created.  So we build our own version here, using
# $(M4_ARG), which suppresses the inclusion of all that information.
# Then we copy it to $(PROTO_DIR)/cf/cf instead of the more intuitive
# $(PROTOETCDIR)/mail/cf/cf to match the transform in the manifest that
# picks up everything under etc/mail/cf/ from cf/ instead.
COMPONENT_POST_INSTALL_ACTION += \
    cd $(@D)/cf/cf ; \
    $(RM) submit.cf ; \
    m4 $(M4_ARG) ../m4/cf.m4 submit.mc > submit.cf ; \
    $(CP) submit.cf $(PROTOETCDIR)/mail ; \
    $(MKDIR) $(PROTO_DIR)/cf/cf ; \
    $(CP) submit.cf $(PROTO_DIR)/cf/cf ; \
    m4 $(M4_ARG) ../m4/cf.m4 sendmail.mc > sendmail.cf

COMPONENT_TEST_ARGS += CC="$(CC)"
COMPONENT_TEST_ARGS += CCOPTS="$(CCOPTS)"
COMPONENT_TEST_ARGS += CCLINK="$(CC)"
COMPONENT_TEST_ARGS += CC_PIC="$(CC_PIC)"
COMPONENT_TEST_ARGS += LDOPTS="$(CC_BITS)"

COMPONENT_TEST_ENV +=	PATH=$(GNUBIN):$(USRBINDIR)

# The transforms below abstract out the 32- or 64-bit specific parts of
# the results, so we can use a common "all" master file.
COMPONENT_TEST_MASTER = $(COMPONENT_TEST_RESULTS_DIR)/results-all.master

COMPONENT_TEST_TRANSFORMS += \
    '-e "s|$(SM_BUILD_SUB_DIR)|\\$$(SM_BUILD_SUB_DIR)|g" ' \
    '-e "s|$(OS_VERSION)|\\$$(OS_VERSION)|g" ' \
    '-e "s|$(ARCH)|\\$$(ARCH)|g" ' \
    '-e "s|^$(CC).*$$|XXX_CC_XXX|g" ' \
    '-e "s|m$(BITS)|m\\$$(BITS)|g" ' \
    '-e "/^XXX_CC_XXX\\$$/d" '

$(INSTALL_32):	$(INSTALL_64)

# common targets
build:		$(BUILD_32_and_64)

install:	$(INSTALL_32_and_64)

test:		$(TEST_32_and_64)

REQUIRED_PACKAGES += database/berkeleydb-5
REQUIRED_PACKAGES += developer/build/makedepend
REQUIRED_PACKAGES += library/libmilter
REQUIRED_PACKAGES += library/openldap
REQUIRED_PACKAGES += library/security/openssl
REQUIRED_PACKAGES += runtime/perl-522
REQUIRED_PACKAGES += SUNWcs
REQUIRED_PACKAGES += system/library
REQUIRED_PACKAGES += system/library/security/libsasl
