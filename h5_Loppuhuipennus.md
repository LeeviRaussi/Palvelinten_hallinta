# h5 Loppuhuipennus

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
> - SSD: 733/951 GB vapaana

> Raportissa esiintyvät aikaleimat viittaavat reaaliseen aikaan, jona komentoja on ajettu, koodeja kirjoitettu ja tietoa etsitty. Itse raportin kirjoittamiseen käytetty aika ei sisälly tähän.

## a) Oma miniprojekti

### Env-virtuaaliympäristön uudelleensijoittaminen

*0:00*

Jatkoin edellisen raporttini (ks. LeeviRaussi 2024b) pohjalta oman moduulini työstämistä. Torstain luennolla saamani vinkit saivat minut muuttamaan Virtualenv-virtuaaliympäristön asennuksen sijaintia suoraan käyttäjän "weppimasteri" omaan kotihakemistoon. Huomasin, että olin näin tehnyt jo Linux-palvelimet-kurssilla (ks. LeeviRaussi 2024a), mutta olin tällä kertaa sijoittanut hakemiston verkkosivuille varattuun hakemistoon. En uskonut, että tällä olisi niin suurta väliä, mutta päätin tehdä muutoksen silti hakemistorakenteen selkiyttämisen takia.

Aloitin poistamalla env-hakemiston Saltin avulla komennolla `sudo salt-call --local state.single file.absent /home/weppimasteri/public_html/env`.

![1 env absent](https://github.com/user-attachments/assets/96aed336-700e-473a-b913-6b9042a892ea)

Otin tämän jälkeen SSH-yhteyden weppimasteriin (`ssh weppimasteri@192.168.88.101`) ja loin kyseisen käyttäjän kotihakemistoon tarvittavat Virtualenv-hakemistot komennolla `virtualenv -p python3 --system-site-packages env`. Muokkasin tämän jälkeen perusta.conf-tiedostoon päivitetyn polun kyseisestä hakemistosta löytyviin osiin.

![2 new path](https://github.com/user-attachments/assets/08d2c43f-fe9e-43a0-9f12-5a6182e15622)

Testasin tämän jälkeen `curl`-komennolla, että kaikki toimisi oikein, mutta sain 500-virheviestin. Kokeilin, auttaisiko Apachen uudelleenkäynnistys (`sudo systemctl restart apache2`), mutta ongelma jatkui edelleen.

![3 500 error](https://github.com/user-attachments/assets/b533cf33-48a9-497a-a75b-2e5de1488b54)

Komento `/sbin/apache2ctl configtest` ilmoitti, että ongelma ei ollut syktaksissa. Päätin muuttaa edellisessä raportissani muokkaamani settings.py-tiedoston DEBUG-asetuksen takaisin päälle, josko tämä auttaisi asiaan.

![4 debug true](https://github.com/user-attachments/assets/9b6c89b0-eca6-4bcc-af07-ecf5f330e87c)

Ongelma ei kuitenkaan ratkennut, joten seuraavaksi päätin ottaa pois päältä (tai kommentoida) kaikki kyseiseen tiedostoon tekemäni muutokset.

![5 additions away](https://github.com/user-attachments/assets/ed0de3c2-4a8e-4849-a957-09b8108da768)

localhost/static/-sivu toimi edelleen ongelmitta, mutta itse localhost-sivu antoi vieläkin virheilmoitusta.

![6 still 500](https://github.com/user-attachments/assets/cb0816e6-ce5f-44f8-9025-6794a41506f4)

Koska en hahmottanut, mistä ongelma johtui, ja koska loppupeleissä vanhan toistoa ei ollut niin paljoa, päätin tuhota koko aiemman rakennelmani ja tehdä moddulin Django-sivut uudelleen. Otin talteen /static/-sivun tiedostot (`mv /home/weppimasteri/public_html/perusta/modduli/static/ /home/weppimasteri/public_html/perusta/`), jonka jälkeen siirryin takaisin pääkäyttäjälle ajamaan vaadittavan Salt-komennon (`sudo salt-call --local state.single file.absent /home/weppimasteri/public_html/perusta/modduli`).

![7 mv static](https://github.com/user-attachments/assets/d0cd3751-4b25-4644-83b6-9a1fcb3552ba)

![8 modduli absent](https://github.com/user-attachments/assets/eed8e6fd-8c59-4ace-8d7d-60d874e4e894)

*0:18*

Palasin takaisin SSH-yhteydellä käyttäjälle weppimasteri ja siirryin ensiksi kotihakemistosta hakemistoon perusta (`cd public_html/perusta/`), jossa testasin saada selville Djangon-versionumeron (`django-admin --version`). Tämä ei kuitenkaan toiminut.

![9 no django](https://github.com/user-attachments/assets/45c1911b-ab73-4ec8-a8d0-4fc082e88db0)

Ajattelin, että Django oli ollut jotenkin sidottuna virtuaaliseen ympäristöön, joten uudelleen tehty asennus olisi vaikuttanut myös siihen. Palasin käyttäjän kotihakemistoon, jonne siirsin myös django-tekstin sisältävän requirements-tekstitiedoston (`mv public_html/requirements.txt /home/weppimasteri/`). Siirryin tämän jälkeen env-virtuaaliympäristöön (`source env/bin/activate`) ja asensin Djangon jälleen kerran komennolla `pip install -r requirements.txt`.

![10 django install](https://github.com/user-attachments/assets/41fe820a-3f23-497c-8497-26edf1f4faaf)

[Jälkihuomio raporttia kirjoittaessa: Edellä olleeseen vaiheeseen liittyen raporttia kirjoittaessa ajoin uudelleen komennon `django-admin --version` niin ei-env-ympäristössä kuin myös env-ympäristössä ollessani hakemistossa `/home/weppimasteri/public_html/perusta/modduli`. Ei-env-ympäristössä ajaettu komento ei taaskaan tunnistanut komentoa, kun taas env-ympäristössä ajettu komento tuntui jäävän jonkinlaiseen looppiin, jonka jouduin keskeyttämään hetken odottelun jälkeen CTRL+C yhdistelmällä. Siirryin tämän jälkeen `cd`-komennolla takaisin kotihakemistoon, jossa olin tehnyt Django-asennuksen, mutta täälläkään versionumeroa hakeva komento ei toiminut, vaan jäi jälleen looppiin. Verkkosivut toimivat kuitenkin oikein `curl`-komennolla, joten tämä asia jää itselleni nyt pieneksi mysteeriksi.]

Koska olin jo edellisessä raportissani (ks. LeeviRaussi 2024b) tehnyt nyt tekemäni asiat, sain suoraviivaistettua asioita hyvin. Siirryin perusta-sivujen hakemistoon (`cd public_html/perusta/`), jossa aloitin uuden Django-projektin (`django-admin startproject modduli`). Tarkastettuani, että uusi hakemisto oli luotu, siirsin evakossa olleen /static/-hakemiston takaisin conf-tiedoston määrittelemään sijaintiin (`mv /home/weppimasteri/public_html/perusta/static/ /home/weppimasteri/public_html/perusta/modduli/`). Itse conf-tiedostoa en lähtenyt muokkaamaan, koska siellä kaiken pitäisi olla oikein. Tämän jälkeen siirryin pois env-ympäristöstä (`deactivate`).

![11 django start](https://github.com/user-attachments/assets/8b7a1046-5081-4b43-8544-e59382844f38)

Poistuin env-ympäristöstä, koska ajattelin `sudo apt-get -y install libapache2-mod-wsgi-py3` komennolla ajetun wsgi-lisäosan asennuksen olevan hyvä merkki siitä, että env-tilan käyttö riitti tältä erää. Asennuksen valmistuttua testasin conf-tiedoston syntaksin (`/sbin/apache2ctl configtest`), jossa ei ollut ongelmia. Tämän jälkeen käynnistin Apachen uudelleen (`sudo systemctl restart apache2`), minkä jälkeen testasin `curl`-komennolla onnistuneesti sivujen toimimisen.

![12 wsgi mod install](https://github.com/user-attachments/assets/9a6660bc-3402-4ba6-82f7-144cf5a6afc0)

Muokkasin asetustiedostoon DEBUGin epätodeksi ja sallituksi isännäksi localhostin (`micro modduli/modduli/settings.py`). Ajoin sitten molemmat komennot `touch wsgi.py` ja `sudo systemctl restart apache2`, jottei ongelmia syntyisi. `curl`-komento tuotti jälleen oikeaa tulosta.

![13 debug false](https://github.com/user-attachments/assets/c01fff5d-1ae7-42ab-81b6-a18e7d6ffaa5)

![14 django hidden](https://github.com/user-attachments/assets/73a1657b-0cc4-4926-a509-cfd0d6b545a1)

Muokkasin lisää edellä mainittua asetustiedostoa lisäten sinne käskyn `import root` ja määritelmän `STATIC_ROOT` (tai ainakin näin sen olisi pitänyt mennä).

![15 style additions](https://github.com/user-attachments/assets/5cafca02-6d73-4d24-a909-75468d5957f7)

Koitin sitten suorittaa komennon `./manage.py collectstatic` suoraan nykyisestä hakemistostani, mutta tämä ei onnistunut. Siirryin sitten oikeaan hakemistoon (`/home/weppimasteri/public_html/perusta/modduli`), jossa myöskään yllätyksekseni komento ei toiminut. Tässä vaiheessa kiinnitin enemmän huomiota virheilmoitukseen ja tajusin, että minun pitäisi ajaa komento env-tilassa, josta olin aiemmin poistunut.

![16 wrong env](https://github.com/user-attachments/assets/6d8da9da-1709-47a7-8b99-64d0375d83bc)

Ajoin uudelleen vaaditun komennon, mutta tällä kertaa homma kaatui huonosti määriteltyyn asetustiedostoon.

![16 5 missing static_root](https://github.com/user-attachments/assets/62585aea-2b3d-4247-94f1-2f9f096e8d34)

Tarkistin edellä tekemiäni muokkauksia huomaten, että olin kirjoittanut vahingossa STATIC_Root. Korjaus isoihin kirjaimiin, jonka jälkeen poistuin jälleen env-ympäristöstä. Ilokseni `curl`-komento antoi edelleen oikeaa tulosta.

![17 fixed static_root](https://github.com/user-attachments/assets/39a88cd1-020d-4e98-879f-8ca34f7a8222)

### Automatisoinnin jatkaminen

*0:33*

Palasin pääkäyttäjän rooliin ja siirryin Saltin modduli-modulin hakemistoon (`cd /srv/salt/modduli/`). Kopioin kyseiseen hakemistoon päivitetyn perusta.conf-tiedoston (`sudo cp /etc/apache2/sites-available/perusta.conf /srv/salt/modduli/`) sekä django-tekstin sisältävän requirements-tekstitiedoston (`sudo cp /home/weppimasteri/requirements.txt /srv/salt/modduli/`).

![18 cp](https://github.com/user-attachments/assets/0c2e2b76-87f6-4c3c-9ce1-eb3bea48b6cb)

Lisäsin sitten moddulin sls-tiedostoon kaksi cmd.run-komentoa, joilla ensin aktivoidaan env-virtuaaliympäristö ja tullaan sitten sieltä pois.

![19 new additions](https://github.com/user-attachments/assets/c22292a4-06eb-4008-ba5c-27996145be8d)

Käynnistin Salt-masterin uudelleen (`sudo systemctl restart salt-master.service`) ja ajoin moddulin Saltilla (`sudo salt '*' state.apply modduli`). Lopputuloksena niin juuri tekemäni lisäykset kuin myös Apache epäonnistuivat.

![20 salt error](https://github.com/user-attachments/assets/413a512a-0ebb-4aad-ad33-efa55fa1052a)

Tajusin, etten ollut koskaan luonut env-hakemistoja, joihin yritin juuri viitata, joten näiden kahden komennon epäonnistuminen olikin odotettua. Kuitenkin Apachen virhettä en täysin ymmärtänyt, koska asia oli toiminut edellisellä kerralla (ks. LeeviRaussi 2024b), eivätkä nyt tekemäni muutokset conf-tiedostoon olleet mielestäni kovin radikaaleja. Koitin ajaa uudelleen komennon `sudo salt '*' state.apply modduli`, mutta Apache ei edelleenkään tehnyt yhteistyötä. Lähdin selaamaan Saltin manuaalia ja erityisesti watch-lisäosaan keskittyvää osiota (ks VMware 2024a). Täältä sain idean, että mahdollisesti ongelma johtui järjestyksestä, missä asiat olivat sls-tiedostossa. Muutiksin Apacheen liittyvän service.running-tilafunktion sijainnin heti asennuksen perään.

![21 new place](https://github.com/user-attachments/assets/60beb86f-8c63-431a-9f34-404e8bb45816)

Tämäkään ei kuitenkaan ratkaissut ongelmaa. Avasin toisen komentorivin ja kirjauduin sillä orjana olevalle t002:lle, jossa ajoin komennot `systemctl status apache2.service` ja `sudo journalctl -xeu apache2.service`, jotka oli mainittu Saltin virheilmoituksen yhteydessä. Jälkimmäisen avulla sain näkyviini tietoja syntaksivirheistä, joita conf-tiedostossa esiintyi.

![22 journalctl apache2](https://github.com/user-attachments/assets/d7e6d4cf-82b7-43ea-9d62-9db13a4e53e7)

Erikoisesti ensimmäinen vastaava ilmoitus oli päivätty 27.11. noin klo 11, jolloin en ole kuitenkaan ollut minkäänlaisessa kosketuksessa koko koneeseen (viimeistelin edellisen raportin 26. päivän illalla), joten vaikutti siltä, että jotain olisi hajonnut itsestään sen jälkeen, kun olin saanut edellisessä raportissani (ks. LeeviRaussi 2024b) asiat päätökseen.

Syntaksiongelma johtui todennäköiseti siitä, että käytin conf-tiedostossa asetuksia "tulevaisuudesta". Päätin luoda tätä tilannetta varten uuden conf-tiedoston (perusta2.conf), jonka sisällön otin suoraan edellisen raporttini manuaalisesta osuudesta (ks. LeeviRaussi 2024b) ennen kuin aloin asentelemaan Djangoa. Tämä vaati myös muutosten tekemistä sls-tiedostoon.

![23 perusta2](https://github.com/user-attachments/assets/ded66575-d0cd-4f4f-897c-b066d4ad2156)

Tämän väliaikaisen ratkaisun ansiosta Apache lähti jälleen toimimaan ja sls-tiedosto muuttui idempotentiksi toisella ajolla.

![24 salt perusta2 working](https://github.com/user-attachments/assets/1964ed6c-30b6-43a5-ae50-5557126731c8)

![25 salt perusta2 idempotent](https://github.com/user-attachments/assets/dd6e546e-729f-4c52-9f2e-fca845f24b77)

*1:17*

Pääsin sitten tietyssä mielessä jatkamaan siitä, mihin edellisessä raportissani (ks. LeeviRaussi 2024b) jäin. Lisäsin edellä kirjoittamieni ja sittemin kommentoimieni env-virtuaaliympäristökomentojen eteen komennon, jolla kyseinen ympäristö luotaisiin. Vaatimuksena olevalla `creates`-parametrilla pyrin siihen, että hakemisto luotaisiin weppimasterin kotihakemistoon.

![26 virtualenv](https://github.com/user-attachments/assets/c0369188-3426-4966-991e-ffe0824a476e)

Sulkiessani sls-tiedoston käyttöjärjestelmä jäi jumiin reiluksi kolmeksi minuutiksi, josta pääsin (todennäköisesti) irti vasta CTRL+C yhdistelmällä. Päätin tämän myötä kuitenkin käynnistää tässä vaiheessa Vagrantin uudestaan.

![27 stuck](https://github.com/user-attachments/assets/28634d46-16c6-4635-a5d5-28b2713ff009)

*1:26*

Päästyäni takaisin koneelle sisään käynnistin Salt-masterin uudelleen (`sudo systemctl restart salt-master.service`) ja ajoin moddulin Saltilla (`sudo salt '*' state.apply modduli`). Kaikki näytti onnistuneen oikein.

![28 salt env wrong](https://github.com/user-attachments/assets/ad9a121f-95f9-477e-ac3e-4e320ac424bc)

Toistin ajon, jotta idempotenttisuus osoittautuisi, mutta näin ei tapahtunutkaan, vaan komento suoritettiin taas. Kirjauduin toisella komentorivillä t002:lle ja tarkistin hakemiston `/home/weppimasteri/` sisällön, ja eihän hakemistoa todellakaan ollut siellä. Tarkastelin tarkemmin edellä ollutta salt-logia, silmiini osui app_data_dir-kohdassa ollut /root/-polku. Tarkastin kansion sisällön ja löysin kadoksissa olleen env-hakemiston. [Kommentti jälkikäteen: Tässä voi sanoa olleen tuuria, koska myöhemmin oikein suoritetun komennon kanssa samassa kohdassa esiintyy sama polku.]

![29 root](https://github.com/user-attachments/assets/17c3cfd6-5314-4817-80ff-8d0eb264e1b6)

Tarkastin t001:llä, että /root/-hakemisto on tyhjä, jonka jälkeen ajoin komennon `sudo salt '*' state.single file.absent /root/env`.

![30 absent root](https://github.com/user-attachments/assets/058589af-bc6d-4902-81c1-3756b300aae1)

Koitin muokata sls-tiedostoon oikean sijainnin määrittämällä tarkasti polun, jonne asennus pitäisi tehdä.

![31 fix location](https://github.com/user-attachments/assets/462a9a30-aeb8-4ee3-bbfa-4e4887ae96a0)

Saltin uudelleenkäynnistys (`sudo systemctl restart salt-master.service`) ja modulin ajo (`sudo salt '*' state.apply modduli`) onnistuivat taas.

![32 salt env correct](https://github.com/user-attachments/assets/c5ffa62f-e0f8-4470-ac47-454ac84e6666)

Toinen ajo osoitti, että tällä kertaa kyseessä oli idempotenttinen rakenne.

![33 salt env idempotent](https://github.com/user-attachments/assets/23fc592c-02dc-41ac-9a7e-f0db421b8fd5)

Nyt env-hakemisto löytyi oikeasta paikasta.

![34 env correct location](https://github.com/user-attachments/assets/33bf8f21-bda6-44c3-b760-47ce497b1a60)

*1:33*

Palasin muokkaamaan sls-tiedostoa ja poistin aiemmin lisäämäni kommenttimerkit virtualenv-komentojen edestä. En ollut varma miten nämä komennot toimisivat etänä, mutta lähdin kokeilemaan asiaa vakioituneilla `sudo systemctl restart salt-master.service` ja `sudo salt '*' state.apply modduli` komennoilla.

![35 env activation fail](https://github.com/user-attachments/assets/0caf870f-7730-4132-9e38-ec712a973dfa)

Virheilmoitukset näyttivät erikoisilta, vaikken niitä osannut kuitenkaan sen tarkemmin tulkita. Päätin kokeilla manuaalisesti env-tilaan siirtymistä t002:lla, joten otin kyseisen koneen weppimasteriin SSH-yhteyden. Yllätyksekseni komento `source env/bin/activate` ei toiminut.

![36 source mia](https://github.com/user-attachments/assets/41c616e8-3500-45a4-9c5d-ffce3eb5a96a)

Tämä toki selitti, miksi asia ei ollut onnistunut Saltilla, mutta avasi suuremman kysymyksen, eli miksi tämä puuttuu. Lähtökohtaisesti kuitenkin t001 (herra) ja t002 (orja) koneiden välillä ei pitäisi olla liiemmin eroa, joten koska tämä toimii t001:n weppimasterilla (uudelleen testattuna) odottaisin sen toimivan myös t002:lla. Lähdin etsimään Googlella tietoa, miten saisin aktivoitua env-ympäristön. Päädyin vanhahkoon larryn (2013) aloittamaan keskusteluun, joka vaikutti sivuavan omaa ongelmaani. Keskustelun kommenttien seasta bongasin Thomasin kommentin, että "." on synonyymi "source" komennolle. Testasin tätä yllättyneen onnistuneesti t001:n weppimasterilla.

![37 dot is source](https://github.com/user-attachments/assets/c8d9f5cd-840f-4cc0-956b-0f1f337dbd11)

Sourcen korvaaminen pisteellä toimi myös t002:n puolella.

![38 dot works](https://github.com/user-attachments/assets/edf07949-8c8d-43bd-93bc-dafcc994d4e3)

Päätinkin korvata sls-tiedostossa sourcen pisteellä.

![39 dot included](https://github.com/user-attachments/assets/2e84f5fd-488f-4f8f-bcee-eead36a8d14b)

Tämän muutoksen seurauksena virtuaaliympäristön aktivointi rupesi toimimaan, mutta deaktivointi ei edelleenkään toiminut.

![40 deactivate fail](https://github.com/user-attachments/assets/c39c5aa6-f864-474a-979c-768b38b70a5f)

Koneen t002 puolella myöskään ei ollut jäänyt env-ympäristö päälle.

![41 not in env](https://github.com/user-attachments/assets/29c38fce-c28b-43d0-8cc5-855a8db3bfbb)

*2:07*

Mietin kokonaan deactive-kohdan poistamista koodista, sillä se ei selvästikään toiminut (sama virheilmoitus komennon puuttumisesta kuin edellä sourcen kanssa). Vaihtoehtoisesti minun pitäisi mahdollisesti etsiä toinen toteutustapa, koska kyseenalaistin sen, kuinka pitkään env-ympäristössä todella pysytään.

*2:45*

## Lähdeluettelo

larry 2013. How can I activate a virtualenv in Linux?. Luettavissa: https://stackoverflow.com/questions/14604699/how-can-i-activate-a-virtualenv-in-linux. Luettu: 1.12.2024.

LeeviRaussi 2024a. h6 Hello Django. Luettavissa: https://github.com/LeeviRaussi/linux-palvelimet/blob/main/h6_Hello_Django.md. Luettu: 30.11.2024.

LeeviRaussi 2024b. h4 Puolikas. Luettavissa: https://github.com/LeeviRaussi/Palvelinten_hallinta/blob/main/h4_Puolikas.md. Luettu: 30.11.2024.

VMware, Inc. 2024a. Requisites and Other Global State Arguments. Luettavissa: https://docs.saltproject.io/en/latest/ref/states/requisites.html. Luettu: 1.12.2024.

VMware, Inc. 2024b. salt.states.cmd. Luettavissa: https://docs.saltproject.io/en/latest/ref/states/all/salt.states.cmd.html. Luettu: 30.11.2024.
