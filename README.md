# wrlinux_ros2_galactic

빌드 환경 : intel 64, Ubuntu 20.04
타겟 보드 : NXP-S32G
타겟 리눅스 : WRLinux-LTS21

1. Wind River Linux setup tool 다운:

        git clone -b WRLINUX_10_21_BASE https://github.com/WindRiver-Labs/wrlinux-x.git


2. WorkSpace 생성

        mkdir wrlinux-ros2-2021
        cd wrlinux-ros2-2021

3. NXP S32G 보드 빌드 디렉토리 생성

        ./wrlinux-x/setup.sh --machines=nxp-s32g --distros=wrlinux --dl-layers
   
4. meta-ros layer 추가

        cd layers
        git clone -b hardknott https://github.com/ros/meta-ros.git
        cd meta-ros
        git checkout 'hardknott@{2021-09-01 12:00:00}'
        cd ..

5. wr-ros layer 추가

        git clone -b WRLINUX_10_21_BASE https://github.com/Wind-River/wr-ros.git
        cd wr-ros
        rm -rf recipes-python/pyqt5 recipes-bbappends/qt-gui-core recipes-imported-bistro/openblas
        curl -O https://github.com/ros/meta-ros/commit/4e7ce48fc62329b36f9a56b5713bd46eb8256b2a.patch
        git am -p2 4e7ce48fc62329b36f9a56b5713bd46eb8256b2a.patch
        curl -O https://github.com/ros/meta-ros/commit/7479cfe0279e7cdc094c02a3cc6f04e8b0a1fb38.patch
        git am -p2 7479cfe0279e7cdc094c02a3cc6f04e8b0a1fb38.patch
        cd ..

6. OpenBLAS 설정 변경

        # wr-ros/recipes-extended/suitesparse/suitesparse-cholmod_5.4.0.bb
        # wr-ros/recipes-extended/suitesparse/suitesparse-spqr_5.4.0.bb
        # 위 파일에서 '-lopenblas' 삭제
        
 
8. nxp-s32g2xx에서 source.codeaurora.org 참조 변경

        # nxp-s32g2xx/recipes-bsp/arm-trusted-firmware/atf-s32g_2.5.bb
        URL ?= "git://github.com/nxp-auto-linux/arm-trusted-firmware.git;protocol=http"
   
        # nxp-s32g2xx/recipes-bsp/u-boot/u-boot-s32_2020.04.bb
        SRC_URI:prepend = "git://github.com/nxp-auto-linux/u-boot.git;protocol=https;branch=release/bsp32.0-2020.04"

9. WorkSpace 디렉토리로 돌아와서 수행

        . ./environment-setup-x86_64-wrlinuxsdk-linux
        . ./oe-init-build-env wrlinux-lts21-ros2

10. bitbake layer 구성

        cd $BUILDDIR
        bitbake-layers add-layer $(readlink -f $BUILDDIR/../layers)/meta-ros/meta-ros-common
        bitbake-layers add-layer $(readlink -f $BUILDDIR/../layers)/meta-ros/meta-ros2
        bitbake-layers add-layer $(readlink -f $BUILDDIR/../layers)/meta-ros/meta-ros2-galactic
        bitbake-layers add-layer $(readlink -f $BUILDDIR/../layers)/wr-ros

11. cd $BUILDDIR/conf/local.conf에 추가

        BB_NO_NETWORK = '0'
        BB_NUMBER_THREAD = "1"
        PARALLEL_MAKE = "-j 1"
        PNWHITELIST_LAYERS = ""
        IMAGE_INSTALL_append += " packagegroup-ros-world"
        
        SYSROOT_DIRS_append += " ${nonarch_libdir} "
        FILES_${PN}-dbg += "${nonarch_libdir}/.debug"
        FILES_${PN}-dev += "${nonarch_libdir}/pkgconfig ${nonarch_libdir}/cmake"
        FILES_${PN} += "${nonarch_libdir}/lib*.so*"
        FILES_${PN} += "${nonarch_libdir}/${ROS_BPN}"
        FILES_${PN} += "${nonarch_libdir}/${PYTHON_DIR}"
        FILES_${PN}-dev += "${nonarch_libdir}/${PYTHON_DIR}/site-packages/*.la"
        FILES_${PN}-staticdev += "${nonarch_libdir}/lib*.a"
        FILES_${PN}-staticdev += "${nonarch_libdir}/${PYTHON_DIR}/site-packages/*.a"
        INSANE_SKIP_${PN}-dbg += "libdir"
        INSANE_SKIP_${PN} += "dev-so libdir"
        
        IMAGE_INSTALL_append += "\
        packagegroup-core-buildessential \
        git"
        
        PNBLACKLIST[rmf-fleet-adapter] ?= "Linker error"
        PNBLACKLIST[tracetools-trace] ?= "Linker error"
        LICENSE_FLAGS_WHITELIST = "commercial"
        RDEPENDS:${PN}:remove += "tango-icons-vendor"
        RDEPENDS:${PN}:remove += "rcss3d-agent"
        RDEPENDS:${PN}:remove += "nao-button-sim"
        RDEPENDS:${PN}:remove += "rmf-fleet-adapter"
        RDEPENDS:${PN}:remove += "rmf-fleet-adapter-python"
        RDEPENDS:${PN}:remove += "action-tutorials-py"
        RDEPENDS:${PN}:remove += "examples-rclpy-minimal-client"
        RDEPENDS:${PN}:remove += "demo-nodes-py"
        RDEPENDS:${PN}:remove += "grbl-ros"
        RDEPENDS:${PN}:remove += "key-teleop"
        RDEPENDS:${PN}:remove += "mouse-teleop"
        RDEPENDS:${PN}:remove += "rmf-demos-tasks"
        RDEPENDS:${PN}:remove += "quality-of-service-demo-py"
        RDEPENDS:${PN}:remove += "rmf-visualization-building-systems"
        RDEPENDS:${PN}:remove += "bno055"
        RDEPENDS:${PN}:remove += "joint-state-publisher"
        RDEPENDS:${PN}:remove += "examples-rclpy-minimal-service"
        RDEPENDS:${PN}:remove += "camera-calibration"
        RDEPENDS:${PN}:remove += "examples-rclpy-minimal-publisher"
        RDEPENDS:${PN}:remove += "simple-launch"
        RDEPENDS:${PN}:remove += "nav2-gazebo-spawner"
        RDEPENDS:${PN}:remove += "rc-reason-clients"
        RDEPENDS:${PN}:remove += "examples-rclpy-minimal-action-server"
        RDEPENDS:${PN}:remove += "examples-tf2-py"
        RDEPENDS:${PN}:remove += "tf2-tools"
        RDEPENDS:${PN}:remove += "tracetools-trace"
        RDEPENDS:${PN}:remove += "suitesparse-spqr"
        RDEPENDS:${PN}:remove += "examples-rclpy-pointcloud-publisher"
        RDEPENDS:${PN}:remove += "examples-rclpy-executors"
        RDEPENDS:${PN}:remove += "examples-rclpy-guard-conditions"
        RDEPENDS:${PN}:remove += "rmf-visualization-fleet-states"
        RDEPENDS:${PN}:remove += "examples-rclpy-minimal-action-client"
        RDEPENDS:${PN}:remove += "joy-teleop"
        RDEPENDS:${PN}:remove += "teleop-twist-keyboard"
        RDEPENDS:${PN}:remove += "examples-rclpy-minimal-subscriber"
        RDEPENDS:${PN}:remove += "nav2-simple-commander"
        RDEPENDS:${PN}:remove += "topic-monitor"
        RDEPENDS:${PN}:remove += "ros2trace"
        RDEPENDS:${PN}:remove += "smacc2"
        RDEPENDS:${PN}:remove += "tracetools-launch"
        
        ROS_WORLD_SKIP_GROUPS += " qt5 "
        # ROS_WORLD_SKIP_GROUPS_remove = "gazebo"
        ROS_WORLD_SKIP_GROUPS += " libomp "

12. Wind River Linux + ROS2 빌드

        bitbake wrlinux-image-std

13. SD카드에 이미지 write

        cd $BUILDDIR/tmp-glibc/deploy/images/nxp-s32g
        sudo dd if=<path/.wic> of=/dev/<sdcard_dev> bs=1M&&sync

