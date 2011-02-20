# Maintainer: Giuseppe Calderaro <giuseppecalderaro@gmail.com>
# Contributor: (misc updates) Michael Evans <mjevans1983@gmail.com>
# Contributor: (RT and misc) Ng Oon-Ee <ng oon ee AT gmail.com>

pkgdesc="The Linux Kernel and modules with tuxonice support and optional bfs/ck patches"
depends=('coreutils' 'module-init-tools' 'mkinitcpio>=0.5.15' 'kernel26-firmware')
pkgext=-ice
pkgname=kernel26$pkgext
pkgver=2.6.37
_minor_patch=1
icever=$pkgver$pkgext
pkgrel=1
makedepends=('xmlto' 'docbook-xsl')
arch=(i686 x86_64)
license=('GPL2')
url="http://www.kernel.org"

### User/Environment defined variables
enable_toi=${enable_toi:-1}
bfs_scheduler=${bfs_scheduler:-0}
ck_patches=${ck_patches:-0}
keep_source_code=${keep_source_code:-0}
menuconfig=${menuconfig:-0}
realtime_patch=${realtime_patch:-0}
local_patch_dir="${local_patch_dir:-}"
use_config="${use_config:-}"
use_config_gz=${use_config_gz:-0}
enable_reiser4=${enable_reiser4:-0} # not yet released for 2.6.37
make_jobs=${make_jobs:-2}
### Compile time defined variables
###

### Files / Versions
file_rt="patch-2.6.33.7.2-rt30.bz2"
file_reiser4="reiser4-for-2.6.36.patch.bz2"
#file_toi="tuxonice-3.2-rc2-for-2.6.36.patch.bz2"
file_toi="current-tuxonice-for-2.6.37.patch_0.bz2"
file_bfs="2.6.37-sched-bfs-363.patch"
patch_rev_ck="ck1"
file_ck="patch-${pkgver}-${patch_rev_ck}.bz2"
###

source=(http://kernel.org/pub/linux/kernel/v2.6/linux-${pkgver}.tar.bz2
 	http://www.kernel.org/pub/linux/kernel/v2.6/patch-${pkgver}.${_minor_patch}.bz2
	http://www.kernel.org/pub/linux/kernel/projects/rt/${file_rt}
	http://www.kernel.org/pub/linux/kernel/people/ck/patches/2.6/${pkgver}/${pkgver}-${patch_rev_ck}/${file_ck}
	http://www.tuxonice.net/files/${file_toi}
	http://ck.kolivas.org/patches/bfs/${pkgver}/${file_bfs}
	config
	config.x86_64
	$pkgname.preset
	mkinitcpio-$pkgname.conf)

md5sums=('c8ee37b4fdccdb651e0603d35350b434'
         '7693d1d32ed39346cc988e0f027e5890'
         'da527aea6a4a374f963f4063e548dc74'
         'd5c93c7df1692d364c15d8eea0b384c9'
         '6b19322620d4fabfb2db1bf6748020eb'
         '3455da009658ce7dd2f5f4ab358d29ee'
         '33946ae31868ea734e7d6750f6e113d1'
         '0c0fe551f217f9ebc762e3f8d4bc68d0'
         '541973d72e24a2def82d33884a781ee1'
         '4ec86e859234dc251dd16884235a9e37')

build() {
    cd ${srcdir}/linux-$pkgver

    # Applying official patch
    if [ "$_minor_patch" != "0" ] ; then
	echo "Applying patch-${pkgver}.${_minor_patch}.bz2"
	patch -Np1 -i ${srcdir}/patch-${pkgver}.${_minor_patch} || return 1
    fi

    if [ -n "${local_patch_dir}" ] && [ -d "${local_patch_dir}" ] ; then
	echo "Applying patches from ${local_patch_dir} ..."
	for my_patch in "${local_patch_dir}"/* ; do
		echo -e "Applying custom patch:\t'${my_patch}'" || true
		patch -Np1 -i "${my_patch}" || return 1
	done
    fi

    # Applying realtime patch
    if [ "$realtime_patch" = "1" ]; then
	echo "Applying real time patch"
	# Strip './Makefile' changes
	bzip2 -dkc ${srcdir}/${file_rt} \
            | sed '/diff --git a\/Makefile b\/Makefile/,/*DOCUMENTATION*/d' \
            | patch -Np1 || return 1
    fi
    
    # applying reiserfs4 patch
    if [ "$enable_reiser4" = "1" ]; then
	echo "Applying ${file_reiser4%.gz}"
	bzip2 -dc ${srcdir}/${file_reiser4} | patch -Np1 || return 1
    fi
    
    # applying tuxonice patch
    if [ "${enable_toi}" = "1" ]; then
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
    fi

    if [ "${bfs_scheduler}" = "1" ] && [ "${ck_patches}" = "0" ]; then
       # applying BFS scheduler patch
	echo "Applying BFS scheduler patch"
        patch -Np1 -i ${srcdir}/${file_bfs} || return 1
    fi
    if [ "${ck_patches}" = "1" ] ; then
	echo "Applying CK patches ${file_ck%.*}"
	# sed out the -ckX version to make kernel naming happy.
	bzip2 -dck ${srcdir}/${file_ck} \
	    | sed 's/+EXTRAVERSION := $(EXTRAVERSION)$(CKVERSION)/+EXTRAVERSION := $(EXTRAVERSION)/' \
	    | patch -Np1 || return 1
    fi
    
    # remove extraversion
    sed -i 's|^EXTRAVERSION = .*$|EXTRAVERSION =|g' Makefile
    
    # load configuration for i686 or x86_64
    if [ "$CARCH" = "x86_64" ]; then
	cat ../config.x86_64 > ./.config
    else
	cat ../config > ./.config
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
    
    # hack to prevent output kernel from being marked as dirty or git
    chmod +x ${srcdir}/linux-$pkgver/scripts/setlocalversion
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
    mkdir -p $pkgdir/{lib/modules,lib/firmware,boot}
    make INSTALL_MOD_PATH=$pkgdir modules_install || return 1
    install -D -m644 System.map $pkgdir/boot/System.map26$pkgext
    install -D -m644 arch/$KARCH/boot/bzImage $pkgdir/boot/vmlinuz26$pkgext
    install -D -m644 Makefile $pkgdir/usr/src/linux-$icever/Makefile
    install -D -m644 kernel/Makefile $pkgdir/usr/src/linux-$icever/kernel/Makefile
    install -D -m644 .config $pkgdir/usr/src/linux-$icever/.config
    install -D -m644 .config $pkgdir/boot/kconfig26$pkgext
    mkdir -p $pkgdir/usr/src/linux-$icever/include
    
    for i in acpi asm-generic config generated linux math-emu media net pcmcia scsi sound trace video xen; do
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
	if ls drivers/media/video/$i/*.h &>/dev/null; then
		mkdir -p $pkgdir/usr/src/linux-$icever/drivers/media/video/$i
		cp -a drivers/media/video/$i/*.h $pkgdir/usr/src/linux-$icever/drivers/media/video/$i
	else
		echo Skipping $i : drivers/media/video/$i/*.h
	fi
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
