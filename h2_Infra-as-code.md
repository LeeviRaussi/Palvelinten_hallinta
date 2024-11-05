# h2 Infra-as-code

> ## Tiivistelmä
>
> Kuvaus

> ### Käytetty laitteisto
>
> Lenovo Ideapad Pro 5 14APH8 kannettava tietokone
> - Prosessori: AMD Ryzen 7 7840HS
> - GPU: Radeon 780M Graphics (prosessoriin integroitu)
> - RAM: 32 GB, 6400 MT/s
> - OS: Windows 11 Home 23H2
> - Näytön resoluutio: 2880x1800 (175% skaalaus)
> - SSD: 745/951 GB vapaana
>
> Virtuaalikoneena VirtualBoxiin asennettu Debian 12 Bookworm.

## x) Tiivistelmät

### Two Machine Virtual Network With Debian 11 Bullseye and Vagrant (Karvinen 4.11.2021)

Karvinen (4.11.2021) esittelee artikkelissaan Vagrant-ohjelmiston käyttöönottamista. Vagrantilla voi luoda virtuaalisia verkkoyhteyksiä virtuaalikoneiden välille hyödyntäen VirtualBoxia. Vagrantia pystyy käyttämään niin Linuxilla, Windowsilla kuin myös macOS:lla. Käyttö onnistuu SSH-yhteyden ylitse, minkä lisäksi Vagrantia voidaan pyörittää suoraan komentoriviltä. Vagrantilla pystyy luomaan ja poistamaan virtuaalikoneita nopeasti, mikä tekee niiden käytöstä helppoa.

### Salt Quickstart – Salt Stack Master and Slave on Ubuntu Linux (Karvinen 28.3.2018)

Tiivistelmä artikkelista löytyy edellisen tehtävän raportistani (ks. LeeviRaussi 29.10.2024), joka löytyy kopioituna alta.

> Karvinen (28.3.2018) esittelee artikkelissaan miten Salt otetaan käyttöön Xubuntu 16.04 käyttöjärjestelmässä, joskin vastaavia komentoja voinee hyödyntää myös muissa käyttöjärjestelmissä. Keskeistä Saltin asentamisessa on, että herrana toimivan koneen osoite on orjien tiedossa, ja kommunikointi tapahtuu porttien 4505 ja 4506 kautta, jotka tulee avata palomuurissa. Herran ei tarvitse tietää orjien osoitteita, mutta jokaisella orjalla tulee olla yksilöivä nimi (id), jotta niitä pystytään komentamaan.

### Hello Salt Infra-as-Code (Karvinen 3.4.2024)

Karvinen (3.4.2024) esittelee artikkelissaan miten Saltia voi käyttää infrastruktuuri koodina. Linuxissa herrana toimivan koneen hakemistoon "/srv/salt/" luodaan eri moduuleja, joita jaetaan orjana oleville koneille. Jokainen moduuli muodostaa oman hakemistonsa, josta löytyvät tarvittavat koodit ja tiedostot, mitä kyseinen moduuli vaatii. Kooditiedostot kirjoitetaan Saltin omalla kielellä, jossa voidaan hyödyntää viittää tärkeintä tilafunktiota: pkg, file, service, user ja cmd.

### Salt Vagrant - automatically provision one master and two slaves (Karvinen 28.3.2023)

Karvinen (28.3.2023) käisttelee artikkelissaan Saltin yhdistämistä Vagrantiin. Infra as Code -kappaleessa luodaan hello-moduuliin init.sls-tiedosto, joka sisältää YAML:llä kirjoitetun koodin:
```
/tmp/infra-as-code:
  file.managed
```
Moduuli saadaan käyttöön komennolla `sudo salt '*' state.apply hello`. Hakemistoon /srv/salt/ voidaan myös luoda top.sls-niminen tiedosto, joka määrittää, mitä tiloja suoritetaan millekin orjalle. Tämäkin tiedosto noudattaa YAML:ää, ja sen koodi näyttää esimerkiksi seuraavanlaiselta:
```
base:
  '*':
    - hello
```
top.sls-tiedostoa hyödyntämällä ei tarvitse erikseen määrittää komennoissa, mitä moduuleja käytetään, vaan voidaan käyttää suoraan komentoa `sudo salt '*' state.apply`.

### Salt overview (VMware 2024)

VMwaren (2024) artikkelissa esitellään Saltia ja YAML merkintäkieltä, johon tässä tiivistelmässä keskitytään. Salt käyttää oletuksena YAML:ää kääntämään kyseisen merkintäkielen datarakenteen Saltin ymmärtämäksi Python datarakenteeksi. YAML rakentuu avain-arvo pareista, jotka erotetaan toisistaan kaksoispisteellä, jota seuraa täsmälleen yksi välilyönti (": "). Näitä avain-arvo pareja on kaikkiaan kolmenlaisia: skalaarinen (arvo voi olla luku, teksti tai totuusarvo), lista (sisältää skalaareja) ja sanakirja (sisältää skalaareja ja listoja). YAML:n rakenne erottelee isot ja pienet kirjaimet toisistaan, minkä lisäksi ainakin yhtä välilyöntiä on käytettävä hyödynnettäessä lista- tai sanakirja blokkirakennetta, joskin kaksi välilyöntiä on vakioitunut standardiksi.

## a) Hello Vagrant!

*0:00*

Koska minulla ei ollut vielä Vagrantia asennettuna, aloitin tehtävän asentamalla sen. Olen tähän asti käyttänyt Linuxia VirtualBoxilla tehdyllä virtuaalikoneella, minkä myötä tehtävänannon vinkin (ks. Karvinen 10.5.2024) takia päätin asentaa Vagrantin isäntäjärjestelmänä olevaan Windows 11:een. Karvisen (4.11.2021) artikkelissa oli linkki Windows version asennukseen (ks. Hashicorp 2024a), josta latasin AMD64-pohjaisen Vagrantin (versio 2.4.2). Puolen minuutin latauksen jälkeen Windowsin Setup Wizard otti asennuksen hoidettavakseen ja itse asennusprosessi eteenpäin-nappien paineluiden kanssa vei reilut kolme minuuttia, minkä jälkeen sain kehoituksen tietokoneen uudelleen käynnistämisestä asennuksen saattamiseksi loppuun. Koneen käynnistyttyä uudelleen kävin läpi aloitusvalikkoni ohjelmat, mutten huomannut Vagrantia siellä. Googlaamalla pääsin Vagrantin etusivulle, josta oli suoraan linkit käyttöön liittyviin ohjeisiin, mistä sain tiedon, että Vagrantia käytetään suoraan komentoriviltä (Hashicorp 2024b). Ajoin komentorivillä komennon `vagrant -v`, joka näytti, että olin onnistuneesti asentanut oikean version ohjelmasta (ks. kuva alla).

![vagrant install](https://github.com/user-attachments/assets/1c2a2c33-775d-4140-b90f-d11d64308c0e)

## b) Linux Vagrant

*0:09*

Komentorivin käyttäminen Windowsissa erosi komennoiltaan jonkin verran Linuxista, joten etsin Googlella oppaan, jolla saisin varmistettua yleisimpien komentojen muodot. Koladen (9.8.2022) ohjesivu tarjosi vastaukset näihin ongelmiini. Navigoin ensin hakemistossa `cd`-komennoilla tämän viikon tehtäväkansiooni, johon loin Karvisen (4.11.2021) artikkelin mukaisesti kansion "twohost" komennolla `mkdir twohost`, ja siirryin sitten kyseiseen hakemistoon `cd twohost`-komennolla. Ei niin yllätyksekseni Windowsissa ei ole nano-editoria, joten käytin komentoa `notepad Vagrantfile.txt`, jolla loin uuden tekstitiedoston. Kopioin kyseiseen tiedostoon Karvisen (4.11.2021) artikkelista sisällön, joskin muutin käyttöjärjestelmän tietoihin "debian/bookworm64" tehtävänannon mukaisesti (ks. Karvinen 10.5.2024). Ajoin sitten artikkelin komennon `vagrant ssh t001`, mutta sain ilmoituksen, ettei ympäristöä Vagrantille ollut vielä luotuna ja minun pitäisi tätä varten ajaa komento `vagrant init`. Tein tämän ja sain ilmoituksen, että Vagrantfile-tiedosto (ei .txt loppuinen) oli luotu kansioon. Tarkistin asian graafisesta näkymästä ja totesin näin olevan asian laita, joten poistin edellä luomani Vagrantfile.txt-tiedoston komennolla `del Vagrantfile.txt` otettuani ensin tiedoston sisällön talteen leikepöydälle. Komennolla `notepad Vagrantfile` avasin oikean tiedoston ja liitin siihen edellä muokkaamani asetukset (ks. ensimmäinen kuva alla). Ajoin sitten uudelleen komennon `vagrant ssh t001`, mutta sain tällä kertaa ilmoituksen, ettei virtuaalikoneita ollut vielä luotu, ja tämä tapahtuisi komennolla `vagrant up`. Ajoin kyseisen komennon, joka aloitti muutaman minuutin pituisen lataus-/asennusprosessin (ks. keskimmäinen alla oleva kuva komennoista tähän asti). Prosessin tultua päätökseen pääsin viimein ajamaan onnistuneesti komennon `vagrant ssh t001`. Ajoin vielä komennon `pwd` varmistuakseni, että olin todella Linux-pohjaisessa virtuaalikoneessa (ks. alimmainen alla oleva kuva).

![vagrantfile](https://github.com/user-attachments/assets/a65bf8a7-bdc2-4673-be51-067c90dba9c6)

![vagrant commands](https://github.com/user-attachments/assets/1d89da66-5225-44f4-b7b6-8934487efa65)

![vagrant login](https://github.com/user-attachments/assets/d809b091-811f-4f26-be9b-59b698f17dd7)

## c) Kaksin kaunihimpi

*0:19*

Koska tehtävässä b) olin jo luonut kaksi virtuaalikonetta, siirryin suoraan testaamaan pingin toimimista. Koneelta t001 suoritin komennon `ping -c 1 192.168.88.102`, joka onnistui ongelmitta. Kirjauduin ulos komennolla `exit` ja siirryin koneelle t002 komennolla `vagrant ssh t002`, jossa pingasin vastavuoroisesti t001 IP-osoitetta komennolla `ping -c 1 192.168.88.101`. Tämäkin onnistui, joten pingasin vielä lopuksi Googlen DNS-palvelinta testimielessä jälleen ilman ongelmia (ks. kuva alla).

![vagrant ping](https://github.com/user-attachments/assets/e4d9a652-ebda-4978-b134-0571c5cc3bbb)

## Lähdeluettelo

Hashicorp 2024a. Install Vagrant. Luettavissa: https://developer.hashicorp.com/vagrant/install. Luettu: 5.11.2024.

Hashicorp 2024b. Install Vagrant. Luettavissa: https://developer.hashicorp.com/vagrant/tutorials/getting-started/getting-started-install. Luettu: 5.11.2024.

Karvinen 28.3.2018. Salt Quickstart – Salt Stack Master and Slave on Ubuntu Linux. Luettavissa: https://terokarvinen.com/2018/03/28/salt-quickstart-salt-stack-master-and-slave-on-ubuntu-linux/. Luettu: 5.11.2024.

Karvinen, T. 4.11.2021. Two Machine Virtual Network With Debian 11 Bullseye and Vagrant. Luettavissa: https://terokarvinen.com/2021/two-machine-virtual-network-with-debian-11-bullseye-and-vagrant/. Luettu: 5.11.2024.

Karvinen, T. 28.3.2023. Salt Vagrant - automatically provision one master and two slaves. Luettavissa: https://terokarvinen.com/2023/salt-vagrant/#infra-as-code---your-wishes-as-a-text-file. Luettu: 5.11.2024.

Karvinen, T. 3.4.2024. Hello Salt Infra-as-Code. Luettavissa: https://terokarvinen.com/2024/hello-salt-infra-as-code/. Luettu: 5.11.2024.

Karvinen, T. 10.5.2024. Palvelinten Hallinta. Luettavissa: https://terokarvinen.com/palvelinten-hallinta/. Luettu: 5.11.2024.

Kolade, C. 9.8.2022. Command Line Commands – CLI Tutorial. Luettavissa: https://www.freecodecamp.org/news/command-line-commands-cli-tutorial/. Luettu: 5.11.2024.

LeeviRaussi 29.10.2024. h1 Viisikko. Luettavissa: https://github.com/LeeviRaussi/Palvelinten_hallinta/blob/main/h1_Viisikko.md. Luettu: 4.11.2024.

VMware, Inc. 2024. Salt overview. Luettavissa: https://docs.saltproject.io/salt/user-guide/en/latest/topics/overview.html#rules-of-yaml. Luettu: 4.11.2024.
