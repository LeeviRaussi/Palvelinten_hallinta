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

## Lähdeluettelo

Karvinen, T. 2018. Pkg-File-Service – Control Daemons with Salt – Change SSH Server Port. Luettavissa: https://terokarvinen.com/2018/04/03/pkg-file-service-control-daemons-with-salt-change-ssh-server-port/?fromSearch=karvinen%20salt%20ssh. Luettu: 15.11.2024.

Salt 2024. Salt manual. Debian 12 käyttöjärjestelmällinen kone. Luettu: 15.11.2024.
