# OS
sys := $(shell uname -s)

ifeq ($(sys), Darwin)
  os = Darwin
  compiler = gcc
  cxx = g++
  so = dylib
  teh = Mac
else
ifeq ($(sys), MINGW32_NT-5.1)
  os = WINNT
  compiler = msvc
  cxx = cl
  so = dll
  teh = Win
else
ifeq ($(sys), MINGW32_NT-6.1)
  os = WINNT
  compiler = msvc
  cxx = cl
  so = dll
  teh = Win
else
ifeq ($(sys), Linux)
  os = Linux
  compiler = gcc
  cxx = g++
  so = so
  teh = Nix
else
  $(error Sorry, your os is unknown/unsupported: $(sys))
endif
endif
endif
endif

# Arch
machine := $(shell uname -m)

ifeq ($(machine), i386)
  arch = x86
else
ifeq ($(machine), i586)
  arch = x86
else
ifeq ($(machine), i686)
  arch = x86
else
ifeq ($(machine), x86_64)
  arch = x86
else
  $(error: Sorry, your architecture is unknown/unsupported: $(machine))
endif
endif
endif
endif

# Target and objects
target = libmediarecorder
so_target = $(target:=.$(so))

# source and path configurations
idl = IMediaRecorder.idl
cpp_sources = MediaRecorder.cpp MediaModule.cpp Convert.cpp \
    VideoSource.cpp $(os)/VideoSource$(teh).cpp \
    AudioSource.cpp $(os)/AudioSource$(teh).cpp \
    VideoSourceCanvas.cpp 

sdkdir ?= ${MOZSDKDIR}
xpidl = $(sdkdir)/bin/xpidl

# FIXME: we don't actually require this for e.g. clean
ifeq ($(sdkdir),)
  $(warning No 'sdkdir' variable given)
  $(warning It should point to the location of the Gecko SDK)
  $(warning For example: "make sdkdir=/foo/bar/baz")
  $(warning Or set the MOZSDKDIR environment variable to point to it)
  $(error )
endif

idl_headers = $(idl:.idl=.h)
idl_typelib = $(idl:.idl=.xpt)
cpp_objects = $(cpp_sources:.cpp=.o)
so_target = $(target:=.$(so))

cppflags = -DRAINBOW_$(teh)
headers = -I. -I$(os) -I$(sdkdir)/include \
    -I$(sdkdir)/include/system_wrappers \
    -I$(sdkdir)/include/xpcom \
    -I$(sdkdir)/include/string \
    -I$(sdkdir)/include/content \
    -I$(sdkdir)/include/dom \
    -I$(sdkdir)/include/nspr \
    -I$(sdkdir)/sdk/include

# libraries
libdirs := $(sdkdir)/lib $(sdkdir)/bin
libs := xpcomglue_s xpcom nspr4 mozalloc plds4 plc4

# compiler and Linker Flags
ifeq ($(os), Darwin)
  libdirs := $(patsubst %,-L%,$(libdirs))
  libs := $(patsubst %,-l%,$(libs))
  headers += -I../lib/Darwin_i386-gcc3/include

  ifeq ($(debug),1)
    cppflags += -g -DDEBUG
  endif

  cppflags += -c -pipe -Os -arch i386 -dynamiclib \
    -mmacosx-version-min=10.5 -DMOZ_NO_MOZALLOC -fPIC \
    -fno-rtti -fno-exceptions -fno-strict-aliasing \
    -fno-common -fshort-wchar -fpascal-strings -pthread \
    -Wall -Werror -Wconversion -Wpointer-arith \
    -Wno-invalid-offsetof \
    -Woverloaded-virtual -Wsynth -Wno-ctor-dtor-privacy \
    -Wno-non-virtual-dtor -Wcast-align -Wno-long-long \
    -include xpcom-config.h $(headers)
  ldflags += -pthread -pipe -bundle \
    -mmacosx-version-min=10.5 -arch i386 \
    -Wl,-executable_path,$(sdkdir)/bin \
    -Wl,-dead_strip \
    $(sdkdir)/lib/XUL \
    ../lib/Darwin_i386-gcc3/libogg.a \
    ../lib/Darwin_i386-gcc3/libvorbis.a \
    ../lib/Darwin_i386-gcc3/libvidcap.a \
    ../lib/Darwin_i386-gcc3/libtheora.a \
    ../lib/Darwin_i386-gcc3/libtheoraenc.a \
    ../lib/Darwin_i386-gcc3/libvorbisenc.a \
    ../lib/Darwin_i386-gcc3/libportaudio.a \
    -framework Carbon \
    -framework CoreAudio \
    -framework QuartzCore \
    -framework QuickTime \
    -framework AudioUnit \
    -framework AudioToolbox \
    $(libdirs) $(libs)
else
ifeq ($(os), WINNT)
  libdirs := $(patsubst %,-LIBPATH:%,$(libdirs))
  libs := $(patsubst %,$(sdkdir)/lib/%.lib,$(libs))
  headers += -I../lib/WINNT_x86-msvc/include

  ifeq ($(debug),1)
    cppflags += -MTd -DDEBUG
  else
    cppflags += -MT
	ldflags = -NODEFAULTLIB:msvcrt
  endif

  cppflags += -c -nologo -Zc:wchar_t- $(headers) \
	-DXP_WIN=1 -DHW_THREADS=1 \
	-DWIN32_LEAN_AND_MEAN=1 -DMOZ_NO_MOZALLOC=1
  ldflags += -DLL -NOLOGO -SUBSYSTEM:WINDOWS \
    -LTCG -MANIFEST $(libdirs) $(libs) \
    ../lib/WINNT_x86-msvc/libogg.lib \
    ../lib/WINNT_x86-msvc/libvorbis.lib \
    ../lib/WINNT_x86-msvc/libtheora.lib \
	winmm.lib user32.lib advapi32.lib \
    strmiids.lib ole32.lib oleaut32.lib \
	user32.lib advapi32.lib winmm.lib $(ldops)
rcflags := -r $(headers)
else
ifeq ($(os), Linux)
  libdirs := $(patsubst %,-L%,$(libdirs))
  libs := $(patsubst %,-l%,$(libs))
  headers += -I../lib/Linux_x86-gcc3/include

  ifeq ($(debug),1)
    cppflags += -g -DDEBUG
  endif

  cppflags += -pipe -Os -shared \
    -DMOZ_NO_MOZALLOC -DMOZILLA_STRICT_API -fshort-wchar \
    -fPIC -fno-rtti -fno-exceptions -fno-strict-aliasing \
    -fno-common -pthread -Wall -Werror -Wconversion \
    -Wpointer-arith -Woverloaded-virtual -Wsynth \
    -Wno-ctor-dtor-privacy -Wno-non-virtual-dtor -Wcast-align \
    -Wno-long-long -include xpcom-config.h $(headers)

  ldflags += -pthread -pipe -m32 \
    -Wl,-z,defs \
    -Wl,-rpath-link,$(sdkdir)/bin \
    $(sdkdir)/lib/libxpcomglue_s.a \
    $(libdirs) $(libs) -lrt -lasound \
    ../lib/Linux_x86-gcc3/libvidcap.a \
    ../lib/Linux_x86-gcc3/libvorbis.a \
    ../lib/Linux_x86-gcc3/libvorbisenc.a \
    ../lib/Linux_x86-gcc3/libogg.a \
    ../lib/Linux_x86-gcc3/libtheora.a
endif
endif
endif

######################################################################

.PHONY: all build clean

all: build

build: $(so_target) $(idl_typelib)

clean:
	rm -f $(so_target) $(so_target).manifest $(cpp_objects) \
	$(target:=.res) libmediarecorder.lib libmediarecorder.exp \
	$(target).pdb $(target).ilk

# rules to build the c headers and .xpt from idl
$(idl_headers): $(idl)
	$(xpidl) -m header -I$(sdkdir)/idl $(@:.h=.idl)

$(idl_typelib): $(idl)
	$(xpidl) -m typelib -I$(sdkdir)/idl $(@:.xpt=.idl)


ifeq ($(os), Darwin)
  $(cpp_objects): $(cpp_sources)
	$(cxx) -o $@ $(cppflags) $(@:.o=.cpp)

  $(so_target): $(idl_headers) $(cpp_objects)
	$(cxx) -o $@ $(ldflags) $(cpp_objects)
	chmod +x $@
else
ifeq ($(os), WINNT)
  $(cpp_objects): $(cpp_sources)
	$(cxx) -Fo$@ -Fd$(@:.o=.pdb) $(cppflags) $(@:.o=.cpp)

  $(so_target): $(idl_headers) $(cpp_objects)
	link -OUT:$@ -PDB:$(@:.dll=.pdb) $(cpp_objects) $(ldflags)
	mt.exe -manifest $(so_target).manifest -outputresource:"$@;2"
	chmod +x $@
else
ifeq ($(os), Linux)
  $(so_target): $(idl_headers)
	$(cxx) $(cppflags) -o $@ $(cpp_sources) $(ldflags)
	chmod +x $@
endif
endif
endif
