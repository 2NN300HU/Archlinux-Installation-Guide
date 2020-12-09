# ARCH LINUX INSTALLATION GUIDE - I3WM - DUAL OS (WINDOWS)
* UEFI 시스템 기준입니다
* I3WM의 환경의 arch linux 를 설치하는 가이드 입니다
* Windows, arch 두 OS 가 공존하는 듀얼 OS 환경을 구축합니다
* 이 글은 아치 리눅스를 설치하는 한가지 방법을 보여주는 것입니다
* 이하는 편의를 위해 반말을 사용하겠습니다

# 검증된 환경 목록
* 아래에 없는 다른 환경에서 검증을 하셨다면 이슈 날려주시면 정말 감사드리겠습니다
* CPU : Intel, GPU : NVIDIA, Internet : wifi

# 설치 준비
## Windows
### Windows 설치
* 윈도우를 아치보다 먼저 설치하는것을 권장
### 윈도우를 UTC 를 사용하게  설정
1. regedit 을 열어 다음 경로로 이동한다
    `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\TimeZoneInformation\`
2. 새로만들기를 통해 32-bit 환경에서는 DWORD, 64-bit 환경에서는 QWORD 값을 만든다
3. 그 값의 이름을 RealTimeIsUniversal 으로 정해준다
4. 수정하여 16진수의 1로 값데이터를 지정

## Partition 축소
* 파티션을 축소하여 아치리눅스를 설치할 공간을 마련하여 준다

## archlinux-날짜-x86_64.iso 담은 usb 준비
1. https://www.archlinux.org/download/ 에서 iso 파일 다운로드 (BitTorrent Download 에서 Magnet link 사용)
2. rufus potable를 다운로드 하여준다
3. GPT 파티션 형식의 UEFI / FAT32 로 설치 usb 생성

## BIOS  
1. arch linux 설치 usb 를 꽂는다
2. BIOS 로 진입해준다
3. secure boot 를 disable 하여준다
4. 부팅순서를 조정하여 usb 로 부팅하도록 설정하여준다
5. Arch Linux install medium (x86_64, UEFI) 를 선택하여준다

# 설치
* 이제부턴 arch linux live 환경이다
* \# 이나 $ 가 붙어있다면 명령어의 시작이라는 뜻이다
* 만약 유저마다 다른 명령어인 경우 예시를 주고, 어떤 값을 자신에 맞게 바꾸어야하는지 적도록 하였다
##인터넷 연결
### Wifi
1. Rfkill 확인 `#rfkill list`
2. 만약 wifi 나 bluetooth 가 soft blocked: yes 되어있다면 3,4 를 진행한다
3. wifi 를 켠다 `#rfkill unblock wifi`
4. 만약 추후 블루투스도 사용하겠다면 `#rfkill unblock Bluetooth`
5. 다음의 명령을 통해 iwctl 을 시작한다
    * `#systemctl enable iwd.service`
    * `#systemctl start iwd.service`
    * `#systemctl start dhcpcd.service`
    * `#iwctl`
6. 다음의 명령을 통해 와이파이에 연결한다
    * `#device list`
      * 디바이스 목록이 나온다
    * `#station wlan0 scan`
      * wlan0 은 위에서 나온 device Name
    * `#station wlan0 get-networks`
      * 연결가능한 와이파이 목록이 나온다
    * `#station wlan0 connect SSID`
        * SSID 는 위에서 검색된 Network name
        * 비밀번호를 입력해야할 수도 있다
        * 탭키를 이용하여 자동완성도 가능
7. 연결되었다면 iwctl 에서 나와준다 `#exit`

8. 인터넷 연결 확인 `#ping -c 2 www.google.com`

### LAN

## Partition 분할
### UEFI 체크
* 다음의 명령어를 통해 UEFI 시스템인지 확인이 가능
* efivars 폴더가 보이면 UEFI 방식이니 계속 진행 `#ls /sys/firmware/efi`     

### 드라이브 및 파티션의 이름 확인 위한 명령어
* 이름이 기억 안나면 언제든지 사용하면 됨 `#lsblk`
* 용량을 이용하여 파티션을 구분하여야 함

### Partition 계획 예시

#### Linux extended boot 활용
* 윈도로 인해 100M가 된 esp 파티션의 용량을 넉넉하게 사용할수 있게 된다


* esp 파티션 / 윈도우의 그것을 사용 (따로 파티션, 포맷 작업하지 않음)
* boot 파티션 / 512 MB / Linux extended boot / FAT 32
* root 파티션 / 나머지 용량 / Linux FileSystem / ext 4

* 이렇게 할 경우 추후에 chroot 진입 후 다음의 명령어를 한번 사용해주어야함 (사용하여야 할 때 다시 언급)

`# bootctl --esp-path=/efi --boot-path=/boot install`


#### 일반적인 경우
* esp 파티션 / 윈도우의 그것을 사용
* root 파티션 / 나머지 용량 / Linux FileSystem / FAT 32


## Partition 작업
1. 다음의 명령어를 통해 파티션 편집을 시작한다 `#cfdisk /dev/sdb`
   * sdb 는 파티션 작업을 할 디스크의 이름으로, 위에서 확인한 값이다
2. 상단에서 Label: gpt 확인. 만약 처음에 물어볼 시 GPT 선택
* 모든 파티션은 Primary 로 진행 하여야 함
    * DELETE : 기존의 파티션 지우기
    * NEW : 새 파티션 만들기
        * 빈 파티션 선택 -> New -> 용량 지정 -> TYPE -> system type 선택
        * 초기값은 파티션 전체의 용량이므로, 나머지 용량을 전부 지정하려는 경우 그대로 진행
3. 전부 진행한 후 WRITE -> QUIT

## Format
* fat32 파일 시스템으로 포맷 `#mkfs.fat -F 32 /dev/sdb5`
  * sdb5 는 포맷할 디스크의 이름이다
* ext4 파일 시스템으로 포맷 `#mkfs.ext4 -j /dev/sdb6`
    * sdb6 은 포맷할 디스크의 이름이다
    
* 생성된 디스크의 이름을 확인하고 진행하여야함
* 용량만으로 파티션을 구별하고 파티션들의 이름이 비슷하니 헷갈리지 않도록 주의

## Disk mount

1. root 파티션 마운트
   * `#mount /dev/sdb6 /mnt`
    * sdb6은 root 파티션

2. boot 파티션 마운트
   1. `#mkdir /mnt/boot`
    2. `#mount /dev/sdb5 /mnt/boot`
        * sdb5는 boot 파티션 경로
        * 만약 따로 boot 파티션을 만들지 않았다면 여기에 windows 의 esp 파티션을 마운트 해주고 3번 항목은 무시
3. esp 파티션 마운트
    1. `#mkdir /mnt/efi`
    2. `#mount /dev/sdb1 /mnt/efi`
        * sdb1 은 esp 파티션
        * windows 의 기본 esp 파티션 크기는 100M 임


## 미러 서버 추가
1. 미러 서버 목록 파일을 열어준다 `#vim /etc/pacman.d/mirrorlist`
2. 아래의 두 줄 맨 위에 추가

`Server = http://ftp.lanet.kr/pub/archlinux/$repo/os/$arch`

`Server = https://ftp.lanet.kr/pub/archlinux/$repo/os/$arch`
* 기초적 vim 사용법
    * i를 누르면 편집모드가 실행되니 입력하면 됨
    * 다 입력한뒤 esc 를 누르고 wq를 누르면 편집 완료
* $repo 와 $arch 는 명령어의 시작이 아닙니다


## Pacstrap
* 다음의 명령어를 통해 기본적인 패키지들을 설치하여 준다 `#pacstrap /mnt base base-devel linux linux-firmware vim networkmanager`
* vim 대신 nano가 익숙하다면 대신 설치하여도 무방



##genfstab
* 마운트 정보 저장 `#genfstab -U /mnt >> /mnt/etc/fstab`


# 시스템 세팅
## arch linux 진입
* 명령어 입력 `#arch-chroot /mnt`

## 비밀번호 설정
* 명령어 입력 `#passwd`
    * 비밀번호를 두번 입력하면 설정 완료
    * 이 비밀번호는 root 비밀번호이기에 충분히 어렵게 설정하여야함
    * 비밀번호를 입력하여도 나오지 않으니 다 입력하고 엔터 입력

## locale 설정
1. locale 파일을 연다 `#vim /etc/locale.gen`
2. `en_US.UTF-8` 앞의 #을 지워 주석을 해제한후 저장
3. 명령어 입력 `#locale-gen`
   
## 언어 설정
* 명령어 입력 `#echo LANG=en_US.UTF-8 > /etc/locale.conf`

## pc 이름 설정
* 명령어 입력 `#echo dog > /etc/hostname`
    * dog 는 pc의 이름이니 자유롭게 정한다 

## 시간 설정
* 시간대를 서울로 맞춘다 `#ln -sf /usr/share/zoneinfo/Asia/Seoul /etc/localtime`
* 하드웨어 시간을 맞춘다 `#hwclock --systohc`
* 만약 이후에 시간이 이상하다면 윈도우에서 설정 - 날짜 및 시간 - 지금 동기화를 해본다
## 일반 사용자 추가
1. 사용자를 추가한다 `#useradd -m -g users -G wheel -s /bin/bash username`
    * username 는 원하는 유저명으로 설정한다
2. 비밀번호를 설정한다 `#passwd username`
   * username 는 아까 추가한 유저의 이름이다
3. 파일을 연다 `#EDITOR=vim visudo`
4. 아래쪽의 다음 항목을 #을 지워 주석처리된것을 풀어준다 `# %wheel ALL = (ALL) ALL`
    * 만약 `# %wheel ALL = (ALL) NOPASSWD: ALL` 을 대신 풀어주면 sudo 명령때마다 비밀번호를 묻지 않는다

## 부트로더 설치
* 이 가이드에서는 systemd-boot 를 사용하도록 하겠다
### systemd-boot 설정
#### Linux extended boot 를 사용하기로 한 경우
1. 부팅 경로를 설정하여 준다 `# bootctl --esp-path=/efi --boot-path=/boot install`
2. 부팅 설정 파일을 열어 준다 `# vim /efi/loader/loader.conf`
3. 다음의 내용으로 만들어 준다 (주석은 당연히 필요 없음)
```
default arch
timeout 3 #이곳의 숫자는 부팅 방식을 선택할 때까지 기다리는 시간이니 적당히 설정해준다
```

#### 일반적인 경우
1. 부팅 경로를 설정하여 준다 `# bootctl --path=/boot install`
2. 부팅 설정 파일을 열어 준다 `# vim /boot/loader/loader.conf`
3.  다음의 내용으로 만들어 준다 (주석은 당연히 필요 없음)
```
default arch
timeout 3 #이곳의 숫자는 부팅 방식을 선택할 때까지 기다리는 시간이니 적당히 설정해준다
```

#### 공통
* 부팅 설정 파일을 만들어준 이후이다
* 부트로더의 부팅 목록 파일에 arch 를 추가하여준다 `# vim /boot/loader/entries/arch.conf`
* 다음의 내용으로 만들어 준다
```
title Arch_Linux
linux /vmlinuz-linux
initrd /initramfs-linux.img
```

* 다음의 명령어를 입력하여 root 파티션을 알려준다
`#echo “options root=PARTUUID=$(blkid -s PARTUUID -o value /dev/sdb6) rw” >> /boot/loader/entries/arch.conf`
    * 이떄 sdb6은 root 파티션의 이름
* 내용이 잘 추가되었는지 확인하여도 좋다 `# vim /boot/loader/entries/arch.conf`


## 재부팅
1. 인터넷 기능을 켜준다. 이것을 켜두어야 재부팅 후에도 인터넷이 가능하다 `#systemctl enable NetworkManager.service`
2. chroot 에서 나온다 `#exit`
3. 마운트 했던 디스크들을 전부 언마운트 해준다 (unmount 가 아님) `#umount -R /mnt`
4. 재부팅 해준다 `#reboot`
5. 종료되면 usb 를 분리하여주어도 된다
* 이제 모든 설치가 완료 된 것이나 마찬가지 입니다
* 이제부턴 window manager 등 사용 환경을 구축할것입니다

## 부팅
* 윈도우와 아치 중 부팅할 운영체제를 선택할 수 있으며, 일정한 시간이 지나면 arch 로 자동 부팅 됩니다
* 다른 운영체제를 선택하고 d 를 누르면 기본 부팅 운영체제가 바뀌게 됩니다
* 로그인은 먼저 유저이름을 입력하고 비밀번호를 입력하면 됩니다

## 데스크탑 환경 설치
* 추천이라고 써진것은 필수는 아니지만 필자가 사용하는 것입니다
### 인터넷 확인
* 다음의 명령으로 인터넷이 잘 연결 되었는지 확인한다 `$ping -c 2 www.google.com`
* 만약 연결이 안되었을 경우 다음의 명령들을 사용해 wifi 에 연결할 수 있다
  * `$ nmcli device wifi list`
  * `$ nmcli device wifi connect SSID password thisispass`
    * SSID 는 wifi 의 이름, thisispass 는 비밀번호이다
    
### Pacman
* pacman 은 강력한 패키지 매니저입니다
* 다음의 명령어를 통해 서버와 동기화하고 모든 패키지를 업데이트 `$sudo pacman -Syu`
* 다음의 명령어를 통해 패키지를 설치 `$sudo pacman -S package`
* 다음의 명령어를 통해 패키지를 의존성 검사 하에 삭제 `$sudo pacman -S package`
    * 위의 package 는 패키지 명이며, package1 package2 와 같이 동시에 여러개를 설치
    
### yay (추천)
* arch 에는 AUR 이라는 또 다른 패키지 방식이 존재함
* 이 가이드에서는 AUR 패키지 매니저로 yay 를 사용할것
1. 우선 git 를 설치합니다 `$sudo pacman -S git`
2. 그 뒤 다음의 명령어들을 사용해준다
   1. `$ git clone https://aur.archlinux.org/yay.git`
   2. `$ cd yay`
   3. `$ makepkg -si`
4. 사용방법은 pacman 과 동일하다 ex)`yay -S package`
5. pacman  AUR 의 저장소를 동기화하고 모든 패키지를 업데이트 하는것을 `$yay` 라는 명령하나로 가능
    
### Reflector(추천)
* 자동으로 미러를 잡아주는 프로그램이다
* 설치 `sudo pacman -S reflector`
* 미러 설정을 자동화 하는 방법 
    * https://m.blog.naver.com/luoive/221727970858
    
### I3wm
#### 그래픽 드라이버 설치
* 다음을 참고 https://wiki.archlinux.org/index.php/xorg#Driver_installatio
* 그래픽 카드 확인 명령어 `lspci -v | grep -A1 -e VGA -e 3D`
* 그래픽 카드 드라이버 설치 `sudo pacman -S xf86-video-intel`
    * 자신의 그래픽 카드에 맞는 드라이버와 opengl 을 설치하면 충분할 것이다 proprietary 와 오픈소스중 하나를 고르면 된다
#### i3 설치
* 설치 `$pacman -S xorg i3-wm`
* 엔터 - 윈도키(메타키를 무엇으로 하냐에 달려있음) - 엔터를 하면 기본 설정 파일이 ~/.config/i3/config 에 생긴다
* 이제 콘솔창을 윈도우키+ 엔터로 열수 있다 
  `sudo vim ~/.config/i3/config` 로 수정 가능

### font (추천)
* 원하는 폰트를 설치
* 이하는 필자가 사용하는 폰트의 목록이다
    * ttf-nanum (AUR)
        * i3 설정파일에서 폰트 설정을 찾아 다음으로 바꿔주면 된다 `font pango:nanum 8`

    * ttf-font-awesome (디자인용 폰트)

### tian (추천)
* 개인적으로 제일 만족스러웠던 한글 입력기 이다
* 옵션 내에서 왼쪽 alt 키를 한영키로 만들어 줄수 있다
* 설치 방법은 사이트에 나와있다 https://www.nimfsoft.com/downloads/

###microcode (추천)
* 안정화 코드
#### intel
1. micro code 를 설치합니다 `$sudo pacman -S intel-ucode`
2. 설정을 변경합니다 `$sudo vim /boot/loader/entries/arch.conf`
3. linux ~~ 다음줄(세번째줄)에 다음 줄을 추가하고 저장한다`initrd /intel-ucode.img`
3. 다음 명령어를 입력한다 `$sudo bootctl update`
4. 재부팅한다 `$reboot`

#### amd
1. micro code 를 설치합니다 `$sudo pacman -S amd-ucode`
2. 설정을 변경합니다 `$sudo vim /boot/loader/entries/arch.conf`
3. linux ~~ 다음줄(세번째줄)에 다음 줄을 추가하고 저장한다`initrd /amd-ucode.img`
3. 다음 명령어를 입력한다 `$sudo bootctl update`
4. 재부팅한다 `$reboot`

### ly(추천)
* ly 는 display manager 의 일종으로 부팅시 로그인 화면을 바꾸어 줌
  * AUR 로 설치 `$ yay -S ly`
  * 매 부팅마다 실행되게 한다 `$systemctl enable ly`
    
### Chrome(선택)
* google-chrome (AUR)
  
### 상단의 파란색 바 없애기(선택)
* 설정파일에 다음을 추가한다
```
default_border none
default_floating_border normal
```
* 여기까지만 해도 사용하는데에 문제는 없을 것이다
// 이하는 미완성
### urxvt
`$sudo pacman -S rxvt-unicode`

### rofi
`$sudo pacman -S rofi`

### polybar
`$yay -S polybar`



