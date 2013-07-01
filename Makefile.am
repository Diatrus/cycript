# Cycript - Optimizing JavaScript Compiler/Runtime
# Copyright (C) 2009-2013  Jay Freeman (saurik)

# GNU General Public License, Version 3 {{{
#
# Cycript is free software: you can redistribute it and/or modify
# it under the terms of the GNU General Public License as published
# by the Free Software Foundation, either version 3 of the License,
# or (at your option) any later version.
#
# Cycript is distributed in the hope that it will be useful, but
# WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with Cycript.  If not, see <http://www.gnu.org/licenses/>.
# }}}

.DELETE_ON_ERROR:

AUTOMAKE_OPTIONS = subdir-objects
CLEANFILES = 
SUBDIRS = 

ACLOCAL_AMFLAGS = -I m4

# XXX: do I really need -fno-common?
AM_CPPFLAGS = -fno-common -I$(srcdir)/include -DYYDEBUG=1
AM_CPPFLAGS += -include $(srcdir)/config.h -include $(srcdir)/unconfig.h

AM_CFLAGS = $(CY_ARCH)
AM_CXXFLAGS = $(CY_ARCH)
AM_OBJCXXFLAGS = $(CY_ARCH)
AM_LDFLAGS = $(CY_ARCH)

CY_LDFLAGS = -no-undefined -avoid-version -export-dynamic

lib_LTLIBRARIES = 

lib_LTLIBRARIES += libcycript.la
libcycript_la_SOURCES = Select.cpp
libcycript_la_LDFLAGS = $(AM_LDFLAGS) $(CY_LDFLAGS)
libcycript_la_LIBADD = -ldl

lib_LTLIBRARIES += libcycript-sys.la
libcycript_sys_la_SOURCES = ConvertUTF.c Highlight.cpp Network.cpp Output.cpp Parser.cpp Replace.cpp
libcycript_sys_la_LDFLAGS = $(AM_LDFLAGS) $(CY_LDFLAGS)
libcycript_sys_la_LIBADD = $(LTLIBFFI) -ldl

libcycript_sys_la_SOURCES += Cycript.tab.cc
libcycript_sys_la_SOURCES += lex.cy.cpp

bin_PROGRAMS = cycript
cycript_SOURCES = Console.cpp Display.cpp
cycript_LDADD = libcycript.la libcycript-sys.la $(LTLIBAPR) $(LTLIBREADLINE) $(LTLIBTERMCAP) -ldl

ldid = true
entitle = $(ldid) -Scycript.xml

filters = $(CY_FILTERS)

if CY_EXECUTE
libcycript_sys_la_SOURCES += sig/ffi_type.cpp sig/parse.cpp sig/copy.cpp
libcycript_sys_la_SOURCES += Bridge.cpp Execute.cpp JavaScriptCore.cpp Library.cpp
libcycript_sys_la_LIBADD += $(LTJAVASCRIPTCORE)

AM_CPPFLAGS += -DCY_EXECUTE
filters += C

Bridge.lo: Bridge.hpp

CLEANFILES += Bridge.gperf
Bridge.gperf: Bridge.def Bridge.sh
	$(srcdir)/Bridge.sh $< >$@

CLEANFILES += Bridge.hpp
Bridge.hpp: Bridge.gperf
	$(GPERF) $< | $(SED) -e 's/defined __GNUC_STDC_INLINE__ || defined __GNUC_GNU_INLINE__/0/' >$@
endif

if CY_OBJECTIVEC
filters += ObjectiveC
libcycript_sys_la_SOURCES += ObjectiveC/Output.mm ObjectiveC/Replace.cpp ObjectiveC/Library.mm
libcycript_sys_la_LIBADD += $(LTOBJECTIVEC)
endif

if CY_MACH
libcycript_sys_la_SOURCES += Handler.mm
cycript_SOURCES += Mach/Inject.cpp

AM_CPPFLAGS += -DCY_ATTACH

Mach/Inject.$(OBJEXT): Trampoline.t.hpp

CLEANFILES += Trampoline.t.lo
Trampoline.t.lo: Trampoline.t.cpp Trampoline.hpp Baton.hpp Standard.hpp
	$(LTCXXCOMPILE) $(CY_ARCH) -c -o $@ $< -fno-stack-protector -fno-exceptions -O0

CLEANFILES += libTrampoline.t.la
libTrampoline.t.la: Trampoline.t.lo
	$(CXXLINK) $(CY_ARCH) -rpath $(libdir) $^

CLEANFILES += Trampoline.t.hpp
Trampoline.t.hpp: libTrampoline.t.la trampoline.sh
	$(srcdir)/trampoline.sh $@ .libs/libTrampoline.t.dylib Trampoline $(SED) $(LIPO) $(NM) $(OTOOL)
	! grep '## symbol stub for:' $@
endif

libcycript_sim_la_SHORTNAME = sim
if CY_IOS_SIMULATOR
lib_LTLIBRARIES += libcycript-sim.la
libcycript_sim_la_SOURCES = $(libcycript_sys_la_SOURCES)
libcycript_sim_la_CPPFLAGS = $(AM_CPPFLAGS)

CY_SIM_FLAGS = -arch i386 -isysroot $(CY_XCODE_IOSSIMSDK)
libcycript_sim_la_CXXFLAGS = $(CY_SIM_FLAGS)
libcycript_sim_la_OBJCXXFLAGS = $(CY_SIM_FLAGS)
libcycript_sim_la_LDFLAGS = $(CY_SIM_FLAGS) $(CY_LDFLAGS)

libcycript_sim_la_LIBADD = $(LTOBJECTIVEC) -ldl
libcycript_sim_la_LIBADD += $(LTOBJECTIVEC) $(LTJAVASCRIPTCORE)

libcycript_sim_la_CPPFLAGS += -idirafter $(CY_XCODE_MACOSXSDK)/usr/include
libcycript_sim_la_LDFLAGS += -Wl,/usr/lib/libffi.dylib

libcycript_sim_la_OBJCXXFLAGS += -fobjc-abi-version=2 -fobjc-legacy-dispatch
endif

CLEANFILES += Cycript.yy
Cycript.yy: Cycript.yy.in
	$(srcdir)/Filter.sh <$< >$@ $(filters)

CLEANFILES += Cycript.l
Cycript.l: Cycript.l.in
	$(srcdir)/Filter.sh <$< >$@ $(filters)

CLEANFILES += lex.cy.cpp
lex.cy.cpp: Cycript.l
	$(FLEX) -t $< | $(SED) -e 's/int yyl;/yy_size_t yyl;/;s/int yyleng_r;/yy_size_t yyleng_r;/' >$@

Console.lo Execute.lo Highlight.lo Library.lo Parser.lo lex.cy.lo: Cycript.tab.hh

CLEANFILES += Cycript.tab.cc Cycript.tab.hh location.hh position.hh stack.hh Cycript.output
Cycript.tab.cc Cycript.tab.hh location.hh position.hh stack.hh Cycript.output: Cycript.yy
	$(BISON) -v --report=state $<
	! grep -n '^ *$$default  reduce using rule [0-9]* (Lex[A-Z][^)]*)$$' Cycript.output -B 2 | grep 'shift, and go to state [0-9]*$$'
	! grep -n '^ *$$default  reduce using rule [0-9]* (Lex[A-Z][^)]*)$$' Cycript.output -B 2 | grep -v '$$default' | grep 'reduce using rule [0-9]* ([^)]*)$$'