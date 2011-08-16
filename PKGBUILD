# Contributor: Giuseppe Calderaro <giuseppecalderaro@gmail.com>
# Contributor: (misc updates) Michael Evans <mjevans1983@gmail.com>
# Maintainer: (RT and misc) Ng Oon-Ee <ngoonee.talk@gmail.com>

pkgdesc="The Linux Kernel and modules with tuxonice support and optional bfs/ck patches"
depends=('coreutils' 'linux-firmware' 'module-init-tools>=3.16' 'mkinitcpio>=0,7')
optdepends=('crda: to set the correct wireless channels of your country')
pkgname=linux-ice
backup=(etc/mkinitcpio.d/$pkgname.preset)
_kernelname=${pkgname#linux}
_basekernel=3.0
_minor_patch=2
pkgver=${_basekernel}
pkgrel=4
install=$pkgname.install
provides=('kernel26-ice')
conflicts=('kernel26-ice')
replaces=('kernel26-ice')
makedepends=('xmlto' 'docbook-xsl')
arch=(i686 x86_64)
license=('GPL2')
url="http://www.kernel.org"

### User/Environment defined variables
_skip_minor_patch=${_skip_minor_patch:-0}
enable_toi=${enable_toi:-1}
bfs_scheduler=${bfs_scheduler:-0}
keep_source_code=${keep_source_code:-0}
gconfig=${gconfig:-0}
xconfig=${xconfig:-0}
menuconfig=${menuconfig:-0}
local_patch_dir="${local_patch_dir:-}"
use_config="${use_config:-}"
use_config_gz=${use_config_gz:-0}
enable_reiser4=${enable_reiser4:-0}
### Compile time defined variables
###

### Files / Versions
file_reiser4="reiser4-for-2.6.38.patch.bz2"
file_toi="current-tuxonice-for-3.0.patch.bz2"
file_bfs="3.0-sched-bfs-406.patch"
###

options=(!strip)
source=(http://kernel.org/pub/linux/kernel/v3.0/linux-${_basekernel}.tar.bz2
        http://www.kernel.org/pub/linux/kernel/v3.0/patch-${_basekernel}.${_minor_patch}.gz
        # http://www.kernel.org/pub/linux/kernel/people/edward/reiser4/reiser4-for-2.6/${file_reiser4}
        http://www.tuxonice.net/files/${file_toi}
        http://ck.kolivas.org/patches/bfs/3.0.0/${file_bfs}
        # the main kernel config files
        config config.x86_64
        # standard config files for mkinitcpio ramdisk
        ${pkgname}.preset
        fix-i915.patch)
md5sums=('398e95866794def22b12dfbc15ce89c0'
         'd4ff10b7c9a5d8d0aa4c97579c81b7f9'
         'afbd01926c57fc5b82ee6034dc9311e5'
         '0399ec35c4c0998ce423e656094b91ac'
         'c466c85e967fe004455aacd26b89e1fb'
         'e932681c602ba4ddfbbb0fa96c5a7bc5'
         '1d27835b55d02f8cc66507a5ae1a6885'
         '263725f20c0b9eb9c353040792d644e5')
         
build() {
  cd ${srcdir}/linux-$_basekernel

  # Applying official patch
  if [ "$_minor_patch" != "0" ] && [ "$_skip_minor_patch" != "1" ] ; then
    echo "Applying patch-${_basekernel}.${_minor_patch}.bz2"
    patch -Np1 -i ${srcdir}/patch-${_basekernel}.${_minor_patch} || { echo Failed patch-${_basekernel}.${_minor_patch}.bz2 ; return 1 ; }
  fi

  if [ -n "${local_patch_dir}" ] && [ -d "${local_patch_dir}" ] ; then
    echo "Applying patches from ${local_patch_dir} ..."
    for my_patch in "${local_patch_dir}"/* ; do
      echo -e "Applying custom patch:\t'${my_patch}'" || true
      patch -Np1 -i "${my_patch}" || { echo -e "Failed custom patch:\t'${my_patch}'" ; return 1 ; }
    done
  fi

  # applying reiserfs4 patch
  if [ "$enable_reiser4" = "1" ]; then
    echo "Applying ${file_reiser4%.gz}"
    bzip2 -dc ${srcdir}/${file_reiser4} | patch -Np1 || { echo "Failed reiser4 patch '${file_reiser4%.gz}'"; return 1 ; }
  fi

  # applying tuxonice patch
  if [ "${enable_toi}" = "1" ]; then
    echo "Applying ${file_toi%.bz2}"
      bzip2 -dck ${srcdir}/${file_toi} \
        | sed 's/printk(KERN_INFO "PM: Creating hibernation image:\\n/printk(KERN_INFO "PM: Creating hibernation image: \\n/' \
        | patch -Np1 -F4 || { echo "Failed TOI"; return 1 ; }
  fi

  if [ "${bfs_scheduler}" = "1" ]; then
    # applying BFS scheduler patch
     echo "Applying BFS scheduler patch"
     patch -Np1 -i ${srcdir}/${file_bfs} || { echo "Failed BFS"; return 1 ; }
  fi

  # remove extraversion
  sed -i 's|^EXTRAVERSION = .*$|EXTRAVERSION =|g' Makefile
  
  # fix #19234 i1915 display size
  patch -Np1 -i ${srcdir}/fix-i915.patch

  # load configuration for i686 or x86_64
  if [ "$CARCH" = "x86_64" ]; then
    cat ../config.x86_64 >./.config
  else
    cat ../config >./.config
  fi

  # use custom config instead
  if [ -n "${use_config}" ] ; then
    echo "Using config: '${use_config}'"
    cat "${use_config}" > ./.config
    make oldconfig
  fi

  # use existing config.gz
  if [ "$use_config_gz" = "1" ]; then
    zcat /proc/config.gz > ./.config
    make oldconfig
  fi

  if [ "${_kernelname}" != "" ]; then
    sed -i "s|CONFIG_LOCALVERSION=.*|CONFIG_LOCALVERSION=\"${_kernelname}\"|g" ./.config
  fi
  # remove the extraversion from Makefile
  # this ensures our kernel version is always 3.X-ARCH
  # this way, minor kernel updates will not break external modules
  # we need to change this soon, see FS#16702
  sed -i 's|^EXTRAVERSION = .*$|EXTRAVERSION = |g' Makefile
  # get kernel version

  # hack to prevent output kernel from being marked as dirty or git
  #chmod +x ${srcdir}/linux-${_basekernel}/scripts/setlocalversion
  #sed 's/head=`git rev-parse --verify --short HEAD 2>\/dev\/null`/0/' \
  #  ${srcdir}/linux-${_basekernel}/scripts/setlocalversion \
  #  > ${srcdir}/linux-${_basekernel}/scripts/setlocalversion.new
  #mv ${srcdir}/linux-${_basekernel}/scripts/setlocalversion.new \
  #  ${srcdir}/linux-${_basekernel}/scripts/setlocalversion

  make prepare
  # load configuration
  # Configure the kernel. Replace the line below with one of your choice.
  if [ "$gconfig" = "1" ]; then
    make gconfig
  else
    if [ "$xconfig" = "1" ]; then
      make xconfig
    else
      if [ "$menuconfig" = "1" ]; then
        make menuconfig
      fi
    fi
  fi
  yes "" | make config
  make prepare # Necessary in case config has been changed
  # build!
  make ${MAKEFLAGS} bzImage modules
}

package_linux-ice() {

  KARCH=x86
  cd ${srcdir}/linux-${_basekernel}
  if [ "$keep_source_code" = "1" ]; then
    echo -n "Copying source code..."
    # Keep the source code
    cd $startdir
    mkdir -p $pkgdir/usr/src || return 1
    cp -a ${srcdir}/linux-${_basekernel} $pkgdir/usr/src/linux-${_kernver} || return 1

    #Add a link from the modules directory
    mkdir -p $pkgdir/lib/modules/${_kernver} || return 1
    cd $pkgdir/lib/modules/${_kernver} || return 1
    rm -f source
    ln -s ../../../usr/src/linux-${_kernver} source || return 1
    echo "OK"
  fi
   
  # get kernel version
  _kernver="$(make kernelrelease)"
  mkdir -p ${pkgdir}/{lib/modules,lib/firmware,boot}
  make INSTALL_MOD_PATH=${pkgdir} modules_install
  cp System.map ${pkgdir}/boot/System.map26${_kernelname}
  cp arch/$KARCH/boot/bzImage ${pkgdir}/boot/vmlinuz-${pkgname}
  # add vmlinux
  install -m644 -D vmlinux ${pkgdir}/usr/src/linux-${_kernver}/vmlinux

  # install fallback mkinitcpio.conf file and preset file for kernel
  install -m644 -D ${srcdir}/${pkgname}.preset ${pkgdir}/etc/mkinitcpio.d/${pkgname}.preset
  # set correct depmod command for install
  sed \
    -e  "s/KERNEL_NAME=.*/KERNEL_NAME=${_kernelname}/g" \
    -e  "s/KERNEL_VERSION=.*/KERNEL_VERSION=${_kernver}/g" \
    -i $startdir/${pkgname}.install
  sed \
    -e "s|source .*|source /etc/mkinitcpio.d/kernel26${_kernelname}.kver|g" \
    -e "s|default_image=.*|default_image=\"/boot/initramfs-${pkgname}.img\"|g" \
    -e "s|fallback_image=.*|fallback_image=\"/boot/initramfs-${pkgname}-fallback.img\"|g" \
    -i ${pkgdir}/etc/mkinitcpio.d/${pkgname}.preset

  echo -e "# DO NOT EDIT THIS FILE\nALL_kver='${_kernver}'" > ${pkgdir}/etc/mkinitcpio.d/${pkgname}.kver
  # remove build and source links
  rm -f ${pkgdir}/lib/modules/${_kernver}/{source,build}
  # add compat symlink for the kernel image
  ln -sf vmlinuz-${pkgname} ${pkgdir}/boot/vmlinuz26${_kernelname}
  # remove the firmware
  rm -rf ${pkgdir}/lib/firmware
  # gzip -9 all modules to safe 100MB of space
  find "$pkgdir" -name '*.ko' -exec gzip -9 {} \;

  mkdir -p ${pkgdir}/lib/modules/${_kernver}
  cd ${pkgdir}/lib/modules/${_kernver}
  ln -sf ../../../usr/src/linux-${_kernver} build
  cd ${srcdir}/linux-$_basekernel
  install -D -m644 Makefile \
    ${pkgdir}/usr/src/linux-${_kernver}/Makefile
  install -D -m644 kernel/Makefile \
    ${pkgdir}/usr/src/linux-${_kernver}/kernel/Makefile
  install -D -m644 .config \
    ${pkgdir}/usr/src/linux-${_kernver}/.config
  mkdir -p ${pkgdir}/usr/src/linux-${_kernver}/include

  for i in acpi asm-generic config crypto drm generated linux math-emu \
    media net pcmcia scsi sound trace video xen; do
    cp -a include/$i ${pkgdir}/usr/src/linux-${_kernver}/include/
  done

  # copy arch includes for external modules
  mkdir -p ${pkgdir}/usr/src/linux-${_kernver}/arch/x86
  cp -a arch/x86/include ${pkgdir}/usr/src/linux-${_kernver}/arch/x86/

  # copy files necessary for later builds, like nvidia and vmware
  cp Module.symvers ${pkgdir}/usr/src/linux-${_kernver}
  cp -a scripts ${pkgdir}/usr/src/linux-${_kernver}
  # fix permissions on scripts dir
  chmod og-w -R ${pkgdir}/usr/src/linux-${_kernver}/scripts
  mkdir -p ${pkgdir}/usr/src/linux-${_kernver}/.tmp_versions

  mkdir -p ${pkgdir}/usr/src/linux-${_kernver}/arch/$KARCH/kernel

  cp arch/$KARCH/Makefile ${pkgdir}/usr/src/linux-${_kernver}/arch/$KARCH/
  if [ "$CARCH" = "i686" ]; then
    cp arch/$KARCH/Makefile_32.cpu ${pkgdir}/usr/src/linux-${_kernver}/arch/$KARCH/
  fi
  cp arch/$KARCH/kernel/asm-offsets.s ${pkgdir}/usr/src/linux-${_kernver}/arch/$KARCH/kernel/

  # add headers for lirc package
  mkdir -p ${pkgdir}/usr/src/linux-${_kernver}/drivers/media/video
  cp drivers/media/video/*.h  ${pkgdir}/usr/src/linux-${_kernver}/drivers/media/video/
  for i in bt8xx cpia2 cx25840 cx88 em28xx et61x251 pwc saa7134 sn9c102; do
   mkdir -p ${pkgdir}/usr/src/linux-${_kernver}/drivers/media/video/$i
   cp -a drivers/media/video/$i/*.h ${pkgdir}/usr/src/linux-${_kernver}/drivers/media/video/$i
  done
  # add docbook makefile
  install -D -m644 Documentation/DocBook/Makefile \
    ${pkgdir}/usr/src/linux-${_kernver}/Documentation/DocBook/Makefile
  # add dm headers
  mkdir -p ${pkgdir}/usr/src/linux-${_kernver}/drivers/md
  cp drivers/md/*.h  ${pkgdir}/usr/src/linux-${_kernver}/drivers/md
  # add inotify.h
  mkdir -p ${pkgdir}/usr/src/linux-${_kernver}/include/linux
  cp include/linux/inotify.h ${pkgdir}/usr/src/linux-${_kernver}/include/linux/
  # add wireless headers
  mkdir -p ${pkgdir}/usr/src/linux-${_kernver}/net/mac80211/
  cp net/mac80211/*.h ${pkgdir}/usr/src/linux-${_kernver}/net/mac80211/
  # add dvb headers for external modules
  # in reference to:
  # http://bugs.archlinux.org/task/9912
  mkdir -p ${pkgdir}/usr/src/linux-${_kernver}/drivers/media/dvb/dvb-core
  cp drivers/media/dvb/dvb-core/*.h ${pkgdir}/usr/src/linux-${_kernver}/drivers/media/dvb/dvb-core/
  # add dvb headers for external modules
  # in reference to:
  # http://bugs.archlinux.org/task/11194
  #mkdir -p ${pkgdir}/usr/src/linux-${_kernver}/include/config/dvb/
  #cp include/config/dvb/*.h ${pkgdir}/usr/src/linux-${_kernver}/include/config/dvb/
  # add dvb headers for http://mcentral.de/hg/~mrec/em28xx-new
  # in reference to:
  # http://bugs.archlinux.org/task/13146
  mkdir -p ${pkgdir}/usr/src/linux-${_kernver}/drivers/media/dvb/frontends/
  cp drivers/media/dvb/frontends/lgdt330x.h ${pkgdir}/usr/src/linux-${_kernver}/drivers/media/dvb/frontends/
  cp drivers/media/video/msp3400-driver.h ${pkgdir}/usr/src/linux-${_kernver}/drivers/media/dvb/frontends/
  # add dvb headers  
  # in reference to:
  # http://bugs.archlinux.org/task/20402
  mkdir -p ${pkgdir}/usr/src/linux-${_kernver}/drivers/media/dvb/dvb-usb
  cp drivers/media/dvb/dvb-usb/*.h ${pkgdir}/usr/src/linux-${_kernver}/drivers/media/dvb/dvb-usb/
  mkdir -p ${pkgdir}/usr/src/linux-${_kernver}/drivers/media/dvb/frontends
  cp drivers/media/dvb/frontends/*.h ${pkgdir}/usr/src/linux-${_kernver}/drivers/media/dvb/frontends/
  mkdir -p ${pkgdir}/usr/src/linux-${_kernver}/drivers/media/common/tuners
  cp drivers/media/common/tuners/*.h ${pkgdir}/usr/src/linux-${_kernver}/drivers/media/common/tuners/
  # add xfs and shmem for aufs building
  mkdir -p ${pkgdir}/usr/src/linux-${_kernver}/fs/xfs
  mkdir -p ${pkgdir}/usr/src/linux-${_kernver}/mm
  cp fs/xfs/xfs_sb.h ${pkgdir}/usr/src/linux-${_kernver}/fs/xfs/xfs_sb.h
  # add headers vor virtualbox
  # in reference to:
  # http://bugs.archlinux.org/task/14568
  #cp -a include/drm $pkgdir/usr/src/linux-${_kernver}/include/
  # add headers for broadcom wl
  # in reference to:
  # http://bugs.archlinux.org/task/14568
  #cp -a include/trace $pkgdir/usr/src/linux-${_kernver}/include/
  # add headers for crypto modules
  # in reference to:
  # http://bugs.archlinux.org/task/22081
  #cp -a include/crypto $pkgdir/usr/src/linux-${_kernver}/include/
  # copy in Kconfig files
  for i in `find . -name "Kconfig*"`; do 
    mkdir -p ${pkgdir}/usr/src/linux-${_kernver}/`echo $i | sed 's|/Kconfig.*||'`
    cp $i ${pkgdir}/usr/src/linux-${_kernver}/$i
  done

  chown -R root.root ${pkgdir}/usr/src/linux-${_kernver}
  find ${pkgdir}/usr/src/linux-${_kernver} -type d -exec chmod 755 {} \;
  # strip scripts directory
  find ${pkgdir}/usr/src/linux-${_kernver}/scripts  -type f -perm -u+w 2>/dev/null | while read binary ; do
  case "$(file -bi "$binary")" in
    *application/x-sharedlib*) # Libraries (.so)
    /usr/bin/strip $STRIP_SHARED "$binary";;
    *application/x-archive*) # Libraries (.a)
    /usr/bin/strip $STRIP_STATIC "$binary";;
    *application/x-executable*) # Binaries
    /usr/bin/strip $STRIP_BINARIES "$binary";;
    esac 
  done 
  # remove unneeded architectures
  if [ "$keep_source_code" = "0" ]; then
    rm -rf ${pkgdir}/usr/src/linux-${_kernver}/arch/{alpha,arm,arm26,avr32,blackfin,cris,frv,h8300,ia64,m32r,m68k,m68knommu,mips,microblaze,mn10300,parisc,powerpc,ppc,s390,sh,sh64,sparc,sparc64,um,v850,xtensa}
  fi
}
