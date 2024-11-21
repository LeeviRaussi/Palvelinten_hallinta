# h3 Demoni

> ## Tiivistelmä
>
> Raportissa tiivistetään ensin Saltin package-file-service hallinnointiin liittyviä asioita. Tämän jälkeen tehdään ensin manuaalisesti Apachen asennus ja konfugirointi, jonka jälkeen prosessi automatisoidaan. Myös package-file-service hallinnoinnin käyttöä harjoitetaan SSH:n avulla.

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

Jatkoin työskentelyä edellisessä raportissani (ks. LeeviRaussi 2024c) Vagrantilla luomieni virtuaalikoneiden t001 (herra) ja t002 (orja) kanssa. Lähdin asentamaan t001:lle Apachea manuaalisesti, jotta voisin tämän jälkeen automatisoida kyseisellä koneella tehdyt välivaiheet sls-tiedostoksi orjia varten. Hyödynsin asennusprosessissa "Linux palvelimet" kurssilla kirjoittamaani raporttia Apachen asentamisesta (ks. LeeviRaussi 2024a).

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

Etsin Googlella tietoa, miten --async pitäisi lisätä komentoon, ja päädyin VMwaren (2024a) ohjeisiin aiheeseen liittyen. Ajoin komennon uudelleen muodossa `sudo salt '*' --async state.apply apache`, mutta tämäkään ei toiminut. Kokeilin sitten edellisestä raportistani komentoa `sudo salt '*' state.apply hello`, joka myös yllätyksekseni ei onnistunut.

![salt time-out hello](https://github.com/user-attachments/assets/a9b11352-b83b-4b4c-9c74-63ee9e714f65)

Kokeilin auttaisiko salt-masterin uudelleenkäynnistys, joten ajoin komennon `sudo systemctl restart salt-master.service` (muokattuna salt-minion muodosta edellisestä raportistani LeeviRaussi 2024c), minkä jälkeen tarkistin oliko uusia avaimia orjilta (komento ` sudo salt-key -A`), jonka jälkeen testasin uudelleen hello-modulin suoritusta, tällä kertaa onnistuneesti. Myös perään ajamani poistahello-moduli (`sudo salt '*' state.apply poistahello`) toimi odotetusti.

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

Käytin tehtävän pohjana Karvisen (2018) artikkelia, koska tästä löytyi mielestäni selkeän tuntuiset ohjeet. Aloitin tehtävän tarkastamalla komennolla `ls /etc/ssh/` kohteena olevan hakemiston sisällön. SSH oli selvästi asennettuna jo koneelle, koska Vagrant-yhteys itsessään luodaan nimenomaan sillä. Hakemistosta löytyikin jo sshd_config-tiedosto, jonka sisältöä tarkastelin komennolla `cat /etc/ssh/sshd_config`. Ajattelin muokata tiedostoa ja ajoin komennon `sudoedit /etc/ssh/sshd_config`, mutta pysähdyin tässä kohtaa. Olin ajatellut tekeväni asiat tehtävän a) kaltaisesti ensin manuaalisesti, jonka jälkeen automatisoisin asiat, mutta toisaalta SSH oli jo asennettuna sekä sshd_config-tiedosto jo olemassa, joten manuaalinen tekeminen oli tietyssä mielessä jo tehty. Tarkastellessani Karvisen (2018) artikkelia enemmän päätin suoraan luoda modulin SSHd:tä varten.

Loin modulin sshd komennolla `sudo mkdir -p /srv/salt/sshd/` ja siirryin hakemiston sisälle komennolla `cd /srv/salt/sshd/`. Loin tänne ensiksi sshd_config-tiedoston, johon kopioin sisällön Karvisen (2018) artikkelista, joskin lisäsin myös portin 22 SSH-yhteyksille.

![sshd_config](https://github.com/user-attachments/assets/cec39d51-ffa1-427e-97a1-28ab5b7bbff6)

Tämän jälkeen loin tarvittavan sls-tiedoston komennolla `sudoedit init.sls`. Tämänkin sisällön otin Karvisen (2018) artikkelista, mutta muutin sshd_config:n sourceksi salt://sshd/sshd_config, jottei sourcea etsittäisi suoraan salt-hakemistosta vaan nimenomaan sshd-modulista.

![sshd sls](https://github.com/user-attachments/assets/cdf3fd52-a6cd-4be6-8122-fba51c095edc)

Ajoin tämän jälkeen komennon `sudo salt '*' state.apply sshd` tarkoituksenani saada moduli käyttöön orjana olevalla t002:lla. Sain kuitenkin samanlaisen timeout ilmoituksen kuin tehtävässä a.

![sshd master fail](https://github.com/user-attachments/assets/ec060bfb-d1cf-4a2d-90ee-2ad2b2c6cfcf)

Ilmeisesti salt-master tulee käynnistää uudelleen erillisten sessioiden välissä (tein tätä tehtävää tehtävää a seuraavana päivänä), sillä ajoin pokkana komennon `sudo systemctl restart salt-master.service`, jonka myötä edellä oleva `salt`-komento toimi kiltisti.

![sshd apply 1](https://github.com/user-attachments/assets/01739970-457b-48fb-b496-b8673c538b20)

![sshd apply 2](https://github.com/user-attachments/assets/133cc6ce-e61d-46e5-9192-47dae88e68a4)

Yritin sitten saada yhteyden portin 8888 kautta Karvisen (2018) artikkelin komennoilla `nc -vz 192.168.88.102 8888` ja `ssh -p 8888 vagrant@192.168.88.102`, mutta kumpikaan ei tuntunut antavan minkäänlaista vastakaikua.

![nc port 8888 fail](https://github.com/user-attachments/assets/5fe3d70f-4dc0-4eb3-8e99-ffd84694e911)

Ajattelin sitten kokeilla uuden käyttäjän luomista t002:lle, jolle kirjautuminen SSH:n ylitse voisi onnistua paremmin kuin Vagrantille, koska en käytännössä tiedä kyseisen "käyttäjän" asetuksista mitään. Ajoinkin komennon `sudo salt '*' state.apply hello`, jonka yhtenä osana luodaan uusi käyttäjä.

![sshd hello](https://github.com/user-attachments/assets/ed39d20e-5621-4710-b1ce-bafbc984d62a)

Yritin komentoa `ssh -p 8888 herrat001@192.168.88.102`, mutta sama vastakaiun puute jatkui edelleen. Kokeilin sitten muuttaa portin tutumpaan 22:een, jonka kautta onnistuin muodostamaan yhteyden, joskaan en päässyt kirjautumaan sisälle, koska en tiennyt juuri luomani käyttäjän salasanaa (yritin edellä olevan luomisprosessin kautta vaihtoehtoina "x", "X", "", mutta mikään ei kelvannut).

![sshd port 22](https://github.com/user-attachments/assets/b8bace8d-76bf-4980-9501-3eff7bdc26c8)

SSH:ssa ei selvästikään ollut ongelmaa, vaan se vaikutti olevan käyttämässäni portissa. Tässä kohtaa tajusin, etten missään kohtaa ollut avannut t002:n palomuurissa porttia 8888, joten ongelma lieni johtuvan tästä. En kokenut järkeväksi kirjautua t002:lle ja avata manuaalisesti kyseinen portti, koska tämän tekeminen mahdollisesti useille koneille ei olisi erityisen tehokasta. Toimenpide kannattaisi pikemminkin liittää osaksi sshd-modulia. Yritin avata Saltin (2024) manuaalin, kuten olin tehnyt edellä tiivistelmissä, mutta jostain syystä mikään yrityksistäni ei suostunut toimimaan, vaan jokainen kerta tuli järjestelmän tappamaksi hetken odottelun jälkeen (todennäköisesti jonkinlainen timeout käytäntö).

![sys state_doc dead](https://github.com/user-attachments/assets/4f4fa7d5-96e2-4a68-b5b5-34206d659484)

Olin onneksi aiemmin törmännyt manuaalin nettiversioon, joten lähdin tarkastelemaan sitä. Service-tilafunktio ei tuntunut olevan etsimäni toiminto, joten päädyin käyttämään puhdasta `cmd.run`-muotoista tilafunktiota VMwaren (2024b) dokumentaation perusteella. Lisäsin tämän sshd-modulin sls-tiedoston loppuun.

![sshd sls edit](https://github.com/user-attachments/assets/61b6a3c5-76f6-4a6e-b9e1-5d416bbc7ef0)

Ajoin jälleen kerran komennon `sudo salt '*' state.apply sshd`, joka näytti, että tekemäni lisäys suoritettiin onnistuneesti.

![sshd apply edit](https://github.com/user-attachments/assets/9fcf206b-598a-4b2a-81c6-352d38e52e28)

Nyt yrittäessäni ajaa komennon `nc -vz 192.168.88.102 8888` sain sentään tiedon, että portti 8888 on auki, joskin itse hostin haku vaikutti epäonnistuvan. SSH-yhteyden ottaminen portin 8888 kautta kuitenkin onnistui nyt, tosin en edelleenkään tiennyt oikeaa salasanaa (jatkossa käyttäjien luonnissa täytyy muistaa määrittää myös salasanat).

![sshd 8888 success](https://github.com/user-attachments/assets/c21b36dc-3dd0-45f6-b74c-c4ae7c50e597)

## c) Oma moduli

Alustavasti teen/automatisoin mahdollisesti jotain Apachella. Tarvitsen kuitenkin hieman enemmän tietoa, mitä modulilta oikein odotetaan, joten aihe saattaa vaihtua vielä radikaalistikin.

## d) VirtualHost

### Manuaalinen asennus

*1:57*

Aloitin luomalla uuden käyttäjän komennolla `sudo adduser weppimasteri` (komentoa muunnettu LeeviRaussi 2024b). En liittänyt käyttäjää sudo-ryhmään, koska ajattelin, ettei tälle olisi tarvetta, koska käyttäjän tehtävä olisi ainoastaan tehdä julkisia verkkosivuja ilman sudo-komentoja. Näin jälkikäteen ehkä ne sudo-oikeudetkin olisi voinut antaa, vaikkei homma nyt sinällään tähän kaadukaan.

![new user](https://github.com/user-attachments/assets/9ed516fc-33e4-4634-8422-c4f3e7fcee0b)

Yritin sitten päästä kirjautumaan kyseiselle käyttäjälle SSH-yhteydellä komennolla `ssh weppimasteri@192.168.88.101`, mutta homma tyssäsi seinään.

![ssh publickey fail](https://github.com/user-attachments/assets/f3c1131c-48f8-4d95-b7d6-c8935a2875cf)

Lähdin selvittämään ongelmaa Googlen avulla ja löysin Ondaran (2023) artikkelin aiheesta. Artikkelin nojaten tarkastin komennolla `cat /etc/ssh/sshd_config` millaiselta SSH:n asetukset näyttävät ja sieltähän löytyi, että salasanakirjautuminen oli estettynä.

![password denied](https://github.com/user-attachments/assets/7aca686e-9195-497f-af24-febce118209c)

Ondaran (2023) artikkelin mukaisesti muokkasin komennolla `sudoedit /etc/ssh/sshd_config` tähän kohtaan "yes", minkä jälkeen käynnistin SSH:n uudelleen komennolla `sudo systemctl restart sshd`. Tämän jälkeen kun yritin kirjautua weppimasterin tilille komennolla `ssh weppimasteri@192.168.88.101` kaikki sujui mallikkaasti.

![weppimasteri success](https://github.com/user-attachments/assets/8ecc8d87-731b-4856-ad1d-ed15a7573c5e)

Otin tässä käyttööni aiemman raporttini aiheesta (ks. LeeviRaussi 2024b). Loin uuden kansion verkkosivuille komennolla `mkdir public_html`. Yritin tämän jälkeen luoda tarvittavan conf-tiedoston komennolla `sudoedit /etc/apache2/sites-available/esimerkki.conf`, mutta tässä tulivat ne sudo-oikeuksien puuttumiset vastaan. Kirjauduin käyttäjältä ulos `exit`-komennolla ja suoritin edellä olleen komennon sudo-oikeudet omaavalla vagrantilla. Tiedoston sisällön kopioin pitkälti aiemmasta raportistani (ks. LeeviRaussi 2024b).

![esimerkki conf](https://github.com/user-attachments/assets/497f45ca-cd31-4993-8e7c-e7ad7fab72d7)

Palasin takaisin weppimasteriksi komennolla `ssh weppimasteri@192.168.88.101` ja siirryin aiemmin luomaani kansioon komennolla `cd public_html`. Loin tänne uuden kansion "esimerkki" (`mkdir esimerkki`), koska olin määritellyt sen conf-tiedostossa verkkosivun sijainniksi. Siirryin kyseiseen kansioon (`cd esimerkki`) ja loin sinne uuden html-tiedoston lyhyellä tekstillä (`micro index.html`).

![index](https://github.com/user-attachments/assets/93e5e4f4-e57f-4792-a6f7-fac219c33a54)

Kirjauduin ulos weppimasterilta ja ajoin komennot `sudo a2dissite 000-default.conf` (Apachen oletussivut alas), `sudo a2ensite esimerkki.conf` (uusi sivu käyttöön) ja `sudo systemctl restart apache2` (Apachen uudelleenkäynnistys asetusten päivittämiseksi). Ajoin sitten komennon `curl localhost`, mutta sainkin eteeni 403-viestin odottamani edellä kirjoittamani tekstin sijasta.

![403 forbidden](https://github.com/user-attachments/assets/f5d33c16-5a0e-4345-ba1b-7fd40239ceb2)

Olin tätäkin asiaa kuitenkin käsitellyt jo aiemmin (ks. LeeviRaussi 2024b), joten siirryin taas kerran weppimasterille hommiin. Ajoin komennot `chmod ugo+x $HOME $HOME/public_html/esimerkki/` ja `chmod ugo+x $HOME $HOME/public_html/esimerkki/index.html`, joiden tarkoituksena oli saattaa kyseinen hakemisto ja tiedosto muidenkin luettavaksi. Ajaessani nyt komennon `curl localhost` sain näkyviini odottamani tekstin ja varmistin asian toimimisen vielä vagrant-käyttäjältä käsin.

![403 fixed](https://github.com/user-attachments/assets/6da99354-ba0f-4900-8071-af329c3a97f8)

### Automatisointi

*2:32*

Lähdin rakentamaan automatisointia edellä luomaani Apache moduliin. Kopioin manuaalisesti luomani esimerkki.conf tiedoston modulin hakemistoon (`sudo cp /etc/apache2/sites-available/esimerkki.conf /srv/salt/apache/`) ja muokkasin sls-tiedostoon manuaalisesti tekemieni välivaiheiden komennot (kuva tästä unohtui, mutta myöhemmistä versioista saa idean mitä oli alkujaan putkessa). Tämän jälkeen ajoin komennon `sudo salt '*' state.apply apache`, ja sain vastaani virheilmoituksen "apache2" id:n tuplakäytöstä.

![1 double apache](https://github.com/user-attachments/assets/2621f177-3f7b-41fc-b39d-db58f4f9db28)

Koitin etsiä Googlen avulla tietoa, saisiko tämän ongelman kierrettyä jotenkin, mutta päädyin lopulta siirtämään tekemäni lisäykset uuteen moduliin. Loin salt-hakemistossa modulin "weppi" (`sudo mkdir weppi`), jonne siirsin esimerkki.confin (`sudo mv /srv/salt/apache/esimerkki.conf /srv/salt/weppi/`), ja kopioin edellä muokkaamani apachen sls-tiedoston (` sudo cp /srv/salt/apache/init.sls /srv/salt/weppi/`), josta poistin Apache-modulin osat.

![2 separate modules](https://github.com/user-attachments/assets/5755dacd-7039-48d9-933c-efb900e18a8e)

Yritin ajaa komennon `sudo salt '*' state.apply apache weppi`, mutta salt-master nukkui jälleen (tein tehtäviä edelleen putkeen, joten en tiedä ollenkaan mistä tämä johtui).

![3 salt-master offline](https://github.com/user-attachments/assets/959238f4-eaf8-47c6-99c1-5b38f319b0e3)

Ajattelin ensin, että ongelma saattoi johtua komennostani, joten ajoin pelkän apache-modulin (`sudo salt '*' state.apply apache`), mutta sain saman TCP timeoutin.

![4 salt-master offline](https://github.com/user-attachments/assets/0bf9351a-abcd-4ab8-9318-6c4ada6be4b8)

Boottasin salt-masterin (`sudo systemctl restart salt-master.service`) ja ajoin uudelleen apache-modulin komennon saaden vastaani saman virheviestin kuin aiemmin modulien ollessa vielä yhdessä.

![4 5 reboot](https://github.com/user-attachments/assets/1e40c192-e7b3-40e8-b1dc-b2d6fd56e9a2)

En ollut muistanut poistaa kyseisestä modulista tekemiäni lisäyksi, mutta niiden poistamisen jälkeen apache-modulin ajaminen sujui ongelmitta.

![5 salt apache works](https://github.com/user-attachments/assets/f7f6d0ee-4a55-4d07-9146-7dec6ca8480b)

Ajoin sitten komennon `sudo salt '*' state.apply weppi` weppi-modulin suorittamiseksi, mutta service.running kohdan watch antoi erroria.

![6 watch error](https://github.com/user-attachments/assets/37761f21-8a97-466a-b0dc-c4df356775fe)

Karvisen (2018) artikkelin avulla huomasin, että olin kirjoittanut virheellisesti sijainnin samalle riville watchin kanssa, joten "korjasin" asian. Tai en oikeasti korjannut, koska unohdin välilyönnin "-" ja "file" välistä.

![7 watch error sls](https://github.com/user-attachments/assets/9bab0f37-c83c-420f-b303-ec5a43426e98)

![8 still watch error](https://github.com/user-attachments/assets/eac6f5de-6eef-43e9-a03e-5e027332f50f)

Näiden korjausten jälkeen sain ajettua komennon `sudo salt '*' state.apply weppi` "onnistuneesti" loppuun asti, joskin virheilmoitusten saattelemana.

![9 weppi errors](https://github.com/user-attachments/assets/2aa3dbc8-95ae-44b0-aa6e-83741f47f379)

![10 weppi errors](https://github.com/user-attachments/assets/e547bc1b-563d-4937-935d-b93f1e50008d)

![11 weppi errors](https://github.com/user-attachments/assets/22b5daa5-da34-4ae3-96fc-8c4b0b524fcd)

Jälkikäteen todettuna järkevintä olisi ollut rakentaa moduli yksi osa kerrallaan, mutta tässä kohtaa se oli myöhäistä. Muokkasin esimerkki.confin oikeaksi sijainniksi muuttuneen weppi-moduulin. Lisäksi päätin jakaa file.managed-tilafunktiot yksittäisiin osiin, josko tämä olisi syynä virheilmoituksiin.

![12 file managed split](https://github.com/user-attachments/assets/df75718c-2e95-4a9d-bb0f-31644f4bfb82)

![13 file managed error](https://github.com/user-attachments/assets/9e02a93c-aacb-403d-9f12-5c88d6cb93f8)

esimerkki.conf-virhe poistui, mutta file.managed virheet jatkuivat. Ajattelin virheeni olleen puuttuvat vaakaviivat, joten lisäsin ne.

![14 file managed slash](https://github.com/user-attachments/assets/4050eea6-cfaf-4b09-a6dd-78b3e45dfaf0)

![15 file managed error](https://github.com/user-attachments/assets/5da4c171-e1ac-416f-8a4c-99d4d3d80b7e)

Sama virhe toistui, joten lähdin etsimään Saltin nettimanuaalista apua. Huomasinkin, että minun pitäisi käyttää tilafunktiona file.directory, kun käsittelen hakemistoja (VMware 2024c). Muokkasin tilafunktiot oikeiksi, jolloin sain uudenlaisen virheilmoituksen.

![16 file directory fix](https://github.com/user-attachments/assets/7f2964c7-0216-4d0b-bfb3-680597909d99)

![17 already exists](https://github.com/user-attachments/assets/79ba4824-b439-45bc-bcae-549d9204f2a2)

Edellä ajamani komennot olivat luoneet tiedoston "public_html", joten samannimistä hakemistoa ei voitu luoda. Ajoin komennon `sudo salt '*' state.single file.absent /home/weppimasteri/public_html`, jolla pääsin eroon kyseisestä tiedostosta.

![18 file absent](https://github.com/user-attachments/assets/9b4447a8-a312-4d9a-84df-2d7f15891aa8)

Nyt ajaessani komennon `sudo salt '*' state.apply weppi` sain vihreää `chown`-komentoon asti.

![19 chown error](https://github.com/user-attachments/assets/c3cfe6bf-9400-415a-9c09-b37844a699ba)

Olin hakenut komennon käyttöön ohjeiksi Marijanin (2024) artikkelin, jota tarkstelin lisää ajatellen, että olin tehnyt taas pienen typon. Huomasin kuitenkin tehneeni isomman typon, sillä olin unohtanut määritellä käyttäjän kenen omistukseen kyseisen hakemiston pitäisi tulla.

![20 chown sls fix](https://github.com/user-attachments/assets/dc9037a8-2ec8-4951-810d-cdba7736a4f6)

Korjattuani tämän sain vihdoin ajettua onnistuneesti komennon `sudo salt '*' state.apply weppi`.

![21 weppi all ok](https://github.com/user-attachments/assets/d302567d-7899-4bfb-aebc-345dec880637)

Yritin komentoa `curl 192.168.88.102`, mutta en saanut minkäänlaista vastakaikua tähän. Tämä johtui todennäköisesti jälleen palomuuriasetuksista.

![22 curl timeooooout](https://github.com/user-attachments/assets/37b8b75b-2902-4b72-9e9e-eb2cc4391774)

Täysin sama toiminto orjalla (kone t002) toimi kuitenkin täysin moitteetta. Yritin sitten päästä kirjautumaan sisälle SSH-yhteydellä t002 koneelta käsin, mutta user.present kohdassa määrittelemääni salasanaa ei hyväksytty.

![23 t002 ssh fail](https://github.com/user-attachments/assets/cd737cbb-cd02-4abc-a696-e64723600590)

Muokkasin sshd_config tiedostoon (`sudoedit /etc/ssh/sshd_config`) lisäyksen "PasswordAuthentication yes", mutta tämäkään ei auttanut.

![23 5 sshd_config edit t002](https://github.com/user-attachments/assets/f6b9c64d-1870-427c-9d99-802a8401648c)

t001 koneelta sain samaa virhettä yrittäessäni muodostaa SSH-yhteyden. Ajattelin asian mahdollisesti johtuvan yksityinen-julkinen avaisparin ongelmasta, joten generoin uuden avainparin komennolla `ssh-keygen -o` ja yritin siirtää julkisen avaimen t002:n weppimasterille komennolla `ssh-copy-id weppimasteri@192.168.88.102` tässä kuitenkaan onnistumatta.

![24 t001 ssh-keygen](https://github.com/user-attachments/assets/35e212c6-0bbe-4b20-8061-700e10723946)

Kirjautuminen t001:llä onnistui kuitenkin saman koneen käyttäjään "weppimasteri", joten mietin oliko asetuksissani jotain vikaa. Selasin tehtävänantoon liittyviä asioita (ks. Karvinen 2024) ja silmääni osui tiivistelmien kohdassa ollut maininta, että SSH-asetustiedostojen pohjana tulisi käyttää oman Linux-version pohjaa, joten ratkaisuni aiemmin kopioida Karvisen (2018) artikkelista asetukset ei välttämättä ollutkaan toimiva ratkaisu. Muokkasin komennolla `sudoedit /etc/ssh/sshd_config` tiedostoon SSH-porteiksi 22 ja 8888.

![25 t001 sshd_config](https://github.com/user-attachments/assets/82716cb9-5db3-4b15-ad7f-0507be310933)

Kopioin tämän jälkeen kyseisen tiedoston komennolla `sudo cp /etc/ssh/sshd_config /srv/salt/sshd/` aiempaan sshd-moduliin ja ajoin komennon `sudo salt '*' state.apply sshd`, jolla sain t002:lle samat SSH-asetukset. Tämä ei kuitenkaan auttanut, vaan sain samaa virheviestiä edelleen.

![26 t001 ssh fail](https://github.com/user-attachments/assets/5155fb8c-a47a-4d3c-9615-206fcd69c64a)

Koska kirjautuminen toimii kuitenkin SSH:lla t001:n "weppimasterille" aloin epäilemään teinkö jotain väärin user.present-tilafunktion kanssa. Törmäsin Gonzalesin (2013) aloittamaan vanhaan keskusteluun, jonka kautta selvisi, ettei salasanaa pitäisi kirjoittaa itse salasanana, vaan hash-muodossa. Tarkistin asian myös Saltin nettimanuaalista (VMware 2024d), ja tosiaan siellä sanotaan, että salasana annetaan hash-muodossa. Artikkelissa oleva komento `mkpasswd -m sha-256` ei toiminut, enkä myöskään komento `sudo apt-get install mkpasswd` onnistunut asentamaan tätä. Googlen kautta löysin sivuston, jolla pystyi generoimaan näitä hash-muotoja (ks. mkpasswd), joten kirjoitin salasanan ja pastesin tuloksen weppi-modulin sls-tiedostoon.

![27 hash password](https://github.com/user-attachments/assets/138ccf70-b304-4ca9-b940-628dbcecdaa8)

Salasana vaihtui onnistuneesti komennolla `sudo salt '*' state.apply weppi`.

![28 password change](https://github.com/user-attachments/assets/509421be-6948-4999-abd9-eb8281c34ee5)

Lopputulosta tämä ei kuitenkaan muuttanut, vaan kirjautuminen epäonnistui edelleen.

![29 no change](https://github.com/user-attachments/assets/0d95bbfc-2600-4634-a82b-76ffafd391de)

Otin radikaalimmat ideat käyttöön Saltin manuaalista (VMware 2024d) ja hyväksyin tyhjät salasanat.

![30 no password](https://github.com/user-attachments/assets/4a900174-c419-44a6-9896-f02ef97ff69c)

Lisäksi myös tällä kertaa deletoin (`sudo salt '*' state.single user.absent weppimasteri`) koko käyttäjän "weppimasteri" ennen kuin ajoin uudelleen komennon `sudo salt '*' state.apply weppi`.

![31 user absent](https://github.com/user-attachments/assets/a69acde7-5469-4b05-a287-1bb7c49b0e1f)

![32 user present](https://github.com/user-attachments/assets/b7f92b93-740c-4663-a618-9e7771eacc03)

Lopputulokseen ei kuitenkaan ollut taaskaan vaikutusta.

![33 doesnt matter](https://github.com/user-attachments/assets/a28bfb42-9cee-4205-b856-9db6f69acc0c)

*4:32*

Tässä kohtaa löin hanskat pöytään osittaisena luovutuksena. Itse nettisivu toimi kyllä, oikeudet olivat oikein (ks. alla oleva kuva), ainoa ongelma vain, ettei käyttäjälle pääse kirjautumaan sisälle. Saman työn voisi varmaan tehdä suoraviivaisesti t002:n vagrantille, jolloin uutta käyttäjää ei tarvitsisi luoda, mutta tämä tuntuu huonolta ratkaisulta pitkässä juoksussa. Ongelma lienee user.present-tilamodulissa, koska t001:llä homma toimi `adduser`-komennolla, mutta mikä tämä ongelman ratkaisu sitten on onkin toinen asia.

![34 owner](https://github.com/user-attachments/assets/4c11ca1d-15ca-4df4-bdae-224083558ae1)

## Lähdeluettelo

Gonzales, R. 2013. setting password apparently does not work in user.present. Luettavissa: https://groups.google.com/g/salt-users/c/6kQlEYqlPis. Luettu: 19.11.2024.

Karvinen, T. 2018. Pkg-File-Service – Control Daemons with Salt – Change SSH Server Port. Luettavissa: https://terokarvinen.com/2018/04/03/pkg-file-service-control-daemons-with-salt-change-ssh-server-port/?fromSearch=karvinen%20salt%20ssh. Luettu: 19.11.2024.

Karvinen, T. 2021. Run Salt Command Locally. Luettavissa: https://terokarvinen.com/2021/salt-run-command-locally/. Luettu: 18.11.2024.

Karvinen, T. 2024. Palvelinten Hallinta. Luettavissa: https://terokarvinen.com/palvelinten-hallinta/. Luettu: 19.11.2024.

LeeviRaussi 2024a. h3 Hello Web Server. Luettavissa: https://github.com/LeeviRaussi/linux-palvelimet/blob/main/h3_Hello_Web_Server.md. Luettu: 19.11.2024.

LeeviRaussi 2024b. h7 Maalisuora. Luettavissa: https://github.com/LeeviRaussi/linux-palvelimet/blob/main/h7_Maalisuora.md. Luettu: 19.11.2024.

LeeviRaussi 2024c. h2 Infra-as-code. Luettavissa: https://github.com/LeeviRaussi/Palvelinten_hallinta/blob/main/h2_Infra-as-code.md. Luettu: 18.11.2024.

lloydbush 2024. [Solved] Error updating using apt/apt-get. Luettavissa: https://forums.debian.net/viewtopic.php?t=158481. Luettu: 18.11.2024.

Marijan, B. 2024. Linux chown Command: Syntax, Options & Examples. Luettavissa: https://phoenixnap.com/kb/linux-chown-command-with-examples. Luettu: 19.11.2024.

mkpasswd. mkpasswd. Luettavissa: https://www.mkpasswd.net/index.php?category=sha. Luettu: 19.11.2024. 

Ondara, W. 2023. How to Fix SSH Permission Denied (Public key) Error in Linux. Luettavissa: https://www.tecmint.com/ssh-permission-denied-publickey/. Luettu: 19.11.2024.

Salt 2024. Salt manual. Debian 12 käyttöjärjestelmällinen kone. Luettu: 19.11.2024.

VMware, Inc 2024a. Salt execution framework. Luettavissa: https://docs.saltproject.io/salt/user-guide/en/latest/topics/execution-framework.html. Luettu: 18.11.2024.

VMware, Inc 2024b. salt.states.cmd. Luettavissa: https://docs.saltproject.io/en/latest/ref/states/all/salt.states.cmd.html. Luettu: 19.11.2024.

VMware, Inc 2024c. salt.states.file. Luettavissa: https://docs.saltproject.io/en/latest/ref/states/all/salt.states.file.html#salt.states.file.directory. Luettu: 19.11.2024.

VMware, Inc 2024d. salt.states.user. Luettavissa: https://docs.saltproject.io/en/latest/ref/states/all/salt.states.user.html. Luettu: 19.11.2024.

Zivanov, S. 2023. How to Set Up Time Synchronization on Debian. Luettavissa: https://phoenixnap.com/kb/debian-time-sync. Luettu: 18.11.2024.
