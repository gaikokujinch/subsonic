Subsonic


ToC
--------
+ [Add jail](#addjail)
+ [Add user sonic](#usersonic)
+ [Add storage](#addstorage)
+ [Install Subsonic](#installsubsonic)
+ [Install SSL](#ssl)
+ [Upgrade Subsonic](#upgradesubsonic)
+ [Upgrade Jail](#upgradejail)




Notes
-----

```
jls
jexec 18 tcsh
```


Inside the jail

Optional: give root a password
```
passwd
```


Source
https://forums.freenas.org/index.php?threads/how-to-install-subsonic-4-8-on-freenas-9-1-1.15016/



**Add jail<a name="#addjail">**

Using WebGUI
Click add Jail and give it a name "Subsonic" and then hit the advanced tab and and un-check "Vanilla" (make sure you have standard selected and VIMAGE ticked, probably a good idea to put in gateway too) and click ok.



**Add user sonic<a name="#usersonic">**

```
adduser
```
Username: sonic
Full name: Sonic
Uid: (1011)
Login group: (default)
Login group is sonic. Invite sonic into other groups:
Login class: (default)
Shell: tcsh
Home directory: (default)
Home directory permissions: (default)
Use password-based authentication: no
Lock out the account after creation: (default)
OK?: yes
Add another user?: no


```
mkdir /music
chown -R sonic:sonic /music
su - sonic
mkdir /home/sonic/install
```

**Add storage<a name="#addstorage">**

Add storage to /music in the NAS gui


**Install Subsonic<a name="#installsubsonic">**


```
su - root
pkg update && pkg upgrade -y
pkg search openjdk
pkg install openjdk-xxxx
pkg install xtrans xproto xextproto javavmwrapper lame flac
portsnap fetch extract
portsnap fetch update
cd /usr/ports/multimedia/ffmpeg
make
```
---- Important ---- At this point you will be presented a list of options, make sure you an X in the LAME section. I also turn on AAC support (top).
```
make install clean
mkdir -p /home/sonic/subsonic/transcode
mkdir /home/sonic/subsonic/standalone
cp /usr/local/bin/lame /home/sonic/subsonic/transcode/
cp /usr/local/bin/flac /home/sonic/subsonic/transcode/
ln /usr/local/bin/ffmpeg /home/sonic/subsonic/transcode/ffmpeg
chown -R subsonic:subsonic /home/sonic/subsonic
su - sonic
cd /tmp


wget https://s3-eu-west-1.amazonaws.com/subsonic-public/download/subsonic-6.1.1-standalone.tar.gz
tar xvzf /tmp/subsonic-6.1.1-standalone.tar.gz -C /home/sonic/subsonic/standalone
```
Edit File /home/sonic/subsonic/standalone/subsonic.sh and change these lines to match your setup:
> SUBSONIC_HOME=/home/sonic/subsonic
> SUBSONIC_DEFAULT_MUSIC_FOLDER=/music
> SUBSONIC_DEFAULT_PODCAST_FOLDER=/music/Podcast
> SUBSONIC_DEFAULT_PLAYLIST_FOLDER=/music/playlists


To start the server automatically at every FreeNAS reboots
```
ps ax  | grep subsonic
```
Only if pid appears
```
kill (pid)
```
```
nano /usr/local/etc/rc.d/subsonic
```
Copy contents of subsonic into that file.

```
chmod ug+x /usr/local/etc/rc.d/subsonic
nano /etc/rc.conf
```
Add lines from rc.conf to that file.


```
/usr/local/etc/rc.d/subsonic start
```


Now open a web browser and go to http://10.0.0.26:4040






**Install SSL<a name="#ssl">**

Source: https://project.altservice.com/issues/761
Install openssl and zip:
```
pkg install openssl zip
```
Generate a strong SSL key and a CSR to send for signing by a CA:
```
cd /usr/local/etc
openssl req -sha512 -out madsonic.example.com.csr -new -newkey rsa:4096 -nodes -keyout madsonic.example.com.key
```
Combine the SSL key, certificate, and CA intermediate certificate files together into a madsonic-bundle.crt file:
```
cat madsonic.example.com.key madsonic.example.com.crt startcom.class1.bundle > madsonic-bundle.crt
```
Next convert it to a format madsonic understands:
```
openssl pkcs12 -in madsonic-bundle.crt -export -out madsonic-bundle.pkcs12
```
When prompted enter madsonic as export password.
Now you should have a madsonic-bundle.pkcs12 file, we need to import this into a keystore for Madsonic to use:
```
keytool -importkeystore -srckeystore madsonic-bundle.pkcs12 -destkeystore madsonic.keystore -srcstoretype PKCS12 -srcstorepass madsonic -srcalias 1 -destalias madsonic
```
When prompted enter madsonic as the password.
Finally we need to put the keystore into the file Madsonic uses to boot:
```
zip /usr/local/madsonic-standalone/madsonic-booter-jar-with-dependencies.jar madsonic.keystore
```
Now set the madsonic_https_port in the rc.conf file and restart madsonic:
```
echo 'madsonic_https_port="4443"' >> /etc/rc.conf
service madsonic restart
```



**Upgrade Subsonic<a name="#upgradesubsonic">**

https://forums.freenas.org/index.php?threads/how-to-install-subsonic-4-8-on-freenas-9-1-1.15016/page-5#post-98300


```
su - sonic
mkdir /usr/home/sonic/subsonic_backup
cp /usr/home/sonic/subsonic /usr/home/sonic/subsonic_backup
ps ax  | grep subsonic
kill (pid)
mkdir /usr/home/sonic/temp
cd /usr/home/sonic/temp
wget https://s3-eu-west-1.amazonaws.com/subsonic-public/download/subsonic-6.1.1-standalone.tar.gz
tar xvzf subsonic-6.1.1-standalone.tar.gz -C /home/sonic/subsonic/standalone
cp /usr/home/sonic/subsonic_backup/standalone/subsonic.properties /home/sonic/subsonic/standalone/subsonic.properties
cp /usr/home/sonic/subsonic_backup/standalone/subsonic.sh 


```
Where is the db folder?
```
/home/sonic/subsonic/standalone/subsonic.sh
cp /usr/home/sonic/subsonic_backup/standalone/db /home/sonic/subsonic/standalone/db
/usr/local/etc/rc.d/subsonic start
```


**Upgrade Jail<a name="#upgradejail">**


https://forums.freenas.org/index.php?threads/how-to-install-subsonic-4-8-on-freenas-9-1-1.15016/page-5#post-134977

```
portsnap fetch extract
portsnap fetch update
cd /usr/ports/ports-mgmt/pkg
make install
```
You will get errors -- look over this thread http://forums.freebsd.org/viewtopic.php?f=5&t=44181&p=245847#p245859 and delete the old pkg stuff as indicated
```
cd /usr/ports/ports-mgmt/pkg
make install
pkg2ng
cd /usr/ports/multimedia/ffmpeg
make
```
---- Important ---- At this point you will be presented a list of options, make sure you an X in the LAME section. I also turn on AAC support (top).
```
make install clean


```



```
```
