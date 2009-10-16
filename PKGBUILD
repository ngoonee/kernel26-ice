# Contributor: Giuseppe Calderaro <giuseppecalderaro@gmail.com>
# Contributor: (misc updates) Michael Evans <mjevans1983@gmail.com>

pkgext=-ice
pkgname=kernel26$pkgext
pkgver=2.6.31
pkgrel=8
pkgdesc="The Linux Kernel and modules with gentoo-sources patchset and tuxonice support"
arch=('i686' 'x86_64')
license=('GPL2')
url="http://www.kernel.org"
backup=('boot/kconfig26$pkgext' etc/mkinitcpio.d/${pkgname}.preset etc/mkinitcpio.d/${pkgname}-fallback.conf)
depends=('coreutils' 'module-init-tools' 'mkinitcpio>=0.5.15' 'kernel26-firmware')
install=$pkgname.install

### User defined variables
bfs_scheduler="0"
enable_fastboot="0"
keep_source_code="0"
menuconfig="0"
realtime_patch="0"
use_config_gz="0"
###

### Files / Versions
file_kernel="linux-2.6.31.tar.bz2"
file_kernel_patch="patch-2.6.31.4.bz2"
file_rt="patch-2.6.31.4-rt14.bz2"
file_reiser4="reiser4-for-2.6.31.patch.gz"
file_toi="current-tuxonice-for-2.6.31.patch-20091009-v1.bz2"
file_bfs="2.6.31-sched-bfs-303.patch"
file_fastboot="Auke-Kok-s-patch-to-kernel-2.6.30.patch"
###

source=(http://kernel.org/pub/linux/kernel/v2.6/${file_kernel}
	http://www.kernel.org/pub/linux/kernel/v2.6/${file_kernel_patch}
	http://www.kernel.org/pub/linux/kernel/projects/rt/${file_rt}
	http://sources.gentoo.org/viewcvs.py/*checkout*/linux-patches/genpatches-2.6/trunk/2.6.31/2700_kworld-plustv-dual-dvb.patch
	http://sources.gentoo.org/viewcvs.py/*checkout*/linux-patches/genpatches-2.6/trunk/2.6.31/4100_dm-bbr.patch
	http://sources.gentoo.org/viewcvs.py/*checkout*/linux-patches/genpatches-2.6/trunk/2.6.31/4200_fbcondecor-0.9.6.patch
	http://sources.gentoo.org/viewcvs.py/*checkout*/linux-patches/genpatches-2.6/trunk/2.6.31/4400_alpha-sysctl-uac.patch
	http://www.kernel.org/pub/linux/kernel/people/edward/reiser4/reiser4-for-2.6/${file_reiser4}
	http://www.tuxonice.net/downloads/all/${file_toi}
	http://ck.kolivas.org/patches/bfs/${file_bfs}
	${file_fastboot}
	config
	config.x86_64
	$pkgname.preset
	mkinitcpio-$pkgname.conf)

md5sums=('84c077a37684e4cbfa67b18154390d8a'
         '02078f4231baee4f0004212f2875df2b'
         'cc6ffc13f5ce52792ddc878e0beac5bb'
         'e9d1d9593503bcf633f47a1c48a578b2'
         'e501d050605a7399e7b12a6b14903631'
         '6906c45acbaf073915fe24ec2632130b'
         '21562518ab45d8be9c67d316aef9399f'
         '62ab025dda9d5c291b6e7ce763e557c1'
         '8066cf922d24d227bf8e849dcba2e0b3'
         '403360f48dd0f85621b357221ed011e3'
         '5bd5c60b7e7664e8794279e99cafd185'
         '5b6618e68db20c720ea6c9351b315c68'
         '4aa04d55ae3cdeeea24fbf078ec33b3e'
         '541973d72e24a2def82d33884a781ee1'
         '07dc6997d19340b654f92c1d6a120cc0')

build() {
    [ "${CARCH}" = "i686" ]   && KARCH=x86
    [ "${CARCH}" = "x86_64" ] && KARCH=x86

    cd $startdir/src/linux-$pkgver

    # Applying official patch
    echo "Applying ${file_kernel_patch%.bz2}"
    patch -Np1 -i $startdir/src/${file_kernel_patch%.bz2} || return 1

    # Applying realtime patch
    if [ "$realtime_patch" = "1" ]; then
       echo "Applying real time patch"
       # Strip './Makefile' changes
       bzip2 -dkc $startdir/src/${file_rt} \
         | sed '/diff --git a\/Makefile b\/Makefile/,/*DOCUMENTATION*/d' \
         | patch -Np1 || return 1
    fi

    # Applying base gentoo patches
    for i in $(ls $startdir/src/[1-3][0-9][0-9][0-9]*); do
    	echo "Applying $i"
	patch -Np1 -i $i || return 1
    done

    if [ "$realtime_patch" = "0" ]; then
      # Applying extra gentoo patches
      for i in $(ls $startdir/src/[4-9][0-9][0-9][0-9]*); do
        echo "Applying $i"
        patch -Np1 -i $i || return 1
      done
    fi

    # applying reiserfs4 patch
    echo "Applying ${file_reiser4%.gz}"
    gzip -dc $startdir/src/${file_reiser4} | patch -Np1 || return 1

    # applying tuxonice patch
    echo "Applying ${file_toi%.bz2}"
    # fix to tuxonice patch to work with rt
    if [ "$realtime_patch" = "1" ]; then
       bzip2 -dck $startdir/src/${file_toi} \
         | sed '/diff --git a\/kernel\/fork.c b\/kernel\/fork.c/,/{/d' \
         | patch -Np1 || return 1
    else
       bzip2 -dck $startdir/src/${file_toi} | patch -Np1 || return 1
    fi

    if [ "$enable_fastboot" = "1" ]; then
       # applying fastboot patch
       echo "Applying fastboot (${file_fastboot})"
       patch -Np1 -i $startdir/src/${file_fastboot} || return 1
    fi

    if [ "$bfs_scheduler" = "1" ]; then
       # applying BFS scheduler patch
       echo "Applying BFS scheduler patch"
       ## Delete the Makefile changes that break patching.
       sed '/Index: linux-2.6.31-bfs\/Makefile/,/To see a list of typical targets execute "make help"/d' \
         $startdir/src/${file_bfs} | patch -Np1 || return 1
    fi

    # remove extraversion
    sed -i 's|^EXTRAVERSION = .*$|EXTRAVERSION =|g' Makefile

    # load configuration for i686 or x86_64
    if [ "$CARCH" = "x86_64" ]; then
	cat ../config.x86_64 > ./.config
    else
	cat ../config > ./.config
    fi

    # use existing config.gz
    if [ "$use_config_gz" = "1" ]; then
      zcat /proc/config.gz > ./.config
      make oldconfig
    fi

    # hack to prevent output kernel from being marked as dirty or git
    sed 's/head=`git rev-parse --verify --short HEAD 2>\/dev\/null`/0/' \
      $srcdir/linux-$pkgver/scripts/setlocalversion \
      > $srcdir/linux-$pkgver/scripts/setlocalversion

    # get kernel version
    make prepare
    _kernver="$(make kernelrelease)"

    # configure kernel
    if [ "$menuconfig" = "1" ]; then
      make menuconfig
    fi
    yes "" | make config

    # get kernel version if it has been changed in make config
    # Is this the best way to do it? Should make config just run before make prepare?
    #      - ngoonee
    make prepare
    _kernver="$(make kernelrelease)"

    if [ "$keep_source_code" = "1" ]; then
	echo -n "Copying source code..."
	# Keep the source code
	cd $startdir || return 1
	mkdir -p $startdir/pkg/usr/src || return 1
	cp -a $startdir/src/linux-$pkgver $startdir/pkg/usr/src/linux-$_kernver || return 1

	#Add a link from the modules directory
	mkdir -p $startdir/pkg/lib/modules/$_kernver || return 1
	cd $startdir/pkg/lib/modules/$_kernver || return 1
	rm -f source
	ln -s ../../../usr/src/linux-$_kernver source || return 1
	echo "OK"
    fi

    cd $startdir/src/linux-$pkgver
    # build kernel
    make bzImage modules || return 1
    mkdir -p $startdir/pkg/{lib/modules,boot}
    make INSTALL_MOD_PATH=$startdir/pkg modules_install || return 1
    install -D -m644 System.map $startdir/pkg/boot/System.map26$pkgext
    install -D -m644 arch/$KARCH/boot/bzImage $startdir/pkg/boot/vmlinuz26$pkgext
    install -D -m644 Makefile $startdir/pkg/usr/src/linux-$_kernver/Makefile
    install -D -m644 kernel/Makefile $startdir/pkg/usr/src/linux-$_kernver/kernel/Makefile
    install -D -m644 .config $startdir/pkg/usr/src/linux-$_kernver/.config
    install -D -m644 .config $startdir/pkg/boot/kconfig26$pkgext
    mkdir -p $startdir/pkg/usr/src/linux-$_kernver/include

    for i in acpi asm-{generic,$KARCH} config linux math-emu media net pcmcia scsi sound trace video; do
	cp -a include/$i $startdir/pkg/usr/src/linux-$_kernver/include/
    done

    # copy arch includes for external modules
    mkdir -p ${pkgdir}/usr/src/linux-${_kernver}/arch/$KARCH
    cp -a arch/$KARCH/include ${pkgdir}/usr/src/linux-${_kernver}/arch/$KARCH/

    # copy files necessary for later builds, like nvidia and vmware
    cp Module.symvers $startdir/pkg/usr/src/linux-$_kernver
    cp -a scripts $startdir/pkg/usr/src/linux-$_kernver

    # fix permissions on scripts dir
    chmod og-w -R $startdir/pkg/usr/src/linux-$_kernver/scripts

    mkdir -p $startdir/pkg/usr/src/linux-$_kernver/arch/$KARCH/kernel

    cp arch/$KARCH/Makefile $startdir/pkg/usr/src/linux-$_kernver/arch/$KARCH/
    if [ "${CARCH}" = "i686" ]; then
	cp arch/$KARCH/Makefile_32.cpu $startdir/pkg/usr/src/linux-$_kernver/arch/$KARCH/
    fi
    cp arch/$KARCH/kernel/asm-offsets.s $startdir/pkg/usr/src/linux-$_kernver/arch/$KARCH/kernel/

    # add headers for lirc package
    mkdir -p $startdir/pkg/usr/src/linux-$_kernver/drivers/media/video
    cp drivers/media/video/*.h  $startdir/pkg/usr/src/linux-$_kernver/drivers/media/video/
    for i in bt8xx cpia2 cx25840 cx88 em28xx et61x251 pwc saa7134 sn9c102 usbvideo zc0301
    do
	mkdir -p $startdir/pkg/usr/src/linux-$_kernver/drivers/media/video/$i
	cp -a drivers/media/video/$i/*.h $startdir/pkg/usr/src/linux-$_kernver/drivers/media/video/$i
    done

    # add dm headers
    mkdir -p $startdir/pkg/usr/src/linux-$_kernver/drivers/md
    cp drivers/md/*.h  $startdir/pkg/usr/src/linux-$_kernver/drivers/md

    # add inotify.h
    mkdir -p $startdir/pkg/usr/src/linux-$_kernver/include/linux
    cp include/linux/inotify.h $startdir/pkg/usr/src/linux-$_kernver/include/linux/

    # add CLUSTERIP file for iptables
    mkdir -p $startdir/pkg/usr/src/linux-$_kernver/net/ipv4/netfilter/
    cp net/ipv4/netfilter/ipt_CLUSTERIP.c $startdir/pkg/usr/src/linux-$_kernver/net/ipv4/netfilter/

    # add wireless headers
    mkdir -p $startdir/pkg/usr/src/linux-$_kernver/net/mac80211/
    cp net/mac80211/*.h $startdir/pkg/usr/src/linux-$_kernver/net/mac80211/

    # add xfs and shmem for aufs building
    mkdir -p $startdir/pkg/usr/src/linux-$_kernver/fs/xfs
    mkdir -p $startdir/pkg/usr/src/linux-$_kernver/mm
    cp fs/xfs/xfs_sb.h $startdir/pkg/usr/src/linux-$_kernver/fs/xfs/xfs_sb.h
    cp mm/shmem.c $startdir/pkg/usr/src/linux-$_kernver/mm/shmem.c

    # add vmlinux
    cp vmlinux $startdir/pkg/usr/src/linux-$_kernver

    # copy in Kconfig files
    for i in $(find . -name "Kconfig*")
    do
	mkdir -p $startdir/pkg/usr/src/linux-$_kernver/$(echo $i | sed 's|/Kconfig.*||')
	cp $i $startdir/pkg/usr/src/linux-$_kernver/$i
    done

    cd $startdir/pkg/usr/src/linux-$_kernver/include && ln -s asm-$KARCH asm

    chown -R root.root $startdir/pkg/usr/src/linux-$_kernver
    find $startdir/pkg/usr/src/linux-$_kernver -type d -exec chmod 755 {} \;
    cd $startdir/pkg/lib/modules/$_kernver && (rm -f source build; ln -sf ../../../usr/src/linux-$_kernver build)

    # install fallback mkinitcpio.conf file and preset file for kernel
    install -m644 -D $startdir/src/$pkgname.preset $startdir/pkg/etc/mkinitcpio.d/$pkgname.preset || return 1
    install -m644 -D $startdir/src/mkinitcpio-$pkgname.conf $startdir/pkg/etc/mkinitcpio.d/$pkgname-fallback.conf || return 1

    # set correct depmod command for install
    sed -i -e "s/KERNEL_VERSION=.*/KERNEL_VERSION=${_kernver}/g" $startdir/$pkgname.install
    echo -e "# DO NOT EDIT THIS FILE\nALL_kver='${_kernver}'" > $startdir/pkg/etc/mkinitcpio.d/$pkgname.kver

    if [ "$keep_source_code" = "0" ]; then
	# remove unneeded architectures
	rm -rf $startdir/pkg/usr/src/linux-$_kernver/arch/{alpha,arm,arm26,avr32,blackfin,cris,frv,h8300,ia64,m32r,m68k,m68knommu,mips,mn10300,parisc,powerpc,ppc,s390,sh,sh64,sparc,sparc64,um,v850,xtensa}
    fi

    # Delete firmware directory
    rm -rf ${pkgdir}/lib/firmware
}
