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

Jatkoin edellisen raporttini (ks. LeeviRaussi 2024b) pohjalta oman moduulini työstämistä. Torstain luennolla saamani vinkit saivat minut muuttamaan Virtualenv-virtuaaliympäristön asennuksen sijaintia suoraan käyttäjän "weppimasteri" omaan kotihakemistoon. Huomasin, että olin näin tehnyt jo Linux-palvelimet-kurssilla (ks. LeeviRaussi 2024a), mutta olin tällä kertaa sijoittanut hakemiston verkkosivuille varattuun hakemistoon. En uskonut, että tällä olisi niin suurta väliä, mutta päätin tehdä muutoksen silti.

Aloitin poistamalla env-hakemiston saltin avulla komennolla `sudo salt-call --local state.single file.absent /home/weppimasteri/public_html/env`.

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

Koska en hahmottanut, mistä ongelma johtui, ja koska loppupeleissä vanhan toistoa ei ollut niin paljoa, päätin tuhota koko aiemman rakennelmani ja tehdä moddulin django-sivut uudelleen. Otin talteen /static/-sivun tiedostot (`mv /home/weppimasteri/public_html/perusta/modduli/static/ /home/weppimasteri/public_html/perusta/`), jonka jälkeen siirryin takaisin pääkäyttäjälle ajamaan vaadittavan Salt-komennon (`sudo salt-call --local state.single file.absent /home/weppimasteri/public_html/perusta/modduli`).

![7 mv static](https://github.com/user-attachments/assets/d0cd3751-4b25-4644-83b6-9a1fcb3552ba)

![8 modduli absent](https://github.com/user-attachments/assets/eed8e6fd-8c59-4ace-8d7d-60d874e4e894)

*0:18*

Siirryin ensiksi kotihakemistosta hakemistoon perusta (`cd public_html/perusta/`), jossa testasin saada selville Djangon-versionumeron (`django-admin --version`). Tämä ei kuitenkaan toiminut.

![9 no django](https://github.com/user-attachments/assets/45c1911b-ab73-4ec8-a8d0-4fc082e88db0)

Ajattelin, että Django oli ollut jotenkin sidottuna virtuaaliseen ympäristöön, joten uudelleen tehty asennus olisi vaikuttanut myös siihen. Palasin käyttäjän kotihakemistoon, jonne siirsin myös django-tekstin sisältävän requirements-tekstitiedoston (`mv public_html/requirements.txt /home/weppimasteri/`). Siirryin tämän jälkeen env-virtuaaliympäristöön (`source env/bin/activate`) ja asensin Djangon jälleen kerran komennolla `pip install -r requirements.txt`.

![10 django install](https://github.com/user-attachments/assets/41fe820a-3f23-497c-8497-26edf1f4faaf)

[Jälkihuomio raporttia kirjoittaessa: Edellä olleeseen vaiheeseen liittyen raporttia kirjoittaessa ajoin uudelleen komennon `django-admin --version` niin ei-env-ympäristössä kuin myös env-ympäristössä ollessani hakemistossa `/home/weppimasteri/public_html/perusta/modduli`. Ei-env-ympäristössä ajaettu komento ei taaskaan tunnistanut komentoa, kun taas env-ympäristössä ajettu komento tuntui jäävän jonkinlaiseen looppiin, jonka jouduin keskeyttämään hetken odottelun jälkeen CTRL+C yhdistelmällä. Siirryin tämän jälkeen `cd`-komennolla takaisin kotihakemistoon, jossa olin tehnyt Django-asennuksen, mutta täälläkään versionumeroa hakeva komento ei toiminut, vaan jäi jälleen looppiin. Verkkosivut toimivat kuitenkin oikein `curl`-komennolla, joten tämä asia jää itselleni nyt pieneksi mysteeriksi.]

Koska olin jo edellisessä raportissani (ks. LeeviRaussi 2024b) tehnyt nyt tekemäni asiat, sain suoraviivaistettua asioita hyvin. Siirryin perusta-sivujen hakemistoon (`cd public_html/perusta/`), jossa aloitin uuden Django-projektin (`django-admin startproject modduli`). Tarkastettuani, että uusi hakemisto oli luotu, siirsin evakossa olleen /static/-hakemiston takaisin conf-tiedoston määrittelemään sijaintiin (`mv /home/weppimasteri/public_html/perusta/static/ /home/weppimasteri/public_html/perusta/modduli/)`. Itse conf-tiedostoa et lähtenyt muokkaamaan, koska siellä kaiken pitäisi olla oikein. Tämän jälkeen siirryin pois env-ympäristöstä (`deactivate`).

![11 django start](https://github.com/user-attachments/assets/8b7a1046-5081-4b43-8544-e59382844f38)

Poistuin env-ympäristöstä, koska ajattelin `sudo apt-get -y install libapache2-mod-wsgi-py3` komennolla ajetun wsgi-lisäosan asennuksen olevan hyvä merkki siitä, että env-tilan käyttö riitti tältä erää. Asennuksen valmistuttua testasin conf-tiedoston syntaksin (`/sbin/apache2ctl configtest`), jossa ei ollut ongelmia. Tämän jälkeen käynnistin Apachen uudelleen (`sudo systemctl restart apache2`), minkä jälkeen testasin `curl`-komennolla onnistuneesti sivujen toimimisen.

![12 wsgi mod install](https://github.com/user-attachments/assets/9a6660bc-3402-4ba6-82f7-144cf5a6afc0)

Muokkasin asetustiedostoon DEBUGin epätodeksi ja sallituksi isännäksi localhostin (`micro modduli/modduli/settings.py`). Ajoin sitten molemmat komennot `touch wsgi.py` ja `sudo systemctl restart apache2`, jottei ongelmia syntyisi. `curl`-komento tuotti jälleen oikeaa tulosta.

![13 debug false](https://github.com/user-attachments/assets/c01fff5d-1ae7-42ab-81b6-a18e7d6ffaa5)

![14 django hidden](https://github.com/user-attachments/assets/73a1657b-0cc4-4926-a509-cfd0d6b545a1)

Muokkasin lisää edellä mainittu asetustiedostoa lisäten sinne käskyn `import root` ja määritelmän `STATIC_ROOT` (tai ainakin näin sen olisi pitänyt mennä).

![15 style additions](https://github.com/user-attachments/assets/5cafca02-6d73-4d24-a909-75468d5957f7)

Koitin sitten suorittaa komennon `./manage.py collectstatic` suoraan nykyisestä hakemistostani, mutta tämä ei onnistunut. Siirryin sitten oikeaan hakemistoon (`/home/weppimasteri/public_html/perusta/modduli`), jossa myöskään yllätyksekseni komento ei toiminut. Tässä vaiheessa kiinnitin enemmän huomiota virheilmoitukseen ja tajusin, että minun pitäisi ajaa komento env-tilassa, josta olin aiemmin poistunut.

![16 wrong env](https://github.com/user-attachments/assets/6d8da9da-1709-47a7-8b99-64d0375d83bc)

Ajoin uudelleen vaaditun komennon, mutta tällä kertaa homma kaatui huonosti määriteltyyn asetustiedostoon.

![16 5 missing static_root](https://github.com/user-attachments/assets/62585aea-2b3d-4247-94f1-2f9f096e8d34)

Tarkistin edellä tekemiäni muokkauksia huomaten, että olin kirjoittanut vahingossa STATIC_Root. Korjaus isoihin kirjaimiin, jonka jälkeen poistuin jälleen env-ympäristöstä. Ilokseni `curl`-komento antoi edelleen oikeaa tulosta.

![17 fixed static_root](https://github.com/user-attachments/assets/39a88cd1-020d-4e98-879f-8ca34f7a8222)

### Automatisoinnin jatkaminen

*0:33*

## Lähdeluettelo

LeeviRaussi 2024a. h6 Hello Django. Luettavissa: https://github.com/LeeviRaussi/linux-palvelimet/blob/main/h6_Hello_Django.md. Luettu: 30.11.2024.

LeeviRaussi 2024b. h4 Puolikas. Luettavissa: https://github.com/LeeviRaussi/Palvelinten_hallinta/blob/main/h4_Puolikas.md. Luettu: 30.11.2024.

VMware, Inc. 2024. salt.states.cmd. Luettavissa: https://docs.saltproject.io/en/latest/ref/states/all/salt.states.cmd.html. Luettu: 30.11.2024.
