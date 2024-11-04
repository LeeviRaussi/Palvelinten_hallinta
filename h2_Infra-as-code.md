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
> - SSD: 761/951 GB vapaana
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

## Lähdeluettelo

Karvinen 28.3.2018. Salt Quickstart – Salt Stack Master and Slave on Ubuntu Linux. Luettavissa: https://terokarvinen.com/2018/03/28/salt-quickstart-salt-stack-master-and-slave-on-ubuntu-linux/. Luettu: 5.11.2024.

Karvinen, T. 4.11.2021. Two Machine Virtual Network With Debian 11 Bullseye and Vagrant. Luettavissa: https://terokarvinen.com/2021/two-machine-virtual-network-with-debian-11-bullseye-and-vagrant/. Luettu: 5.11.2024.

Karvinen, T. 28.3.2023. Salt Vagrant - automatically provision one master and two slaves. Luettavissa: https://terokarvinen.com/2023/salt-vagrant/#infra-as-code---your-wishes-as-a-text-file. Luettu: 5.11.2024.

Karvinen, T. 3.4.2024. Hello Salt Infra-as-Code. Luettavissa: https://terokarvinen.com/2024/hello-salt-infra-as-code/. Luettu: 5.11.2024.

Karvinen, T. 10.5.2024. Palvelinten Hallinta. Luettavissa: https://terokarvinen.com/palvelinten-hallinta/. Luettu: 5.11.2024.

LeeviRaussi 29.10.2024. h1 Viisikko. Luettavissa: https://github.com/LeeviRaussi/Palvelinten_hallinta/blob/main/h1_Viisikko.md. Luettu: 4.11.2024.

VMware, Inc. 2024. Salt overview. Luettavissa: https://docs.saltproject.io/salt/user-guide/en/latest/topics/overview.html#rules-of-yaml. Luettu: 4.11.2024.
