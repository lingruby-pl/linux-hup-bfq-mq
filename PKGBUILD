# Maintainer: Jacek Bienias <sp7ezd@gmail.com>
# Maintainer: Piotr Gorski <lucjan.lucjanov@gmail.com>
# Contributor: Tobias Powalowski <tpowa@archlinux.org>
# Contributor: Thomas Baechler <thomas@archlinux.org>

### BUILD OPTIONS
# Set these variables to ANYTHING that is not null to enable them

### Use tentative patches from https://groups.google.com/forum/#!forum/bfq-iosched
### Thx to Tom 'monotykamary' Nguyen
_use_tentative_patches=y

### Tweak kernel options prior to a build via nconfig
_makenconfig=

### Tweak kernel options prior to a build via menuconfig
_makemenuconfig=

### Tweak kernel options prior to a build via xconfig
_makexconfig=

### Tweak kernel options prior to a build via gconfig
_makegconfig=

### Setting GCC Flags for CONFIG_MHASWELL
_gcc_haswell=y

### Set performance governor as default
_per_gov=y

# NUMA is optimized for multi-socket motherboards.
# A single multi-core CPU actually runs slower with NUMA enabled.
# See, https://bugs.archlinux.org/task/31187
_NUMAdisable=y

# Compile ONLY probed modules
# As of mainline 2.6.32, running with this option will only build the modules
# that you currently have probed in your system VASTLY reducing the number of
# modules built and the build time to do it.
#
# WARNING - ALL modules must be probed BEFORE you begin making the pkg!
#
# To keep track of which modules are needed for your specific system/hardware,
# give module_db script a try: https://aur.archlinux.org/packages/modprobed-db
# This PKGBUILD will call it directly to probe all the modules you have logged!
#
# More at this wiki page ---> https://wiki.archlinux.org/index.php/Modprobed_db
_localmodcfg=

# Use the current kernel's .config file
# Enabling this option will use the .config of the RUNNING kernel rather than
# the ARCH defaults. Useful when the package gets updated and you already went
# through the trouble of customizing your config options.  NOT recommended when
# a new kernel is released, but again, convenient for package bumps.
_use_current=

### Disable Deadline I/O scheduler
_deadline_disable=y

### Disable CFQ I/O scheduler
_CFQ_disable=y

### Disable Kyber I/O scheduler
_kyber_disable=y

### Enable MQ scheduling 
_mq_enable=y

### Running with a 1000 HZ tick rate 
_1k_HZ_ticks=y

### Do not edit below this line unless you know what you're doing

pkgbase=linux-hup-bfq-mq-git
# pkgname=('linux-hup-bfq-mq-git-kernel' 'linux-hup-bfq-mq-git-headers' 'linux-hup-bfq-mq-git-docs')
_srcname=linux-stable
pkgver=4.16.6.0.g22bc2b8a6aa4
pkgrel=1
arch=('x86_64')
url="https://github.com/Algodev-github/bfq-mq/"
license=('GPL2')
options=('!strip')
makedepends=('kmod' 'inetutils' 'bc' 'libelf' 'git')
_bfq_sq_mq_ver='20180412'
_bfq_sq_mq_patch="4.16-bfq-sq-mq-git-${_bfq_sq_mq_ver}.patch"
#_lucjanpath="https://raw.githubusercontent.com/sirlucjan/kernel-patches/master/4.16"
_lucjanpath="https://gitlab.com/sirlucjan/kernel-patches/raw/master/4.16"
_gcc_name="kernel_gcc_patch"
_gcc_rel='20180310'
_gcc_path="https://github.com/graysky2/kernel_gcc_patch/archive"
_gcc_patch="enable_additional_cpu_optimizations_for_gcc_v4.9+_kernel_v4.13+.patch"

source=('git://git.kernel.org/pub/scm/linux/kernel/git/stable/linux-stable.git#branch=linux-4.16.y'
        "${_lucjanpath}/${_bfq_sq_mq_patch}"
        "${_lucjanpath}/0100-Check-presence-on-tree-of-every-entity-after-every-a.patch"
        "${_lucjanpath}/bfq-pf/0900-block-bfq-lower-bound-the-estimated-peak-rate-to-1.patch"
        "${_lucjanpath}/blk-pf/0910-bfq-blk-fixes-from-pfkernel.patch"
        "${_gcc_name}-${_gcc_rel}.tar.gz::${_gcc_path}/${_gcc_rel}.tar.gz"
         # the main kernel config files
        'config'
         # pacman hook for depmod
        '60-linux.hook'
         # pacman hook for initramfs regeneration
        '90-linux.hook'
         # pacman hook for remove initramfs
        '99-linux.hook'
         # standard config files for mkinitcpio ramdisk
        'linux.preset'
        '0001-add-sysctl-to-disallow-unprivileged-CLONE_NEWUSER-by.patch'
        '0002-drm-i915-edp-Only-use-the-alternate-fixed-mode-if-it.patch'        
        ### UKSM ###
        "https://raw.githubusercontent.com/dolohow/uksm/master/uksm-4.16.patch"
	### PDS ###
        "https://bitbucket.org/alfredchen/linux-gc/downloads/v4.16_pds098n.patch"

	)
        
_kernelname=${pkgbase#linux}
: ${_kernelname:=-linux-hup-bfq-mq-git}
        
pkgver() {
    cd "${_srcname}"
        
        git describe --long --tags | sed 's/^v//;s/-/./g'
} 

prepare() {
    cd "${_srcname}"
    
    ### Disable USER_NS for non-root users by default
        msg "Disable USER_NS for non-root users by default"
        patch -Np1 -i ../0001-add-sysctl-to-disallow-unprivileged-CLONE_NEWUSER-by.patch
    
    ### Fix https://bugs.archlinux.org/task/56711
        msg "Fix #56711"
        patch -Np1 -i ../0002-drm-i915-edp-Only-use-the-alternate-fixed-mode-if-it.patch
    
    ### Add block-bfq fixes
        msg "Add block-bfq fixes"
        patch -Np1 -i ../0900-block-bfq-lower-bound-the-estimated-peak-rate-to-1.patch
        patch -Np1 -i ../0910-bfq-blk-fixes-from-pfkernel.patch    
    ### Patch source with BFQ-SQ-MQ
        _ver="$(cat Makefile | grep -m1 -e SUBLEVEL | grep -o "[[:digit:]]*")"
        msg "Fix naming schema in BFQ-SQ-MQ patch"
        sed -i -e "s|SUBLEVEL = 0|SUBLEVEL = ${_ver}|g" \
            -i -e "s|EXTRAVERSION = -bfq|EXTRAVERSION =|g" \
            -i -e "s|EXTRAVERSION =-mq|EXTRAVERSION =|g" ../${_bfq_sq_mq_patch}
        msg "Patching source with BFQ-SQ-MQ patches"
        patch -Np1 -i ../${_bfq_sq_mq_patch}
        
    ### Patches related to BUG_ON(entity->tree && entity->tree != &st->active) in __bfq_requeue_entity();
        if [ -n "$_use_tentative_patches" ]; then
        msg "Apply tentative patches"
        for p in "${srcdir}"/0100*.patch*; do 
        msg " $p"
        patch -Np1 -i "$p"; done
        fi
        
    ### Patch source to enable more gcc CPU optimizatons via the make nconfig
        msg "Patching source with gcc patch to enable more cpus types"
	patch -Np1 -i ../${_gcc_name}-${_gcc_rel}/${_gcc_patch}

    ### UKSM ###
	patch -Np1 -i "${srcdir}/uksm-4.16.patch"
    ### PSD ###
	patch -Np1 -i "${srcdir}/v4.16_pds098n.patch"       


    ### Clean tree and copy ARCH config over
	msg "Running make mrproper to clean source tree"
	make mrproper
	
	cat ../config - >.config <<END
CONFIG_LOCALVERSION="${_kernelname}"
CONFIG_LOCALVERSION_AUTO=n
END
        
    ### Optionally use running kernel's config
	# code originally by nous; http://aur.archlinux.org/packages.php?ID=40191
	if [ -n "$_use_current" ]; then
		if [[ -s /proc/config.gz ]]; then
			msg "Extracting config from /proc/config.gz..."
			# modprobe configs
			zcat /proc/config.gz > ./.config
		else
			warning "You kernel was not compiled with IKCONFIG_PROC!"
			warning "You cannot read the current config!"
			warning "Aborting!"
			exit
		fi
	fi    
        
    ### Set GCC flags
        if [ -n "$_gcc_haswell" ]; then
                msg "Setting GCC flags for Intel Haswell CPU..."
                sed -i -e s'/^CONFIG_GENERIC_CPU=y/# CONFIG_GENERIC_CPU is not set/' \
                    -i -e s'/^# CONFIG_MHASWELL is not set/CONFIG_MHASWELL=y/' ./.config
        fi  
        
    ### Set performance governor
        if [ -n "$_per_gov" ]; then
                msg "Setting performance governor.."
                sed -i -e s'/^CONFIG_CPU_FREQ_DEFAULT_GOV_SCHEDUTIL=y/# CONFIG_CPU_FREQ_DEFAULT_GOV_SCHEDUTIL is not set/' \
                    -i -e s'/^# CONFIG_CPU_FREQ_DEFAULT_GOV_PERFORMANCE is not set/CONFIG_CPU_FREQ_DEFAULT_GOV_PERFORMANCE=y/' ./.config
                msg "Disabling uneeded governors..."
                sed -i -e s'/^CONFIG_CPU_FREQ_GOV_ONDEMAND=m/# CONFIG_CPU_FREQ_GOV_ONDEMAND is not set/' \
                    -i -e s'/^CONFIG_CPU_FREQ_GOV_CONSERVATIVE=m/# CONFIG_CPU_FREQ_GOV_CONSERVATIVE is not set/' \
                    -i -e s'/^CONFIG_CPU_FREQ_GOV_USERSPACE=m/# CONFIG_CPU_FREQ_GOV_USERSPACE is not set/' \
                    -i -e s'/^CONFIG_CPU_FREQ_GOV_SCHEDUTIL=y/# CONFIG_CPU_FREQ_GOV_SCHEDUTIL is not set/' ./.config
        fi 
        
    ### Disable Deadline I/O scheduler
	if [ -n "$_deadline_disable" ]; then
		msg "Disabling Deadline I/O scheduler..."
		sed -i -e s'/^CONFIG_IOSCHED_DEADLINE=y/# CONFIG_IOSCHED_DEADLINE is not set/' \
		    -i -e s'/^CONFIG_MQ_IOSCHED_DEADLINE=y/# CONFIG_MQ_IOSCHED_DEADLINE is not set/' ./.config
	fi
	
    ### Disable CFQ I/O scheduler
	if [ -n "$_CFQ_disable" ]; then
		msg "Disabling CFQ I/O scheduler..."
		sed -i -e s'/^CONFIG_IOSCHED_CFQ=y/# CONFIG_IOSCHED_CFQ is not set/' \
		    -i -e s'/^CONFIG_CFQ_GROUP_IOSCHED=y/# CONFIG_CFQ_GROUP_IOSCHED is not set/' ./.config	
	fi
	
    ### Disable Kyber I/O scheduler
	if [ -n "$_kyber_disable" ]; then
		msg "Disabling Kyber I/O scheduler..."
		sed -i -e s'/^CONFIG_MQ_IOSCHED_KYBER=y/# CONFIG_MQ_IOSCHED_KYBER is not set/' ./.config
	fi
	
    ### Enable MQ scheduling
	if [ -n "$_mq_enable" ]; then
		msg "Enable MQ scheduling..."
		sed -i -e s'/^# CONFIG_SCSI_MQ_DEFAULT is not set/CONFIG_SCSI_MQ_DEFAULT=y/' \
		    -i -e s'/^# CONFIG_DM_MQ_DEFAULT is not set/CONFIG_DM_MQ_DEFAULT=y/' ./.config
        fi
        
    ### Optionally set tickrate to 1000 
	if [ -n "$_1k_HZ_ticks" ]; then
		msg "Setting tick rate to 1k..."
		sed -i -e 's/^CONFIG_HZ_300=y/# CONFIG_HZ_300 is not set/' \
                    -i -e 's/^# CONFIG_HZ_1000 is not set/CONFIG_HZ_1000=y/' \
                    -i -e 's/^CONFIG_HZ=300/CONFIG_HZ=1000/' ./.config
	fi    

	### Optionally disable NUMA for 64-bit kernels only
        # (x86 kernels do not support NUMA)
        if [ -n "$_NUMAdisable" ]; then
            msg "Disabling NUMA from kernel config..."
            sed -i -e 's/CONFIG_NUMA=y/# CONFIG_NUMA is not set/' \
                -i -e '/CONFIG_AMD_NUMA=y/d' \
                -i -e '/CONFIG_X86_64_ACPI_NUMA=y/d' \
                -i -e '/CONFIG_NODES_SPAN_OTHER_NODES=y/d' \
                -i -e '/# CONFIG_NUMA_EMU is not set/d' \
                -i -e '/CONFIG_NODES_SHIFT=6/d' \
                -i -e '/CONFIG_NEED_MULTIPLE_NODES=y/d' \
                -i -e '/# CONFIG_MOVABLE_NODE is not set/d' \
                -i -e '/CONFIG_USE_PERCPU_NUMA_NODE_ID=y/d' \
                -i -e '/CONFIG_ACPI_NUMA=y/d' ./.config
        fi

	### Set extraversion to pkgrel and empty localversion
        sed -e "/^EXTRAVERSION =/s/=.*/= -${pkgrel}/" \
            -e "/^EXTRAVERSION =/aLOCALVERSION =" \
            -i Makefile

	### Don't run depmod on 'make install'. We'll do this ourselves in packaging
	sed -i '2iexit 0' scripts/depmod.sh

	### Get kernel version
	msg "Running make prepare for you to enable patched options of your choosing"
	make prepare

	### Optionally load needed modules for the make localmodconfig
	# See https://aur.archlinux.org/packages/modprobed-db
		if [ -n "$_localmodcfg" ]; then
		msg "If you have modprobe-db installed, running it in recall mode now"
		if [ -e /usr/bin/modprobed-db ]; then
			[[ -x /usr/bin/sudo ]] || {
                        echo "Cannot call modprobe with sudo. Install sudo and configure it to work with this user."
                        exit 1; }
			sudo /usr/bin/modprobed-db recall
		fi
		msg "Running Steven Rostedt's make localmodconfig now"
		make localmodconfig
	fi

	### Running make nconfig
	
	[[ -z "$_makenconfig" ]] ||  make nconfig
	
	### Running make menuconfig
	
	[[ -z "$_makemenuconfig" ]] || make menuconfig
	
	### Running make xconfig
	
	[[ -z "$_makexconfig" ]] || make xconfig
	
	### Running make gconfig
	
	[[ -z "$_makegconfig" ]] || make gconfig
	
	### Rewrite configuration
	yes "" | make config >/dev/null

	### Save configuration for later reuse
	cat .config > "${startdir}/config.last"
}

build() {
  cd ${_srcname}

  make bzImage modules
}

_package-kernel() {
    pkgdesc='Linux Kernel and modules with the  BFQ-SQ-MQ scheduler. Fourth Gen Intel Core i3/i5/i7 optimized.'
    depends=('coreutils' 'linux-firmware' 'mkinitcpio>=0.7')
    optdepends=('crda: to set the correct wireless channels of your country' 'modprobed-db: Keeps track of EVERY kernel module that has ever been probed - useful for those of us who make localmodconfig')
    groups=('linux-hup-bfq-mq-git')
    backup=("etc/mkinitcpio.d/${pkgbase}.preset")
    install=linux.install

    cd ${_srcname}

    # get kernel version
    _kernver="$(make kernelrelease)"
    _basekernel=${_kernver%%-*}
    _basekernel=${_basekernel%.*}

    mkdir -p "${pkgdir}"/{boot,usr/lib/modules}
    make INSTALL_MOD_PATH="${pkgdir}/usr" modules_install
    cp arch/x86/boot/bzImage "${pkgdir}/boot/vmlinuz-${pkgbase}"

    # make room for external modules
    local _extramodules="extramodules-${_basekernel}${_kernelname}"
    ln -s "../${_extramodules}" "${pkgdir}/usr/lib/modules/${_kernver}/extramodules"

    # add real version for building modules and running depmod from hook
    echo "${_kernver}" |
        install -Dm644 /dev/stdin "${pkgdir}/usr/lib/modules/${_extramodules}/version"

    # remove build and source links
    rm "${pkgdir}"/usr/lib/modules/${_kernver}/{source,build}

    # now we call depmod...
    depmod -b "${pkgdir}/usr" -F System.map "${_kernver}"

    # add vmlinux
    install -Dt "${pkgdir}/usr/lib/modules/${_kernver}/build" -m644 vmlinux

    # sed expression for following substitutions
    local _subst="
        s|%PKGBASE%|${pkgbase}|g
        s|%KERNVER%|${_kernver}|g
        s|%EXTRAMODULES%|${_extramodules}|g
    "

    # hack to allow specifying an initially nonexisting install file
    sed "${_subst}" "${startdir}/${install}" > "${startdir}/${install}.pkg"
    true && install=${install}.pkg

    # install mkinitcpio preset file
    sed "${_subst}" ../linux.preset |
        install -Dm644 /dev/stdin "${pkgdir}/etc/mkinitcpio.d/${pkgbase}.preset"

    # install pacman hooks
    sed "${_subst}" ../60-linux.hook |
        install -Dm644 /dev/stdin "${pkgdir}/usr/share/libalpm/hooks/60-${pkgbase}.hook"
    sed "${_subst}" ../90-linux.hook |
        install -Dm644 /dev/stdin "${pkgdir}/usr/share/libalpm/hooks/90-${pkgbase}.hook"
    sed "${_subst}" ../99-linux.hook |
        install -Dm644 /dev/stdin "${pkgdir}/usr/share/libalpm/hooks/99-${pkgbase}.hook"    
}

_package-headers() {
    pkgdesc='Header files and scripts to build modules for linux-hup-bfq-mq-git. Fourth Gen Intel Core i3/i5/i7 optimized.'
    depends=('linux-hup-bfq-mq-git-kernel')
    groups=('linux-hup-bfq-mq-git')

    cd ${_srcname}
    local _builddir="${pkgdir}/usr/lib/modules/${_kernver}/build"

    install -Dt "${_builddir}" -m644 Makefile .config Module.symvers
    install -Dt "${_builddir}/kernel" -m644 kernel/Makefile

    mkdir "${_builddir}/.tmp_versions"

    cp -t "${_builddir}" -a include scripts

    install -Dt "${_builddir}/arch/x86" -m644 arch/x86/Makefile
    install -Dt "${_builddir}/arch/x86/kernel" -m644 arch/x86/kernel/asm-offsets.s

    cp -t "${_builddir}/arch/x86" -a arch/x86/include

    install -Dt "${_builddir}/drivers/md" -m644 drivers/md/*.h
    install -Dt "${_builddir}/net/mac80211" -m644 net/mac80211/*.h

    # http://bugs.archlinux.org/task/13146
    install -Dt "${_builddir}/drivers/media/i2c" -m644 drivers/media/i2c/msp3400-driver.h

    # http://bugs.archlinux.org/task/20402
    install -Dt "${_builddir}/drivers/media/usb/dvb-usb" -m644 drivers/media/usb/dvb-usb/*.h
    install -Dt "${_builddir}/drivers/media/dvb-frontends" -m644 drivers/media/dvb-frontends/*.h
    install -Dt "${_builddir}/drivers/media/tuners" -m644 drivers/media/tuners/*.h

    # add xfs and shmem for aufs building
    mkdir -p "${_builddir}"/{fs/xfs,mm}

    # copy in Kconfig files
    find . -name Kconfig\* -exec install -Dm644 {} "${_builddir}/{}" \;

    # add objtool for external module building and enabled VALIDATION_STACK option
    install -Dt "${_builddir}/tools/objtool" tools/objtool/objtool

    # remove unneeded architectures
    local _arch
    for _arch in "${_builddir}"/arch/*/; do
        [[ ${_arch} == */x86/ ]] && continue
        rm -r "${_arch}"
    done

    # remove files already in linux-hup-bfq-mq-git-docs package
    rm -r "${_builddir}/Documentation"
    
    # remove now broken symlinks
    find -L "${_builddir}" -type l -printf 'Removing %P\n' -delete

    # Fix permissions
    chmod -R u=rwX,go=rX "${_builddir}"

    # strip scripts directory
    local _binary _strip
    while read -rd '' _binary; do
        case "$(file -bi "${_binary}")" in
        *application/x-sharedlib*)  _strip="${STRIP_SHARED}"   ;; # Libraries (.so)
        *application/x-archive*)    _strip="${STRIP_STATIC}"   ;; # Libraries (.a)
        *application/x-executable*) _strip="${STRIP_BINARIES}" ;; # Binaries
        *) continue ;;
        esac
        /usr/bin/strip ${_strip} "${_binary}"
    done < <(find "${_builddir}/scripts" -type f -perm -u+w -print0 2>/dev/null)
}

_package-docs() {
    pkgdesc='Kernel hackers manual - HTML documentation that comes with the linux-hup-bfq-mq-git kernel. Fourth Gen Intel Core i3/i5/i7 optimized.'
    depends=('linux-hup-bfq-mq-git-kernel')
    groups=('linux-hup-bfq-mq-git')
  
    cd ${_srcname}
    local _builddir="${pkgdir}/usr/lib/modules/${_kernver}/build"

    mkdir -p "${_builddir}"
    cp -t "${_builddir}" -a Documentation

    # Fix permissions
    chmod -R u=rwX,go=rX "${_builddir}"
}

pkgname=("${pkgbase}-kernel" "${pkgbase}-headers" "${pkgbase}-docs")
for _p in ${pkgname[@]}; do
  eval "package_${_p}() {
    $(declare -f "_package${_p#${pkgbase}}")
    _package${_p#${pkgbase}}
  }"
done

sha512sums=('SKIP'
            '5b860dd67312af2327b9c6abc62d08e285cb81b5e64207f879178dac673cfe67249321ab9bf61e07e6fb855b0d7388061e8bbd4cead0e078886b91beb38599c9'
            '0f96fa9ad784709973b32eea82075ceb3e9dc2482df6441a4607612806f069254e63508b1b562279622394e4a1fbebef1b87af8401c0b1210d5d0de9954245c8'
            '23cde8ab10fef1552f12869b6bbe11e3c072ea49155746804ccf1c3d550e5d2721cd142559bc2845f11c598435531788f1b876bfa8989bc749e6513bdd95331c'
            'babffa9b2486507293ce22923b0625957aea35ffef155342276f50c5fd28a6a49f9cc1b99878abf130794ec476bccc843cf8d5e7d2533a0c12fd6a1fbda83dea'
            '079e34ec7bf3ef36438c648116e24c51e00ea8608a1d8b5776164478522d6a96dcab5fe0431e8e9a6282c11a1edd177e1b68fc971a81717b297e199efc101963'
            'db4447ef564ee6bedf62efa20bef3724192d8c97c0ae7778a7dee8405c55afaf3fb2ac450ee6b6a84b06fa9b93c77e385f7b99c30c8f04f9f8567d434850e3fe'
            '7ad5be75ee422dda3b80edd2eb614d8a9181e2c8228cd68b3881e2fb95953bf2dea6cbe7900ce1013c9de89b2802574b7b24869fc5d7a95d3cc3112c4d27063a'
            '4a8b324aee4cccf3a512ad04ce1a272d14e5b05c8de90feb82075f55ea3845948d817e1b0c6f298f5816834ddd3e5ce0a0e2619866289f3c1ab8fd2f35f04f44'
            '6346b66f54652256571ef65da8e46db49a95ac5978ecd57a507c6b2a28aee70bb3ff87045ac493f54257c9965da1046a28b72cb5abb0087204d257f14b91fd74'
            '2dc6b0ba8f7dbf19d2446c5c5f1823587de89f4e28e9595937dd51a87755099656f2acec50e3e2546ea633ad1bfd1c722e0c2b91eef1d609103d8abdc0a7cbaf'
            '52006c0b490a3eddbaa67b4afa12cff1e9bb14a1a3619f41b59568af9440b40498eab04c2fc28cbe36824b909df6e5f1197d57a187cc960c3d070f1c1688beeb'
            '08a9a9546eddd032ff0235441d70e475cc525f8e60931aeb076fa087af3fcecde4d8f70fe5d1fb0612ec641206aa50ab8319dfce635d51a1af96bce5b60205d5'
            'ab962157c2c20faf0bd164a82c2e28ce35fd581d640409ae6419dc2a64c82b7f49e7215149de0bc028dd3d517434160d68127b05137f6b6611641cc871f6c76e'
            'e4a8879e81280da66647023760f270e25bc3633ccc399f84c4a8a61b677b67f6274273d1f9c0dfaef99d442c9b9323d5c729788e966089ef4221803f0a5970e2')
