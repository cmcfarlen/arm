# Makefile for easy setup of arm toolchain
#
#

.PHONY = brew

BASEDIR=/usr/local
BINDIR=$(BASEDIR)/bin
DLDIR=$(HOME)/Downloads

PREFIX=$(BASEDIR)/arm

#URLS
BINUTILS_VERSION=2.23.2
GCC_VERSION=4.6.4
NEWLIB_VERSION=2.0.0

BINUTILS_URL=http://ftpmirror.gnu.org/gnu/binutils/binutils-$(BINUTILS_VERSION).tar.gz
GCC_URL=http://mirror.team-cymru.org/gnu/gcc/gcc-4.6.4/gcc-4.6.4.tar.bz2
NEWLIB_URL=ftp://sourceware.org/pub/newlib/newlib-2.0.0.tar.gz

BINUTILS_TBZ=$(DLDIR)/binutils-$(BINUTILS_VERSION).tar.gz
GCC_TBZ=$(DLDIR)/gcc-$(GCC_VERSION).tar.bz2
NEWLIB_TBZ=$(DLDIR)/newlib-$(NEWLIB_VERSION).tar.gz


# homebrew if for some reason this isn't installed
$(BINDIR)/brew:
	ruby -e "$$(curl -fsSL https://raw.github.com/mxcl/homebrew/go)"

brew: $(BINDIR)/brew

brew-deps: brew
	brew install mpc mpfr gmp libmpc texinfo libusb-compat libftdi wget

$(BINUTILS_TBZ):
	curl -L -o $(BINUTILS_TBZ) $(BINUTILS_URL)

$(GCC_TBZ):
	curl -L -o $(GCC_TBZ) $(GCC_URL)

$(NEWLIB_TBZ):
	curl -L -o $(NEWLIB_TBZ) $(NEWLIB_URL)


$(PREFIX)/bin/arm-elf-as: $(BINUTILS_TBZ)
	mkdir -p ./build
	rm -rf build/binutils*
	gzip -cd $(BINUTILS_TBZ) | (cd build; tar -xf -)
	mkdir build/binutils-$(BINUTILS_VERSION)/build
	cd build/binutils-$(BINUTILS_VERSION)/build && \
		../configure --prefix=$(PREFIX) --target=arm-elf --enable-interwork --enable-multilib --with-gnu-as --with-gnu-ld --disable-nls && \
		make -j5 all && \
		make install

binutils: $(PREFIX)/bin/arm-elf-as

$(PREFIX)/bin/arm-elf-gcc: $(GCC_TBZ)
	rm -rf build/gcc*
	bzip2 -cd $(GCC_TBZ) | (cd build; tar -xf -)
	mkdir build/gcc-$(GCC_VERSION)/build
	cd build/gcc-$(GCC_VERSION)/build && \
		../configure --prefix=$(PREFIX)  --target=arm-elf  --enable-interwork  --enable-multilib  --enable-languages="c,c++"  --with-newlib  --without-headers  --disable-shared  --with-gnu-as  --with-gnu-ld && \
		make -j5 all-gcc && \
		make install-gcc

gcc-base: binutils $(PREFIX)/bin/arm-elf-gcc

$(PREFIX)/arm-elf/lib/libc.a: $(NEWLIB_TBZ)
	rm -rf build/newlib*
	gzip -cd $(NEWLIB_TBZ) | (cd build; tar -xf -)
	mkdir build/newlib-$(NEWLIB_VERSION)/build
	cd build/newlib-$(NEWLIB_VERSION)/build && \
		../configure --prefix=$(PREFIX) --target=arm-elf --enable-interwork --with-gnu-ld --with-gnu-as --disable-shared && \
		make -j5 CFLAGS_FOR_TARGET="-ffunction-sections -fdata-sections -DPREFER_SIZE_OVER_SPEED -D__OPTIMIZE_SIZE__ -Os -fomit-frame-pointer -D__BUFSIZ__=256" all && \
		make install

newlib: gcc-base $(PREFIX)/arm-elf/lib/libc.a

$(PREFIX)/arm-elf/lib/libstdc++.a:
	cd build/gcc-$(GCC_VERSION)/build && \
		make -j5 all && \
		make install

toolchain: newlib $(PREFIX)/arm-elf/lib/libstdc++.a

clean:
	rm -rf build
