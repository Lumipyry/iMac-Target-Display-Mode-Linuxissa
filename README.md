# Näin käytät iMac konetta 2-näyttönä Linuxissa
![imac-target-display-mode-hero](https://github.com/user-attachments/assets/f5948c13-a0b7-4bae-abd1-c50fb6f6df32)

_In english: [    Step-by-step-instructions-for-using-iMac-in-Target-Display-Mode](https://github.com/Lumipyry/Step-by-step-instructions-for-using-iMac-in-Target-Display-Mode)_

Tämä on asennusohje [smc_util](https://github.com/floe/smc_util):lle sisältäen FreekMank:n [tuen virtanäppäimen käytölle](https://github.com/floe/smc_util/pull/11/commits).

Kohdenäyttötila (Target Display Mode) käynnistetään ja sammutetaan painamalla virtanäppäintä.

Soveltuu iMac malleihin [2009-2014](https://support.apple.com/fi-fi/105126). Ulkoisena näyttönä käytettävässä koneessa tulee olla sekä Linux että Apple High Sierra tai aikaisempi.

Toimii todistettavasti MiniDisplay Port kaapelilla. Ei henkilökohtaisia kokemuksia Thunderbolt kaapelilla. Testattu iMac malleilla 2009-2011.

***
HUOM: Luultavasti haluat jättää `rc.local`:n pois asennuksesta (`Kohta 16`) - jos ja kun haluat myös käyttää näyttökoneen Linux käyttöjärjestelmää. (`rc.local` käynnistää koneen suoraan kohdenäyttötilaan ja oma kokemukseni on että TDM:n pois laittaminen johtaa tyhjään näyttöön)

Ilman `rc.local`:a kone käynnistyy ensin Linuxissa.
***
Asennus tapahtuu seuraavilla komennoilla käyttämällä päätettä (Terminal)

1.Lataa smc_util 2-näyttökoneen kotihakemistoon
```
git clone https://github.com/floe/smc_util.git
```
2.Siirry hakemistoon smc_util
```
cd smc_util
```
3.Käännä SmcDumpkey GCC:llä
```
gcc -O2 -o SmcDumpKey SmcDumpKey.c -Wall
```
4.Luo tiedostot tdm_toggle.sh, powerbutton, powerbutton.sh ja rc.local
```
touch tdm_toggle.sh powerbutton powerbutton.sh rc.local
```
5.Vaihda tiedoston tdm_off.sh sisältö seuraavanlaiseksi 

(käyttämällä päätteessä komentoa `nano ~/smc_util/tdm_off.sh` tai avaamalla tiedosto tekstinkäsittelyohjelmalla)
```
#!/bin/bash
rm -f tdm_started
./SmcDumpKey MVHR 0
sleep 1
./SmcDumpKey MVMR 2
sleep 2
DISPLAY=:0.0 xrandr --output eDP --auto

6.Change content of file tdm_on.sh

#!/bin/bash
./SmcDumpKey MVHR 1
sleep 1
./SmcDumpKey MVMR 2
sleep 2
DISPLAY=:0.0 xrandr --output eDP --off
touch tdm_started
```
6.Vaihda tiedoston tdm_on.sh sisältö seuraavanlaiseksi
```
#!/bin/bash
./SmcDumpKey MVHR 1
sleep 1
./SmcDumpKey MVMR 2
sleep 2
DISPLAY=:0.0 xrandr --output eDP --off
touch tdm_started
```
7.Luo seuraava sisältö tiedostoon tdm_toggle.sh
```
#!/bin/bash
pushd $(dirname "${BASH_SOURCE[0]}")

if [[ -f "tdm_started" ]]; then
  echo "Switching off TDM"
  source tdm_off.sh
else
  echo "Switch on TDM"
  source tdm_on.sh
fi

popd

8.Create content to file powerbutton

event=button/power PBTN
action=/etc/acpi/powerbutton.sh
```
8.Luo seuraava sisältö tiedostoon powerbutton
```
event=button/power PBTN
action=/etc/acpi/powerbutton.sh
```
9.Luo seuraava sisältö tiedostoon powerbutton.sh. `MUISTA VAIHTAA XXXXXXXXX omaan käyttäjänimeesi (KAHDESSA kohdassa scriptissä)`
```
#!/bin/bash

pushd $(dirname "${BASH_SOURCE[0]}")

FILE=powerbutton_pressed
NOW=$(date +%s)

if [[ -f "$FILE" ]]; then
  # Read timestamp of previous powerbutton press from file
  echo "File exists"
  typeset -i PREV=$(cat $FILE)
  echo Compare $NOW and $PREV

  # if two powerbutton presses were <1 seconds apart -> shutdown
  if [[ $(($NOW-$PREV)) -lt 2 ]]; then
    # Shutdown
    echo "Powerbutton pressed twice in a row: Shutting down"
    shutdown now
  else
    echo "Toggle TDM"
    /home/XXXXXXXXX/smc_util/tdm_toggle.sh &
  fi
else
  echo "Toggle TDM"
  /home/XXXXXXXXX/smc_util/tdm_toggle.sh &
fi

echo $NOW > $FILE

popd
```
10.Luo seuraava sisältö tiedostoon rc.local. `MUISTA VAIHTAA XXXXXXXXX omaan käyttäjänimeesi (KAHDESSA kohdassa scriptissä)`
```
#!/bin/bash

# Start Target Display Mode such that the pc is used as external monitor right away
# Note: The Power button toggles TDM (see /etc/acpi/events)
pushd /home/XXXXXXXXX/smc_util
./tdm_on.sh
popd
```
11.Poista SMC kernel ajuri konfliktien välttämiseksi
```
sudo rmmod applesmc
```
12.Muuta tdm_toggle.sh, rc.local ja powerbutton.sh suoritettaviksi (`vaihda XXXXXXXX`)
```
sudo chmod +x /home/XXXXXXXXX/smc_util/tdm_toggle.sh /home/XXXXXXXXX/smc_util/rc.local /home/XXXXXXXXX/smc_util/powerbutton.sh
```
13.Asenna build-essential ja acpid
```
sudo apt install build-essential acpid
```
14. Kopioi tiedosto powerbutton hakemistoon /etc/acpi/events (`vaihda XXXXXXXXXX`)
```
sudo cp /home/XXXXXXXXX/smc_util/powerbutton /etc/acpi/events
```
15.Kopioi tiedosto powerbutton.sh hakemistoon /etc/acpi (`vaihda XXXXXXXX`)
```
sudo cp /home/XXXXXXXXX/smc_util/powerbutton.sh /etc/acpi
```
16.Kopioi tiedosto rc.local hakemistoon /etc (`vaihda XXXXXXXX`)
```
sudo cp /home/XXXXXXXXX/smc_util/rc.local /etc/rc.local
```
17.Käynnistä `acpid` uudelleen
```
sudo systemctl restart acpid
```
18.Vaihda virtanäppäimen toiminta asetusten osassa Virranhallinta seuraavanlaiseksi

`Älä tee mitään`
***
TDM:n sammutus eli "Off": Kytke ensin ulkoinen näyttö pois käytöstä pääkoneen näyttöasetuksista ja paina sitten virtanäppäintä näyttönä toimivassa iMacissa.

***
Vinkki: Näyttönä toimivaan iMaciin kannattaa asentaa Samba palvelin ja kytkeä koneet yhteen Ethernet-piuhalla. Näin voit käyttää näyttökoneen tiedostojärjestelmää tallennustilana ja siirtää tiedostoja koneiden välillä. [Ohje löytyy täältä](https://github.com/Lumipyry/Luo-kotiverkko-ja-asenna-Samba-palvelin)
***
