# h2 Infra-as-code

> ## Tiivistelmä
>
> Raportissa esitetään ensin tiivistelmät viidestä artikkelista. Tämän jälkeen Windows 11 koneelle asennetaan Vagrant-ohjelmisto, jolla luodaan kaksi Debian 12 Bookworm pohjaista virtuaalikonetta, joiden välille rakennetaan Saltin avulla herra-orja arkkitehtuuri.

> ### Käytetty laitteisto
>
> Lenovo Ideapad Pro 5 14APH8 kannettava tietokone
> - Prosessori: AMD Ryzen 7 7840HS
> - GPU: Radeon 780M Graphics (prosessoriin integroitu)
> - RAM: 32 GB, 6400 MT/s
> - OS: Windows 11 Home 23H2
> - Näytön resoluutio: 2880x1800 (175% skaalaus)
> - SSD: 745/951 GB vapaana

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

## d) Herra-orja verkossa

Hyödynsin Salt herra-orja arkkitehtuurin asentamisessa edellisellä viikolla kirjoittamaani raporttia (ks. LeeviRaussi 29.10.2024).

### Salt-master

*0:21*

Päätin tehtä edellä luomastani t001-virtuaalikoneesta arkkitehtuurin herran, joten kirjauduin järjestelmään komennolla `vagrant ssh t001`. Ajoin ensin komennon `sudo mkdir -p /etc/apt/keyrings`, jota seurasin komennolla `sudo curl -fsSL -o /etc/apt/keyrings/salt-archive-keyring-2023.gpg https://repo.saltproject.io/salt/py3/debian/12/amd64/SALT-PROJECT-GPG-PUBKEY-2023.gpg`. Sain kuitenkin virheilmoituksen, ettei kyseistä komentoa löytynyt, joten asensin sen komennoilla `sudo apt-get update` ja `sudo apt-get install curl`. Ajoin sitten uudelleen edellä olleen `curl`-komennon, mutta sain tällä kertaa ilmoituksen, ettei osoitetta repo.saltproject.io pystytty selvittämään. `curl www.google.com` toimi kuitenkin kuitenkin ongelmitta, joten lähdin tarkastelemaan virheellistä osoitetta tarkemmin. Googlen kautta löysin Majeedin (21.3.2024) artikkelin aiheeseen liittyen ja testasin edellä olevassa komennossa olevaa osoitetta suoraan selaimeen, mutten onnistunut muodostamaan yhteyttä. Pohdittuani asiaa päätin tarkistaa edellisen viikon tehtävissä olleen linkin Saltin asennukseen (ks. VMware 2024b) ja huomasin, että asennuksessa käytettävän sivun osoite oli erilainen. Testasin osoitetta selaimella ja sivu latasi suoraan koneelleni julkisen avaimen. Päätin tämän myötä noudattaa tässä kohtaa VMwaren (2024b) asennusohjeita edellisessä raportissani käyttämieni ohjeiden sijasta (ks. LeeviRaussi 29.10.2024), ja ajoin komennot `curl -fsSL https://packages.broadcom.com/artifactory/api/security/keypair/SaltProjectKey/public | sudo tee /etc/apt/keyrings/salt-archive-keyring.pgp` ja `curl -fsSL https://github.com/saltstack/salt-install-guide/releases/latest/download/salt.sources | sudo tee /etc/apt/sources.list.d/salt.sources` ongelmitta. Seurasin tätä päivittämällä saatavilla olevat paketit komennolla `sudo apt-get update`, jonka jälkeen asensin salt-masterin komennolla `sudo apt-get install salt-master`. Lainasin edellisestä raportistani komennon `sudo systemctl enable salt-master && sudo systemctl start salt-master`, jolla laitoin herran päälle ja käynnistin sen uudelleen. Lopuksi varmistin vielä komennolla `sudo salt-call --version`, että olin asentanut Saltin onnistuneesti, mikä piti paikkansa (ks. ensimmäinen kuva alla). Asensin tässä yhteydessä virtuaalikoneelle myös Micro-editorin (`sudo apt-get install micro`) sekä ufw-palomuurin, josta avasin portit 4505 ja 4506 herra-orja arkkitehtuuria varten sekä portin 22 SSH-yhteyttä varten (ks. jälkimmäinen kuva komennoista). Lopuksi suljin SSH-yhteyden komennolla `exit`.

![salt-master install](https://github.com/user-attachments/assets/8902218b-afe0-4032-878a-a3457c34328c)

![master ufw](https://github.com/user-attachments/assets/da9dfb94-6e01-4947-81c0-ef85e281d80c)

### Salt-minion

*0:39*

Siirryin sitten komennolla `vagrant ssh t002` toiselle virtuaalikoneelleni, josta aioin rakentaa arkkitehtuurin orjan. Salt-masterin asennusprosessista viisastuneena aloitin päivittämällä saatavilla olevat paketit (`sudo apt-get update`) ja asentamalla curlin (`sudo apt-get install curl`). Seurasin tätä komentosarjalla
```
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://packages.broadcom.com/artifactory/api/security/keypair/SaltProjectKey/public | sudo tee /etc/apt/keyrings/salt-archive-keyring.pgp
curl -fsSL https://github.com/saltstack/salt-install-guide/releases/latest/download/salt.sources | sudo tee /etc/apt/sources.list.d/salt.sources
```
ilman ongelmia. Päivitin jälleen saatavilla olevat paketit (`sudo apt-get update`), minkä jälkeen asensin salt-minionin komennolla `sudo apt-get install salt-minion`. Poimin edellisestä raportistani (ks. LeeviRaussi 29.10.2024) komennon `sudo systemctl enable salt-minion && sudo systemctl start salt-minion`, jolla laitoin salt-minionin päälle ja käynnistin sen uudelleen. Varmistin asennuksen vielä komennolla `sudo salt-call --version` (ks. alla oleva kuva). Asensin sitten tällekin koneelle Micro-editorin ja ufw-palomuurin tehden samat asetukset kuin toisellekin koneelle.

![salt-minion install](https://github.com/user-attachments/assets/f22d8cf2-eccf-4fa5-8642-1f28f3f6d28a)

### Käskytys

*0:46*

Tehtävän lopuksi otin jälleen mallia edellisestä raportistani (ks. LeeviRaussi 29.10.2024) ja ajoin komennon `sudoedit /etc/salt/minion`, jotta pääsin muokkaamaan orjan asetuksia. Asetin laitteen id:ksi "orjat002" sekä herran osoitteeksi aiemmin määritellyn IP-osoitteen 192.168.88.101 (ks. ensimmäinen kuva alla). Käynnistin salt-minionin uudelleen komennolla `sudo systemctl restart salt-minion.service`, minkä jälkeen siirryin takaisin koneen t001 puolelle. Hyväksyin orjan avaimen komennolla `sudo salt-key -A`, minkä jälkeen käskytin orjaa onnistuneesti herra-koneelta käsin komennolla `sudo salt '*' cmd.run 'whoami'` (ks. jälkimmäinen kuva alla).

![salt-minion settings](https://github.com/user-attachments/assets/0feb946f-bd59-4042-a6e7-43f10bd0fb7e)

![master-slave commands](https://github.com/user-attachments/assets/cf859ac4-280a-4165-8992-3a5b92866c2e)

## e) Hei infrakoodi!

*0:50*

Infrakoodin kirjoittamiseen otin avukseni Karvisen (3.4.2024) artikkelin aiheesta. Herrallani (t001) ajoin komennon `sudo mkdir -p /srv/salt/hello/`, jolla loin sekä salt-hakemiston, johon kaikki orjille asennettevat moduulit tulevat, että ensimmäisen moduulin "hello". Siirryin sitten kyseiseen hakemistoon komennolla `cd /srv/salt/hello/`. Loin kooditiedoston komennolla `sudoedit init.sls`, johon kirjoitin yksinkertaisen koodin varmistaa, että hakemistossa /tmp/ on tiedosto "helloleevi". Ajoin sitten moduulin komennolla `sudo salt-call --local state.apply hello`, mikä näytti onnistuneen. Varmistin tuloksen vielä komennolla `ls /tmp/`, enkä kohdannut tässäkään ongelmia (ks. alla oleva kuva kokonaisuudesta, joskin komennot `cd` ja `sudoedit` mysteerisesti puuttuvat välistä).

![salt-call hello](https://github.com/user-attachments/assets/96d30667-0bf7-42be-b7d0-43b905574d44)

## f) sls-tiedosto verkon yli

*0:52*

Lähdin kokeilemaan suoraviivaista ratkaisua tehtävään hyödyntäen edellä verkon yli ajamaani `whoami`-komennon runkoa muuttaen nyt komennon muotoon `sudo salt '*' state.apply hello` ja ajaen sen herralla (t001). Kone orjat002 antoi ilmoituksen, että se oli onnistuneesti saanut suoritettua moduulin (ks. ensimmäinen kuva alla) ja varmistin tämän toisella komentorivi-ikkunalla avaamastani koneen t002 näkymästä komennon `ls /tmp/` avulla (ks. jälkimmäinen kuva alla).

![network state apply](https://github.com/user-attachments/assets/1882e825-39a9-4492-884e-715c3fa10ed0)

![network ls](https://github.com/user-attachments/assets/0fac90f6-094e-40b9-b810-66db8ff5fd79)

## g) Useampi tilafunktio

*0:53*

Lähdin muokkaamaan edellä luomaani hello-moduulin init.sls-tiedostoa aikeenani lisätä siihen toinen tilafunktio. Lisäsin tilafunktion user.present, jonka tehtävä on tarkistaa, että käyttöjärjestelmästä löytyy käyttäjä "herrat001" (ks. ensimmäinen alla oleva kuva). Tämän jälkeen ajoin jälleen komennon `sudo salt '*' state.apply hello`, joka näytti, että hakemistossa /tmp/ oli jo olemassa tiedosto "helloleevi", minkä lisäksi käyttäjä "herrat001" luotiin myös, koska tätä ei ollut vielä olemassa (ks. toinen kuva alla). Orjalla ajettu komento `ls /home/` näytti, että uudelle käyttäjälle oli luotu oma kotihakemisto, minkä lisäksi ajoin vielä komennon `cat /etc/passwd | grep /home/` (Ondara 12.4.2024) nähdäkseni tiedot järjestelmän käyttäjistä (ks. kolmas alla oleva kuva). Ajoin herralla komennon vielä kahdesti osoittaakseni, ettei erikoisempia muutoksia enää synny, eli luotu sls-tiedosto on idempotentti (ks. alimmainen kuva alla).

![multi tila](https://github.com/user-attachments/assets/032441e0-5303-4349-8a30-53b37241406d)

![multi tila first](https://github.com/user-attachments/assets/2cf53275-e43d-4fd3-85c7-f97c023c9270)

![ls multi tila](https://github.com/user-attachments/assets/73d06f5d-5105-4a85-a969-af3ba0808121)

![multi tila idempotency](https://github.com/user-attachments/assets/5d36a862-ef3e-4aeb-9071-4128f757fb29)

*0:56*

Pienenä valmisteluna tehtävää h) varten loin vielä uuden moduulin "poistahello", minkä tarkoituksena on kumota kaikki asiat, mitä moduuli "hello" tekee. Loin uuden hakemiston komennolla `sudo mkdir -p /srv/salt/poistahello/`, johon loin `sudoedit init.sls` komennolla hello-moduulia muistuttavan sls-tiedoston, mutta tilojen present muodot muutin absent:iksi. Ajoin tämän jälkeen komennon `sudo salt '*' state.apply poistahello`, joka näytti suorittavan kaiken onnistuneesti (ks. ensimmäinen kuva alla). Tarkastaessani asian orjakoneelta t002 huomasin, että kotihakemisto käyttäjälle "herrat001" oli jäänyt olemaan, vaikka selvästi käyttäjä oli järjestelmästä poistettu (ks. toinen kuva alla). Muokkasin poistahellon sls-tiedostoa siten, että lisäsin siihen käyttäjän kotihakemiston poiston (ks. kolmas kuva alla), minkä jälkeen ajoin taas komennon `sudo salt '*' state.apply poistahello` onnistuneesti (ks. neljäs kuva alla). Tämän myötä poistetun käyttäjän kotihakemisto myös poistui järjestelmästä (ks. viimeinen kuva alla).

![poistahello creation](https://github.com/user-attachments/assets/b66dbe4e-52b3-4307-9197-e0ed9f011f06)

![poistahello ls](https://github.com/user-attachments/assets/16679d9c-960a-430a-b305-f5612d380980)

![poistahello yaml](https://github.com/user-attachments/assets/336d8c65-c49f-401c-9b32-0291d27a5a84)

![poistahello modified](https://github.com/user-attachments/assets/958fc44c-b4ee-41c5-af17-7c84611daf2d)

![poistahello modified ls](https://github.com/user-attachments/assets/6bded491-3a78-4408-bd94-6a33e3cc6587)

## h) Top file

*0:58*

Aloitin luomalla moduulille nsnake hakemiston komennolla `sudo mkdir nsnake` ollessani hakemistossa /srv/salt/. Siirryin sitten kyseiseen hakemistoon luomaan sls-tiedoston, jolla asennetaan nsnake-ohjelma (ks. alla oleva kuva), minkä jälkeen palasin takaisin hakemistoon /srv/salt/.

![nsnake moduuli](https://github.com/user-attachments/assets/0ffc8753-1bb9-4b38-a8cf-18df29bd2e39)

Loin seuraavaksi top.sls-tiedoston komennolla `sudoedit top.sls`, johon määritin suoritettaviksi moduuleiksi hellon ja nsnaken (ks. ensimmäinen alla oleva kuva). Tämän jälkeen ajoin herralla komennon `sudo salt '*' state.apply`, jonka tarkoitus oli suorittaa moduulit orjalla, mikä onnistuikin (ks. kaksi alinta kuvaa alla).

![top](https://github.com/user-attachments/assets/40ca1288-478b-4750-a2a4-ad6df5b4fe80)

![top apply 1](https://github.com/user-attachments/assets/42d715b5-bd61-4704-a8fc-4a7801d5aac5)

![top apply 2](https://github.com/user-attachments/assets/ab3662bb-01df-4587-8a34-c00df2812e28)

Tarkistaessani asian orjalla (t002) näin, että helloleevi-tiedosto ja käyttäjä herrat001 oli luotu onnistuneesti (ks. ensimmäinen kuva alla), sekä nsnake asennus toimi myös (ks. jälkimmäinen kuva alla).

![top check](https://github.com/user-attachments/assets/00119094-1504-43e1-a7ac-8f5340e8f759)

![nsnake ajo](https://github.com/user-attachments/assets/0e421b5b-5ca7-445f-9339-bd723cf36b91)

*1:04*

Valmis.

## Lähdeluettelo

Hashicorp 2024a. Install Vagrant. Luettavissa: https://developer.hashicorp.com/vagrant/install. Luettu: 5.11.2024.

Hashicorp 2024b. Install Vagrant. Luettavissa: https://developer.hashicorp.com/vagrant/tutorials/getting-started/getting-started-install. Luettu: 5.11.2024.

Karvinen 28.3.2018. Salt Quickstart – Salt Stack Master and Slave on Ubuntu Linux. Luettavissa: https://terokarvinen.com/2018/03/28/salt-quickstart-salt-stack-master-and-slave-on-ubuntu-linux/. Luettu: 6.11.2024.

Karvinen, T. 4.11.2021. Two Machine Virtual Network With Debian 11 Bullseye and Vagrant. Luettavissa: https://terokarvinen.com/2021/two-machine-virtual-network-with-debian-11-bullseye-and-vagrant/. Luettu: 6.11.2024.

Karvinen, T. 28.3.2023. Salt Vagrant - automatically provision one master and two slaves. Luettavissa: https://terokarvinen.com/2023/salt-vagrant/#infra-as-code---your-wishes-as-a-text-file. Luettu: 6.11.2024.

Karvinen, T. 3.4.2024. Hello Salt Infra-as-Code. Luettavissa: https://terokarvinen.com/2024/hello-salt-infra-as-code/. Luettu: 6.11.2024.

Karvinen, T. 10.5.2024. Palvelinten Hallinta. Luettavissa: https://terokarvinen.com/palvelinten-hallinta/. Luettu: 6.11.2024.

Kolade, C. 9.8.2022. Command Line Commands – CLI Tutorial. Luettavissa: https://www.freecodecamp.org/news/command-line-commands-cli-tutorial/. Luettu: 5.11.2024.

LeeviRaussi 29.10.2024. h1 Viisikko. Luettavissa: https://github.com/LeeviRaussi/Palvelinten_hallinta/blob/main/h1_Viisikko.md. Luettu: 4.11.2024.

Majeed, U. 21.3.2024. cURL Error 6: Could Not Resolve Host. Luettavissa: https://sendlayer.com/docs/curl-error-6-could-not-resolve-host/. Luettu: 6.11.2024.

Ondara, W. 12.4.2024. How to List Users in Linux [With Examples]. Luettavissa: https://www.cherryservers.com/blog/linux-list-users. Luettu: 6.11.2024.

VMware, Inc. 2024a. Salt overview. Luettavissa: https://docs.saltproject.io/salt/user-guide/en/latest/topics/overview.html#rules-of-yaml. Luettu: 4.11.2024.

Vmware, Inc. 2024b. Linux (DEB). Luettavissa: https://docs.saltproject.io/salt/install-guide/en/latest/topics/install-by-operating-system/linux-deb.html#install-deb. Luettu: 6.11.2024.
