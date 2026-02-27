# H6 Hardware hacking

Tehtävässä käytetty virtuaalikone

<img width="452" height="242" alt="image" src="https://github.com/user-attachments/assets/27f815b0-93f1-4c59-8d53-222ad83fb3d1" />


## 1.) Decrypt firmware image

Aloitetaan kloonaamalla ```https://github.com/robbins/tp-link-decrypt``` sekä lataamalla annetut tiedostot TapoV3 firmware binary ja camera dump-file

Read me ohjeistaa ajamaan skriptit ```./extract_keys.sh``` ja ```./extract_keys.sh```

Make komennolla sain outoja virheilmoituksia joihin yiritin tekoälyllä löytää ratkaisua mutta turhaan...
Virheitä oli kymmeniä rivejä 
<img width="674" height="269" alt="image" src="https://github.com/user-attachments/assets/b800aa54-1a78-4dc9-8d66-7fa28993cf25" />
Tässä tappelin niin kauan, että päätin vain siirtyä eteenpäin

Yirtin katsoa apua muilta kurssilla olleilta opiskelijoilta, mutta tuloksetta.

## 2.) Analyse the image file

Yirtin binwalking avulla tutkia tiedostoa, mutta tuloksetta

<img width="494" height="83" alt="image" src="https://github.com/user-attachments/assets/ac5ed448-3855-4ccb-b73e-c120ad4e74c2" />

Yritin strings komennon avulla, mutta siitäkään ei ottanut selvää

## 3.)  Extract rootfs from the dump file

Dump tiedosto on ilmeisesti raakakopio flash-muistista ja sisältää laitteen sisäisen tiedostojärjestelmän
Täältä pitäisi löytää rivi squashfs, sillä se on rootfs:n aloituskohta

```binwalk dump-tapo-c200v3-1.4.2.bin```

<img width="362" height="119" alt="image" src="https://github.com/user-attachments/assets/91f7ea03-610f-4e9b-85d1-1067f2400201" />

Erotetaan rootfs dumpista: ```dd if=dump-tapo-c200v3-1.4.2.bin of=rootfs.bin bs=1 skip=4456448```

Puretaan se binwalkin avulla ```binwalk -e rootfs.bin```

Huomataan, että onnistuimme
<img width="853" height="154" alt="image" src="https://github.com/user-attachments/assets/4e86c8b3-8848-4efc-9cd9-a450b6bd7727" />

Squashfs-root hakemistosta löydetään rootfs:n sisältö

<img width="943" height="244" alt="image" src="https://github.com/user-attachments/assets/50fc3cd8-e078-41cb-b43a-39fbb87cd2fc" />

## 4.) Extract rootfs from the image file

Tähän jumituin ensimmäisen vaiheen takia, sillä image filen kanssa oli ongelmia, enkä saanut sitä decryptattua.


## 5.) Search available applications

Tarkoituksena penkoa 3 purettua dump tiedostoa, etsiä sieltä ohjelmia, konfiguraatiota ja skriptejä

<img width="823" height="82" alt="image" src="https://github.com/user-attachments/assets/371b4c50-dfa9-4eb4-8795-7987ddc23f3b" />

dmesg näyttää kernel-logit
logcat näyttää sovellusten lokit
main on yleensä pääohjelma

Etc hakemistossa on seuraavaa: <img width="464" height="146" alt="image" src="https://github.com/user-attachments/assets/70f717e3-6a19-4501-8c46-0c96f402ba3d" />

default_isp kertoo kameran kuvankäsittelyparametrien ja isp säädösten rakenteen

webrtc_profile.ini paljastaa miten video kulkee cloud yhteydellä

## 6.) Analyse and try to open root password

Tarkoituksena löytää salasana

Aloitin menemällä aikaisemmassa tehtävässä löydettyyn pääohjelmaan, kokeilemaan stringsin avulla löydetäänkö salasana

<img width="1377" height="628" alt="image" src="https://github.com/user-attachments/assets/0bee8351-67d2-4d7f-bb3c-e699db08184f" />

Täältä ei kuitenkaan löydetty salasanaa, vaan se on salattu

Tässä ilmeisesti käytetään PasswordDigest protokollaa (SALT+password+nonce) = digesti
käytetään RSTP/ONVIF autentikaatiossa

Lisäksi HMACIa salasanan ja tokenin käsittelyyn, tämä ilmeisesti estää replay hyökkäykset ja varmistaa salasanan

Tässä vaiheessa tehtävään oli mennyt jo useita tunteja, enkä keksinyt mitään lähestymistapaa, joten jätin tehtävän tähän.


## Lähteet

Kaiser, Q. (2025, July 25). Rooting the Tapo C200.
https://quentinkaiser.be/security/2025/07/25/rooting-tapo-c200/

https://hhmoodle.haaga-helia.fi/course/view.php?id=45171&section=3#tabs-tree-start

Tehtävässä käytetty Microsoft Copilotia apuna koodin tulkitsemisessa


