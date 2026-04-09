Arch Linux install
==================


1.부팅준비
---------

certutil -hashfile [filename] sha256

iso 를 풀어서 안에있는 파일을 USB에 쓴다. LABEL 도 일치시킨다.

UEFI모드로 부팅됐는지 확인

    ls /sys/firmware/efi/efivars  

인터넷 연결체크

    ping archlinux.org -c 3

WIFI로 인터넷 연결
 
    ip link
      rfkill
      iwctl
      device list
      station wlan0 scan
      station wlan0 get-networks
      station wlan0 connect [SSID]
      exit
    
    ping archlinux.org -c 3
    timedatectl status
    timedatectl list-timezones | grep Seoul
    timedatectl set-timezones Asia/Seoul
    timedatectl set-ntp true

    fdisk -l
      /dev/sda? 
      /dev/nvme0n1p?

    fdisk /dev/sda
      n   ; new partition
      5   ; partition number
      +4G ; size
      19  ; linux swap
      p   ; list partition
      w   ; write & end

    mkswap /dev/sda5
    mkfs.ext4 /dev/sda6
    lsblk -f /dev/sda
    mount /dev/sda6 /mnt
    mount --mkdir /dev/sda1 /mnt/efi
    mount -l |grep /dev/sda
    swapon /dev/sda5
    swapon -s


빠른 미러서버 확보
    reflector --save /etc/pacman.d/mirrorlist --protocol https --country KR,JP --sort rate --latest 10 --number 5

    cat /etc/pacman.d/mirrorlist

필수 패키지 설치(커널 등)

    pacstrap -K /mnt base linux linux-firmware vim nano man-db man-pages texinfo

인터넷 연결에 필요한 요소들 추출

    cp /etc/systemd/network/* /mnt/etc/systemd/network

 부팅시 아치리눅스 파티션 자동마운트 처리

    genfstab -U /mnt /mnt/etc/fstab

    arch-chroot /mnt
    ln -sf /usr/share/zoneinfo/Asia/Seoul /etc/localtime
    hwclock --systohc

지역설정 & 언어설정

    nano /etc/locale.gen
        ko_KR.UTF-8 을 찾아서 주석해제후 저장
    locale-gen
    nano /etc/locale.conf
        LANG=ko_KR.UTF-8

인터넷 끊김 대비
    pacman -S dhcpcd iwd

    nano /etc/hostname
        firebat

    nano /etc/hosts
        127.0.0.1   localhost
        ::1         localhost

        127.0.1.1   firebat

root 계정 비밀번호 어렵게 설정
GRUB 설치
    pacman -S intel-ucode
    pacman -S grub efibootmgr os-prober
    nano /usr/bin/grub-mkconfig
        GRUB_DISABLE_OS_PROBER="true" -> false
    grub-install --target=x86_64-efi --efi-directory=/efi --bootloader-id=firebat1st
    grub-mkconfig -o /boot/grub/grug.cfg
    LC_ALL=C grub-mkconfig -o /boot/grub/grub.cfg

chroot 종료
    exit
    umount -R /mnt
    poweroff
        


2.실기세팅
---------

    lang=en_US.UTF-8
    ping -c google.com
    systemctl start iwd.service
    ip link
        iwctl
        device list
        station wlan0 scan 
        ...

    systemctl start systemd-networkd.service
    systemctl start systemd-resolved.service

    passwd --status --all
    pacman -S sudo
    EDITOR=nano visudo
    %wheel ALL=(ALL:ALL) ALL 찾아서 코멘트 해제
    useradd gomwing -m -U -s /bin/bash -c 'gomwing,,,'
    chmod g+rx /home/gomwing
    usermod gomwing -aG wheel
    passwd gomwing
    pacman -S noto-fonts-cjk
    pacman -S fontconfig
    fc-cache -f -v
    echo -e "\nEDITOR=nano">>/etc/environment
    echo -e "VISUAL=nano">>/etc/environment
    echo -e "\nauth optional pam_faildelay.so delay=4000000">>/etc/pam.d/system-login
    nano /etc/security/faillock.conf
        even_deny_root
    nano /etc/pacman.conf
        ParallelDownloads=5

2. GUI 설치(Wayland+GNOME)
--------------------------
    
    pacman -S gnome gnome-tweaks
    systemctl enable gdm.service
    systemctl stasrt gdm.service
...
    sudo passwd --lock root


네트워크

    sudo pacman -S networkmanager
    sudo systemctl enable NetworkManager.service
    sudo systemctl start  NetworkManager.service

방화벽

    sudo pacman -S ufw gufw
    sudo systemctl enable ufw.service
    sudo systemctl start  ufw.service
    gufw --> enable


블루투스

    sudo pacman -S bluez bluez-utils
    sudo lsmod | grep btusb
    sudo systemctl enable bluetooth.service
    sudo systemctl start  bluetooth.service

한글키보드

    sudo pacman -S ibus ibus-hangul libhangul
    sudo nano /etc/environment
        QT_IM_MODULE=ibus
        XMODEFIERS=@im=ibus
    sudo reboot

    설정 -> 키보드 -> 입력소스 ; 한국어 추가,영어 지우기

yay

    sudo pacman -S --needed base-devel git
    cd /usr/local/src
    sudo mkdir aur
    cd aur
    sudo git clone https://aur.archlinux.org/yay.git
    sudo chown -R gomwing:gomwing yay
    cd yay
    makepkg -si

package manager gui

    LC_ALL=C yay -S pamac-all
        GUI: pamac -> 환경설정 -> 서드파티 -> aur 지원 ; on, 업데이트 확인 : on
        GUI: pamac -> google chrome 검색후 설치
        TXT: yay -Scc ; 캐시 지우기

chrome

    korean web ime 검색해서 html 로 저장한다.
    nano /home/gomwing/.local/applications/menulibre-onlinekoreanime.desktop
        [Desktop Entry]
        Type=Application
        Name=OnlineKoreanIME
        keywords=hangul;hangeul;korean;ime;keyboard;
        icon=applications-other
        Exec=/usr/bin/google-chrome-stable "path/to/IME.html"
        hangul 로 검색해서 IME아이콘을 대시보드에 고정

기본 스토어 업데이트 비활성화

reflector 설치

    sudo pacman -S reflector
    sudo nano /etc/xdg/reflector/reflector.conf
        --country "South Korea"
        --latest 10
        --number 5
        --sort rate
    sudo systemctl enable reflector.service
    sudo systemctl start  reflector.service
    cat /etc/pacman.d/mirrorlist

단축키
    GUI: 설정 -> 키보드 -> 바로가기 사용자설정
        시스템 정보(gnome-system-monitor);gnome-system-monitor;shift+ctrl+esc
        탐색기(nautilus);nautilus;super+E
        chrome(google-chrome);google-chrome-stable;super+C
        terminal(gnome-terminal | kgx);kgx;super+T

ETC
    sudo pacman -S fuse2
    gsettings set org.gnome.SessionManager logout-prompt false
    SEARCH : tweaks ; 
        제목표시줄 단추 -> 최대화 : on
        날짜및 시각 ; 평일 : on (요일 나옴) ; 주번호 : on
    sudo pacman -S wireplumber
    systemctl --user --now enable wireplumber

윈도우를 기본 부팅으로

    sudo nano /etc/default/grub
        GRUB_DEFAULT= 값변경
    LC_ALL=C sudo grub-mkconfig -o /boot/grub/grub.cfg
        














