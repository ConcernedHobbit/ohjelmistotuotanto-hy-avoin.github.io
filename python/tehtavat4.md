---
layout: page
title: Viikko 4
inheader: no
permalink: /python/tehtavat4/
---

## Viikko 4

_Alla olevien tehtävien deadline on maanantaina 23.11. klo 23:59_

Apua tehtävien tekoon kurssin [Telegram](https://telegram.me/ohjelmistotuotanto)-kanavalla sekä zoom-pajassa:

- Maanantai 14-16 [zoom](https://helsinki.zoom.us/j/63962392550?pwd=RzluTjZWYmNLb0g4bjRxb0ZlckRkUT09)
- Perjantai 10-12 [zoom](https://helsinki.zoom.us/j/64396759243)

Muista myös tämän viikon [monivalintatehtävät]({{site.stats_url}}/quiz/4), joiden deadline on sunnuntaina 22.11. klo 23:59:00.

Tehtävissä 1-4 tutustutaan yksikkötestausta helpottavaan Mockito-kirjastoon. Tehtävissä 5 ja 6 refaktoroidaan sisäiseltä laadultaan heikossa kunnossa olevaa koodia.

### Typoja tai epäselvyyksiä tehtävissä?

{% include typo_instructions.md path="/python/tehtavat4.md" %}

### Tehtävien palauttaminen

Tehtävät palautetaan GitHubiin, sekä merkitsemällä tehdyt tehtävät palautussovellukseen <{{site.stats_url}}>

Katso tarkempi ohje palautusrepositorioita koskien [täältä](/tehtavat1#teht%C3%A4vien-palautusrepositoriot).

### 1. Yksikkötestaus ja riippuvuudet: mock-kirjasto, osa 1

Useimmilla luokilla on riippuvuuksia toisiin luokkiin. Esim. [viikon 2](/python/tehtavat2/#8-riippuvuuksien-injektointi-osa-3-verkkokauppa) laskarien verkkokaupan luokka `Kauppa` riippui `Pankista`-, `Varastosta`- ja `Viitegeneraattori`-luokista. Riippuvuuksien injektion avulla saimme mukavasti purettua riippuvuudet luokkien väliltä.

Vaikka luokilla ei olisikaan riippuvuuksia toisiin luokkiin, on tilanne edelleen se, että luokan oliot käyttävät joidenkin toisten luokkien olioiden palveluita. Tämä tekee yksikkötestauksesta välillä hankalaa. Miten esim. luokkaa `Kauppa` tulisi testata? Tuleeko kaupan testeissä olla mukana toimivat versiot kaikista sen riippuvuuksista?


Olemme jo muutamaan otteeseen (esim. NHL-tilastot-tehtävässä [viikolla 1](/python/tehtavat1#15-riippuvuuksien-injektointi-osa-2-nhl-tilastot) ratkaisseet asian ohjelmoimalla riippuvuuden korvaavan "tynkäkomponentin". Pythonille kuten kaikille muillekin kielille on tarjolla myös valmiita kirjastoja tynkäkomponenttien, toiselta nimeltään _mock-olioiden_ luomiseen.

Kuten pian huomaamme, mock-oliot eivät ole pelkkiä "tynkäolioita", mockien avulla voi myös varmistaa, että testattava metodi tai funktio kutsuu olioiden metodeja asiaankuuluvalla tavalla.

Tutustumme nyt unittest-moduulin [mock](https://docs.python.org/3/library/unittest.mock.html)-kirjastoon. Kirjastosta voidaan tuoda luokka [Mock](https://docs.python.org/3/library/unittest.mock.html#unittest.mock.Mock). Kasotaan mitä luokalla voi tehdä käynnistämällä interaktiivinen Python-komentorivi komennolla `python3` (virtuaaliympäristölle ei ole tarvetta, koska emme käytä ulkoisia riippuvuuksia):

```python
>>> from unittest.mock import Mock
>>> mock = Mock()
>>> mock
<Mock id='4568521696'>
```

Anna komentoriville syötteet yksi kerrallaan. Enter-painikkeen painallus suorittaa annetun syötteen. Muuttuja `mock` sisältää siis `Mock`-luokan olion. `Mock`-luokan olioilla on se mielenkiintoinen piirre, että niillä kaikki mahdolliset toteutukset. Mitä tällä tarkoitetaan? Kokeillaan:

```python
>>> mock.foo
<Mock name='mock.foo' id='4568521648'>
>>> mock.foo.bar()
<Mock name='mock.foo.bar()' id='4570560112'>
```

Kaikki annetut operaatiot palauttavat siis uuden `Mock`-olion. Voimme antaa olion metodeille haluttuja palautusarvoja [return_value](https://docs.python.org/3/library/unittest.mock.html#unittest.mock.Mock.return_value)-attribuutin avulla:

```python
>>> mock.foo.bar.return_value = "Foobar"
>>> mock.foo.bar()
'Foobar'
```

Voimme myös antaa metodeille haluttuja toteutuksia [side_effect](https://docs.python.org/3/library/unittest.mock.html#unittest.mock.Mock.side_effect)-attribuutin avulla:

```python
>>> mock.foo.bar.side_effect = lambda name: f"{name}: Foobar"
>>> mock.foo.bar("Kalle")
'Kalle: Foobar'
```

Attribuutin `side_effect` arvo pitää olla kutsuttavissa, kuten funktio, metodi, tai lambda. Huomaa, että `Mock`-oliota voi käyttää myös funktion kaltaisesti:

```python
>>> get_name_mock = Mock(return_value = "Matti")
>>> get_name_mock()
'Matti'
```

Kuten edellä jo mainittiin, mockeille voi määritellä toteutuksien lisäksi oletuksia. Voimme esimerkiksi olettaa, että `Mock`-oliota on kutsuttu:

```python
>>> mock.foo.bar.assert_called()
>>> mock.foo.doo.assert_called()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "/Users/kalleilv/.pyenv/versions/3.9.0/lib/python3.9/unittest/mock.py", line 876, in assert_called
    raise AssertionError(msg)
AssertionError: Expected 'doo' to have been called.
```

Voimme siis kutsua tarkasteltavalle metodille [assert_called](https://docs.python.org/3/library/unittest.mock.html#unittest.mock.Mock.assert_called)-metodia. Huomaa, että `mock.foo.bar`-metodia on kutsuttu, mutta `mock.foo.doo`-metodia sen sijaan ei ole. Voimme myös tarkistaa, että metodia on kutsuttu oikeilla argumenteilla käyttämällä [assert_called_with](https://docs.python.org/3/library/unittest.mock.html#unittest.mock.Mock.assert_called_with)-metodia.

Kun `Mock`-oliot ovat tulleet tutuksi, voit sulkea komentorivin syöttämällä `exit()`.

Hae seuraavaksi [kurssirepositorion](https://github.com/ohjelmistotuotanto-hy-avoin/python-kevat-2021) hakemistossa _koodi/viikko4/mock-demo_ oleva projekti. Kyseessä on yksinkertaistettu versio verkkokauppaesimerkistä.

Kaupan toimintaperiaate on yksinkertainen:

```python
my_net_bank = Pankki()
viitteet = Viitegeneraattori()
kauppa = Kauppa(my_net_nank, viitteet)

kauppa.aloita_ostokset()
kauppa.lisaa_ostos(5)
kauppa.lisaa_ostos(7)
kauppa.maksa("1111")
```

Ostokset aloitetaan tekemällä metodikutsu `aloita_ostokset`. Tämän jälkeen "ostoskoriin" lisätään tuotteita, joiden hinta kerrotaan metodin `lisaa_ostos` parametrina. Ostokset lopetetaan kutsumalla metodia `maksa` joka saa parametriksi tilinumeron miltä summa veloitetaan.

Kauppa tekee veloituksen käyttäen tuntemaansa luokan `Pankki` olioa. Viitenumerona käytetään luokan `Viitegeneraattori` generoimaa numeroa.

Projektiin on kirjoitettu kuusi `Mock`-luokkaa hyödyntävää testiä. Testit testaavat, että kauppa tekee ostoksiin liittyvän veloituksen oikein, eli että se kutsuu `Pankki`-luokan metodia `maksa` oikeilla parametreilla, ja että jokaiselle laskutukselle on kysytty viitenumero `Viitegeneraattori`-luokan metodilta `uusi`. Testit siis eivät kohdistu olion pankki tilaan vaan sen muiden olioiden kanssa käymän interaktion oikeellisuuteen. Testeissä kaupan riippuvuudet (`Pankki` ja `Viitegeneraattori`) on määritelty `Mock`-olioina.

Seuraavassa testi, joka testaa, että kauppa kutsuu pankin metodia oikealla tilinumerolla ja summalla:

```python
def test_kutsutaan_pankkia_oikealla_tilinumerolla_ja_summalla(self):
    pankki_mock = Mock()
    viitegeneraattori_mock = Mock(wraps=Viitegeneraattori())

    kauppa = Kauppa(pankki_mock, viitegeneraattori_mock)

    kauppa.aloita_ostokset()
    kauppa.lisaa_ostos(5)
    kauppa.lisaa_ostos(5)
    kauppa.maksa("1111")

    # katsotaan, että ensimmäisen ja toisen parametrin arvo on oikea
    pankki_mock.maksa.assert_called_with("1111", 10, ANY)
```

Testi siis aloittaa luomalla kaupan riippuvuuksista mock-oliot:

```python
pankki_mock = Mock()
viitegeneraattori_mock = Mock(wraps=Viitegeneraattori())

kauppa = Kauppa(pankki_mock, viitegeneraattori_mock)
```

`Mock`-luokan [konstruktorin](https://docs.python.org/3/library/unittest.mock.html#unittest.mock.Mock) `wraps`-parametrin avulla voimme määritellä, minkä olion `Mock`-olio toteuttaa. Tämä mahdollistaa sen, ettei esimerkiksi `uusi`-metodille tarvitse määritellä toteutusta, vaan voimme käyttää sen oikeaa toteutusta.

Testi tarkastaa, että kaupalle tehdyt metodikutsut aiheuttavat sen, että pankin `Mock`-olion metodia `maksa` on kutsuttu oikeilla parametreilla. Kolmanteen parametriin, eli viitenumeroon ei kiinnitetä huomiota:

```python
pankki_mock.maksa.assert_called_with("1111", 10, ANY)
```

Kuten edellisissä esimerkeissä tuli ilmi, `Mock`-olioille tehtyjen metodikutsujen paluuarvot on mahdollista määrittää. Seuraavassa määritellään, että viitegeneraattori palauttaa arvon `55` kun sen metodia `uusi` kutsutaan:

```python
def test_kaytetaan_maksussa_palautettua_viitetta(self):
    pankki_mock = Mock()
    viitegeneraattori_mock = Mock()

    # palautetaan aina arvo 55
    viitegeneraattori_mock.uusi.return_value = 55

    kauppa = Kauppa(pankki_mock, viitegeneraattori_mock)

    kauppa.aloita_ostokset()
    kauppa.lisaa_ostos(5)
    kauppa.lisaa_ostos(5)
    kauppa.maksa("1111")

    # katsotaan, että kolmannen parametrin arvo on oikea
    pankki_mock.maksa.assert_called_with(ANY, ANY, 55)
```

Testin lopussa varmistetaan, että pankin `Mock`-oliota on kutsuttu oikeilla parametrinarvoilla, eli kolmantena parametrina tulee olla viitegeneraattorin palauttama arvo.

Tutustu projektiin ja sen kaikkiin testeihin. Asenne projektin riippuvuudet komennolla `poetry install` ja suorita sen jälkeen testit virtuaaliympäristössä komennolla `pytest`. Riko joku testi, esimerkiksi jokin edellä mainituista, muuttamalla sen ekspektaatiota:

```python
pankki_mock.maksa.assert_called_with(ANY, ANY, 1000)
```

Ja varmista että testi eivät mene läpi. Katso miltä virheilmoitus näyttää.

Voit tutusta aiheeseen tarkemmin lukemalla mock-kirjaston [dokumentaatiota](https://docs.python.org/3/library/unittest.mock.html).

### 2. Yksikkötestaus ja riippuvuudet: mock-kirjasto, osa 2

Hae [kurssirepositorion](https://github.com/ohjelmistotuotanto-hy-avoin/python-kevat-2021) hakemistossa _koodi/viikko4/maksukortti-mock_ oleva projekti.

Tässä tehtävässä on tarkoitus testata ja täydennetään luokkaa `Kassapaate`. **`Maksukortin koodiin ei tehtävässä saa koskea ollenkaan! Testeissä ei myöskään ole tarkoitus luoda konkreettisia instansseja maksukortista, testien tarvitsemat kortit tulee luoda mock-kirjaston avulla.**

Projektissa on valmiina kaksi testiä:

```python
import unittest
from unittest.mock import Mock, ANY
from kassapaate import Kassapaate, HINTA
from maksukortti import Maksukortti


class TestKassapaate(unittest.TestCase):
    def setUp(self):
        self.kassa = Kassapaate()

    def test_kortilta_velotetaan_hinta_jos_rahaa_on(self):
        maksukortti_mock = Mock(wraps=Maksukortti(10))
        self.kassa.osta_lounas(maksukortti_mock)

        maksukortti_mock.osta.assert_called_with(HINTA)

    def test_kortilta_ei_veloteta_jos_raha_ei_riita(self):
        maksukortti_mock = Mock(wraps=Maksukortti(4))
        self.kassa.osta_lounas(maksukortti_mock)

        maksukortti_mock.osta.assert_not_called()
```

Ensimmäisessä testissä varmistetaan, että jos kortilla on riittävästi rahaa, kassapäätteen metodin `osta_lounas` kutsuminen velottaa summan kortilta.

Testi ottaa siis kantaa ainoastaan siihen miten kassapääte kutsuu maksukortin metodeja. Maksukortin saldoa ei erikseen tarkasteta, sillä oletuksena on, että maksukortin omat testit varmistavat kortin toiminnan.

Toinen testi varmistaa, että jos kortilla ei ole riittävästi rahaa, kassapäätteen metodin `osta_lounas` kutsuminen _ei_ veloita kortilta rahaa.

**Testit eivät mene läpi. Korjaa kassapäätteen metodi `osta_lounas`.**

**Tee tämän jälkeen samaa periaatetta noudattaen seuraavat testit:**

- Kassapäätteen metodin `lataa` kutsu lisää maksukortille ladattavan rahamäärän käyttäen kortin metodia `lataa` jos ladattava summa on positiivinen
- Kassapäätteen metodin `lataa` kutsu ei tee maksukortille mitään jos ladattava summa on negatiivinen

Korjaa kassapäätettä siten, että testit menevät läpi.

### 3. Yksikkötestaus ja riippuvuudet: mock-kirjasto, osa 3

Testataan [viikolla 2](/python/tehtavat2/#8-riippuvuuksien-injektointi-osa-3-verkkokauppa) tutuksi tulleen verkkokaupan luokkaa `Kauppa`.

- Jos et tehnyt tehtävää, sovellus löytyy [kurssirepositorion](https://github.com/ohjelmistotuotanto-hy-avoin/python-kevat-2021) hakemistossa _koodi/viikko4/verkkokauppa_.

Kaupalle injektoidaan konstruktorissa `Pankki`-, `Viitelaskuri` ja `Varasto`-oliot. Tehdään näistä testeissä mock-kirjaston avulla mockatut versiot.

Seuraavassa esimerkkinä testi, joka testaa, että ostostapahtuman jälkeen pankin metodia `tilisiirto` on kutsuttu:

```python
import unittest
from unittest.mock import Mock, ANY
from kauppa import Kauppa
from viitegeneraattori import Viitegeneraattori
from varasto import Varasto
from tuote import Tuote

class TestKauppa(unittest.TestCase):
    def test_ostoksen_paaytyttya_pankin_metodia_tilisiirto_kutsutaan(self):
        pankki_mock = Mock()
        viitegeneraattori_mock = Mock()

        # palautetaan aina arvo 42
        viitegeneraattori_mock.uusi.return_value = 42

        varasto_mock = Mock()

        # tehdään toteutus saldo-metodille
        def varasto_saldo(tuote_id):
            if tuote_id == 1:
                return 10

        # tehdään toteutus hae_tuote-metodille
        def varasto_hae_tuote(tuote_id):
            if tuote_id == 1:
                return Tuote(1, "maito", 5)

        # otetaan toteutukset käyttöön
        varasto_mock.saldo.side_effect = varasto_saldo
        varasto_mock.hae_tuote.side_effect = varasto_hae_tuote

        # alustetaan kauppa
        kauppa = Kauppa(varasto_mock, pankki_mock, viitegeneraattori_mock)

        # tehdään ostokset
        kauppa.aloita_asiointi()
        kauppa.lisaa_koriin(1)
        kauppa.tilimaksu("pekka", "12345")

        # varmistetaan, että metodia tilisiirto on kutsuttu
        pankki_mock.tilisiirto.assert_called()
        # toistaiseksi ei välitetä kutsuun liittyvistä argumenteista
```

Aloita siten, että saat esimerkkitestin toimimaan. Tee sen jälkeen seuraavat testit:

- Aloitetaan asiointi, koriin lisätään tuote, jota varastossa on ja suoritetaan ostos, eli kutsutaan metodia kaupan `tilimaksu`, varmista että kutsutaan pankin metodia `tilisiirto` oikealla asiakkaalla, tilinumeroilla ja summalla
  - Tämä siis on muuten copypaste esimerkistä, mutta `assert_called_with`-metodia käytettävä, jotta voidaan tarkastaa, että parametreilla on oikeat arvot
- Aloitetaan asiointi, koriin lisätään kaksi eri tuotetta, joita varastossa on ja suoritetaan ostos, varmista että kutsutaan pankin metodia `tilisiirto` oikealla asiakkaalla, tilinumerolla ja summalla
- Aloitetaan asiointi, koriin lisätään kaksi samaa tuotetta, jota on varastossa tarpeeksi ja suoritetaan ostos, varmista että kutsutaan pankin metodia `tilisiirto` oikealla asiakkaalla, tilinumerolla ja summalla
- Aloitetaan asiointi, koriin lisätään tuote, jota on varastossa tarpeeksi ja tuote joka on loppu ja suoritetaan ostos, varmista että kutsutaan pankin metodia `tilisiirto` oikealla asiakkaalla, tilinumerolla ja summalla

Muista, että kaikille testeille yhteiset alustukset on mahdollista tehdä `setUp`-metodissa, joka toistetaan ennen jokaista testiä:

```python
class TestKauppa(unittest.TestCase):
    def setUp(self):
        self.pankki_mock = Mock()
        # ...
```

### 4. Yksikkötestaus ja riippuvuudet: mock-kirjasto, osa 4

Jatketaan edellisen tehtävän koodin testaamista

- Varmista, että metodin `aloita_asiointi` kutsuminen nollaa edellisen ostoksen tiedot (eli edellisen ostoksen hinta ei näy uuden ostoksen hinnassa), katso tarvittaessa apua projektin mock-demo testeistä!
- Varmista, että kauppa pyytää uuden viitenumeron jokaiselle maksutapahtumalle, katso tarvittaessa apua projektin mock-demo testeistä!

Tarkasta viikoilla 1 ja 2 käytetyn coveragen avulla mikä on luokan `Kauppa` testauskattavuus.

Jotain taitaa puuttua. Lisää testi, joka nostaa kattavuuden noin sataan prosenttiin!

### Mock-olioiden käytöstä

Mock-oliot saattoivat tuntua hieman monimutkaisilta edellisissä tehtävissä. Mockeilla on kuitenkin paikkansa. Jos testattavana olevan olion riippuvuutena oleva olio on monimutkainen, kuten esimerkiksi verkkokauppaesimerkissä luokka `Pankki`, kannattaa testattavana oleva olio testata ehdottomasti ilman todellisen riippuvuuden käyttöä testissä. Valeolion voi toki tehdä myös "käsin", mutta tietyissä tilanteissa mock-kirjastoilla tehdyt mockit ovat käsin tehtyjä valeolioita kätevämpiä, erityisesti jos on syytä tarkastella testattavan olion riippuvuuksille tekemiä metodikutsuja.

### 5. IntJoukon testaus ja siistiminen

[Kurssirepositorion](https://github.com/ohjelmistotuotanto-hy-avoin/python-kevat-2021) hakemistossa _koodi/viikko4/int-joukko_ on Python-versiona aloittelevan ohjelmoijan ratkaisu syksyn 2011 Ohjelmoinnin jatkokurssin [viikon 2 tehtävään 3](http://www.cs.helsinki.fi/u/wikla/ohjelmointi/jatko/s2011/harjoitukset/2/).

Koodi jättää hieman toivomisen varaa sisäisen laatunsa suhteen. Refaktoroi luokan `IntJoukko` koodi mahdollisimman siistiksi:

- Poista copypaste
- Vähennä monitkaisuutta
- Anna muuttujille selkeät nimet
- Tee metodeista pienempiä ja hyvän koheesion omaavia

Koodissa on joukko yksikkötestejä, jotka helpottavat refaktorointia.

**HUOM:** Suorita refaktorointi mahdollisimman pienin askelin, pidä koodi koko ajan toimivana. Suorita testit jokaisen refaktorointiaskeleen jälkeen!

### 6. Tenniksen pisteenlaskun refaktorointi

[Kurssirepositorion](https://github.com/ohjelmistotuotanto-hy-avoin/python-kevat-2021) hakemistossa _koodi/viikko4/tennis_, löytyy ohjelma, joka on tarkoitettu tenniksen [pisteenlaskentaan](https://github.com/emilybache/Tennis-Refactoring-Kata#tennis-kata).

Pisteenlaskennan rajapinta on yksinkertainen. Metodi `get_score` kertoo voimassa olevan tilanteen tenniksessä käytetyn pisteenlaskennan määrittelemän tavan mukaan. Sitä mukaa kun jompi kumpi pelaajista voittaa palloja, kutsutaan metodia `won_point`, jossa parametrina on pallon voittanut pelaaja.

Esim. käytettäessä pisteenlaskentaa seuraavasti:

```python
game = TennisGame("player1", "player2")

print(game.get_score())

game.won_point("player1")
print(game.get_score())

game.won_point("player1")
print(game.get_score())

game.won_point("player2")
print(game.get_score())

game.won_point("player1")
print(game.get_score())

game.won_point("player1")
print(game.get_score())
```

tulostuu

```
Love-All
Fifteen-Love
Thirty-Love
Thirty-Fifteen
Forty-Fifteen
Win for player1
```

Tulostuksessa siis kerrotaan mikä on pelitilanne kunkin pallon jälkeen kun _player1_ voittaa ensimmäiset 2 palloa, _player2_ kolmannen pallon ja _player1_ loput 2 palloa.

Pisteenlaskentaohjelman koodi toimii ja sillä on erittäin kattavat testit. Koodi on kuitenkin sisäiseltä laadultaan kelvotonta.

Tehtävänä on refaktoroida koodi luettavuudeltaan mahdollisimman ymmärrettäväksi. Koodissa tulee välttää ["taikanumeroita"](https://en.wikipedia.org/wiki/Magic_number_(programming)) ja huonosti nimettyjä muuttujia. Koodi kannattaa jakaa moniin pieniin metodeihin, jotka nimennällään paljastavat oman toimintalogiikkansa.

Etene refaktoroinnissa _todella pienin askelin_. Suorita testejä mahdollisimman usein. Yritä pitää ohjelma koko ajan toimintakunnossa.

Jos haluat käyttää jotain muuta kieltä kuin Pythonia, löytyy koodista ja testeistä versioita useilla eri kielillä osoitteesta [https://github.com/emilybache/Tennis-Refactoring-Kata](https://github.com/emilybache/Tennis-Refactoring-Kata)

Tehtävä on kenties hauskinta tehdä pariohjelmoiden. Itse tutustuin tehtävään kesällä 2013 Extreme Programming -konferenssissa järjestetyssä Coding Dojossa, jossa tehtävä tehtiin satunnaisesti valitun parin kanssa pariohjelmoiden.

Lisää samantapaisia refaktorointitehtäviä löytyy Emily Bachen [GitHubista](https://github.com/emilybache).

{% include submission_instructions.md %}
