# h1 Viisikko

> ## Tiivistelmä
>
> Raportissa esitetään ensin lyhyet tiivistelmät kolmesta Tero Karvisen artikkelista. Tämän jälkeen luodaan uusi Debian 12 käyttöjärjestelmällinen virtuaalikone, johon asennetaan Salt, ja harjoitellaan kaikkein keskeisimpiä tilafunktioiden käyttöä paikallisesti.

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

## b) Salt-asennus

0:00 Lähdin asentamaan Saltia hyödyntämällä VMwaren (2024) ohjeita. Käytin asennuksen ensimmmäisessä vaiheessa tehtävänannon vinkeissä olleita komentoja (Karvinen 5.10.2024), jotka oli tarkistettu toimiviksi. Ajoin komennot `sudo mkdir -p /etc/apt/keyrings`, `sudo curl -fsSL -o /etc/apt/keyrings/salt-archive-keyring-2023.gpg https://repo.saltproject.io/salt/py3/debian/12/amd64/SALT-PROJECT-GPG-PUBKEY-2023.gpg` ja `echo "deb [signed-by=/etc/apt/keyrings/salt-archive-keyring-2023.gpg arch=amd64] https://repo.saltproject.io/salt/py3/debian/12/amd64/latest bookworm main" | sudo tee /etc/apt/sources.list.d/salt.list`, jotka mahdollistivat Saltiin liittyvät lataukset, koska näitä ei ole saatavilla normaalisti `sudo apt-get install [ohjelma]` komennolla. Päivitin seuraavaksi saatavilla olevat paketit `sudo apt-get update`, minkä jälkeen asensin VMwaren (2024) ohjeiden mukaisesti orjan komennolla `sudo apt-get install salt-minion`. Komennolla `sudo systemctl enable salt-minion && sudo systemctl start salt-minion` laitoin orjan päälle, minkä myötä Saltin (salt-minion) asennus tuli valmiiksi. Varmistin tämän vielä komennolla `sudo salt-call --version` (Karvinen 28.10.2021). Asensin vielä lisäksi samassa yhteydessä herran (salt-master) koneelle (asennus `sudo apt-get install salt-master`, käynnistys `sudo systemctl enable salt-master && sudo systemctl start salt-master`), koska VMwaren (2024) ohjeissa tämä oli esitetty suoraviivaisesti ja uskoin tämän olevan aiheellista tulevaisuutta ajatellen. Alla olevasta kuvassa näkyy asennusprosessin viimeiset vaiheet.

![salt asennus](https://github.com/user-attachments/assets/9b914d7f-8c3f-412c-897c-81b36e46803c)

## c) Saltin tilafunktiot

Käytin tilafunktioiden tietojen selvittämiseen Karvisen (28.10.2021) artikkelista löytyvää komentoa, johon putkitin perään esitystavaksi lessin: `sudo salt-call --local sys.state_doc | less`. Pääsin näin lukemaan Saltin manuaalia, jota käytin seuraavien tehtävien vastauksiin.

### pkg

0:05 Saltin pkg-tilafunktiota käytetään hallitsemaan orja-koneella olevia paketteja. Hyvänä esimerkkinä tästä toimivat Karvisen (28.10.2021) artikkelissa esitetyt komennot `sudo salt-call --local -l info state.single pkg.installed tree`, joka asentaa paketin "tree" (ks. ensimmäinen kuva alla), ja `sudo salt-call --local -l info state.single pkg.removed tree`, jolla juuri asennettu paketti saadaan poistettua (ks. jälkimmäinen kuva alla). Alla olevista kuvista näkyy, että prosessissa saadaan tietoon mm. mistä paketista oli kyse, mitä sille tehtiin, mikä versio oli kyseessä, ja montako muutosta koneeseen tehtiin.

![pkg install](https://github.com/user-attachments/assets/ae8773fe-0132-4bc7-8de2-14aa5d1ad418)

![pkg remove](https://github.com/user-attachments/assets/b5ff1dc3-de94-4b98-a4d6-b7be31cc9d17)

### file

0:12 Saltin file-tilafunktiota käytetään tiedostojen manipuloimiseen. Erityisesti komentoa `file.managed` käytetään lataamaan tiedostoja herralta orjalle. Karvisen (28.10.2021) artikkelista löytyvä esimerkkikomento `sudo salt-call --local -l info state.single file.managed /tmp/hellotero` näyttää, kuinka orjalle luodaan hakemistoon /tmp/ hellotero-tiedosto (ks. ensimmäinen alla oleva kuva). Tässä nimenomaan kyseiseen kohteeseen luotiin tämä tiedosto, koska sitä ei ollut vielä olemassa. Lisäehdoilla olisi myös mahdollista suoraan kirjoittaa sisältöä tiedostoon sen luonnin yhteydessä. Vastaavasti `file.absent` komentoa voidaan hyödyntää varmistamaan, ettei jotain tiedostoa löydy kohteesta, ja jos se on olemassa, niin se poistetaan, kuten kävi kun ajoin komennon `sudo salt-call --local -l info state.single file.absent /tmp/hellotero` (ks. jälkimmäinen kuva alla).

![file managed](https://github.com/user-attachments/assets/28bf37ee-103b-4c20-9478-b97ec3a1e95d)

![file absent](https://github.com/user-attachments/assets/b7bda750-218a-4db6-b91a-cb0be7f27957)

### service

0:18 Saltin service-tilafunktiota käytetään ohjelmien käynnistämiseen ja sammuttamiseen. Keskeisenä ovat erityisesti komennot `service.running`, jolla käynnistetään ohjelma, sekä `service.dead`, joka sulkee ohjelman. Karvisen (28.10.2021) artikkelissa olevat esimerkkikomennot `sudo salt-call --local -l info state.single service.running apache2 enable=True` (ks. ensimmäinen kuva alla, käynnistää Apachen) ja  `sudo salt-call --local -l info state.single service.dead apache2 enable=False` (ks. jälkimmäinen kuva alla, sulkee Apachen) kuvaavat hyvin näitä tapahtumia. Mikäli koneelle olisi asennettu Apache, tämä käynnistyisi ensimmäisellä komennolla, mutta nyt saadaan virheilmoitus, kun kyseistä ohjelmaa ei löydy koneelta. Mielenkiintoisesti jälkimmäinen komento tulkitaan onnistuneeksi, vaikkei ohjelmaa tosiaan löydy koneelta. Tämän voisi tulkita niin, että `service.dead` tarkoitus on ainoastaan varmistaa, ettei kyseinen ohjelma ole käynnissä enää.

![service running](https://github.com/user-attachments/assets/c96b995a-56e5-442c-94aa-c630739cff8e)

![service dead](https://github.com/user-attachments/assets/31c916fb-c99a-4b2c-ac9e-f8086dd5df2a)

### user

0:24 Saltin user-tilafunktiota käytetään käyttäjien luomiseen ja poistamiseen. `user.present` komento varmistaa, että kohteena oleva käyttäjä on olemassa ja tarvittaessa luo tällaisen käyttäjän. Vastaavasti `user.absent` varmistaa, ettei haettua käyttäjää ole olemassa ja tarvittaessa poistaa käyttäjän ryhmineen. Karvisen (28.10.2021) artikkelin esimerkkikomennot `sudo salt-call --local -l info state.single user.present terote08` (ks. ensimmäinen kuva alla, käyttäjän luonti) ja  `sudo salt-call --local -l info state.single user.absent terote08` (ks. jälkimmäinen kuva alla, käyttäjän poistaminen) demonstroivat näitä tapahtumia hyvin.

![user present](https://github.com/user-attachments/assets/ad1f9ab9-9a56-4492-b4cb-a325506f0c42)

![user absent](https://github.com/user-attachments/assets/48649267-5f03-433c-8f05-d1e773602d59)

### cmd

0:27 Saltin cmd-tilafunktiota käytetään suorittamaan erinäisiä komentoja. Karvinen (28.10.2021) korostaa artikkelissaan, että muita tilafunktioita tulisi suosia, ja mikäli cmd-tilafunktiota käyttää, tulisi pitää huolta, että kyseessä on idempotentti komento. Karvisen (28.10.2021) artikkelin esimerkkikomento `sudo salt-call --local -l info state.single cmd.run 'touch /tmp/foo' creates="/tmp/foo"` luo hakemistoon /tmp/ foo-tiedoston, mikä olisi voitu tehdä myös `file.managed`-komennolla (ks. kuva alla).

![cmd run](https://github.com/user-attachments/assets/88e472a3-71ee-4790-ab75-34f7c2d27c54)

## d) Idempotentti

0:30 Idempontenssisen komennon idea on, että sen tuottama lopputulos on aina sama. Edellä olevista tilafunktioista hyvänä esimerkkinä idempotentti-komennosta on `file.managed`-komento. Antaessani komennon `sudo salt-call --local -l info state.single file.managed /tmp/helloleevi` komento `file.managed` tarkistaa onko tiedostoa "helloleevi" olemassa hakemistossa /tmp/, ja koska ei ole, se luodaan (ks. alla oleva kuva).

![idempotenssi 1](https://github.com/user-attachments/assets/49644acb-37b4-42c3-ad2c-952f2410cee8)

Ajaessani saman komennon uudelleen, sain tiedon, että kyseinen tiedosto on jo olemassa, joten muutoksia ei tarvinnut tehdä (ks. alla oleva kuva).

![idempotenssi 2](https://github.com/user-attachments/assets/7b2038a8-e03c-4232-8eb2-cce40d2e376a)

Komento ei myöskään koske tiedoston sisältöön, vaan se ainoastaan tarkistaa, onko tiedosto olemassa. Testasin tämän kirjoittamalla helloleevi-tiedostoon tekstiä ja ajamalla edellä olleen `file.managed`-komennon uudelleen. Alla oleva kuva näyttää, miten komento ainoastaan tarkastaa tiedoston olemassaolon, muttei välitä sen sisällöstä. Kyseessä on siis idempotenttinen komento.

![idempotenssi 3](https://github.com/user-attachments/assets/70a45527-efb9-4bf3-90c6-30b23fbdff79)

## e) Herra-orja

0:31 Koska olin jo tehtävässä b) asentanut salt-masterin, siirryin suoraan toteuttamaan Karvisen (28.3.2018) artikkelin esimerkkejä näyttääkseni, että herra-orja arkkitehtuuri toimii paikallisesti. Käytin ensin komentoa `sudoedit /etc/salt/minion`, jolla annoin orjan id:ksi "localorja" sekä määritin herran sijainniksi "localhost". Käynnistin tämän jälkeen orjan uudelleen komennolla `sudo systemctl restart salt-minion.service`. Seuraavaksi herran tuli hyväksyä avain uudelta orjaltaan, mikä tapahtui komennon `sudo salt-key -A` avulla. Tämän jälkeen herra pystyi ajamaan komentoja orjallaan, joten ajoin komennon `sudo salt '*' cmd.run 'whoami'` selvittääkseni, kuka on orjan käyttäjänä. Alla oleva kuva näyttää kaikki suoritetut komennot onnistuneina.

![herra-orja](https://github.com/user-attachments/assets/92e4ab4c-2b64-4411-ab2e-166b49b75646)

## Lähdeluettelo

Karvinen, T. 4.6.2006. Raportin kirjoittaminen. Luettavissa: https://terokarvinen.com/2006/06/04/raportin-kirjoittaminen-4/. Luettu: 28.10.2024.

Karvinen, T. 28.3.2018. Salt Quickstart – Salt Stack Master and Slave on Ubuntu Linux. Luettavissa: https://terokarvinen.com/2018/03/28/salt-quickstart-salt-stack-master-and-slave-on-ubuntu-linux/. Luettu: 29.10.2024.

Karvinen, T. 28.10.2021. Run Salt Command Locally. Luettavissa: https://terokarvinen.com/2021/salt-run-command-locally/. Luettu: 29.10.2024.

Karvinen, T. 5.10.2024. Palvelinten Hallinta. Luettavissa: https://terokarvinen.com/palvelinten-hallinta/. Luettu: 29.10.2024.

LeeviRaussi 28.8.2024. h1 Oma Linux. Luettavissa: https://github.com/LeeviRaussi/linux-palvelimet/blob/main/h1_Oma_Linux.md. Luettu: 28.10.2024.

VMware, Inc. 2024. Debian - Salt install guide. Luettavissa: https://docs.saltproject.io/salt/install-guide/en/latest/topics/install-by-operating-system/debian.html#install-salt-on-debian-12-bookworm-amd64. Luettu: 29.10.2024.
