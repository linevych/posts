Title: Виправляємо баг з драйвеом NVIDIA та розширенням монітора LG Flatron L1953S
Date: 2-02-2016
Slug: fixing-nvidia-resolution-problems
Category: Linux
Lang: uk
Image: /images/fuck-you-nvidia.jpg
Featured_image: /images/fuck-you-nvidia.jpg
Tags: linux, bugs, nvidia, debian, LG, hardware, drivers
Summary: Через терен до зірок, або як налаштувати монітор якщо твоя відеокарта надто стара.

## Проблематика

В моєму робочому ПК 2 монітори &mdash; LG Flatron L1953S та Samsung SyncMaster 710N, обидва з
розширенням 1280x1024 (майже музейні експонати :)). Після встановлення пропрієтарного драйвера
NVIDIA розширення у LG стає 640x480 і збільшити його ніяк не можна, тоді як Samsung працює як
годинник.  Жодні маніпуляції, зі зміною драйвера не домогли, трюки з xrandr теж не дають взагалі
ніякого результату.

## Розв’язок проблеми

Якщо  змінити місцями порти до яких підключені монітори то налаштування теж змінюються місцями
(на LG стає нормальне зображення, а на Samsung 640x480), але це лише до першого перезавантаження.

Будь-які маніпуляції з налаштуваннями розширення після зміни портів ведуть до скидання параметрів,
але якщо після зміни портів виконати `sudo nvidia-xconfig` то &mdash; о диво, в налаштуваннях
з’явиться можливість вибирати розширення для обох моніторів!

Для закріплення результату рекомендується виконати `sudo nvidia-settings` і сконфігрувавши все так
як потрібно, натиснути `Save to X Configuration File` &mdash; вуаля все працює так як слід.

## Отриманий xorg.conf
Про всяк випадок залишу тут також отриманий xorg.conf:

    Section "ServerLayout"
        Identifier     "Layout0"
        Screen      0  "Screen0" 0 0
        InputDevice    "Keyboard0" "CoreKeyboard"
        InputDevice    "Mouse0" "CorePointer"
        Option         "Xinerama" "0"
    EndSection
    
    Section "Files"
    EndSection
    
    Section "InputDevice"
    
        Identifier     "Mouse0"
        Driver         "mouse"
        Option         "Protocol" "auto"
        Option         "Device" "/dev/psaux"
        Option         "Emulate3Buttons" "no"
        Option         "ZAxisMapping" "4 5"
    EndSection
    
    Section "InputDevice"
    
        # generated from default
        Identifier     "Keyboard0"
        Driver         "kbd"
    EndSection
    
    Section "Monitor"
        Identifier     "Monitor0"
        VendorName     "Unknown"
        ModelName      "Samsung SyncMaster"
        HorizSync       30.0 - 81.0
        VertRefresh     56.0 - 75.0
        Option         "DPMS"
    EndSection
    
    Section "Device"
        Identifier     "Device0"
        Driver         "nvidia"
        VendorName     "NVIDIA Corporation"
        BoardName      "GeForce GT 440"
    EndSection
    
    Section "Screen"
        Identifier     "Screen0"
        Device         "Device0"
        Monitor        "Monitor0"
        DefaultDepth    24
        Option         "Stereo" "0"
        Option         "nvidiaXineramaInfoOrder" "CRT-0"
        Option         "metamodes" "DVI-I-0: nvidia-auto-select +0+0, VGA-0: 1280x1024 +1280+0"
        Option         "SLI" "Off"
        Option         "MultiGPU" "Off"
        Option         "BaseMosaic" "off"
        SubSection     "Display"
            Depth       24
        EndSubSection
    EndSection


## P.S

Спеціально для тих, хто зайшов сюди лише, щоб побачити це відео: 
  
<div class="embed-responsive embed-responsive-16by9">
  <iframe class="embed-responsive-item" src="https://www.youtube.com/embed/_36yNWw_07g"></iframe>
</div>
