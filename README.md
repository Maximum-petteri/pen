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

Ajan *searchsploit asustor*

Searchsploit palauttaa haulle:

![Screenshot from 2020-11-30 10-00-33](https://user-images.githubusercontent.com/54954455/100583578-8c774300-32f3-11eb-8e58-0394e33cf9f5.png)

Näyttäisi siltä että palvelimella olevalle versiolle ei tällä hetkellä olisi tunnettuja haavoittuvuuksia. Hyvä niin! :)

## b) Tiedustele ja analysoi 5 htb konetta perusteellisesti.

### HTB Time (10.10.10.214)

Skannaan nmapilla seuraavan ip:n käyttäen yllä mainittuja menetelmiä:

    sudo nmap -sC -sV -oA htbtime 10.10.10.214

Tämä palauttaa:

    22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.1 (Ubuntu Linux; protocol 2.0)
    80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
    |_http-server-header: Apache/2.4.41 (Ubuntu)
    |_http-title: Online JSON parser
    Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel 

Apache näyttäisi olevan kutakuinkin päivitetty kuin openssh. Tarkistin vielä searchsploitista, mutta en löytänyt mitään. 
Kokeilen ajaa:
    
    sudo nmap -v -script vuln -oA htb_vuln_time 10.10.10.214

Mutta tämäkään ei tuota tulosta. Lähtisin sitten purkamaan tuota selaimen kautta, jossa näyttäisi pyörivän Online JSON Parser.

### HTB APT (10.10.10.213)

    80/tcp  open  http    Microsoft IIS httpd 10.0
    | http-methods: 
    |   Supported Methods: OPTIONS TRACE GET HEAD POST
    |_  Potentially risky methods: TRACE
    |_http-server-header: Microsoft-IIS/10.0
    |_http-title: Gigantic Hosting | Home
    135/tcp open  msrpc   Microsoft Windows RPC
    Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Tässä olikin jo mielenkiintoisempi skannaus. Goolella sain haettua Cross Site Tracing (https://owasp.org/www-community/attacks/Cross_Site_Tracing), joka liittyy tuohon TRACE-metodiin. Tämä olisi varmasti päälimmäisin asia mitä lähtisin kokeilemaan.

### HTB Laboratory (10.10.10.216)

    22/tcp  open  ssh      OpenSSH 8.2p1 Ubuntu 4ubuntu0.1 (Ubuntu Linux; protocol 2.0)
    80/tcp  open  http     Apache httpd 2.4.41
    | http-methods: 
    |_  Supported Methods: GET HEAD POST OPTIONS
    |_http-server-header: Apache/2.4.41 (Ubuntu)
    |_http-title: Did not follow redirect to https://laboratory.htb/
    443/tcp open  ssl/http Apache httpd 2.4.41 ((Ubuntu))
    | http-methods: 
    |_  Supported Methods: GET POST OPTIONS HEAD
    |_http-server-header: Apache/2.4.41 (Ubuntu)
    |_http-title: The Laboratory
    | ssl-cert: Subject: commonName=laboratory.htb
    | Subject Alternative Name: DNS:git.laboratory.htb
    | Issuer: commonName=laboratory.htb
    | Public Key type: rsa
    | Public Key bits: 4096
    | Signature Algorithm: sha256WithRSAEncryption
    | Not valid before: 2020-07-05T10:39:28
    | Not valid after:  2024-03-03T10:39:28
    | MD5:   2873 91a5 5022 f323 4b95 df98 b61a eb6c
    |_SHA-1: 0875 3a7e eef6 8f50 0349 510d 9fbf abc3 c70a a1ca
    | tls-alpn: 
    |_  http/1.1
    Service Info: Host: laboratory.htb; OS: Linux; CPE: cpe:/o:linux:linux_kernel

En nyt löydä ainakaan tästä skannauksesta mitään poikkeavaa? Yritin mennä selaimella tonne, mutta selain ei löydä palvelinta.
Tässä varmaan on asetukset jotenkin pilalla, joka voisi olla ehkä hyökkäysvektori, mutta en osaa päätellä miten sitä lähtisi tekemään.

### HTB OpenKeys (10.10.10.199)

    22/tcp open  ssh     OpenSSH 8.1 (protocol 2.0)
    | ssh-hostkey: 
    |   3072 5e:ff:81:e9:1f:9b:f8:9a:25:df:5d:82:1a:dd:7a:81 (RSA)
    |_  256 64:7a:5a:52:85:c5:6d:d5:4a:6b:a7:1a:9a:8a:b9:bb (ECDSA)
    80/tcp open  http    OpenBSD httpd
    |_http-title: Site doesn't have a title (text/html).
    
Tässä nyt ei hirveästi infoa tullut. Nimen perusteella korkkaus liittyisi avaimiin mutta nuo hostkeyt ei tietääkseni ole mitenkään poikkeavia. Kokeilin ajaa vielä
nmappia -A vivulla tohon porttiin 80, jos siitä vaikka saisi lisätietoa:

    80/tcp open  http    OpenBSD httpd
    |_http-title: Site doesn't have a title (text/html).
    Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
    Device type: general purpose|firewall
    Running (JUST GUESSING): OpenBSD 4.X|6.X|5.X|3.X (95%), FreeBSD 10.X|7.X (91%), Cisco AsyncOS 7.X (87%)
    OS CPE: cpe:/o:openbsd:openbsd:4.4 cpe:/o:openbsd:openbsd:6 cpe:/o:openbsd:openbsd:5 cpe:/o:openbsd:openbsd:3 cpe:/o:freebsd:freebsd:10.0 cpe:/o:freebsd:freebsd:7.0 cpe:/h:cisco:ironport_c650 cpe:/o:cisco:asyncos:7.0.1
    Aggressive OS guesses: OpenBSD 4.4 - 4.5 (95%), OpenBSD 6.0 - 6.1 (95%), OpenBSD 5.0 - 5.8 (95%), OpenBSD 4.1 (93%), OpenBSD 5.0 (93%), OpenBSD 4.2 (93%), OpenBSD 4.0 (93%), OpenBSD 3.8 - 4.7 (92%), OpenBSD 4.6 (92%), OpenBSD 4.7 (92%)
    No exact OS matches for host (test conditions non-ideal).

Ei tuokaan mitään totuutta tuottanut :( Searchploit palauttaa liikaa openbsd tuloksia, että tuon puolesta tämä näyttäisi olevan vähän hakuammuntaa.

### HTB Luanne (10.10.10.218)

    22/tcp   open  ssh     OpenSSH 8.0 (NetBSD 20190418-hpn13v14-lpk; protocol 2.0)
    | ssh-hostkey: 
    |   3072 20:97:7f:6c:4a:6e:5d:20:cf:fd:a3:aa:a9:0d:37:db (RSA)
    |   521 35:c3:29:e1:87:70:6d:73:74:b2:a9:a2:04:a9:66:69 (ECDSA)
    |_  256 b3:bd:31:6d:cc:22:6b:18:ed:27:66:b4:a7:2a:e4:a5 (ED25519)
    80/tcp   open  http    nginx 1.19.0
    | http-auth: 
    | HTTP/1.1 401 Unauthorized\x0D
    |_  Basic realm=.
    | http-methods: 
    |_  Supported Methods: GET HEAD POST
    | http-robots.txt: 1 disallowed entry 
    |_/weather
    |_http-server-header: nginx/1.19.0
    |_http-title: 401 Unauthorized
    9001/tcp open  http    Medusa httpd 1.12 (Supervisor process manager)
    | http-auth: 
    | HTTP/1.1 401 Unauthorized\x0D
    |_  Basic realm=default
    |_http-server-header: Medusa/1.12
    |_http-title: Error response
    Service Info: OS: NetBSD; CPE: cpe:/o:netbsd:netbsd

Taaskaan en osaa nyt analysoida tätä oikein. Selain vaatii heti kirjautumista tässä osoitteessa. Silmään pistää kumminkin tuo Medusa (supervisor process manager),
mutta ainakaan searchsploit ei löydä aiheeseen mitään. Yritin asentaa vielä Metasploitin koska tämä rupeaa olemaan vähän toivotonta, mutta tämäkin ei nyt ihan onnistunut helposti.


# H5 Sitting Ducks

## z) Lue artikkelit ja katso videot, tee kustakin muistiinpanot (muutama ranskalainen viiva per artikkeli/video).

### Miksi Metasploit?
* Avoin lähdekoodi.
    * Käyttäjillä vapaus lisätä omia moduuleja.
* Helppokäyttöinen
    * Voidaan tehdä testejä isoille verkoille nopeasti
    * Testaamisessa voidaan vaihtaa payloadeja helposti
* Jättää vähemmän jälkiä
    * Metasploit ei kaada palveluita yhtä todennäköisesti kuin custom-koodi

(tässä vaiheessa hirveästi ongelmia metasploitin databasen kanssa)

## a) Metasploitable. Asenna Metasploitable 2. Murtaudu sille useilla tavoilla




        
