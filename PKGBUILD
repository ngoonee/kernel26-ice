# Maintainer: Giuseppe Calderaro <giuseppecalderaro@gmail.com>
# Contributor: (misc updates) Michael Evans <mjevans1983@gmail.com>
# Contributor: (RT and misc) Ng Oon-Ee <ng oon ee AT gmail.com>

pkgdesc="The Linux Kernel and modules with gentoo-sources patchset and tuxonice support"
depends=('coreutils' 'module-init-tools' 'mkinitcpio>=0.5.15' 'kernel26-firmware')
pkgext=-ice
pkgname=kernel26$pkgext
pkgver=2.6.35
_minor_patch=4
icever=$pkgver$pkgext
pkgrel=3
makedepends=('xmlto' 'docbook-xsl')
arch=(i686 x86_64)
license=('GPL2')
url="http://www.kernel.org"

### User/Environment defined variables
bfs_scheduler=${bfs_scheduler:-0}
keep_source_code=${keep_source_code:-0}
menuconfig=${menuconfig:-0}
realtime_patch=${realtime_patch:-0}
use_config_gz=${use_config_gz:-0}
enable_reiser4=${enable_reiser4:-0}
enable_anti_stall=${enable_anti_stall:-0}
make_jobs=${make_jobs:-2}
### Compile time defined variables
###

### Files / Versions
file_rt="patch-2.6.33.7-rt29.bz2"
file_reiser4="reiser4-for-2.6.35.patch.bz2"
file_toi="tuxonice-3.2-rc1-for-2.6.35.patch.bz2"
file_bfs="2.6.35-sched-bfs-330.patch"
###

source=(http://kernel.org/pub/linux/kernel/v2.6/linux-${pkgver}.tar.bz2
	http://www.kernel.org/pub/linux/kernel/v2.6/patch-${pkgver}.${_minor_patch}.bz2
	http://www.kernel.org/pub/linux/kernel/projects/rt/${file_rt}
	http://sources.gentoo.org/viewcvs.py/*checkout*/linux-patches/genpatches-2.6/trunk/$pkgver/2900_xconfig-with-qt4.patch
	http://sources.gentoo.org/viewcvs.py/*checkout*/linux-patches/genpatches-2.6/trunk/$pkgver/4200_fbcondecor-0.9.6.patch
	http://www.kernel.org/pub/linux/kernel/people/edward/reiser4/reiser4-for-2.6/${file_reiser4}
	http://www.tuxonice.net/downloads/all/${file_toi}
	http://ck.kolivas.org/patches/bfs/${pkgver}/${file_bfs}
	vanilla-2.6.35-anti-io-stalling.patch
	config
	config.x86_64
	$pkgname.preset
	mkinitcpio-$pkgname.conf)

md5sums=('091abeb4684ce03d1d936851618687b6'
         '738f762746488345b1a8707d00895eef'
         'b59bd4ce52c54e639f9fd2d85c7cc951'
         'aa68610ca948e3c17aab8c8686baba76'
         'b31ec9691fdf2e5c2897ea1348c55600'
         '9d2bf8ef27b79559a0a7e09e59b41817'
         '0c378c843a7fa717a5a866cb58b6c871'
         'a6c4cce147143da9837d089d5e5d6ac5'
         'be68bdf00d287e6328226a174429fbb7'
         '70b5593b4cc2a0c29457b7c10e39d036'
         '05e289d000abfa2249de5c02934db799'
         '541973d72e24a2def82d33884a781ee1'
         '07dc6997d19340b654f92c1d6a120cc0')

build() {
    cd ${srcdir}/linux-$pkgver

    # Applying official patch
    if [ -n "${file_kernel_patch%.bz2}" ] ; then
	echo "Applying ${file_kernel_patch%.bz2}"
	patch -Np1 -i ${srcdir}/patch-${pkgver}.${_minor_patch}.bz2 || return 1
    fi
    
    # Applying realtime patch
    if [ "$realtime_patch" = "1" ]; then
	echo "Applying real time patch"
	# Strip './Makefile' changes
	bzip2 -dkc ${srcdir}/${file_rt} \
            | sed '/diff --git a\/Makefile b\/Makefile/,/*DOCUMENTATION*/d' \
            | patch -Np1 || return 1
    fi
    
    if [ "$realtime_patch" = "0" ]; then
      # Applying base and extra gentoo patches
	for i in $(ls ${srcdir}/[1-9][0-9][0-9][0-9]*); do
            echo "Applying $i"
            patch -Np1 -i $i || return 1
	done
    else
      # Applying only those specific patches which work with RT patchset
	for i in $(ls ${srcdir}/{1900,2700,4100,4400}*); do
            echo "Applying $i"
            patch -Np1 -i $i || return 1
	done
    fi
    
    # applying reiserfs4 patch
    if [ "$enable_reiser4" = "1" ]; then
	echo "Applying ${file_reiser4%.gz}"
	bzip2 -dc ${srcdir}/${file_reiser4} | patch -Np1 || return 1
    fi
    
    # applying tuxonice patch
    echo "Applying ${file_toi%.bz2}"
    # fix to tuxonice patch to work with rt
    if [ "$realtime_patch" = "1" ]; then
	bzip2 -dck ${srcdir}/${file_toi} \
            | sed '/diff --git a\/kernel\/fork.c b\/kernel\/fork.c/,/{/d' \
            | sed 's/printk(KERN_INFO "PM: Creating hibernation image:\\n/printk(KERN_INFO "PM: Creating hibernation image: \\n/' \
            | patch -Np1 || return 1
    else
	bzip2 -dck ${srcdir}/${file_toi} \
            | sed 's/printk(KERN_INFO "PM: Creating hibernation image:\\n/printk(KERN_INFO "PM: Creating hibernation image: \\n/' \
            | patch -Np1 -F4 || return 1
    fi
    
    if [ "$bfs_scheduler" = "1" ]; then
       # applying BFS scheduler patch
	echo "Applying BFS scheduler patch"
       ## Delete the Makefile changes that break patching.
	sed '/Index: linux-2.6.32-ck1\/Makefile/,/To see a list of typical targets execute "make help"/d' \
            ${srcdir}/${file_bfs} | patch -Np1 || return 1
    fi

    if [ "$enable_anti_stall" = "1" ]; then
	# vanilla-2.6.35-anti-io-stalling.patch
	echo "Applying vanilla-2.6.35-anti-io-stalling patch"
	patch -Np1 -i ${srcdir}/vanilla-2.6.35-anti-io-stalling.patch || return 1
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
	${srcdir}/linux-$pkgver/scripts/setlocalversion \
	> ${srcdir}/linux-$pkgver/scripts/setlocalversion.new
    mv ${srcdir}/linux-$pkgver/scripts/setlocalversion.new \
	${srcdir}/linux-$pkgver/scripts/setlocalversion
    
    make prepare
    
    # configure kernel
    if [ "$menuconfig" = "1" ]; then
	make menuconfig
    fi
    yes "" | make config
        
    cd ${srcdir}/linux-$pkgver
    # build kernel
    make -j${make_jobs} bzImage modules || return 1
}

package_kernel26-ice() {
    pkgdesc="The Linux Kernel and modules"
    groups=('base')
    backup=(etc/mkinitcpio.d/$pkgname.preset)
    depends=('coreutils' 'linux-firmware' 'module-init-tools' 'mkinitcpio>=0.5.20')
    replaces=('kernel24' 'kernel24-scsi' 'kernel26-scsi'
        'alsa-driver' 'ieee80211' 'hostap-driver26'
        'pwc' 'nforce' 'squashfs' 'unionfs' 'ivtv'
        'zd1211' 'kvm-modules' 'iwlwifi' 'rt2x00-cvs'
        'gspcav1' 'atl2' 'wlan-ng26' 'rt2500' 'nouveau-drm')
    install=$pkgname.install
    optdepends=('crda: to set the correct wireless channels of your country')
    
    KARCH=x86
    
    if [ "$keep_source_code" = "1" ]; then
	echo -n "Copying source code..."
	# Keep the source code
	cd $startdir
	mkdir -p $pkgdir/usr/src || return 1
	cp -a ${srcdir}/linux-$pkgver $pkgdir/usr/src/linux-$icever || return 1
	
	#Add a link from the modules directory
	mkdir -p $pkgdir/lib/modules/$icever || return 1
	cd $pkgdir/lib/modules/$icever || return 1
	rm -f source
	ln -s ../../../usr/src/linux-$icever source || return 1
	echo "OK"
    fi

    cd $srcdir/linux-$pkgver
    # get kernel version
    mkdir -p $pkgdir/{lib/modules,boot}
    make INSTALL_MOD_PATH=$pkgdir modules_install || return 1
    install -D -m644 System.map $pkgdir/boot/System.map26$pkgext
    install -D -m644 arch/$KARCH/boot/bzImage $pkgdir/boot/vmlinuz26$pkgext
    install -D -m644 Makefile $pkgdir/usr/src/linux-$icever/Makefile
    install -D -m644 kernel/Makefile $pkgdir/usr/src/linux-$icever/kernel/Makefile
    install -D -m644 .config $pkgdir/usr/src/linux-$icever/.config
    install -D -m644 .config $pkgdir/boot/kconfig26$pkgext
    mkdir -p $pkgdir/usr/src/linux-$icever/include
    
    for i in acpi asm-generic config generated linux math-emu media net pcmcia scsi sound trace video; do
  	cp -a include/$i $pkgdir/usr/src/linux-$icever/include/
    done
    
    # copy arch includes for external modules
    mkdir -p $pkgdir/usr/src/linux-$icever/arch/$KARCH
    cp -a arch/$KARCH/include $pkgdir/usr/src/linux-$icever/arch/$KARCH/
    
    # copy files necessary for later builds, like nvidia and vmware
    cp Module.symvers $pkgdir/usr/src/linux-$icever
    cp -a scripts $pkgdir/usr/src/linux-$icever
    
    # fix permissions on scripts dir
    chmod og-w -R $pkgdir/usr/src/linux-$icever/scripts
    
    mkdir -p $pkgdir/usr/src/linux-$icever/arch/$KARCH/kernel
    
    cp arch/$KARCH/Makefile $pkgdir/usr/src/linux-$icever/arch/$KARCH/
    if [ "${CARCH}" = "i686" ]; then
  	cp arch/$KARCH/Makefile_32.cpu $pkgdir/usr/src/linux-$icever/arch/$KARCH/
    fi
    cp arch/$KARCH/kernel/asm-offsets.s $pkgdir/usr/src/linux-$icever/arch/$KARCH/kernel/
    
    # add headers for lirc package
    mkdir -p $pkgdir/usr/src/linux-$icever/drivers/media/video
    cp drivers/media/video/*.h  $pkgdir/usr/src/linux-$icever/drivers/media/video/
    for i in bt8xx cpia2 cx25840 cx88 em28xx et61x251 pwc saa7134 sn9c102 usbvideo zc0301
    do
  	mkdir -p $pkgdir/usr/src/linux-$icever/drivers/media/video/$i
  	cp -a drivers/media/video/$i/*.h $pkgdir/usr/src/linux-$icever/drivers/media/video/$i
    done
    
    # add dm headers
    mkdir -p $pkgdir/usr/src/linux-$icever/drivers/md
    cp drivers/md/*.h  $pkgdir/usr/src/linux-$icever/drivers/md
    
    # add inotify.h
    mkdir -p $pkgdir/usr/src/linux-$icever/include/linux
    cp include/linux/inotify.h $pkgdir/usr/src/linux-$icever/include/linux/
    
    # add CLUSTERIP file for iptables
    mkdir -p $pkgdir/usr/src/linux-$icever/net/ipv4/netfilter/
    cp net/ipv4/netfilter/ipt_CLUSTERIP.c $pkgdir/usr/src/linux-$icever/net/ipv4/netfilter/
    
    # add wireless headers
    mkdir -p $pkgdir/usr/src/linux-$icever/net/mac80211/
    cp net/mac80211/*.h $pkgdir/usr/src/linux-$icever/net/mac80211/
    
    # add xfs and shmem for aufs building
    mkdir -p $pkgdir/usr/src/linux-$icever/fs/xfs
    mkdir -p $pkgdir/usr/src/linux-$icever/mm
    cp fs/xfs/xfs_sb.h $pkgdir/usr/src/linux-$icever/fs/xfs/xfs_sb.h
    cp mm/shmem.c $pkgdir/usr/src/linux-$icever/mm/shmem.c
    
    # add vmlinux
    cp vmlinux $pkgdir/usr/src/linux-$icever
    
    # copy in Kconfig files
    for i in $(find . -name "Kconfig*")
    do
  	mkdir -p $pkgdir/usr/src/linux-$icever/$(echo $i | sed 's|/Kconfig.*||')
  	cp $i $pkgdir/usr/src/linux-$icever/$i
    done
    
    chown -R root.root $pkgdir/usr/src/linux-$icever
    find $pkgdir/usr/src/linux-$icever -type d -exec chmod 755 {} \;
    cd $pkgdir/lib/modules/$icever && (rm -f source build;
    ln -sf ../../../usr/src/linux-$icever build)
    
    # install fallback mkinitcpio.conf file and preset file for kernel
    install -m644 -D ${srcdir}/$pkgname.preset $pkgdir/etc/mkinitcpio.d/${pkgname}.preset || return 1
    install -m644 -D ${srcdir}/mkinitcpio-$pkgname.conf $pkgdir/etc/mkinitcpio.d/$pkgname-fallback.conf || return 1
    
    # set correct depmod command for install
    sed -i -e "s/KERNEL_VERSION=.*/KERNEL_VERSION=$icever/g" $startdir/$pkgname.install
    echo -e "# DO NOT EDIT THIS FILE\nALL_kver='$icever'" > $pkgdir/etc/mkinitcpio.d/$pkgname.kver
    
    if [ "$keep_source_code" = "0" ]; then
  	# remove unneeded architectures
  	rm -rf $pkgdir/usr/src/linux-$icever/arch/{alpha,arm,arm26,avr32,blackfin,cris,frv,h8300,ia64,m32r,m68k,m68knommu,mips,mn10300,parisc,powerpc,ppc,s390,sh,sh64,sparc,sparc64,um,v850,xtensa}
    fi
    
    # Delete firmware directory
    rm -rf $pkgdir/lib/firmware
}
