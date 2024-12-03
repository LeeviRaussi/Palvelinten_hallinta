# h5 Loppuhuipennus

> ## Tiivistelmä
>
> Tässä raportissa jatketaan oman Salt modulin tekemistä. Tavoitteena on luoda moduli, joka automatisoisi Djangon-asentamisen Apachea hyödyntävään verkkosivuympäristöön.

> ### Käytetty laitteisto
>
> Lenovo Ideapad Pro 5 14APH8 kannettava tietokone
> - Prosessori: AMD Ryzen 7 7840HS
> - GPU: Radeon 780M Graphics (prosessoriin integroitu)
> - RAM: 32 GB, 6400 MT/s
> - OS: Windows 11 Home 23H2
> - Näytön resoluutio: 2880x1800 (175% skaalaus)
> - SSD: 733/951 GB vapaana

> Raportissa esiintyvät aikaleimat viittaavat reaaliseen aikaan, jona komentoja on ajettu, koodeja kirjoitettu ja tietoa etsitty. Itse raportin kirjoittamiseen käytetty aika ei sisälly tähän. Modulia ja tätä raporttia on työstetty aikavälillä 30.11.-3.12.2024.

## a) Oma miniprojekti

### Env-virtuaaliympäristön uudelleensijoittaminen

*0:00*

Jatkoin edellisen raporttini (ks. LeeviRaussi 2024c) pohjalta oman moduulini työstämistä. Torstain luennolla saamani vinkit saivat minut muuttamaan Virtualenv-virtuaaliympäristön asennuksen sijaintia suoraan käyttäjän "weppimasteri" omaan kotihakemistoon. Huomasin, että olin näin tehnyt jo Linux-palvelimet-kurssilla (ks. LeeviRaussi 2024a), mutta olin tällä kertaa sijoittanut hakemiston verkkosivuille varattuun hakemistoon. En uskonut, että tällä olisi niin suurta väliä, mutta päätin tehdä muutoksen silti hakemistorakenteen selkiyttämisen takia.

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

Koska olin jo edellisessä raportissani (ks. LeeviRaussi 2024c) tehnyt nyt tekemäni asiat, sain suoraviivaistettua asioita hyvin. Siirryin perusta-sivujen hakemistoon (`cd public_html/perusta/`), jossa aloitin uuden Django-projektin (`django-admin startproject modduli`). Tarkastettuani, että uusi hakemisto oli luotu, siirsin evakossa olleen /static/-hakemiston takaisin conf-tiedoston määrittelemään sijaintiin (`mv /home/weppimasteri/public_html/perusta/static/ /home/weppimasteri/public_html/perusta/modduli/`). Itse conf-tiedostoa en lähtenyt muokkaamaan, koska siellä kaiken pitäisi olla oikein. Tämän jälkeen siirryin pois env-ympäristöstä (`deactivate`).

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

Tajusin, etten ollut koskaan luonut env-hakemistoja, joihin yritin juuri viitata, joten näiden kahden komennon epäonnistuminen olikin odotettua. Kuitenkin Apachen virhettä en täysin ymmärtänyt, koska asia oli toiminut edellisellä kerralla (ks. LeeviRaussi 2024c), eivätkä nyt tekemäni muutokset conf-tiedostoon olleet mielestäni kovin radikaaleja. Koitin ajaa uudelleen komennon `sudo salt '*' state.apply modduli`, mutta Apache ei edelleenkään tehnyt yhteistyötä. Lähdin selaamaan Saltin manuaalia ja erityisesti watch-lisäosaan keskittyvää osiota (ks VMware 2024a). Täältä sain idean, että mahdollisesti ongelma johtui järjestyksestä, missä asiat olivat sls-tiedostossa. Muutiksin Apacheen liittyvän service.running-tilafunktion sijainnin heti asennuksen perään.

![21 new place](https://github.com/user-attachments/assets/60beb86f-8c63-431a-9f34-404e8bb45816)

Tämäkään ei kuitenkaan ratkaissut ongelmaa. Avasin toisen komentorivin ja kirjauduin sillä orjana olevalle t002:lle, jossa ajoin komennot `systemctl status apache2.service` ja `sudo journalctl -xeu apache2.service`, jotka oli mainittu Saltin virheilmoituksen yhteydessä. Jälkimmäisen avulla sain näkyviini tietoja syntaksivirheistä, joita conf-tiedostossa esiintyi.

![22 journalctl apache2](https://github.com/user-attachments/assets/d7e6d4cf-82b7-43ea-9d62-9db13a4e53e7)

Erikoisesti ensimmäinen vastaava ilmoitus oli päivätty 27.11. noin klo 11, jolloin en ole kuitenkaan ollut minkäänlaisessa kosketuksessa koko koneeseen (viimeistelin edellisen raportin 26. päivän illalla), joten vaikutti siltä, että jotain olisi hajonnut itsestään sen jälkeen, kun olin saanut edellisessä raportissani (ks. LeeviRaussi 2024c) asiat päätökseen.

Syntaksiongelma johtui todennäköiseti siitä, että käytin conf-tiedostossa asetuksia "tulevaisuudesta". Päätin luoda tätä tilannetta varten uuden conf-tiedoston (perusta2.conf), jonka sisällön otin suoraan edellisen raporttini manuaalisesta osuudesta (ks. LeeviRaussi 2024c) ennen kuin aloin asentelemaan Djangoa. Tämä vaati myös muutosten tekemistä sls-tiedostoon.

![23 perusta2](https://github.com/user-attachments/assets/ded66575-d0cd-4f4f-897c-b066d4ad2156)

Tämän väliaikaisen ratkaisun ansiosta Apache lähti jälleen toimimaan ja sls-tiedosto muuttui idempotentiksi toisella ajolla.

![24 salt perusta2 working](https://github.com/user-attachments/assets/1964ed6c-30b6-43a5-ae50-5557126731c8)

![25 salt perusta2 idempotent](https://github.com/user-attachments/assets/dd6e546e-729f-4c52-9f2e-fca845f24b77)

*1:17*

Pääsin sitten tietyssä mielessä jatkamaan siitä, mihin edellisessä raportissani (ks. LeeviRaussi 2024c) jäin. Lisäsin edellä kirjoittamieni ja sittemin kommentoimieni env-virtuaaliympäristökomentojen eteen komennon, jolla kyseinen ympäristö luotaisiin. Vaatimuksena olevalla `creates`-parametrilla pyrin siihen, että hakemisto luotaisiin weppimasterin kotihakemistoon.

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

*2:55*

Pitkällisen mietinnän ja tiedon etsimisen jälkeen päätin yrittää rakentaa yhtenäisen kokonaisuuden env-virtuaaliympäristön aktivoimisesta, Djangon asentamisesta ja Django-projektin aloituksesta. Saltin cmd-tilafunktion manuaalin (VMware 2024b) avulla lähdin hakemaan parametreja, joita uskoin tarvitsevani `cmd.run`-komennon kanssa. Lisäksi tutkin require-parametrin toimintaa Requisites manuaalista (VMware 2024a). Huomioiden, miten esim. tiedosto ja paketti vaatimukset oli merkitty, tein arvauksen, että komentojen ehdon kirjoitusmuoto olisi "cmd". Lisäksi, koska tarvitsisin requirements-tekstitiedostoa Djangon asentamiseen, tein tätä varten file.managed-tilafunktion. Lisäksi kommentoin edellä kirjoittamani activate- ja deactivate-koodipätkät.

![42 startproject require](https://github.com/user-attachments/assets/547abf2a-ab8f-4e6d-b6ed-44b2a7a7c977)

Tutut komennot `sudo systemctl restart salt-master.service` ja `sudo salt '*' state.apply modduli` näyttivät, ettei homma mennyt putkeen.

![43 require missing](https://github.com/user-attachments/assets/34d68731-a485-460a-8b31-d36ab7682a26)

Saadun virheilmoituksen perusteella tulkitsin, että komentojen esiintyminen ei pelkästään riittäisi, vaan ne tulisi erottaa myös omiksi komennoikseen, joten tein näin.

![44 require separate](https://github.com/user-attachments/assets/e0d00ee8-7738-4cc1-b310-521678734d9a)

Tuloksena activate-komento onnistui odotetusti, mutta Djangon asennus epäonnistui, minkä myötä myös kokonaisuus epäonnistui.

![45 pip fail](https://github.com/user-attachments/assets/858363ff-49e0-4c89-9446-0040b56fdad1)

Epäilin, että ongelma liittyi jälleen virtuaaliympäristöön, koska en edelleenkään uskonut, että env-tilassa pysyttiin onnistuneesti. Kommentoin kokonaan edellä kirjoittamani asennuskokonaisuuden sekä deactivate-koodipätkän ja lisäsin testimielessä `which pip`-komennon koodiin.

![46 which pip](https://github.com/user-attachments/assets/473ac1ed-76e5-4881-93d6-b05d363d72d1)

Tämäkin epäonnistui, joskin mitään selvää virheilmoitusta ei tällä kertaa tullut toisin kuin edellisissä kohdissa.

![46 5 which pip fail](https://github.com/user-attachments/assets/fd3c2d31-b9e2-4bd8-a8ad-d0e15b61934f)

Muokkasin koodin luomaan tekstitiedoston, johon tulostuisi komennon sisältö. Ajattelin tällä tavalla saavani selville Pipin version.

![47 which txt](https://github.com/user-attachments/assets/443c2307-e829-4a3a-9813-29aabfb9cffe)

Tämäkin epäonnistui.

![48 which txt fail](https://github.com/user-attachments/assets/01b6a60a-a230-428c-9f69-15e6cc964877)

Tarkastin t002:lla miltä asia näytti sillä puolella, ja vaikka tekstitiedosto oli luotu, ei sillä ollut minkäänlaista sisältöä.

![49 txt no content](https://github.com/user-attachments/assets/b722d52d-78b7-47bf-9a34-468b4cbd256e)

Tämä viittasi mielestäni selvästi siihen, että ajettaessa komento `which pip` ei oltu enää env-ympäristössä. Ongelma olisi siis, että Saltilla ajettu komento `. /home/weppimasteri/env/bin/activate` oli vain hetkellinen eikä pidempään jatkuva. Etsin Googlen avulla apua vaihtoehtoiseen toteutukseen ja päädyin Saltin manuaaliin virtualenv-tilafunktiosta (VMware 2024c). En saanut pelkällä manuaalilla täydellistä kuvaa, miten se toimisi todellisuudessa, joten yritin etsiä Googlella lisää apua tässä kuitenkaan onnistumatta. Tämän takia lähdin tekemään omaa viritystäni aiheesta manuaalin pohjalta. Kommentoin myös muut edellä kirjoittamani koodipätkät, jotka saattoivat liittyä asiaan.

![50 virtualenv_mod managed](https://github.com/user-attachments/assets/3e0279d8-7aaf-4e3b-8a9e-be865e84b971)

![51 mod not available](https://github.com/user-attachments/assets/3bfc09bc-72c3-413c-8ca5-f72fe05b7581)

Uusi virheviesti, jonka syytä en saanut tulkittua Googlen avulla. Hieman sivuavaa keskustelua aiheesta tuli vastaan, mutta kaikki selaamani sivut tuntuivat käsittelevän vähän eri asiaa. Päädyinkin muokkaamaan hieman name-parametrin polkua, koska ajattelin ymmärtäneeni manuaalin (VMware 2024c) väärin.

![52 mod name change](https://github.com/user-attachments/assets/82dbef28-8e05-4a61-9dc3-806aa4136340)

Sama virheviesti jatkui edelleen, joten jatkoin Googlen selaamista. Kiinnitin huomiota tässä vaiheessa siihen, että vaikka olin käyttänyt hakusanaa "virtualenv_mod" sain hakutuloksissa suhteellisen paljon pelkkää "virtualenv" muotoa. Toisaalta myös manuaalin "VMware 2024c) sivun nimi oli jälkimmäisessä muodossa, vaikka sivulla olevissa komennoissa oli käytetty mod-muotoa. Päätin muokata tämän omassa koodissani pelkkään "virtualenv"-muotoon.

![53 virtualenv change](https://github.com/user-attachments/assets/3738f323-f881-45fc-a103-b65794b062f5)

Virheviesti muuttui tällä kertaa, joten tältä kannalta muutos lieni oikea.

![54 virtualenv fail](https://github.com/user-attachments/assets/d0686c49-a06e-446d-91d1-1ca53a44ac6f)

Koska virheilmoituksessa valitettiin sitä, että kohde oli jo olemassa, muutin nimenä olevana olevaa polkua.

![55 virtualenv name change](https://github.com/user-attachments/assets/aab1410a-ffe0-429d-81c3-2919eee3eacc)

Taas uusi virheviesti. Tällä kertaa syynä oikeuksien puuttuminen kyseiseen kansioon.

![55 5 env bin fail](https://github.com/user-attachments/assets/97dfe64c-c2f3-410f-bf2b-84855f5c64ea)

Muutin sijainniksi nyt kotihakemiston, minkä lisäksi lisäsin manuaalista (VMware 2024c) venv_bin-parametrin.

![56 venv](https://github.com/user-attachments/assets/a243fb7b-f49e-4d7d-b9ea-85c371a07d04)

Yhteydessä oli viivettä, joten odottelin hetken ennen kahden tutun komennon ajamista uudelleen. Lopputuloksena oli erittäin pitkä virheviesti, joka ei mahtunut kokonaisuudessaan ruudulleni.

![57 lag](https://github.com/user-attachments/assets/3a1e4c7d-f0cd-43b7-ae9b-113c7fddb36f)

![58 module missing](https://github.com/user-attachments/assets/886e08f9-904e-4778-97fd-80242ea547d1)

### Modulin viimeistely

*5:02*

Koska minulla ei ollut minkäänlaista ideaa, miten ratkaista ongelma, ja deadlinen lähestyessä päätin luovuttaa virtuaaliympäristön rakentamisen ja koota nykyisestä toteutuksestani toimiva, idempotenttinen kokonaisuus. Päätin, että koko prosessi olisi hyvä aloittaa palomuurin, `curl`-komennon ja Micro-editorin asentamisilla, koska näillä pääsee hyvään alkuun verkkosivujen suojauksen ja testaamisen kanssa. Googlen avulla löysin Boucheronin, Camisson ja Abidin (2024) artikkelin, josta uskoin saavani tietooni palomuurin asetusten sijainnin, ja artikkelin alusta bongasin heti sijainnin `/etc/default/ufw`, jonka sisällön tarkistin. Tämä ei kuitenkaan sisältänyt avaamiani portteja, joten en uskonut olevani oikeassa paikassa. Mieleeni tuli sitten, että Apachen asetukset olivat hakemistossa `/etc/apache2/`, joten tarkastin vastaavan ufw-hakemiston sisällön.

![59 ls etc ufw](https://github.com/user-attachments/assets/58d639e4-2f78-4357-aaba-bc44fd358520)

Täältä tarkastin tiedostojen sysctl.conf ja ufw.conf sisällöt, mutta kumpikaan ei näyttänyt oikealta. Jälleen Googlen kautta löysin Bennettin (2013) aloittaman keskustelun, josta bongasin sijainnin `/lib/ufw/`

![60 ls lib ufw](https://github.com/user-attachments/assets/d4cdd24f-afcf-41c7-8073-85849a2178a2)

Nämä tiedostot näyttivät olevan vielä enemmän väärässä, mutta samaisesta keskustelusta (Bennett 2013) alempaa huomasin, että asetukset löytyisivät tiedostosta user.rules, joka oli ollut `/etc/ufw/`-hakemistossa. Tarkastin sisällön ja totesin tiedoston oikeaksi, joten kopioin sen moddulin omaan hakemistoon. Lisäsin myös moddulin sls-tiedostoon ufw-asennukseen, jonka jälkeen ajoin salt-komennon onnistuneesti.

![61 cp user rules](https://github.com/user-attachments/assets/c2b865fb-ae29-4dec-b710-8ac71c991ad7)

![62 salt ufw](https://github.com/user-attachments/assets/004dc9b1-2cdf-4a6b-8f0c-7718344b6332)

Lisäsin sitten asetustiedoston siirron oikeaan hakemistoon orjakoneella, jonka jälkeen toistin jälleen moddulin ajon.

![63 add file managed user rules](https://github.com/user-attachments/assets/adc428ac-a4f7-490e-84a4-e2ec5b582243)

![64 salt user rules fail](https://github.com/user-attachments/assets/57461ec5-e389-496a-843e-04c8a6c9bbbc)

Virheviesti oli erittäin outo, sillä tiedosto oli selvästi oikeassa paikassa.

![65 it is there](https://github.com/user-attachments/assets/08d0eb42-e304-4f5a-8306-6ae5ddcb5b17)

Uudella yrityksellä sain saman virheviestin. Loin hakemistoon uuden tiedoston testi.txt, ja muokkasin tämän tiedoston siirtymään user.rules sijasta kohteeseen.

![66 new file](https://github.com/user-attachments/assets/3dde66e2-3d01-43b7-b6eb-6ec1de6c034d)

![67 new file works](https://github.com/user-attachments/assets/68f1a2ec-dabe-49e8-8fbd-926d4c8073b2)

Tämä toimi, joten polussa ei ainakaan näyttänyt olevan vikaa. Samalla myös t002-koneen user.rules-tiedosto käytännössä tyhjeni, koska en ollut muuttanut kohdetiedoston polkua. Muokkasin taas kerran sls-tiedostoon user.rulesin mukaan ja yritin epäonnistuen taas salt-komennon ajon.

![68 back to og](https://github.com/user-attachments/assets/341c74e7-24b9-4486-b3fb-f9cf3616fcf4)

![69 og not working](https://github.com/user-attachments/assets/c45c67fa-0b1c-4dff-b83b-ee47862c3ff5)

Ajattelin, että ehkä ufw jotenkin estää tiedoston muokkaamisen, koska se on päällä [Jälkihuomio: testi.txt tosin muokkasi tiedostoa ihan hyvin vain hetkeä aiemmin...], joten suljin sen suoraan t002-koneelta.

![70 ufw disable](https://github.com/user-attachments/assets/494aa27d-0c24-45dc-b364-d8871967b109)

Tämä ei [tietenkään] vaikuttanut salt-komennon ajamiseen.

![71 still not working](https://github.com/user-attachments/assets/8e9d2d51-a523-4dea-80f2-dd02746ed927)

![72 ufw not active](https://github.com/user-attachments/assets/42a52984-afec-40e5-9b2a-aa7069e40270)

Korjasin tässä kohtaa manuaalisesti edellä "tuhoamani" user.rules-tiedoston avaamalla manuaalisesti portit 22, 4505 ja 4506.

![73 fix rules](https://github.com/user-attachments/assets/6957f017-be72-46b3-b511-458b48c4941a)

Googlasin apua ongelmaan ja löysin CanItFryn (2021) keskustelun Redditistä, joka käsitteli täysin samaa ongelmaa (joskin hieman eri konfiguraatiolla). Valitettavasti ongelmaan ei ollut keskustelussa saatu ratkaisua, mutta CanItFry oli kirjoittanut vaihtoehtoisen menetelmän kiertää ongelma. Koska tämä ratkaisu ei ollut mitenkään elegantti, päätin yrittää vielä ongelman ratkaisemista itse ennen kuin turvautuisin keskustelun purkkaratkaisuun. Koska edellä testi-tekstitiedoston siirto oli onnistunut korvaamaan user.rules-tiedoston, ajattelin, että voisin hyödyntää tätä. Kopioin user.rules-tiedoston user-tekstitiedostoksi. Lisäksi CanItFryn (2021) keskustelun pohjalta siirsin hakemistoon `/etc/ufw`-hakemistosta myös user6.rules- ja ufw.conf-tiedostot.

![74 user txt](https://github.com/user-attachments/assets/6cf1d851-ef04-4bc9-8a25-4b7a95598592)

![75 user6 txt](https://github.com/user-attachments/assets/7a2d62fa-4784-4114-811a-6cfbd113b8bd)

![76 cp ufw conf](https://github.com/user-attachments/assets/445e149c-71ef-4881-bb0c-3bc33dad122e)

Muokkasin sls-tiedostoon uudet tiedostopolut, jonka jälkeen ajoin moddulin salt-komentona.

![77 changed sls](https://github.com/user-attachments/assets/6c480fb7-095d-4bc6-97d9-a01cc153bd46)

![78 still not working](https://github.com/user-attachments/assets/f8e85edc-5358-4c1e-97a4-01d4bebbf19a)

Edes uudet tekstitiedostot eivät ratkaisseet ongelmaa. Hetken pohdinnan (josko luovuttaisin tämänkin) jälkeen ajoin komennon `ls -l`, koska halusin saada lisää tietoa hakemiston tiedostoista.

![79 missing r](https://github.com/user-attachments/assets/97f726ff-8fb7-494b-8eb9-1f338792eb43)

Muilta puuttui juuri nikotteleviin tiedostoihin lukuoikeudet. En tiennyt, johtuiko ongelma tästä, mutta suhtauduin asiaan siten, että ongelma todennäköisesti johtui juuri tästä. Ilmeisesti lukuoikeuksia näihin tiedostoihin ei ole annettu niiden luonnin yhteydessä, mutta toisaalta samassa hakemistossa olleeseen ufw.conf-tiedostoon lukuoikeudet oli annettu, joten selvää sääntöä en pystynyt hahmottamaan. Annoin lukuoikeudet kahdelle tiedostolle, minkä lisäksi poistin turhaksi käyneen user-tekstitiedoston sekä nimesin user6-tekstitiedoston uudelleen.

![80 add r](https://github.com/user-attachments/assets/571cf209-49e3-4887-a7bf-3f81f80cb135)

Korjasin sls-tiedoston polunnimet (tosin unohdin user6-tiedostosta numeron 6) ja ajoin moddulin taas kerran salt-komentona.

![81 fix sls](https://github.com/user-attachments/assets/6b7c51f4-07a5-4913-b33c-5a06954d10b8)

![82 salt working](https://github.com/user-attachments/assets/b28f7e78-0cc5-48d5-b4d1-d38f6a86e00b)

Homma toimi viimein ja lopputulos oli myös idempotentti.

![83 salt idempotent](https://github.com/user-attachments/assets/ff6596c8-c28d-458c-87e5-341bfdc6e6a0)

Palomuuri ei kuitenkaan ollut mennyt päälle t002:lla.

![84 ufw inactive](https://github.com/user-attachments/assets/42bd7213-31c1-477b-a303-457d67b934e0)

Hyödynsin tähän ongelmaan CanItFryn (2021) ratkaisua lisäten `ufw enable`-komennon cmd.run-tilafunktiona. Lisäksi lisäsin watch-parametrin ufw:n tarkkailemaan, mikäli muutoksia sen asetuksiin tehtäisiin.

![85 fixed sls](https://github.com/user-attachments/assets/3dcb7344-2b7b-458e-a5d7-6b645b8c7fea)

Salt-komento ajettiin onnistuneesti ensimmäisellä kerralla, mutta idempontettitestissä ilmeni jälleen kerran hidastelua.

![86 salt working](https://github.com/user-attachments/assets/d6a3015f-f118-4434-9c8a-153be0133022)

![87 idempotent time out](https://github.com/user-attachments/assets/4277ed09-6e35-45d8-9236-0c7f1f8fde2a)

Hetken odottelun jälkeen tarkastin t002:lla komennolla `sudo ufw status`, mikä palomuurin tila oikein oli.

![88 ufw active](https://github.com/user-attachments/assets/5058fd62-c040-4273-adc3-fe09d754f7f6)

Kaikki näytti olevan kunnossa, joten odotin vielä hetken ennen kuin ajoin komennon `sudo systemctl restart salt-master.service`, jota seurasin modulin "hello" ajolla onnistuneesti.

![89 salt hello](https://github.com/user-attachments/assets/32b8b105-cc3d-4981-a954-76695095c16c)

Poistin hello-modulilla tehdyt asiat ajamalla poistahello-modulin (`sudo salt '*' state.apply poistahello`), minkä jälkeen käynnistin salt-masterin taas uudestaan (`sudo systemctl restart salt-master.service`). Nyt ajettu moddulin ajo salt-komennolla (`sudo salt '*' state.apply modduli`) toimi ja tulos oli idempotentti.

![90 modduli idempotent](https://github.com/user-attachments/assets/d5fa4fee-19be-43b2-97ec-7a4dd6dc53c9)

Lisäsin sls-tiedostoon curl- ja micro-pakettien asennukset, mikä ei muuttanut idempotenttitilaa.

![91 curl micro](https://github.com/user-attachments/assets/b6c39fa8-3484-49f6-9d0b-6afc03974d9d)

![92 still idempotent](https://github.com/user-attachments/assets/a2de95fc-befd-4e89-b076-b814b8e136f0)

*6:29*

Vaikken saanut automatisoitua koko prosessia, halusin edes jättää ohjeet siitä, miten maaliin asti päästäisiin (toivottavasti). Näin ollen tarkoitukseni oli saattaa prosessi loppuun manuaalisesti t002:n weppimasterilla ja kirjoittaa tämän pohjalta ohjeet, jotka jaan modulin mukana. Tätä varten lisäsin sls-tiedostoon file.managed-tilafunktion perusta.conf-tiedostolle, jota käytetään lopullisessa konfiguraatiossa.

![93 perusta conf](https://github.com/user-attachments/assets/1b5f07be-7f01-4bcc-b6d1-f761748b1418)

Komennon `sudo salt '*' state.apply modduli` ajo oli suoraan idempotentti, koska olin jo aiemmin projektissa käyttänyt kyseistä tiedostoa.

![94 perusta conf idempotent](https://github.com/user-attachments/assets/f93c85c4-0fef-4736-bf3d-6a1a44fa8636)

Otin sitten SSH-yhteyden t002:n weppimasteriin tarkoituksenani saattaa asennusprosessi maaliin manuaalisesti. Siirryin aiemmin löytämälläni vaihtoehtoisella komennolla `. env/bin/activate` env-ympäristöön, jossa koitin ajaa komennon `pip install -r requirements.txt`. Asennus väitti onnistuneensa, mutta eteeni tuli silti varoituksia, joita en ollut saanut aiemmissa manuaalisissa asennuksissani (vrt. LeeviRaussi 2024c). Myös perään ajamani testi `django-admin --version` epäonnistui, mikä lisäsi skeptisyyttäni asiaan. Tutkin varoituksissa mainittua hakemistoa, mutta erinäisten kokeilujen jälkeenkään en tullut yhtään viisaammaksi.

![94 5 django problems](https://github.com/user-attachments/assets/c26a15a6-5d33-4acb-bbdf-0944dd288d70)

Huomioiden, että t001:n weppimasterin (luotu `adduser`-komennolla) ja t002:n weppimasterin (luotu Saltin `user.present`-tilafunktiolla) näkymät ovat myös hyvin erilaiset, epäilen, että tässä on jotain rikki, mikä liittynee `user.present`-tilafunktion käyttöön, mutten tiedä mikä siinä sitten on rikki. On siis mahdollista, että vaikka modduli toimii idempotenttisesti, sen taustalla on jotain perustavanlaatuisesti hajalla.

Päätin kuitenkin jatkaa ideaani ohjeiden kirjoittamisesta mahdollista käyttäjää ajatellen. Nyt ohjeet vain tulisivat perustumaan aiempaan manuaaliseen asennukseeni. Loin todo.md-tiedoston (https://github.com/LeeviRaussi/modduli/blob/main/modduli/todo.md), minkä lisäksi tiedoston kirjoituksen yhteydessä tajusin myös kopioida settings.py tiedoston moddulin hakemiston sisälle (`sudo cp /home/weppimasteri/public_html/perusta/modduli/modduli/settings.py /srv/salt/modduli`), koska tarvitsisin sitäkin. Lisäsin sls-tiedostoon molemmille tiedostoille file-tilafunktiot.

![95 todo md and settings py](https://github.com/user-attachments/assets/b08d290f-5d35-41de-9c8e-49b640515ddb)

Salt-masterin uudelleenkäynnistys (`sudo systemctl restart salt-master.service`) ja kaksi kertaa modulin ajo (`sudo salt '*' state.apply modduli`) näyttivät, että kaikki toimii idempotenttisesti.

![96 working](https://github.com/user-attachments/assets/fe505e4e-deac-4da4-9b18-53dd15c6195c)

![97 idempotent](https://github.com/user-attachments/assets/76b0b696-bb37-4874-8dea-2731e367faaf)

*7:29*

Viimeistelin tehtävän luomalla uuden virtuaalikoneen Vagrantilla. Lisäsin Vagrantfileen pitkälti kopioi-liitä-tekniikalla uuden koneen tiedot, joskin muutin koneen nimeksi t003 ja IP-osoitteen yksilöin.

![98 t003](https://github.com/user-attachments/assets/ac566bd3-43b6-405d-8da8-529dead18f56)

Komennot `vagrant reload` ja `vagrant up`, joiden jälkeen kirjautuminen komennolla `vagrant ssh t003` onnistui ongelmitta. Salt-minionin asennus piti tehdä koneelle ensin manuaalisesti, mutta seurasin tässä kuuliaisesti aiempaa raporttiani (ks. LeeviRaussi 2024b), josta sain selvät ohjeet ja komennot prosessiin. Ajoin komennot `sudo mkdir -p /etc/apt/keyrings` ja `curl -fsSL https://packages.broadcom.com/artifactory/api/security/keypair/SaltProjectKey/public | sudo tee /etc/apt/keyrings/salt-archive-keyring.pgp`, mutta jälkimmäinen näistä epäonnistui, koska koneella ei ollut `curl`-komentoa. Asensin tämän `sudo apt-get update` ja `sudo apt-get install -y curl` komennoilla, tosin tässä periaatteessa suoritetaan jo yksi modulin tehtävistä.

![99 prepare salt-minion](https://github.com/user-attachments/assets/40a2981d-dacb-48c0-86a9-126643984719)

Tämän jälkeen ajoin onnistuneesti komentosarjan

```
curl -fsSL https://packages.broadcom.com/artifactory/api/security/keypair/SaltProjectKey/public | sudo tee /etc/apt/keyrings/salt-archive-keyring.pgp
curl -fsSL https://github.com/saltstack/salt-install-guide/releases/latest/download/salt.sources | sudo tee /etc/apt/sources.list.d/salt.sources
sudo apt-get update
sudo apt-get install salt-minion
sudo systemctl enable salt-minion && sudo systemctl start salt-minion
sudo salt-call --version
sudoedit /etc/salt/minion
```

joilla asensin niin Saltin kuin Salt-minionin koneelle, sekä lopuksi muokkasin orjan herran IP-osoitteeksi t001:n IP-osoitteen.

![100 master ip](https://github.com/user-attachments/assets/10c86fa0-3854-49fa-9678-147dcb98835e)

Hyväksyin orjan avaimen t001:llä komennolla `sudo salt-key -A`.

![101 accept key](https://github.com/user-attachments/assets/fdbe8ed1-fca5-4100-addc-56c311659cde)

Tämän jälkeen käynnistin salt-masterin varmuudeksi uudelleen (`sudo systemctl restart salt-master.service`) ja ajoin moddulin salt-komennon kohteenani pelkkä t003 (`sudo salt t003 state.apply modduli`). En yllättynyt kauheasti, kun sain tutun timeout-ilmoituksen.

![102 time out modduli](https://github.com/user-attachments/assets/ca688c3c-0cef-43d1-b1fc-550b56a80290)

Odottelin noin viitisen minuuttia, jonka jälkeen koitin kahta edellistä komentoa uudelleen, mutta sain taas saman ilmoituksen. Odottelin vielä muutaman minuutin, minkä jälkeen vilkaisin miltä t003 puolella näyttää, ja siellä vaikutti kaikki olevan kunnossa.

![103 files in](https://github.com/user-attachments/assets/7131b219-a793-4dcd-8951-f4790bb961c1)

Vielä kerran komennot `sudo systemctl restart salt-master.service` ja `sudo salt t003 state.apply modduli` ja tällä kertaa sain suoraan ilmoituksen, että modduli ei ollut tehnyt enää mitään muutoksia, vaan se oli idempotentti.

![104 idempotent](https://github.com/user-attachments/assets/c6cadc66-16ca-4be7-990d-aae1e2be5456)

*7:50*

Tämän myötä koin saaneeni modulini "valmiiksi", joskaan varsinaisesti tyytyväinen lopputulokseen en voinut sanoa olevani. Linkki Github-varastoon, josta moduli on saatavilla, löytyy alta tehtävän b) kohdalta.

## b) Etusivu

Modulille on luotu Github-varasto osoitteeseen https://github.com/LeeviRaussi/modduli.

## Lähdeluettelo

Bennett, S. 2013. Where does UFW (uncomplicated firewall) save command-line rules to? Luettavissa: https://serverfault.com/questions/475468/where-does-ufw-uncomplicated-firewall-save-command-line-rules-to. Luettu: 3.12.2024.

Boucheron, B., Camisso, J. & Abid, E. 2024. How to Set Up a Firewall with UFW on Ubuntu. Luettavissa: https://www.digitalocean.com/community/tutorials/how-to-set-up-a-firewall-with-ufw-on-ubuntu. Luettu: 3.12.2024.

CanItFry 2021. failing to configure ufw with salt. Luettavissa: https://www.reddit.com/r/saltstack/comments/rbqd7w/failing_to_configure_ufw_with_salt/. Luettu: 3.12.2024.

larry 2013. How can I activate a virtualenv in Linux?. Luettavissa: https://stackoverflow.com/questions/14604699/how-can-i-activate-a-virtualenv-in-linux. Luettu: 1.12.2024.

LeeviRaussi 2024a. h6 Hello Django. Luettavissa: https://github.com/LeeviRaussi/linux-palvelimet/blob/main/h6_Hello_Django.md. Luettu: 30.11.2024.

LeeviRaussi 2024b. h2 Infra-as-code. Luettavissa: https://github.com/LeeviRaussi/Palvelinten_hallinta/blob/main/h2_Infra-as-code.md. Luettu: 3.12.2024.

LeeviRaussi 2024b. h4 Puolikas. Luettavissa: https://github.com/LeeviRaussi/Palvelinten_hallinta/blob/main/h4_Puolikas.md. Luettu: 3.12.2024.

VMware, Inc. 2024a. Requisites and Other Global State Arguments. Luettavissa: https://docs.saltproject.io/en/latest/ref/states/requisites.html. Luettu: 2.12.2024.

VMware, Inc. 2024b. salt.states.cmd. Luettavissa: https://docs.saltproject.io/en/latest/ref/states/all/salt.states.cmd.html. Luettu: 2.12.2024.

VMware, Inc. 2024c. salt.states.virtualenv. Luettavissa: https://docs.saltproject.io/en/latest/ref/states/all/salt.states.virtualenv_mod.html. Luettu: 2.12.2024.
