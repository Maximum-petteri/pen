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

Katsoin kyseisen videon koska se oli tarkoitettu alunperin helpoksi kohteeksi, mutta hyvin pian putosin kärryiltä hetkeksi.
Videolla käytettiin paljon uusia työkaluja joista en ollut kuullut. Alun tiedustelu suoritettiin nmapilla ajamalla
        
        nmap -sC- sV -oA [ip]
        
Eli tässä ajettiin vain default scriptat ja kaivettiin mahdollisten palveluiden versiot softista tiedostoon.
Ymmärsin mitä tässä tapahtuu aina siihen asti kunnes ruvettiin ajelemaan crackmapexec ja metasploittua jotta saataisiin aikaiseksi winRM logini.
Lopullinen korkkaus saatiin aikaiseksi ymmärtääkseni seuraavasti:

* katsottiin mitä softaa palvelimella oli asennettu ja löydettiin Firefox
* luettiin Firefoxilla ajetut prosessit ja niiden IDt
* ladattiin microsoftin sysinternals suite
* upittiin edellämainitusta palvelimella korkatun käyttäjän hakemistoon procdum64.exe
* tämä ajamalla saatiin tehtyä Firefoxin prosessista tiedosto, jota lukemalla selvisi admin salasana.
    
Tässä oli todella paljon asioita mitä en ole käynyt läpi ja tämä oli hämmentävää katsottavaa varsinkin kun videolla selostaja itse
ajautuu umpikujaan. Muutamia uusia työkaluja mitä tästä videosta jäi mieleen oli:

* Metasploit
* Crackmapexec
* Hashcat (ssh kracken?)
* Cisco Type 7 password decrypter
* Evil winrm

### HackTheBox - Buff (https://www.youtube.com/watch?v=-KBm3tBNK74)

Ajattelin katsoa tämän kyseisen palvelimen korkkaamisen, kun tämä oli se mitä katseltiin oppitunneillakin niin jäi kiinnostamaan.
Lopputulos kumminkin oli se, etten itse olisi kylläkään keksinyt jo opituilla taidoilla miten tämän saisi korkattua. Video oli pitkä ja jälleen putosin täysin kärryiltä kun tässä on niin monta tmux-ikkunaa auki, että tätä oli vaikea seurata. Kumminkin mieleen jäi joitain tekniikoita ja työkaluja.

Palveimen avointen porttien tiedustelu alkoi samalla tavalla kuin edellisenkin:

    sudo nmap -v -sC -sV -oA nmap/buff 10.10.10.198
    
Sitten ajettiin taustalla gobuster, mutta tämän tuottamia tietoja ei kai käytetty missään vaiheessa:

    gobuster dir -u http://10.10.10.198:8080/ -w /opt/SecLists/Discovery/Web-Content/raft-small-words.txt -x php -o gobustet.out

Mielenkiintoinen kikka oli kokeilla ladata wgetillä nettisivuilta bannerin kuva, ja katsoa exiftoolilla kuvan tietoa. Analyysillä pystyttiin esim. kertomaan milloin kuva on asetettu palvelimelle ja tästä voitaisiin päätellä onko framework vanhentunut tms.

Selattiin palvelimella eri sivuja jos löytyisi nimiä, joita kokella loginiin, mutta löydettiin millä sivusto on tehty: Gym management software 1.0 ja 
kokeillaan etsiä haavoittuvuutta searchsploitilla. 

        searchsploit gym management

Löydetään tälle versiolle haavoittuvuus ja lukastaan mitä koodi tekee. Muokataan python koodia siten että käytetään localhostia proxynä. Sen jälkeen ajetaan python tiedosto urlia vasten.

* Burb suitessa muokattiin ulosmenevä requesti siten että saadaan kiinni sekä GET ja POST shell_excecillä
* Vaihdetaan ulosmenevät requestit postiksi: tästä ei jää jälkiä apachen access logiin
* Yritetään saavuttaa reverse shell.. *tässä vaiheessa minä tipahdan kärryiltä*
* netcat ajettiin kai kohdeserverillä ja tarjottiin powershell yhteyttä hyökkääjälle?

Loppupuolisko videosta oli hyvin hämmentävä, enkä hirveästi kyennyyt seuraamaan mitä tässä tarkalleen tehtiin. Kumminkin poimin uusia työkaluja mitä tutkia mm.

* Gobuster
* Burb suite
* Chisel
* Msfvenom (?)
* winPEAS ?
* meterpeter / staged / stageless

## a) Etsi ja kokeile 5 uutta työkalua jostain lukemastasi/katsomastasi läpikävelystä.

### Gobuster

Kokeilen ensin tiedustella sisäverkossani tiedostopalvelimen avoimia portteja yllämainitulla tavalla.

    sudo nmap -v -sC -sV -oA asus 192.168.68.105
    
Sitten asensin gobusterin, mutta totesin että minun pitää asentaa myös wordlist. Tässä menikin hetki mutta tajusin asentaa SecLists paketin.

![Screenshot from 2020-11-30 01-31-41](https://user-images.githubusercontent.com/54954455/100556761-31baf880-32ad-11eb-8833-d19e07b2d577.png)

Tämän jälkeen kurkkasin mitä tohon output-tiedostoon oli kirjoitettu:

![Screenshot from 2020-11-30 01-44-31](https://user-images.githubusercontent.com/54954455/100556822-a42bd880-32ad-11eb-9602-e1cf49bb3ed7.png)

Ilmeoisesti löydettiin muutama kansio, mutta näihin ei ainakaan pääse selaimen kautta suoraan.

### Searchsploit

Tiedostopalvelimen hallintasofta näyttäisi olevan Asustor ADM 3.5 niin kokeilen etsiä searchsploitilla sille haavoittuvuutta.

![Screenshot from 2020-11-30 09-51-18](https://user-images.githubusercontent.com/54954455/100583830-ec6de980-32f3-11eb-8000-b5f4be2e92fa.png)

Ajan ´searchsploit asustor´

Searchsploit palauttaa haulle:

![Screenshot from 2020-11-30 10-00-33](https://user-images.githubusercontent.com/54954455/100583578-8c774300-32f3-11eb-8e58-0394e33cf9f5.png)


## b) Tiedustele ja analysoi 5 htb konetta perusteellisesti.

## c) Nimeä 1-3 walktrough:ta, joissa tunkeudutaan samantapaisiin palveluihin, joita käsittelit kohdassa b.


        

        
        
