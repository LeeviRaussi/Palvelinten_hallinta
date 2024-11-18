# h3 Demoni

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
> - SSD: 740/951 GB vapaana

## x) Tiivistelmät

### Pkg-File-Service – Control Daemons with Salt – Change SSH Server Port (Karvinen 2018)

Karvinen (2018) esittelee artikkelissaan Package-file-service demonien hallinnointimetodin. Systeemi koostuu kolmesta vaiheesta: ohjelmiston asennuksesta (package), laitteistomääritysten laatiminen (file) ja uusien asetusten käyttöönotto uudelleenkäynnistyksen avulla (service). Järjestelmä hyödyntää siis kolme Saltin tilafunktiota, joiden sisältö määritellään moduulin sls-tiedostossa. Esimerkkinä Karvinen näyttää artikkelissaan miten SSH:n käyttöön avataan uusi portti.

### Komento `sudo salt-call --local sys.state_doc pkg` (Salt 2024)

Saltin tilafunktiota `pkg` käytetään hallinnoimaan ohjelmistojen asennuksiin liittyviä asioita. Tilafunktio hyödyntää käyttöjärjestelmän omaa paketinhallintajärjestelmää (esim. yum tai apt-get) asennusten tekemiseen. Komento `pkg.installed` huolehtii siitä, että ohjelmisto löytyy kohteesta, eli se tarvittaessa asentaa ohjelmiston mikäli sitä ei löydy ennestään. Komento `pkg.purged` huolehtii, ettei ohjelmisto ole asennettuna ja tarvittaessa poistaa asennuksen yhdessä asetustiedostojen kanssa. Tilamodulia `pkgs` voi hyödyntää edellisten komentojen yhteydessä tekemään saman toiminnon kerralla useammalle paketille.

### Komento `sudo salt-call --local sys.state_doc file` (Salt 2024)

Saltin tilafunktiota `file` käytetään tiedostojen ja hakemistojen muokkaamiseen. Komento `file.managed` pitää huolen, että tiedosto löytyy orjakoneelta määrätystä hakemistosta ja se tarvittaessa ladataan sinne herralta, orjan toisesta hakemistosta, tai HTTP- tai FTP-palvelimelta. Vastavuoroisesti komento `file.absent` poistaa tiedoston tai hakemiston määrätystä sijainnista. Komento `file.symlink` luo symbolisen linkin, eli määrätty tiedosto osoittaa johonkin toiseen sijaintiin.

### sudo salt-call --local sys.state_doc service (Salt 2024)

Saltin tilafunktiota `service` käytetään demonien ja palveluiden käynnissäolon ja uudelleenkäynnistämisen hallinnoimiseen. Komento `service.running` tarkistaa, että kohde on käynnissä ja tarvittaessa käynnistää sen. Vastavuoroisesti komento `service.dead` tarvittaessa sammuttaa käynnissä olevan kohteen. Tilamoduulia `enable` voi hyödyntää automatisoimaan demonin käynnistyminen järjestelmän käynnistymisen yhteydessä.

## Apache easy mode

### Manuaalinen asennus

*0:00*

Jatkoin työskentelyä edellisessä raportissani (ks. LeeviRaussi 2024b) Vagrantilla luomieni virtuaalikoneiden t001 (herra) ja t002 (orja) kanssa. Lähdin asentamaan t001:lle Apachea manuaalisesti, jotta voisin tämän jälkeen automatisoida kyseisellä koneella tehdyt välivaiheet sls-tiedostoksi orjia varten. Hyödynsin asennusprosessissa "Linux palvelimet" kurssilla kirjoittamaani raporttia Apachen asentamisesta (ks. LeeviRaussi 2024a).

Päivitin ensiksi saatavilla olevat paketit komennolla `sudo apt-get update`, minkä jälkeen asensin Apachen komennolla `sudo apt-get -y install apache2`. Asennuksen päätyttyä muokkasin Apachen oletussivulle tekstin "Oletussivu" komennolla `echo "Oletussivu"|sudo tee /var/www/html/index.html`. Tarkistin tämän jälkeen, että kaikki oli mennyt hyvin komennolla `curl localhost`, joka näytti, että niin asennus kuin sivun muokkaaminen oli tehty onnistuneesti.

![Apache installed](https://github.com/user-attachments/assets/0fd31170-bcf6-4605-a3e1-07181d36cf13)

### Automatisointi

*0:02*

Aloitin luomalla uuden modulin Apachea varten komennolla `sudo mkdir -p /srv/salt/apache/`. Tämän jälkeen siirryin komennolla `cd /srv/salt/apache/` kyseiseen hakemistoon ja loin sinne komennolla `sudoedit init.sls` tarvittavan sls-tiedoston. Ajatukseni oli hyödyntää pkg.installed ja file.managed tilafunktioita ensin Apachen asennuksen varmistamiseen ja sitten Apachen oletussivun muokkaukseen. Selatessani file-tilafunktion manuaalia (Salt 2024), olin huomannut "source" ehdon, jonka ajattelin tarkoittavan sourcena olevan tiedoston kopioimista nimenä olevana hakemistoon. En ollut kuitenkaan varma, olinko ymmärtänyt täysin oikein, miten source pitäisi määrittää, mutta päätin yrittää kirjoittamallani sls-tiedostolla.

![apache init og](https://github.com/user-attachments/assets/030a852b-7378-4c99-9207-ca990b3942e2)

Ajoin komennon `sudo salt-call --local state.apply apache`, joka näytti, että Apachen asennus löytyi onnistuneesti, mutta oletussivun tiedoston tarkistus epäonnistui.

![apache init og error](https://github.com/user-attachments/assets/1a28d1e9-6f49-4610-b226-b5547e84ce06)

Pyöriteltyäni file-tilafunktion manuaalia (Salt 2024) edessäni jonkin aikaa päädyin ajatukseen, että mahdollisesti virheilmoituksessa mainittu "base" viittaisi hakemistoon /srv/salt, eli kyseiseen hakemistoon, tai pikemminkin moduliin, voisi liittää tarvittavia tiedostoja. Edellä manuaalisesti luodun tiedoston kopioiminen modulin hakemistoon ei kuitenkaan tuntunut käytännölliseltä, joten aloin etsimään vaihtoehtoista ratkaisua. Löysin Karvisen (2021) artikkelista, miten file-tilafunktioon yhdistetään sisällön kirjoittaminen tiedostoon ja yhdessä file-tilafunktion manuaalin (Salt 2024) kanssa muokkasin sls-tiedostoa.

![apache init edit](https://github.com/user-attachments/assets/8e31a30a-6612-4f6d-9abc-d814b14f771c)

Tällä kertaa komento `sudo salt-call --local state.apply apache` suoritettiin onnistuneesti.

![apache init edit run](https://github.com/user-attachments/assets/c49d8fef-051f-458d-ac74-8db2a567a4a5)

Ajoin seuraavaksi komennon `sudo salt '*' state.apply apache` tarkoituksenani tehdä koko homma orjana olevalla t002:lla, mutta homma katkesi aikaviiveeseen.

![salt time-out](https://github.com/user-attachments/assets/44dad06e-1dc9-448b-83f7-68e405ac038a)

Etsin Googlella tietoa, miten --async pitäisi lisätä komentoon, ja päädyin VMwaren (2024) ohjeisiin aiheeseen liittyen. Ajoin komennon uudelleen muodossa `sudo salt '*' --async state.apply apache`, mutta tämäkään ei toiminut. Kokeilin sitten edellisestä raportistani komentoa `sudo salt '*' state.apply hello`, joka myös yllätyksekseni ei onnistunut.

![salt time-out hello](https://github.com/user-attachments/assets/a9b11352-b83b-4b4c-9c74-63ee9e714f65)

Kokeilin auttaisiko salt-masterin uudelleenkäynnistys, joten ajoin komennon `sudo systemctl restart salt-master.service` (muokattuna salt-minion muodosta edellisestä raportistani LeeviRaussi 2024b), minkä jälkeen tarkistin oliko uusia avaimia orjilta (komento ` sudo salt-key -A`), jonka jälkeen testasin uudelleen hello-modulin suoritusta, tällä kertaa onnistuneesti. Myös perään ajamani poistahello-moduli (`sudo salt '*' state.apply poistahello`) toimi odotetusti.

![salt state hello](https://github.com/user-attachments/assets/d0175f0c-b9ce-4db5-af24-ccec577e46aa)

Ajoin sitten uudelleen komennon `sudo salt '*' state.apply apache`, ja tällä kertaa sain ilmoituksen, että pakettien asennuksessa oli ongelmia.

![salt time-out apache2](https://github.com/user-attachments/assets/4dcae667-e6ff-4cf8-b066-6b1d8d5cdd9a)

Hetken pohdittuani ajoin ensin orjalta käsin komennon `sudo apt-get update`, jonka jälkeen yritin edellä tekemääni komentoa uudelleen lopputuloksen ollessa sama. Koitin sitten ajaa herralta käsin saman komennon muodossa `sudo salt '*' cmd.run 'apt-get update'` ja perään pakettien päivittämisen (`sudo salt '*' cmd.run 'apt-get upgrade'`).

![salt apt-get update](https://github.com/user-attachments/assets/e44878d3-3be9-4d22-8f02-06f7bb83f10e)

Tämä ei kuitenkaan tuottanut tulosta, vaan samat virheilmoitukset jatkuivat. Lähdin googlettamaan, mikä hommassa oikein on ongelma ja löysin lloydbushin (2024) keskustelun etsimilläni hakusanoilla. Silmiini osui kommentti järjestelmän aikamäärityksistä, mikä oli ollut tietyssä mielessä läsnä myös virheilmoituksissa. Päädyinkin ajamaan komennon `date`.

![date1](https://github.com/user-attachments/assets/2056d1c6-3cf8-49c0-b86e-0b5962b8ba75)

Jostain syystä järjestelmä eli viime viikkoa. Aloin selvittämään, miten saisin korjattua tämän selvän synkronointiongelman ja löysin Zivanovin (2023) artikkelin aiheesta. Artikkelin ohjeiden mukaisesti muutin ensin t001:n aikavyöhykkeeksi Helsingin komennolla `sudo timedatectl set-timezone Europe/Helsinki` ja ajoin uudelleen komennon `date` nähdäkseni korjaantuiko ongelma.

![date helsinki](https://github.com/user-attachments/assets/bab3e8a1-ff4a-46ce-8d1a-3a4d93498209)

Aikavyöhyke oli selvästi muuttunut, mutta kone eli edelleen viime viikkoa. Siirryin seuraavaksi tarkistamaan ntp:n statuksen komennolla `sudo systemctl status ntp` saadakseni ilmoituksen, ettei kyseistä palvelua ole saatavilla. Erikoisesti komento `timedatectl` ilmoitti NTP-palvelun olevan kuitenkin aktiivinen, joskaan ei selvästikään toimiva, koska kello ei ole synkronoituna.

![ntp mia](https://github.com/user-attachments/assets/88cc2cd1-8144-434e-a273-03679a82d7ca)

Zivanovin (2023) artikkelissa mainittiin timesyncd-ohjelma vaihtoehtona ntp:lle, joten asensin sen komennolla `sudo apt-get install systemd-timesyncd`. Melko pitkän asennusprosessin jälkeen laitoin ohjelman käyntiin komennolla `sudo systemctl start systemd-timesyncd` ja tarkistin sen statuksen komennolla `sudo systemctl status systemd-timesyncd`. Nyt viimein kone oli siirtynyt nykyhetkeen.

![time correct](https://github.com/user-attachments/assets/714d98f8-b8b6-4f73-8746-8b5fa2b56593)

Tein samat aikasäädöt myös t002:lla, jottei sekään kärsisi näistä ongelmista. Tämän jälkeen uskaltauduin kokeilemaan taas kerran komentoa `sudo salt '*' state.apply apache`, joka vihdoin onnistui.

![apache init success](https://github.com/user-attachments/assets/7beb9c48-499a-4580-8704-4fa23b63ad33)

Testasin vielä varmuudeksi t002:lla komennon `curl localhost`, joka näytti, että Apachen muokattu oletussivu näkyi.

![t002 localhost](https://github.com/user-attachments/assets/c5d27618-85c0-4fa9-9869-8255789c1429)

## SSHouto

*1:15*

## Lähdeluettelo

Karvinen, T. 2018. Pkg-File-Service – Control Daemons with Salt – Change SSH Server Port. Luettavissa: https://terokarvinen.com/2018/04/03/pkg-file-service-control-daemons-with-salt-change-ssh-server-port/?fromSearch=karvinen%20salt%20ssh. Luettu: 15.11.2024.

Karvinen, T. 2021. Run Salt Command Locally. Luettavissa: https://terokarvinen.com/2021/salt-run-command-locally/. Luettu: 18.11.2024.

LeeviRaussi 2024a. h3 Hello Web Server. Luettavissa: https://github.com/LeeviRaussi/linux-palvelimet/blob/main/h3_Hello_Web_Server.md. Luettu: 18.11.2024.

LeeviRaussi 2024b. h2 Infra-as-code. Luettavissa: https://github.com/LeeviRaussi/Palvelinten_hallinta/blob/main/h2_Infra-as-code.md. Luettu: 18.11.2024.

lloydbush 2024. [Solved] Error updating using apt/apt-get. Luettavissa: https://forums.debian.net/viewtopic.php?t=158481. Luettu: 18.11.2024.

Salt 2024. Salt manual. Debian 12 käyttöjärjestelmällinen kone. Luettu: 15.11.2024.

VMware, Inc 2024. Salt execution framework. Luettavissa: https://docs.saltproject.io/salt/user-guide/en/latest/topics/execution-framework.html. Luettu: 18.11.2024.

Zivanov, S. 2023. How to Set Up Time Synchronization on Debian. Luettavissa: https://phoenixnap.com/kb/debian-time-sync. Luettu: 18.11.2024.
