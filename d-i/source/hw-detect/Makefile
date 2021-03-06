DEB_HOST_ARCH ?= $(shell dpkg-architecture -qDEB_HOST_ARCH)
DEB_HOST_ARCH_OS ?= $(shell dpkg-architecture -qDEB_HOST_ARCH_OS 2>/dev/null)

# Take account of old dpkg-architecture output.
ifeq ($(DEB_HOST_ARCH_OS),)
  DEB_HOST_ARCH_OS := $(subst -gnu,,$(shell dpkg-architecture -qDEB_HOST_GNU_SYSTEM))
  ifeq ($(DEB_HOST_ARCH_OS),gnu)
    DEB_HOST_ARCH_OS := hurd
  endif
endif

CC = gcc
CFLAGS = -W -Wall -Os -g
CFLAGS_ARCH = -DCPU_TEXT='"$(DEB_HOST_ARCH)"'

bindir=$(DESTDIR)/bin/
hpdir=$(DESTDIR)/etc/hotplug.d
udevdir=$(DESTDIR)/etc/udev/rules.d
udevlibdir=$(DESTDIR)/lib/udev
didir=$(DESTDIR)/lib/debian-installer.d
fidir=$(DESTDIR)/usr/lib/finish-install.d
sharedir=$(DESTDIR)/usr/share/hw-detect/
postbaseinstallerdir=$(DESTDIR)/usr/lib/post-base-installer.d/

INSTALL=install
INSTALL_DATA = ${INSTALL} -m 644

all: archdetect

clean:
	rm -f *~
	rm -f *.o
	rm -f archdetect

install: install-hw-detect install-ethdetect install-disk-detect install-archdetect install-archdetect-deb

install-hw-detect: hw-detect.sh
	$(INSTALL) -d $(bindir)
ifeq ($(DEB_HOST_ARCH_OS),linux)
	$(INSTALL) hw-detect.sh $(bindir)/hw-detect
	$(INSTALL) hotplug-pcmcia.sh $(bindir)/hotplug-pcmcia
	$(INSTALL) sysfs-update-devnames.sh $(bindir)/sysfs-update-devnames
	$(INSTALL) -d $(udevdir)
	$(INSTALL_DATA) net-hotplug.rules $(udevdir)/010_net-hotplug.rules
	$(INSTALL) -d $(udevlibdir)
	$(INSTALL) firmware.agent $(udevlibdir)
	$(INSTALL) check-missing-firmware.sh $(bindir)/check-missing-firmware
	$(INSTALL) -d $(postbaseinstallerdir)
	$(INSTALL) post-base-installer $(postbaseinstallerdir)/50hw-detect
	$(INSTALL) -d $(hpdir)/net
	$(INSTALL) net-hotplug.sh $(hpdir)/net/hw-detect.hotplug
	$(INSTALL) -d $(fidir)
	$(INSTALL) hw-detect.finish-install $(fidir)/30hw-detect
ifeq ($(DEB_HOST_ARCH),powerpc)
	$(INSTALL) discover-mac-io.sh $(bindir)/discover-mac-io
	$(INSTALL) discover-ibm.sh $(bindir)/discover-ibm
endif
ifeq ($(DEB_HOST_ARCH),sparc)
	$(INSTALL) discover-sbus.sh $(bindir)/discover-sbus
	$(INSTALL) -d $(sharedir)
	$(INSTALL_DATA) sbus.list $(sharedir)/sbus.list
endif
else
	$(INSTALL) detect-stub.sh $(bindir)/hw-detect
endif

install-ethdetect: ethdetect.sh
	$(INSTALL) -d $(bindir)
	$(INSTALL) ethdetect.sh $(bindir)/ethdetect

install-disk-detect: disk-detect.sh
	$(INSTALL) -d $(bindir)
	$(INSTALL) disk-detect.sh $(bindir)/disk-detect

install-archdetect: archdetect
	$(INSTALL) -d $(bindir)
	$(INSTALL) archdetect $(bindir)

install-archdetect-deb: archdetect
	$(INSTALL) -d $(bindir)
	$(INSTALL) archdetect $(bindir)

archdetect: archdetect.o 
	${CC} ${LDFLAGS} -o $@ $^ -ldebian-installer

archdetect.o: %.o:%.c
	$(CC) $(CFLAGS) $(CFLAGS_ARCH) -c $<

