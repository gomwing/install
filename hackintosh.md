hackintosh 설치하기 
==================

USB port mapping
----------------


opencore simplify 설치
----------------------

    1. select hardware report

        E export hardware report

            25 macOS tahoe

                1. AppleALC

                1. AirportItlwm (with OCLP)

    6. export

        using OCLP yes

    select codec (green string)    


OCAT(opencore aux tool) 이용하여 설정변경
----------------------------------------

    check enableWriteUnprotector

    uncheck RebuildAppleMemoryMap


AMFI(apple mobile file integrity) 수정
--------------------------------------

    unckeck AMFIpass.kext

~~NVRAM boot-args 에 amfi=0x80 추가~~

    NVRAM boot-args 에 -amfipassbeta 추가

    잘안되면 ipc_control_port_options=0 추가

    csr-active-config 030a0000 -> 03080000



해킨설치
--------

Hackintool 설치 [https://github.com/benbaker76/Hackintool/releases]

OCAT 설치

OCLP old[https://nightly.link/lzhoang2801/OpenCore-Legacy-Patcher/workflows/build-app-wxpython/tahoe-patchset?preview]

OCLP beta new[https://github.com/kgp-macPro/OCLP-lzhoang2801-amfipassbeta]



