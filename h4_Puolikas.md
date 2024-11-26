# h4 Puolikas

> ## Tiivistelmä
>
> Raportissa asennetaan manuaalisesti Djangon kehitysympäristö Debian 12 Bookworm -pohjaiselle virtuaalikoneelle. Tämän jälkeen tätä prosessia automatisoidaan, jotta se voitaisiin ajaa etänä Saltilla herra-orja-infrastruktuuria hyödyntäen.

> ### Käytetty laitteisto
>
> Lenovo Ideapad Pro 5 14APH8 kannettava tietokone
> - Prosessori: AMD Ryzen 7 7840HS
> - GPU: Radeon 780M Graphics (prosessoriin integroitu)
> - RAM: 32 GB, 6400 MT/s
> - OS: Windows 11 Home 23H2
> - Näytön resoluutio: 2880x1800 (175% skaalaus)
> - SSD: 742/951 GB vapaana

> Raportissa esiintyvät aikaleimat viittaavat reaaliseen aikaan, jona komentoja on ajettu ja koodeja kirjoitettu. Itse raportin kirjoittamiseen käytetty aika ei sisälly tähän.

## a) Puolikas

Tehtävässä oli tarkoituksena tehdä ensimmäinen versio kurssilla rakennettavasta omasta modulista. Ajatuksenani oli automatisoida Djangon käyttöönotto Apachella, joten minun piti ensin tehdä tämä manuaalisesti, jonka jälkeen voisin lähteä automatisoimaan prosessin moduliksi. En uskonut, että tulisin saamaan kokonaan valmista automatisoinnin osalta tässä vaiheessa, koska Djangon asennuksessa hyödynnetään virtuaalisia ympäristöjä. Tämä oli kuitenkin odotettuakin, sillä mahdollisiin ongelmiin saisi apua seuraavalla luennolla.

### Manuaalinen asennus

*0:00*

Alustin ensin tilanteeni lähelle puhdasta Apachen asennusta. Otin alas edellisessä raportissani (ks. LeeviRaussi 2024b) luomani sivun (`sudo a2dissite esimerkki.conf`) sekä laitoin päälle muokkaamani Apachen oletussivun (`sudo a2ensite 000-default.conf`). Tämän jälkeen käynnistin Apachen uudelleen komennolla `sudo systemctl restart apache2` ja tarkistin, että oletussivu oli jälleen päällä komennolla `curl localhost`.

![1 a2ensite default](https://github.com/user-attachments/assets/3b75b644-846b-43da-b41d-8089c4059754)

*0:01*

Hyödynsin asennusprosessissa Karvisen (2022) artikkelia Djangon asentamisesta. Jotta välttäisin mahdolliset omistajuuteen liittyvät ongelmat, päätin tehdä asennusprosessin käyttäjältä "weppimasteri" käsin, jolle olin antanut sudo-oikeudet edellisen kirjoittamani raportin jälkeen. Kirjauduin käyttäjätunnukselle komennolla `ssh weppimasteri@192.168.88.101`, minkä jälkeen loin Karvisen raportin mukaiset pohjat uudelle sivulle, jota tultaisiin myös hyödyntämään Djangon kanssa.

![2 base site](https://github.com/user-attachments/assets/09c523bf-255d-4572-969b-aba20035005f)

Komennolla `sudoedit /etc/apache2/sites-available/perusta.conf` loin tätä sivua varten uuden conf-tiedoston.

![3 base site conf](https://github.com/user-attachments/assets/6c4e4ea0-d571-4b0b-b590-ab75ac4b4a33)

Otin Apachen oletussivut alas (`sudo a2dissite 000-default.conf`) ja laitoin uudet sivut päälle (`sudo a2ensite perusta.conf`). Testasin asetustiedoston toimivuutta (`/sbin/apache2ctl configtest`) eikä syntaksissa ollut vikaa. Tämän jälkeen käynnistin Apachen uudelleen (`sudo systemctl restart apache2`) ja yritin tarkastella uutta luomaani sivua (`curl http://localhost/static/`).

![4 403 permissions](https://github.com/user-attachments/assets/f83163b6-232b-4b8b-b3f7-8cb585c588cd)

Ongelmat tuntuivat tutuilta, joten tarkastin Linux-Palvelimet kurssin raportistani (LeeviRaussi 2024a), miten olin ratkonut ne. Lisäsin oikeuksia komennolla `chmod -R ugo+x $HOME $HOME/public_html/perusta/static/index.html`, jonka jälkeen ajoin uudelleen komennon `curl http://localhost/static/`. Huomasin sitten, että olin aiemmassa raportissani (ks. LeeviRaussi 2024a) käyttänyt muotoa `curl localhost/static/`, joten muutin komentoa.

![5 404 not found](https://github.com/user-attachments/assets/cfd82239-560a-49ad-ad48-8c1ad7926f27)

404-virheilmoituksesta ensimmäisenä mieleeni tuli typo conf-tiedostossa, ja sieltähän tietenkin oli tippunut yksi e-kirjain pois, jonka lisäsin sinne.

![6 typo fix](https://github.com/user-attachments/assets/988f3fee-8281-4277-91ce-1df1b3800f76)

Käynnistin varmuuden vuoksi Apachen uudelleen (`sudo systemctl restart apache2`) ja ajoin komennon `curl localhost/static`, mutta sain edelleen 404-virheilmoituksen. Huomasin sitten, että olin unohtanut `curl`-komennon lopusta /-merkin, jonka kirjoittamalla sain näkyviini haluamani tekstin.

![7 base site working](https://github.com/user-attachments/assets/008a7308-2d0c-4797-984c-0ba9418e73d7)

*0:15*

Palasin Karvisen (2022) artikkelin ohjeisiin ja asensin seuraavaksi Virtualenv-paketin virtuaaliympäristön luomista varten komennolla `sudo apt-get -y install virtualenv`. Loin uuden virtuaaliympäristön public_html-hakemistoon komennolla `virtualenv -p python3 --system-site-packages env` ja siirryin tämän sisälle komennolla `source env/bin/activate` tarkistaen tämän komennon `which pip` avulla. Micro-editorilla loin tekstitiedoston, johon kirjoitin "django", minkä jälkeen asensin Djangon komennolla `pip install -r requirements.txt`. Komento `django-admin --version` näytti, että parin kuukauden takaisesta (vrt. LeeviRaussi 2024a) ohjelma oli päivittynyt hieman.

![8 django install](https://github.com/user-attachments/assets/2fae0d18-d17d-4adb-8108-824cfb421176)

*0:19*

Lähdin Karvisen (2022) ohjeiden mukaisesti seuraavaksi luomaan uutta Django projektia komennolla `django-admin startproject perusta`. Sain kuitenkin virheilmoituksen, että kyseinen kansio oli jo olemassa, olinhan sen luonut jo edellä. Pohtiessani asiaa tarkistin samalla oikean polun tarvittaviin python-paketteihin `ls`-komentojen avulla. Saatuani tämän tehtyä päätin siirtyä "perusta"-hakemistoon (`cd perusta/`), jonne loin Django projektin "modduli" (`django-admin startproject modduli`). Muokkasin aiemmin luomaani perustan conf-tiedostoa komennolla `sudoedit /etc/apache2/sites-available/perusta.conf`.

![9 start project](https://github.com/user-attachments/assets/8285d098-3e18-44a6-9f63-fcffe04e8fad)

Käytetyt polut tuottivat ongelmia, sillä tarkastellessani `ls` komennolla hakemistojen sisältöjä tuli ilmi, että modduli-projektin hakemistoon oli luotu vielä toinen modduli-hakemisto.

![9 5 ls check](https://github.com/user-attachments/assets/0aa07804-e104-459c-990b-ab5ae09b0df9)

Tämä jätti erityisesti conf-tiedostosta löytyvän TDIR-määritelmän polun avonaiseksi, koska static-hakemisto sijaitsi aivan toisessa sijainnissa. Verratessani toteutustani Linux-Palvelimet kurssilta (ks. LeeviRaussi 2024a) päädyin siirtämään kyseisen static hakemiston modduli-projektin hakemistoon (`mv /home/weppimasteri/public_html/perusta/static/ /home/weppimasteri/public_html/perusta/modduli/`), vaikkei tämä tuntunutkaan kaikkeen tyylikkäimmältä ratkaisulta. Toisaalta tämän sivun käyttö on myös melko olematon, joten annettakoon se nyt tällä kertaa anteeksi.

![11 mv static](https://github.com/user-attachments/assets/c0d6e799-e2fc-4c22-b2fb-9ad07ae97f4d)

Tämän myötä sain conf-tiedoston seuraavaan muotoon:

```
Define TDIR /home/weppimasteri/public_html/perusta/modduli
Define TWSGI /home/weppimasteri/public_html/perusta/modduli/modduli/wsgi.py
Define TUSER weppimasteri
Define TVENV /home/weppimasteri/public_html/env/lib/python3.11/site-packages

<VirtualHost *:80>
        Alias /static/ ${TDIR}/static/
        <Directory ${TDIR}/static/>
                Require all granted
        </Directory>

        WSGIDaemonProcess ${TUSER} user=${TUSER} group=${TUSER} threads=5 python-path="${TDIR}:${TVENV}"
        WSGIScriptAlias / ${TWSGI}
        <Directory ${TDIR}>
             WSGIProcessGroup ${TUSER}
             WSGIApplicationGroup %{GLOBAL}
             WSGIScriptReloading On
             <Files wsgi.py>
                Require all granted
             </Files>
        </Directory>

</VirtualHost>

Undefine TDIR
Undefine TWSGI
Undefine TUSER
Undefine TVENV
```

![10 edited conf](https://github.com/user-attachments/assets/53efa0cd-9ef0-43d3-b4a6-6e322081c815)

*0:25*

Palasin jälleen Karvisen (2022) ohjeisiin ajoin komennon `sudo apt-get -y install libapache2-mod-wsgi-py3`, jotta Apachen WSGI-moduulin osat alkaisivat toimia. Komento `/sbin/apache2ctl configtest` näytti, ettei syntaksissa ollut ongelmia, jonka jälkeen käynnistin Apachen uudelleen (`sudo systemctl restart apache2`). Testasin tämän jälkeen muutamalla eri `curl`-komennolla, että Apache näkyy etusivulla ja edellä luotu "staattinen"-sivu osoitteessa /localhost/static/.

![12 django working](https://github.com/user-attachments/assets/55700743-982e-4173-8484-65c9140d763a)

*0:27*

Karvisen (2022) ohjeissa seuraavaksi laitetaan pois päältä debug-tila, joten tein itsekin tämän. Tutkin `ls`-komennon avulla, mistä löytäisin tarvittavan settings.py-tiedoston ja löysin sen hakemistosta /home/weppimasteri/public_html/perusta/modduli/modduli/. Muokkasin asetuksiin debugin epätodeksi sekä sallituksi isäntäkoneeksi localhostin.

![14 settings](https://github.com/user-attachments/assets/90a4810d-1f96-4e0a-9f52-6d061f307a15)

![13 debug false](https://github.com/user-attachments/assets/21b7707c-271c-40b5-aab8-ae134b9ec4c8)

Tämän jälkeen minun piti käynnistää Apache uudelleen (`sudo systemctl restart apache2`), mutta edetessäni ohjeita orjallisesti ajoin ensin komennon `touch wsgi.py`. Tämän jälkeen kuitenkin komento `curl -s localhost|grep title` näytti, että etusivu oli piilotettu onnistuneesti.

![15 curl](https://github.com/user-attachments/assets/f4e95342-58a5-415b-8197-2ec71d430b24)

*0:30*

Siirryin seuraavaksi korjaamaan graafisia bugeja, joita sivulla esiintyy. Tai ainakin esiintyi Linux-Palvelimet kurssilla, nyt en tällä hetkellä näe muuta kuin komentorivinäkymän. Muokkasin `micro settings.py` komennolla asetuksiin kaksi uutta kohtaa hyödyntäen Linux-Palvelimet kurssilla tekemiäni virheitä (ks. LeeviRaussi 2024a).

![16 import os](https://github.com/user-attachments/assets/0744a5ad-9c07-42c4-a737-5f9a426d1cb8)

Palasin takaisin modduli-projektikansioon, jossa ajoin komennon `./manage.py collectstatic` tyyliasetusten käyttöönottamiseksi.

![17 collectstatic](https://github.com/user-attachments/assets/004cacfa-3e3b-4d5a-a4fa-3b81a9f0d24f)

Kuten edellä totesin, en tiedä, miten voisin varmistaa tekemieni muutosten toimivan graafisessa näkymässä. Tästä johtuen päätin, että tämä on hyvä kohta päättää tältä kerralta manuaalinen asennus ja siirtyä automatisoimaan kaikki tekemäni välivaiheet. Luennolla voisin kysyä apua tähän asiaan, minkä lisäksi saisin varmaan vahvistuksen, onko ajatukseni CRM-ohjelman lisäämisestä tähän Django-projektiin riittävä.

### Automatisointi

*0:33*

Siirryin ensiksi Saltin omaan hakemistoon (`cd /srv/salt/`), jonne loin modduli-modulia varten hakemiston (`sudo mkdir modduli`). Siirryin kyseiseen hakemistoon ja kopioin (`sudo cp /srv/salt/apache/init.sls /srv/salt/modduli/`) sinne edellisessä raportissani (ks. LeeviRaussi 2024b) kirjoittamani sls-tiedoston, jota aioin käyttää pohjana tälle modulille. Muokkasin sls-tiedostoa (`sudoedit init.sls`) sopimaan uuteen käyttötarkoitukseensa.

![18 salt dir modduli](https://github.com/user-attachments/assets/0e9ba01e-41dc-48dd-b9eb-654991e066f9)

![19 modduli sls](https://github.com/user-attachments/assets/2b8354d7-c996-4483-80dc-b0804361f084)

Huomasin tässä vaiheessa, että olin edelleen virtualenvin-virtuaaliympäristössä, joten poistuin tästä komennolla `deactivate`. Kopioin sitten tarvittavan conf-tiedoston (`sudo cp /etc/apache2/sites-available/perusta.conf /srv/salt/modduli/`) ja "staattisen" html-sivun (`sudo cp /home/weppimasteri/public_html/perusta/modduli/static/index.html /srv/salt/modduli/`) modulin sisälle, jotta source-osoitteet olisivat oikein. Ajoin tämän jälkeen komennon `sudo salt '*' state.apply modduli` testatakseni, että kaikki olisi kunnossa.

![20 cp files](https://github.com/user-attachments/assets/eda5f48b-ccea-4f8c-9190-01649ce6208d)

Sain tällä kurssilla tutuksi tulleen TCP Timeout -viestin, joten käynnistin sen enempää miettimättä salt-masterin uudelleen (`sudo systemctl restart salt-master.service`) ja komento `sudo salt '*' state.apply modduli` uusiksi. Hetken odottelun jälkeen yhteys jälleen keskeytettiin. Tässä kohtaa mieleeni tuli, että voisiko ongelma johtua siitä, että koitan tehdä asioita weppimasteri-käyttäjällä normaalin "vagrantin" sijasta, joten palasin takaisin kyseiselle käyttäjälle `exit` komennolla. Salt-masterin uudelleenkäynnistys ja modulin ajo tuottivat uuden virheviestin.

![21 wrong user](https://github.com/user-attachments/assets/f083b442-fc06-41f8-b7a4-3e87e6a97df0)

Ajoin virheilmoituksessa olleen komennon `salt-run jobs.lookup_jid 20241126135915139322`, mutta en saanut siitä oikein mitään irti.

![21 5 jid](https://github.com/user-attachments/assets/b4098617-6a9f-41ea-a17f-705076cb4e06)

Tässä välissä kirjauduin toisella komentorivillä orja t002:lle, ja yllätyksekseni huomasin, että modulissa määriteltyjä hakemistoja ja tiedostoja oli kuitenkin luotu.

![22 files exists](https://github.com/user-attachments/assets/7921d9eb-0ad8-438c-aad1-002e66194242)

Palasin takaisin herra t001:lle ja testasin salt-masterin uudelleenkäynnistystä ja modulin ajamista uudelleen. Tällä kertaa solmu, mikä lie ollutkaan, oli ilmeisesti purkautunut, koska moduli suoritettiin lähes onnistuneesti.

![23 after boot](https://github.com/user-attachments/assets/7253342f-bc21-4fd0-8c3b-d6e48c38f1cd)

Muutama vanha sijainti aiemmasta modulista oli jäänyt kummittelemaan, jotka korjasin sudoeditillä.

![24 typo fix](https://github.com/user-attachments/assets/bbc403dc-4fd8-4e3f-a991-293a790aca96)

Ajoin tämän jälkeen jälleen komennon `sudo salt '*' state.apply modduli`. Nyt päädyin kuitenkin uuden äärelle, sillä useammankaan minuutin odottelun jälkeen en saanut TCP Timeout -ilmoitusta, mutta vaikutti myös, ettei mitään konkreettista tapahdu. Yritin sulkea prosessin CTRL+C näppäinyhdistelmällä, mutta tämänkin responsiivisuus oli erittäin tahmea. Jälleen muutaman minuutin odottelun jälkeen prosessi sulkeutui viimein.

![25 slooooooow](https://github.com/user-attachments/assets/01c0fc04-ff18-457a-abf6-a2d048b2bc42)

Päätin käynnistää koko Vagrantin uudelleen, joten kirjauduin järjestelmästä ulos komennolla `exit`, jonka jälkeen käynnistin Vagrantin uudelleen komennolla `vagrant reload`. Tämäkään prosessi ei ollut erityisen sujuva, vaan vei jonkin aikaa, mutta päättyi lopulta onnistuneesti.

Mietin, johtuivatko ongelmat mahdollisesti siitä, että 1) olin tehnyt joitain tiedostoja virtualenv-virtuaaliympäristössä, tai 2) olin tehnyt salt-komentoja ei-vagrantina. Myös nyt raporttia kirjoittaessa tajusin, että koska käytän conf-tiedostoa tulevaisuudesta, sen testaaminen esimerkiksi "staattinen"-sivun näkymiseen ei tule toimimaan, koska sijainti ei ole oikea.

*1:05*

Päästyäni takaisin Vagrant-herra t001:lle totesin suoraan, että ennemmin kuin alkaisin pohtia ongelmien ratkaisua, olisi helpompaa vain tehdä koko asia uudelleen nollasta. Otin modduli-modulin sls-tiedoston sisällön talteen, jonka jälkeen ajoin komennon `sudo salt-call --local state.single file.absent /srv/salt/modduli/` poistaen aiemman tekeleeni.

![26 delete unnecessary](https://github.com/user-attachments/assets/ca7d788e-c19c-4ace-a3a6-ba8b7bc437d4)

Loin sitten uudelleen moddulille hakemiston, johon tein sls-tiedoston liittämällä siihen edellä talteen ottamani koodin, sekä kopioimalla tarvitsemani conf- ja verkkosivutiedostot.

![27 from a scratch](https://github.com/user-attachments/assets/b66ee552-d445-4176-b9bb-cbc326f4ad6d)

Käynnistin Salt-masterin uudelleen (`sudo systemctl restart salt-master.service`) ja ajoin modulin salt-komentona (`sudo salt '*' state.apply modduli`).

![28 not idempotent](https://github.com/user-attachments/assets/61609202-50fb-4663-ada1-2a9459a17b46)

Tulos ei ollut vieläkään idempotentti, ja huomasinkin tehdyistä välivaiheista, että olin edelleen unohtanut vanhoja asioita sls-tiedostoon, joten muokkasin sitä taas.

![29 more ghosts](https://github.com/user-attachments/assets/9442abb7-10b6-46b2-b9a9-f3eaf22fdba8)

Salt-masterin uudelleenkäynnistys (`sudo systemctl restart salt-master.service`) ja modulin ajo (`sudo salt '*' state.apply modduli`) tuottivat viimein oikeannäköistä tulosta.

![30 correct conf activated](https://github.com/user-attachments/assets/7712045d-5412-4535-a788-e9c040f0dce0)

Toistettuna idempotenttisuus myös todistui.

![31 idempotent](https://github.com/user-attachments/assets/7895afd4-135b-4e45-bd55-d9b271aaafaa)

*1:12*

Seuraavaksi automatisoinnin kohteena oli virtuaaliympäristö Virtualenv. Sen sijaan, että olisin järkevästi tehnyt yhden asian kerrallaan, lähdin jo tässä vaiheessa tutkimaan, miten saisin ajettua asioita kyseisessä virtuaaliympäristössä. Tuhlattuani aikaani hyvän tovin rikastumatta ideallisesti yhtään päätin tehdä ensin pelkästään tarvittavan paketin asennuksen. Lisäsin siis sls-tiedostoon tarvittavan pkg.installed-tilafunktion.

![32 virtualenv pkg](https://github.com/user-attachments/assets/1eb9c697-48b3-48f5-8399-12fc9215cd92)

Löin perään komennon `sudo salt '*' state.apply modduli`, ja jäin tuijottamaan ruutua. Jonkin ajan odottamisen jälkeen ilman minkäänlaista elonmerkkiä prosessista keskeytin prosessin näppäinyhdistelmällä CTRL+C ja käynnistin salt-masterin uudelleen (`sudo systemctl restart salt-master.service`) toistaen modulin ajon tämän jälkeen.

![33 take it slow](https://github.com/user-attachments/assets/a79c697c-b197-461e-8549-801b69ead6b1)

Orjalla oli edelleen kesken edellä tekemäni prosessi, joten laitoin jäätä hattuun ja odottelin hetken. Muutaman minuutin jälkeen toistin edellä tekemäni uudelleenkäynnistys- ja ajokomennet, ja tällä kertaa idempotenttisuus oli jo saavutettu, eli virtualenv oli asentunut onnistuneesti ensimmäisellä yrityksellä.

![34 went through](https://github.com/user-attachments/assets/e78ade57-38b6-421f-848b-652091307ed7)

*1:27*

Nyt eteeni tuli todella virtuaaliympäristöön siirtyminen mikäli seuraisin manuaalista asennusprosessia. Ongelmana tässä on, että manuaalisesti ajamani komento `virtualenv -p python3 --system-site-packages env` tehtiin tietyssä hakemistossa, mutten tiedä, onko tämä mahdollista Saltilla. Toisaalta, Linux-Palvelimet kurssilla (ks. LeeviRaussi 2024a) siirsin tekemäni Django-projektin toiseen sijaintiin ilman sen erikoisempia ongelmia, joten pitäisi mahdollisena, että käytännössä koko hakemistoa `/home/weppimasteri/public_html/perusta/` olisi aiheellista ajatella pelkästään siirreltävinä tiedostoina. Tämän myötä voisikin olla aiheellista siirtää koko tämä hakemistopuu modduli-modulin sisälle, josta se sitten vain kopioitaisiin file.managed-tilafunktion sourcena haluttuun kohteeseen. Tällöin kaikki tarvittavat tiedostot olisivat oikeissa paikoissa varmasti.

Tämä jättää kuitenkin edelleen auki sen, miten Django tulisi asentaa, koska tämäkin toteutettiin pip-asennuksena hyödyntäen tietyssä tiedostossa olevaa tekstitiedostoa. Myös Django-projektin aloittaminen saattaa olla haastavaa, joskin tämän voi mahdollisesti kiertää aloittamalla projekti, jonka jälkeen vasta edellä kuvaamani hakemistopuu kopioitaisiin oikeaan polkuun.

Jätänkin automatisoinnin nyt tähän ja pyrin saamaan seuraavalla luennolla apua ja vastauksia seuraaviin ongelmiini:

- Miten voisin tarkastella asioiden toimivan graafisen näkymän kautta (erityisesti osoite "localhost/admin" tulee olemaan oleellinen myöhemmin, koska täällä pitäisi todentaa CRM:n toimiminen)?
- Onko tekemäni modulin kehityssuunta hyvä ja riittääkö CRM-lisäosa tähän tehtävään?
- Miten Virtualenv ja Django tulisi suorittaa? Riittääkö, että nämä löytyvät koneelta ja hakemistopuu on valmiina kehitystyötä varten?
- Miten tehdä (mitä parametreja tarvitaan) edellä olevien asioiden asentamiseen tarvittavista mahdollisista cmd.run-tilafunktioista idempotentteja?

## Lähdeluettelo

Karvinen, T. 2022. Deploy Django 4 - Production Install. Luettavissa: https://terokarvinen.com/2022/deploy-django/. Luettu: 26.11.2024.

LeeviRaussi 2024a. h6 Hello Django. Luettavissa: https://github.com/LeeviRaussi/linux-palvelimet/blob/main/h6_Hello_Django.md. Luettu: 26.11.2024.

LeeviRaussi 2024b. h3 Demoni. Luettavissa: https://github.com/LeeviRaussi/Palvelinten_hallinta/blob/main/h3_Demoni.md. Luettu: 26.11.2024.
