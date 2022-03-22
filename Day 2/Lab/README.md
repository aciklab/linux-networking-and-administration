## Alıştırma 5.1 - Anahtar tabanlı SSH kimlik doğrulamasını ayarlayın.

### Bir SSH anahtarıyla localhost makinenize bağlanın.

Not: Bu laboratuvar, root'un ssh aracılığıyla bir parola ile oturum açmasına izin verildiğini varsayar. Bu davranışı değiştirebilen bir konfigürasyon parametresi bulunmaktadır. Bu parametre dağıtımdan dağıtıma ve sürümden sürüme değişebilir. */etc/ssh/sshd_config* dosyasında *PermitRootLogin* parametresinin *yes* olarak ayarlandığını kontrol edin. Değilse, parametreyi ayarlayın ve sshd sunucusunu yeniden başlatın.

## Çözüm 5.1

### 1. Bir SSH anahtarı oluşturun 

```
$ ssh-keygen -t rsa -f $HOME/.ssh/id-rsa
```

### 2. SSH pubkey'i manuel olarak kopyalayın veya ssh-copy-id kullanın.

```
$ ssh-copy-id sysadmin@localhost
$ ssh sysadmin@localhost
$ id
$ exit
```

## Alıştırma 5.2 - OpenSSH istemci yapılandırma değişikliklerini yapın.

### Varsayılan kullanıcı adını değiştirin ve *$HOME/.ssh/config* dosyasını kullanarak host alias oluşturun.

## Çözüm 5.2

### 1. *$HOME/.ssh/config* dosyasını aşağıdaki içeriklerle düzenleyin.

```
host pardusserver
hostname localhost
user root
host *
ForwardX11 yes
```

### 2. Yeni host alias'a bağlanmak için OpenSSH istemcisini kullanın.

```
$ ssh pardusserver
$ hostname
$ id
$ exit
```

## Alıştırma 5.3 - OpenSSH daemon'ını güvenli hale getirin.

### Yalnızca anahtar tabanlı SSH kimlik doğrulamasını etkinleştirin.

## Çözüm 5.3

### 1. */etc/ssh/sshd_config* dosyasını düzenleyin ve şu satırın mevcut olduğundan emin olun. 

```
PermitRootLogin without-password
```

### sshd deamon'ı yeniden başlatın.

```
# systemctl restart sshd.service
```

Not: Ubuntu'da ssh hizmeti sshd değil ssh olarak adlandırılır.

### Kök olarak oturum açmayı deneyin. Başarısız olmalı.

```
# systemctl restart sshd.service
```

### */home/student/.ssh/authorized_keys* dosyasını */root/.ssh/* dizinine kopyalayın ve kök kullanıcıya ve gruba ait olduğundan emin olun.

```
# cat /home/student/.ssh/authorized_keys >> /root/.ssh/authorized_keys
# chown root.root /root/.ssh/authorized_keys
# chmod 640 /root/.ssh/authorized_keys
```

### İşe yaradığını kanıtlamak için ana bilgisayar garply'de tekrar oturum açın.

```
$ ssh pardusserver
$ hostname
$ id
$ exit
```

## Alıştırma 5.4 - Uzak bir X11 uygulamasını yerel olarak başlatın.

### Uzak bir sistemde xeyes programını başlatın ve yerel olarak görüntüleyin.

Not: xeyes programını kurmanız gerekebilir. 

## Çözüm 5.4

### 1. Uzak bir sunucuya bağlanın ve X11 tünelini açın. 

```
$ ssh -X student@server pinta
```

Not: IPv6 desteği olmadan OpenSUSE kullanıyorsanız, */etc/ssh/sshd_config dosyası:* dosyasında aşağıdaki satırın bulunması gerekir.

```
AddressFamily inet
```

## Alıştırma 5.4 - Uzak bir X11 uygulamasını yerel olarak başlatın.

### Uzak bir sistemde xeyes programını başlatın ve yerel olarak görüntüleyin.

Not: xeyes programını kurmanız gerekebilir. 

## Çözüm 5.4

### 1. Uzak bir sunucuya bağlanın ve X11 tünelini açın. 

```
$ ssh -X student@server xeyes
```

Not: IPv6 desteği olmadan OpenSUSE kullanıyorsanız, */etc/ssh/sshd_config dosyası:* dosyasında aşağıdaki satırın bulunması gerekir.

```
AddressFamily inet
```

## Alıştırma 5.5 - Uzak bir Masaüstüne RDP ile bağlanın.
### Ana bilgisayarınızdaki RDP ya da Remmina gibi uygulamaları kullanarak host üzerindeki XRDP'ye bağlanın
## Çözüm 5.5
### 1. xrdp ve gerekli paketleri indirin

```
sudo apt install xrdp tigervnc-standalone-server

```
### 2. xrdp grubuna ssl yerkisi verin

```
sudo usermod -a -G ssl-cert xrdp
```

### 3. /etc/xrdp/xrdp.ini dosyasona aşağıdaki metni koyun
```
[Globals]
; xrdp.ini file version number
ini_version=1

; fork a new process for each incoming connection
fork=true

; ports to listen on, number alone means listen on all interfaces
; 0.0.0.0 or :: if ipv6 is configured
; space between multiple occurrences
;
; Examples:
;   port=3389
;   port=unix://./tmp/xrdp.socket
;   port=tcp://.:3389                           127.0.0.1:3389
;   port=tcp://:3389                            *:3389
;   port=tcp://<any ipv4 format addr>:3389      192.168.1.1:3389
;   port=tcp6://.:3389                          ::1:3389
;   port=tcp6://:3389                           *:3389
;   port=tcp6://{<any ipv6 format addr>}:3389   {FC00:0:0:0:0:0:0:1}:3389
;   port=vsock://<cid>:<port>
port=3389

; 'port' above should be connected to with vsock instead of tcp
; use this only with number alone in port above
; prefer use vsock://<cid>:<port> above
use_vsock=false

; regulate if the listening socket use socket option tcp_nodelay
; no buffering will be performed in the TCP stack
tcp_nodelay=true

; regulate if the listening socket use socket option keepalive
; if the network connection disappear without close messages the connection will be closed
tcp_keepalive=true

; set tcp send/recv buffer (for experts)
#tcp_send_buffer_bytes=32768
#tcp_recv_buffer_bytes=32768

; security layer can be 'tls', 'rdp' or 'negotiate'
; for client compatible layer
security_layer=negotiate

; minimum security level allowed for client for classic RDP encryption
; use tls_ciphers to configure TLS encryption
; can be 'none', 'low', 'medium', 'high', 'fips'
crypt_level=high

; X.509 certificate and private key
; openssl req -x509 -newkey rsa:2048 -nodes -keyout key.pem -out cert.pem -days 365
; note this needs the user xrdp to be a member of the ssl-cert group, do with e.g.
;$ sudo adduser xrdp ssl-cert
certificate=
key_file=

; set SSL protocols
; can be comma separated list of 'SSLv3', 'TLSv1', 'TLSv1.1', 'TLSv1.2', 'TLSv1.3'
ssl_protocols=TLSv1.2, TLSv1.3
; set TLS cipher suites
#tls_ciphers=HIGH

; Section name to use for automatic login if the client sends username
; and password. If empty, the domain name sent by the client is used.
; If empty and no domain name is given, the first suitable section in
; this file will be used.
autorun=

allow_channels=true
allow_multimon=true
bitmap_cache=true
bitmap_compression=true
bulk_compression=true
#hidelogwindow=true
max_bpp=32
new_cursors=true
; fastpath - can be 'input', 'output', 'both', 'none'
use_fastpath=both
; when true, userid/password *must* be passed on cmd line
#require_credentials=true
; You can set the PAM error text in a gateway setup (MAX 256 chars)
#pamerrortxt=change your password according to policy at http://url

;
; colors used by windows in RGB format
;
blue=80001c
grey=d1d2d4
#black=000000
#dark_grey=808080
#blue=08246b
#dark_blue=08246b
#white=ffffff
#red=ff0000
#green=00ff00
#background=626c72

;
; configure login screen
;

; Login Screen Window Title
ls_title=Aciklab XRDP

; top level window background color in RGB format
ls_top_window_bg_color=113b6a

; width and height of login screen
ls_width=350
ls_height=430

; login screen background color in RGB format
ls_bg_color=d1d2d4

; optional background image filename (bmp format).
#ls_background_image=

; logo
; full path to bmp-file or file in shared folder
ls_logo_filename=
ls_logo_x_pos=135
ls_logo_y_pos=55

; for positioning labels such as username, password etc
ls_label_x_pos=30
ls_label_width=65

; for positioning text and combo boxes next to above labels
ls_input_x_pos=110
ls_input_width=210

; y pos for first label and combo box
ls_input_y_pos=220

; OK button
ls_btn_ok_x_pos=142
ls_btn_ok_y_pos=370
ls_btn_ok_width=85
ls_btn_ok_height=30

; Cancel button
ls_btn_cancel_x_pos=237
ls_btn_cancel_y_pos=370
ls_btn_cancel_width=85
ls_btn_cancel_height=30

[Logging]
LogFile=xrdp.log
LogLevel=DEBUG
EnableSyslog=true
SyslogLevel=DEBUG
; LogLevel and SysLogLevel could by any of: core, error, warning, info or debug

[Channels]
; Channel names not listed here will be blocked by XRDP.
; You can block any channel by setting its value to false.
; IMPORTANT! All channels are not supported in all use
; cases even if you set all values to true.
; You can override these settings on each session type
; These settings are only used if allow_channels=true
rdpdr=true
rdpsnd=true
drdynvc=true
cliprdr=true
rail=true
xrdpvr=true
tcutils=true

; for debugging xrdp, in section xrdp1, change port=-1 to this:
#port=/tmp/.xrdp/xrdp_display_10

; for debugging xrdp, add following line to section xrdp1
#chansrvport=/tmp/.xrdp/xrdp_chansrv_socket_7210


;
; Session types
;

; Some session types such as Xorg, X11rdp and Xvnc start a display server.
; Startup command-line parameters for the display server are configured
; in sesman.ini. See and configure also sesman.ini.
[Xorg]
name=Pardus XFCE
lib=libxup.so
username=ask
password=ask
ip=127.0.0.1
port=-1
code=20

; You can override the common channel settings for each session type
#channel.rdpdr=true
#channel.rdpsnd=true
#channel.drdynvc=true
#channel.cliprdr=true
#channel.rail=true
#channel.xrdpvr=true
```

### 4. Türkçe klavye desteği için /etc/xrdp/km-0000041f.ini dosyası oluşturulur ve içerisine aşağıdaki metin yazılır. 
```
[noshift]
Key8=65406:0
Key9=65307:27
Key10=49:49
Key11=50:50
Key12=51:51
Key13=52:52
Key14=53:53
Key15=54:54
Key16=55:55
Key17=56:56
Key18=57:57
Key19=48:48
Key20=42:42
Key21=45:45
Key22=65288:8
Key23=65289:9
Key24=113:113
Key25=119:119
Key26=101:101
Key27=114:114
Key28=116:116
Key29=121:121
Key30=117:117
Key31=697:305
Key32=111:111
Key33=112:112
Key34=699:287
Key35=252:252
Key36=65293:13
Key37=65507:0
Key38=97:97
Key39=115:115
Key40=100:100
Key41=102:102
Key42=103:103
Key43=104:104
Key44=106:106
Key45=107:107
Key46=108:108
Key47=442:351
Key48=105:105
Key49=34:34
Key50=65505:0
Key51=44:44
Key52=122:122
Key53=120:120
Key54=99:99
Key55=118:118
Key56=98:98
Key57=110:110
Key58=109:109
Key59=246:246
Key60=231:231
Key61=46:46
Key62=65506:0
Key63=65450:42
Key64=65513:0
Key65=32:32
Key66=65509:0
Key67=65470:0
Key68=65471:0
Key69=65472:0
Key70=65473:0
Key71=65474:0
Key72=65475:0
Key73=65476:0
Key74=65477:0
Key75=65478:0
Key76=65479:0
Key77=65407:0
Key78=65300:0
Key79=65429:0
Key80=65431:0
Key81=65434:0
Key82=65453:45
Key83=65430:0
Key84=65437:0
Key85=65432:0
Key86=65451:43
Key87=65436:0
Key88=65433:0
Key89=65435:0
Key90=65438:0
Key91=65439:0
Key92=65377:0
Key93=0:0
Key94=60:60
Key95=65480:0
Key96=65481:0
Key97=65360:0
Key98=65362:0
Key99=65365:0
Key100=65361:0
Key101=0:0
Key102=65363:0
Key103=65367:0
Key104=65364:0
Key105=65366:0
Key106=65379:0
Key107=65535:127
Key108=65421:13
Key109=65508:0
Key110=65299:0
Key111=65377:0
Key112=65455:47
Key113=65027:0
Key114=269025049:0
Key115=65515:0
Key116=65516:0
Key117=0:0
Key118=269025153:0
Key119=269025093:0
Key120=269025094:0
Key121=269025095:0
Key122=269025096:0
Key123=0:0
Key124=65027:0
Key125=0:0
Key126=65469:61
Key127=0:0
Key128=0:0
Key129=0:0
Key130=0:0
Key131=0:0
Key132=0:0
Key133=0:0
Key134=0:0
Key135=0:0
Key136=0:0
Key137=0:0

[shift]
Key8=65406:0
Key9=65307:27
Key10=33:33
Key11=39:39
Key12=94:94
Key13=43:43
Key14=37:37
Key15=38:38
Key16=47:47
Key17=40:40
Key18=41:41
Key19=61:61
Key20=63:63
Key21=95:95
Key22=65288:8
Key23=65056:0
Key24=81:81
Key25=87:87
Key26=69:69
Key27=82:82
Key28=84:84
Key29=89:89
Key30=85:85
Key31=73:73
Key32=79:79
Key33=80:80
Key34=683:286
Key35=220:220
Key36=65293:13
Key37=65507:0
Key38=65:65
Key39=83:83
Key40=68:68
Key41=70:70
Key42=71:71
Key43=72:72
Key44=74:74
Key45=75:75
Key46=76:76
Key47=426:350
Key48=681:304
Key49=233:233
Key50=65505:0
Key51=59:59
Key52=90:90
Key53=88:88
Key54=67:67
Key55=86:86
Key56=66:66
Key57=78:78
Key58=77:77
Key59=214:214
Key60=199:199
Key61=58:58
Key62=65506:0
Key63=65450:42
Key64=65511:0
Key65=32:32
Key66=65509:0
Key67=65470:0
Key68=65471:0
Key69=65472:0
Key70=65473:0
Key71=65474:0
Key72=65475:0
Key73=65476:0
Key74=65477:0
Key75=65478:0
Key76=65479:0
Key77=65407:0
Key78=65300:0
Key79=65463:55
Key80=65464:56
Key81=65465:57
Key82=65453:45
Key83=65460:52
Key84=65461:53
Key85=65462:54
Key86=65451:43
Key87=65457:49
Key88=65458:50
Key89=65459:51
Key90=65456:48
Key91=65452:44
Key92=65377:0
Key93=0:0
Key94=62:62
Key95=65480:0
Key96=65481:0
Key97=65360:0
Key98=65362:0
Key99=65365:0
Key100=65361:0
Key101=0:0
Key102=65363:0
Key103=65367:0
Key104=65364:0
Key105=65366:0
Key106=65379:0
Key107=65535:127
Key108=65421:13
Key109=65508:0
Key110=65299:0
Key111=65377:0
Key112=65455:47
Key113=65027:0
Key114=269025049:0
Key115=65515:0
Key116=65516:0
Key117=0:0
Key118=269025153:0
Key119=269025093:0
Key120=269025094:0
Key121=269025095:0
Key122=269025096:0
Key123=0:0
Key124=65027:0
Key125=65513:0
Key126=65469:61
Key127=65515:0
Key128=65517:0
Key129=0:0
Key130=0:0
Key131=0:0
Key132=0:0
Key133=0:0
Key134=0:0
Key135=0:0
Key136=0:0
Key137=0:0

[altgr]
Key8=65406:0
Key9=65307:27
Key10=62:62
Key11=163:163
Key12=35:35
Key13=36:36
Key14=189:189
Key15=190:190
Key16=123:123
Key17=91:91
Key18=93:93
Key19=125:125
Key20=92:92
Key21=124:124
Key22=65288:8
Key23=65289:9
Key24=64:64
Key25=16777215:0
Key26=8364:8364
Key27=182:182
Key28=16785594:8378
Key29=2299:8592
Key30=251:251
Key31=238:238
Key32=244:244
Key33=16777215:0
Key34=65111:168
Key35=126:126
Key36=65293:13
Key37=65507:0
Key38=226:226
Key39=223:223
Key40=16777215:0
Key41=170:170
Key42=16777215:0
Key43=16777215:0
Key44=65121:0
Key45=16777215:0
Key46=16777215:0
Key47=180:180
Key48=39:39
Key49=60:60
Key50=65505:0
Key51=96:96
Key52=171:171
Key53=187:187
Key54=162:162
Key55=2770:8220
Key56=2771:8221
Key57=110:110
Key58=181:181
Key59=215:215
Key60=183:183
Key61=65110:729
Key62=65506:0
Key63=65450:42
Key64=65513:0
Key65=32:32
Key66=65509:0
Key67=65470:0
Key68=65471:0
Key69=65472:0
Key70=65473:0
Key71=65474:0
Key72=65475:0
Key73=65476:0
Key74=65477:0
Key75=65478:0
Key76=65479:0
Key77=65407:0
Key78=65300:0
Key79=65429:0
Key80=65431:0
Key81=65434:0
Key82=65453:45
Key83=65430:0
Key84=65437:0
Key85=65432:0
Key86=65451:43
Key87=65436:0
Key88=65433:0
Key89=65435:0
Key90=65438:0
Key91=65439:0
Key92=65377:0
Key93=0:0
Key94=124:124
Key95=65480:0
Key96=65481:0
Key97=65360:0
Key98=65362:0
Key99=65365:0
Key100=65361:0
Key101=0:0
Key102=65363:0
Key103=65367:0
Key104=65364:0
Key105=65366:0
Key106=65379:0
Key107=65535:127
Key108=65421:13
Key109=65508:0
Key110=65299:0
Key111=65377:0
Key112=65455:47
Key113=65027:0
Key114=269025049:0
Key115=65515:0
Key116=65516:0
Key117=0:0
Key118=269025153:0
Key119=269025093:0
Key120=269025094:0
Key121=269025095:0
Key122=269025096:0
Key123=0:0
Key124=65027:0
Key125=0:0
Key126=65469:61
Key127=0:0
Key128=0:0
Key129=0:0
Key130=0:0
Key131=0:0
Key132=0:0
Key133=0:0
Key134=0:0
Key135=0:0
Key136=0:0
Key137=0:0

[shiftaltgr]
Key8=65406:0
Key9=65307:27
Key10=161:161
Key11=178:178
Key12=179:179
Key13=188:188
Key14=2756:8540
Key15=16777215:0
Key16=16777215:0
Key17=16777215:0
Key18=177:177
Key19=176:176
Key20=191:191
Key21=16777215:0
Key22=65288:8
Key23=65056:0
Key24=2009:937
Key25=16777215:0
Key26=16777215:0
Key27=174:174
Key28=16777215:0
Key29=165:165
Key30=219:219
Key31=206:206
Key32=212:212
Key33=16777215:0
Key34=65112:176
Key35=65108:175
Key36=65293:13
Key37=65507:0
Key38=194:194
Key39=16777215:0
Key40=16777215:0
Key41=16777215:0
Key42=16777215:0
Key43=16777215:0
Key44=65122:0
Key45=16777215:0
Key46=16777215:0
Key47=65105:180
Key48=65114:711
Key49=176:176
Key50=65505:0
Key51=65104:96
Key52=60:60
Key53=62:62
Key54=169:169
Key55=2768:8216
Key56=2769:8217
Key57=78:78
Key58=186:186
Key59=16777215:0
Key60=247:247
Key61=65110:729
Key62=65506:0
Key63=65450:42
Key64=65511:0
Key65=32:32
Key66=65509:0
Key67=65470:0
Key68=65471:0
Key69=65472:0
Key70=65473:0
Key71=65474:0
Key72=65475:0
Key73=65476:0
Key74=65477:0
Key75=65478:0
Key76=65479:0
Key77=65407:0
Key78=65300:0
Key79=65463:55
Key80=65464:56
Key81=65465:57
Key82=65453:45
Key83=65460:52
Key84=65461:53
Key85=65462:54
Key86=65451:43
Key87=65457:49
Key88=65458:50
Key89=65459:51
Key90=65456:48
Key91=65452:44
Key92=65377:0
Key93=0:0
Key94=166:166
Key95=65480:0
Key96=65481:0
Key97=65360:0
Key98=65362:0
Key99=65365:0
Key100=65361:0
Key101=0:0
Key102=65363:0
Key103=65367:0
Key104=65364:0
Key105=65366:0
Key106=65379:0
Key107=65535:127
Key108=65421:13
Key109=65508:0
Key110=65299:0
Key111=65377:0
Key112=65455:47
Key113=65027:0
Key114=269025049:0
Key115=65515:0
Key116=65516:0
Key117=0:0
Key118=269025153:0
Key119=269025093:0
Key120=269025094:0
Key121=269025095:0
Key122=269025096:0
Key123=0:0
Key124=65027:0
Key125=65513:0
Key126=65469:61
Key127=65515:0
Key128=65517:0
Key129=0:0
Key130=0:0
Key131=0:0
Key132=0:0
Key133=0:0
Key134=0:0
Key135=0:0
Key136=0:0
Key137=0:0

[capslock]
Key8=65406:0
Key9=65307:27
Key10=49:49
Key11=50:50
Key12=51:51
Key13=52:52
Key14=53:53
Key15=54:54
Key16=55:55
Key17=56:56
Key18=57:57
Key19=48:48
Key20=42:42
Key21=45:45
Key22=65288:8
Key23=65289:9
Key24=81:81
Key25=87:87
Key26=69:69
Key27=82:82
Key28=84:84
Key29=89:89
Key30=85:85
Key31=73:73
Key32=79:79
Key33=80:80
Key34=683:286
Key35=220:220
Key36=65293:13
Key37=65507:0
Key38=65:65
Key39=83:83
Key40=68:68
Key41=70:70
Key42=71:71
Key43=72:72
Key44=74:74
Key45=75:75
Key46=76:76
Key47=426:350
Key48=681:304
Key49=34:34
Key50=65505:0
Key51=44:44
Key52=90:90
Key53=88:88
Key54=67:67
Key55=86:86
Key56=66:66
Key57=78:78
Key58=77:77
Key59=214:214
Key60=199:199
Key61=46:46
Key62=65506:0
Key63=65450:42
Key64=65513:0
Key65=32:32
Key66=65509:0
Key67=65470:0
Key68=65471:0
Key69=65472:0
Key70=65473:0
Key71=65474:0
Key72=65475:0
Key73=65476:0
Key74=65477:0
Key75=65478:0
Key76=65479:0
Key77=65407:0
Key78=65300:0
Key79=65429:0
Key80=65431:0
Key81=65434:0
Key82=65453:45
Key83=65430:0
Key84=65437:0
Key85=65432:0
Key86=65451:43
Key87=65436:0
Key88=65433:0
Key89=65435:0
Key90=65438:0
Key91=65439:0
Key92=65377:0
Key93=0:0
Key94=60:60
Key95=65480:0
Key96=65481:0
Key97=65360:0
Key98=65362:0
Key99=65365:0
Key100=65361:0
Key101=0:0
Key102=65363:0
Key103=65367:0
Key104=65364:0
Key105=65366:0
Key106=65379:0
Key107=65535:127
Key108=65421:13
Key109=65508:0
Key110=65299:0
Key111=65377:0
Key112=65455:47
Key113=65027:0
Key114=269025049:0
Key115=65515:0
Key116=65516:0
Key117=0:0
Key118=269025153:0
Key119=269025093:0
Key120=269025094:0
Key121=269025095:0
Key122=269025096:0
Key123=0:0
Key124=65027:0
Key125=0:0
Key126=65469:61
Key127=0:0
Key128=0:0
Key129=0:0
Key130=0:0
Key131=0:0
Key132=0:0
Key133=0:0
Key134=0:0
Key135=0:0
Key136=0:0
Key137=0:0

[capslockaltgr]
Key8=65406:0
Key9=65307:27
Key10=62:62
Key11=163:163
Key12=35:35
Key13=36:36
Key14=189:189
Key15=190:190
Key16=123:123
Key17=91:91
Key18=93:93
Key19=125:125
Key20=92:92
Key21=124:124
Key22=65288:8
Key23=65289:9
Key24=64:64
Key25=16777215:0
Key26=8364:8364
Key27=182:182
Key28=16785594:8378
Key29=2299:8592
Key30=219:219
Key31=206:206
Key32=212:212
Key33=16777215:0
Key34=65111:168
Key35=126:126
Key36=65293:13
Key37=65507:0
Key38=194:194
Key39=7838:0
Key40=16777215:0
Key41=170:170
Key42=16777215:0
Key43=16777215:0
Key44=65121:0
Key45=16777215:0
Key46=16777215:0
Key47=180:180
Key48=39:39
Key49=60:60
Key50=65505:0
Key51=96:96
Key52=171:171
Key53=187:187
Key54=162:162
Key55=2770:8220
Key56=2771:8221
Key57=78:78
Key58=924:0
Key59=215:215
Key60=183:183
Key61=65110:729
Key62=65506:0
Key63=65450:42
Key64=65513:0
Key65=32:32
Key66=65509:0
Key67=65470:0
Key68=65471:0
Key69=65472:0
Key70=65473:0
Key71=65474:0
Key72=65475:0
Key73=65476:0
Key74=65477:0
Key75=65478:0
Key76=65479:0
Key77=65407:0
Key78=65300:0
Key79=65429:0
Key80=65431:0
Key81=65434:0
Key82=65453:45
Key83=65430:0
Key84=65437:0
Key85=65432:0
Key86=65451:43
Key87=65436:0
Key88=65433:0
Key89=65435:0
Key90=65438:0
Key91=65439:0
Key92=65377:0
Key93=0:0
Key94=124:124
Key95=65480:0
Key96=65481:0
Key97=65360:0
Key98=65362:0
Key99=65365:0
Key100=65361:0
Key101=0:0
Key102=65363:0
Key103=65367:0
Key104=65364:0
Key105=65366:0
Key106=65379:0
Key107=65535:127
Key108=65421:13
Key109=65508:0
Key110=65299:0
Key111=65377:0
Key112=65455:47
Key113=65027:0
Key114=269025049:0
Key115=65515:0
Key116=65516:0
Key117=0:0
Key118=269025153:0
Key119=269025093:0
Key120=269025094:0
Key121=269025095:0
Key122=269025096:0
Key123=0:0
Key124=65027:0
Key125=0:0
Key126=65469:61
Key127=0:0
Key128=0:0
Key129=0:0
Key130=0:0
Key131=0:0
Key132=0:0
Key133=0:0
Key134=0:0
Key135=0:0
Key136=0:0
Key137=0:0

[shiftcapslock]
Key8=65406:0
Key9=65307:27
Key10=33:33
Key11=39:39
Key12=94:94
Key13=43:43
Key14=37:37
Key15=38:38
Key16=47:47
Key17=40:40
Key18=41:41
Key19=61:61
Key20=63:63
Key21=95:95
Key22=65288:8
Key23=65056:0
Key24=113:113
Key25=119:119
Key26=101:101
Key27=114:114
Key28=116:116
Key29=121:121
Key30=117:117
Key31=697:305
Key32=111:111
Key33=112:112
Key34=699:287
Key35=252:252
Key36=65293:13
Key37=65507:0
Key38=97:97
Key39=115:115
Key40=100:100
Key41=102:102
Key42=103:103
Key43=104:104
Key44=106:106
Key45=107:107
Key46=108:108
Key47=442:351
Key48=105:105
Key49=201:201
Key50=65505:0
Key51=59:59
Key52=122:122
Key53=120:120
Key54=99:99
Key55=118:118
Key56=98:98
Key57=110:110
Key58=109:109
Key59=246:246
Key60=231:231
Key61=58:58
Key62=65506:0
Key63=65450:42
Key64=65511:0
Key65=32:32
Key66=65509:0
Key67=65470:0
Key68=65471:0
Key69=65472:0
Key70=65473:0
Key71=65474:0
Key72=65475:0
Key73=65476:0
Key74=65477:0
Key75=65478:0
Key76=65479:0
Key77=65407:0
Key78=65300:0
Key79=65463:55
Key80=65464:56
Key81=65465:57
Key82=65453:45
Key83=65460:52
Key84=65461:53
Key85=65462:54
Key86=65451:43
Key87=65457:49
Key88=65458:50
Key89=65459:51
Key90=65456:48
Key91=65452:44
Key92=65377:0
Key93=0:0
Key94=62:62
Key95=65480:0
Key96=65481:0
Key97=65360:0
Key98=65362:0
Key99=65365:0
Key100=65361:0
Key101=0:0
Key102=65363:0
Key103=65367:0
Key104=65364:0
Key105=65366:0
Key106=65379:0
Key107=65535:127
Key108=65421:13
Key109=65508:0
Key110=65299:0
Key111=65377:0
Key112=65455:47
Key113=65027:0
Key114=269025049:0
Key115=65515:0
Key116=65516:0
Key117=0:0
Key118=269025153:0
Key119=269025093:0
Key120=269025094:0
Key121=269025095:0
Key122=269025096:0
Key123=0:0
Key124=65027:0
Key125=65513:0
Key126=65469:61
Key127=65515:0
Key128=65517:0
Key129=0:0
Key130=0:0
Key131=0:0
Key132=0:0
Key133=0:0
Key134=0:0
Key135=0:0
Key136=0:0
Key137=0:0

[shiftcapslockaltgr]
Key8=65406:0
Key9=65307:27
Key10=161:161
Key11=178:178
Key12=179:179
Key13=188:188
Key14=2756:8540
Key15=16777215:0
Key16=16777215:0
Key17=16777215:0
Key18=177:177
Key19=176:176
Key20=191:191
Key21=16777215:0
Key22=65288:8
Key23=65056:0
Key24=2009:937
Key25=16777215:0
Key26=16777215:0
Key27=174:174
Key28=16777215:0
Key29=165:165
Key30=251:251
Key31=238:238
Key32=244:244
Key33=16777215:0
Key34=65112:176
Key35=65108:175
Key36=65293:13
Key37=65507:0
Key38=226:226
Key39=16777215:0
Key40=16777215:0
Key41=16777215:0
Key42=16777215:0
Key43=16777215:0
Key44=65122:0
Key45=16777215:0
Key46=16777215:0
Key47=65105:180
Key48=65114:711
Key49=176:176
Key50=65505:0
Key51=65104:96
Key52=60:60
Key53=62:62
Key54=169:169
Key55=2768:8216
Key56=2769:8217
Key57=110:110
Key58=186:186
Key59=16777215:0
Key60=247:247
Key61=65110:729
Key62=65506:0
Key63=65450:42
Key64=65511:0
Key65=32:32
Key66=65509:0
Key67=65470:0
Key68=65471:0
Key69=65472:0
Key70=65473:0
Key71=65474:0
Key72=65475:0
Key73=65476:0
Key74=65477:0
Key75=65478:0
Key76=65479:0
Key77=65407:0
Key78=65300:0
Key79=65463:55
Key80=65464:56
Key81=65465:57
Key82=65453:45
Key83=65460:52
Key84=65461:53
Key85=65462:54
Key86=65451:43
Key87=65457:49
Key88=65458:50
Key89=65459:51
Key90=65456:48
Key91=65452:44
Key92=65377:0
Key93=0:0
Key94=166:166
Key95=65480:0
Key96=65481:0
Key97=65360:0
Key98=65362:0
Key99=65365:0
Key100=65361:0
Key101=0:0
Key102=65363:0
Key103=65367:0
Key104=65364:0
Key105=65366:0
Key106=65379:0
Key107=65535:127
Key108=65421:13
Key109=65508:0
Key110=65299:0
Key111=65377:0
Key112=65455:47
Key113=65027:0
Key114=269025049:0
Key115=65515:0
Key116=65516:0
Key117=0:0
Key118=269025153:0
Key119=269025093:0
Key120=269025094:0
Key121=269025095:0
Key122=269025096:0
Key123=0:0
Key124=65027:0
Key125=65513:0
Key126=65469:61
Key127=65515:0
Key128=65517:0
Key129=0:0
Key130=0:0
Key131=0:0
Key132=0:0
Key133=0:0
Key134=0:0
Key135=0:0
Key136=0:0
Key137=0:0
```

### 5. XRDP servisini yeniden başlatın
```
sudo systemctl restart xrdp
```
