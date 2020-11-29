# H1

HTTP Basics

Go!-nappula lähettää POST-requestin, jota voidaan tarkistella devtoolssien Network-tabilla.
Requestin sisältä löytyy kolme arvoa, joista yksi on magic_num.
Developer tools

4: Ajetaan webgoat.customjs.phoneHome() funktio devtoolssin console-tabilla, joka palauttaa tekstin "Response is xxxxx.
6: Go!-nappi lähettää POST-requestin, jonka sisällöstä löydetään muttuja networkNum.
CIA-triad

1: 3.
2: 1.
3: 4.
4: 2.

Darknet Diaries Ep. 52 Magecart

Mielenkiintoinen episodi jossa käsitellään luottokorttiskimmausta netissä.
Skimmausmekanismina ollaan käytetty muutamaa riviä javaScriptiä, joka lähettää luottokorttitiedot eteenpäin parsittavaksi.
Kyseinen javaScript-koodi on istutettu tiedostoon joka käsittelee sekä nettisivun, että mobiiliaplikaation ostot.
Magecartista kärsineet nimekkäimmät nettisivut olivat brittishairways.com ja newegg.com.



# H2:

OWASP:

A2 - Broken authentication

Sivusto on altis hyökkäykselle kun se:
1. mahdollistaa automaatisen autentikoinnin, jolloin voidaan kokeilla listasta suosittuja salasanoja ja käyttäjänimiä
2. mahdollistaa brute force-hyökkäyksen
3. ei palauta käyttäjälle tietoa salasanan vahvuudesta
4. ei vaihda sessio-id:tä tarpeeksi usein

A3 - Sensitive data exposure

Sivusto on altis hyökkäykselle kun se:
1. lähettää salattavaa tietoja clear textinä
2. tallentaa salattavaa tietoja kantaan clear textinä
3. käyttää vanhentuneita enkryptaus metodeita

A7 - Cross-site scripting

1. sivusto/api mahdollistaa käyttäjän "siivoamattoman" inputin ajamisen
2. sivusto/api tallentaa käyttäjän siivoamattoman inputin joka tulee muille käyttäjille näkyväksi/ajettavaksi

Webgoat

A2 - Authentication bypasses:

Tutkitaan sivuston HTML-koodia inspectorilla ja kokeillaan manuaalisesti vaihtaa kysymysten muuttujat:
    
    "secQuestion0" ja "secQuestion1" => "secQuestion2" ja "secQuestion3"

Näitä todennäköisesti ei ole olemassa, ja näiden arvoiksi "test".

Toimii. 
En kyllä osaa hahmottaa miksi, kyllähän sivuston pitäisi osata tarkistaa invalidit muuttujat myös?

Secure passwords:

käytän salasanan sijaan "salalausetta" ja yleensä suomeksi.

    Estimated guesses needed to crack your password: 15000000000000002000000000000000
    Score: 4/4
    Estimated cracking time: 292471208677 years 195 days 15 hours 30 minutes 7 secondsScore: 4/5 

A3 -Insecure Login:

Käynnistetään Wireshark loopback-asetuksella.
Painoin login-nappulaa ja menen wiresharkkiin etsimään HTTP-protokollalla POST-viestiä.
Wiresharkista löytyy kaksi POST-riviä ja avaan aikajanalla ensimmäisen näistä. Skrollailen ja luen viestin sisältöä ja 
viimeinen rivi näyttää epäilyttävältä:
    
    {"username":"CaptainJack","password":"BlackPearl"}.

A7 - Try It! Reflected XSS:

Kokeilin kirjoittaa kaikkiin mahdollisiin input-kenttiin <script>alert('foo');</script> ja luottokorttinumerossa tämä toimi.

Identify potential for DOM-Based XSS:

inspectorin network-tabilla filtteröin sivun latauksen kaikki js-tiedostot.
Tiedosto "GoatRouter.js" iski silmään ja tämän sisältö on luettavissa ilman de-obfuskointia.
Löysin koodista objektin "GoatAppRouter" jonka sisällöstä kohdan 'test/:param': 'testRoute',.

Kokeilin laittaa vastaukseksi tuon "testRoute", mutta tämä ei auttanut.

Sitten skrollatessani samaa js-tiedostoa eteenpäin löysin rivin:

    testRoute: function (param) {
            this.lessonController.testHandler(param);

jolloin päätin kurkata mitä tuolla LessonController.js-tiedostosta löytyisi.
Käyn tätä läpi mutta mikään ei viittaa mihinkään testaamiseen paitsi 

    this.testHandler = function(param) {
                console.log('test handler');
                this.lessonContentView.showTestParam(param);

mutta en ole varma mitä tällä nyt tulisi tehdä.
Lueskelin uudestaan tuon sivuston ohjeen liittyen urliin. Jos start.mvc#lesson/ on urlin "base route" ja sen
jälkeen tuleva "CrossSiteScripting.lesson/9" on parametri, 
niin aijemmin GoatRouterista löytynyt tavara sisältää tekstin ":param". Voisin tätä kokeilla seuravaavaa:

        start.mvc#test/

A8 -Cross-Site Request Forgeries:

Laitoin ohjeen mukaan http-serverin päälle: python3 -m http.server 
ja tein html-tiedoston johon kopioin webgoatista inspectorilla submit-napin form-tägin:

    <form accept-charset="UNKNOWN" id="basic-csrf-get" method="POST" name="form1" target="_blank" successcallback="" action="/WebGoat/csrf/basic-get-flag"              
    enctype="application/json;charset=UTF-8">
    <input name="csrf" type="hidden" value="false">
    <input type="submit" name="submit">
    </form>

mutta editoin action="" kohdan siten että lisäsin täyden urlin actionin eteen, eli

    action="http://localhost:8080/WebGoat/csrf/basic-get-flag"

navigoin tuolle html-tiedostolle ja painoin nappia, jolloin toi action palautti tarvittavan numeron.

Post a review on someone else’s behalf:

kokeilen samaa tekniikkaa kuin yllä mutta kopioin myös input-kenttien htll-koodin, ja vaihdan actionin urlin yllämainitulla tavalla.
Tämä onnistui "osittain", sillä webgoatin sivua päivitettäessä sinne ilmestyi rivi:

        "petteri / null stars"

ja pitkään aikaan en onnistunut replikoimaan tätä kunnes huomasin etten ollut laittanut actionin endpointtiin "http://".
sitten lähti toimimaan

# H3

## Active recon:

* porttiskannaus
    * nmap: suosituin porttiskannaussofta
    * masscan: nopein skannata isoja määriä ip-osoitteita
    * udpprotoscanner: UDP porttiskanneri
* haavoittuvuusskannaus
    * nmap --script vuln
    
## Nmap

Kokeilen skannata tiedostopalveintani, jonka osoitteen asetan wiresharkissa valmiiksi filtteriksi.

        -sT -p 80 [ip osoite]

Tämä suorittaa ns. "three way handshaken", eli wiresharkissa näkyy kolme pakettia, [SYN], [SYN, ACK] ja [ACK]. Eli keskuetelu menisi about näin:
        
        Minä - Heippa, olis vähän asiaa portilla 80, ookkonää hereillä [SYN] 
        Servu - Moikka, joo oon mää täällä whassup? [SYN, ACK]
        Minä - Hei hieno juttu hei! [ACK]
        

        
Sitten kokeillaan vähän salakavalampaa tapaa

        -sS -p 80 [ip osoite]
        
Tämä käytännössä tiputtaa yllämainitusta keskustelusta viimeisen minun lähettämän ACK-paketin, jolloin palvelinten palomuurit ei ehkä kiinnitä huomiota tähän kun "täyttä yhteyttä" ei saavutettu?

        -sN  -p 80 [ip osoite]
        
Wireshark näyttää että kaksi pakettia ilman mitään asetuksia olisi lähtenyt koneeltani, paketin headeri on tyhjä. En tiedä mitä tämä auttaa.

        -Pn -p 80 [ip osoite]
        
Tämä näyttää wiresharkissa täysin vastaavalta kuin -sS asetuksella ajettu.

        -sV -p 80 [ip osoite]
        
Palauttaa kyseisen palvelun softan nimen, tässä tapauksessa Apache httpd. Wiresharkissa tapahtuu liikaa tämän kyselyn ansiosta.

        -p1-100, --top-ports 5
        
Palauttaa "suosituimmat" 5 porttia ikkunalla 1-80.

        -oA foo
        
Tallentaa skannauksen tuloksen foo.gnmap, nmap ja xml -tiedostoihin.

HackTheBox

Avaan vpn-yhteyden komennolla sudo openvpn [nimi].ovpn ja menen htb sivustolle "access" osioon jossa näkyy että olen yhdistettynä htb lab -verkkoon.
Tarkistan verkon ip-osoitteen sivustolta joka näyttää olevan 10.10.14.xx.
Ajan sudo nmap -sS 10.10.14.1/24

hmm nyt jotain menee pieleen. :(

# H4 Walkthrough

## z) Lue artikkelit ja katso videot, tee kustakin muistiinpanot (muutama ranskalainen viiva per artikkeli/video).

### HackTheBox - Heist (https://www.youtube.com/watch?v=fmBb6BgLsC8)

nmap -sC- sV -oA nmap/heist ip

portit 80 ms iis v. 10.0
cookie flags
PHPSESSID <- ?? ms ajaisi yleensä asp
login.php

->tsekataan portti 80 ip/login.php

Cisco type 7 passowrd decrypter
hashcat
crackmapexec
metasploit -> winRM login


evil winrm
hashes.org

upmpikuja. firefox prosessit dumpataan

ms sysinternals suite

upitaan procdump64 remoteshelliin
procdumpilla dumpataan firefoxin prosessi ja ladataan se tarkisteltavaksi
 

## a) Etsi ja kokeile 5 uutta työkalua jostain lukemastasi/katsomastasi läpikävelystä.

## b) Tiedustele ja analysoi 5 htb konetta perusteellisesti.

## c) Nimeä 1-3 walktrough:ta, joissa tunkeudutaan samantapaisiin palveluihin, joita käsittelit kohdassa b.


        

        
        
