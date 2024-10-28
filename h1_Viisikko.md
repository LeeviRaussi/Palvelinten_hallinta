# h1 Viisikko

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

## x) Tiivistelmät

### Run Salt Command Locally (Karvinen 28.10.2021)

Karvinen (28.10.2021) esittelee artikkelissaan viisi tärkeintä tilafunktiota, joita käytetään Saltin hallinnoimiseen: `pkg`, `file`, `service`, `user` ja `cmd`. Vaikka Saltia käytetään normaalisti hallinnoimaan useita verkossa olevia orjakoneita, näitä komentoja voi käyttää myös paikallisesti harjoittelemiseen ja toiminnan testaamiseen. Lisäksi kaikki tilafunktiot toimivat niin Linuxissa kuin myös Windowsissakin.

### Salt Quickstart – Salt Stack Master and Slave on Ubuntu Linux (Karvinen 28.3.2018)

Karvinen (28.3.2018) esittelee artikkelissaan miten Salt otetaan käyttöön Xubuntu 16.04 käyttöjärjestelmässä, joskin vastaavia komentoja voinee hyödyntää myös muissa käyttöjärjestelmissä. Keskeistä Saltin asentamisessa on, että herrana toimivan koneen osoite on orjien tiedossa, ja kommunikointi tapahtuu porttien 4505 ja 4506 kautta, jotka tulee avata palomuurissa. Herran ei tarvitse tietää orjien osoitteita, mutta jokaisella orjalla tulee olla yksilöivä nimi (id), jotta niitä pystytään komentamaan.

### Raportin kirjoittaminen (Karvinen 4.6.2006)

Karvinen (4.6.2006) esittelee artikkelissaan, miten teknisten testien raportoinnin tulisi tapahtua. Raportointi tulisi tehdä täsmällisesti, jotta kaikki työvaiheet pystytään tarvittaessa toistamaan mahdollisten ongelmien selvittämiseksi. Tämä tarkoittaa myös sitä, että mahdolliset ongelmat ja ratkaisut niihin tulee kertoa rehellisesti. Raporttia ei siis saa hienostella oikomalla mahdollisia ongelmia tai muutenkaan sepittää asioita tehdyiksi, vaikkei niitä todellisuudessa ole tehty. Raportti tulisi myös kirjoittaa käyttäen hyvää kieliasua unohtamatta lähdeviittauksia. Näin taataan, että raportti on helppolukuinen kaikille kuin myös ettei kirjoittaja ota kunniaa asioista, jotka ovat todellisuudessa jonkun toisen tekemiä.

## a) Debian 12 Bookworm asennus

Seurasin "Linux Palvelimet" kurssilla kirjoittamaani raporttia Linuxin asentamisesta (ks. LeeviRaussi 28.8.2024), enkä kohdannut minkäänlaisia ongelmia luodessani uuden virtuaalikoneen, johon asensin Debian 12 Bookwormin käyttöjärjestelmäksi. Virtuaalikoneelle annoin käyttöön 8 gigaa muistia, 60 gigaa kovalevytilaa ja 4 prosessoriydintä.

## Lähdeluettelo

Karvinen, T. 4.6.2006. Raportin kirjoittaminen. Luettavissa: https://terokarvinen.com/2006/06/04/raportin-kirjoittaminen-4/. Luettu: 28.10.2024.

Karvinen, T. 28.3.2018. Salt Quickstart – Salt Stack Master and Slave on Ubuntu Linux. Luettavissa: https://terokarvinen.com/2018/03/28/salt-quickstart-salt-stack-master-and-slave-on-ubuntu-linux/. Luettu: 29.10.2024.

Karvinen, T. 28.10.2021. Run Salt Command Locally. Luettavissa: https://terokarvinen.com/2021/salt-run-command-locally/. Luettu: 29.10.2024.

LeeviRaussi 28.8.2024. h1 Oma Linux. Luettavissa: https://github.com/LeeviRaussi/linux-palvelimet/blob/main/h1_Oma_Linux.md. Luettu: 28.10.2024.
