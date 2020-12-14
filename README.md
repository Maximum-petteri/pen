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

Aloitetaan porttiskannauksella metasploitissa:

    db_nmap -sC -sV -oA meta2 192.168.56.101

Ja tämä antaakin paljon kaikkea ihmeteltävää, laitan tähän -sS -sV tuloksen:

    PORT     STATE SERVICE     VERSION
    21/tcp   open  ftp         vsftpd 2.3.4
    22/tcp   open  ssh         OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)
    23/tcp   open  telnet      Linux telnetd
    25/tcp   open  smtp        Postfix smtpd
    53/tcp   open  domain      ISC BIND 9.4.2
    80/tcp   open  http        Apache httpd 2.2.8 ((Ubuntu) DAV/2)
    111/tcp  open  rpcbind     2 (RPC #100000)
    139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
    445/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
    512/tcp  open  exec        netkit-rsh rexecd
    513/tcp  open  login
    514/tcp  open  shell       Netkit rshd
    1099/tcp open  java-rmi    GNU Classpath grmiregistry
    1524/tcp open  bindshell   Metasploitable root shell
    2049/tcp open  nfs         2-4 (RPC #100003)
    2121/tcp open  ftp         ProFTPD 1.3.1
    3306/tcp open  mysql       MySQL 5.0.51a-3ubuntu5
    5432/tcp open  postgresql  PostgreSQL DB 8.3.0 - 8.3.7
    5900/tcp open  vnc         VNC (protocol 3.3)
    6000/tcp open  X11         (access denied)
    6667/tcp open  irc         UnrealIRCd
    8009/tcp open  ajp13       Apache Jserv (Protocol v1.3)
    8180/tcp open  http        Apache Tomcat/Coyote JSP engine 1.1
    MAC Address: 08:00:27:7A:99:C0 (Oracle VirtualBox virtual NIC)   
    
Yhdellä silmäyksellä voisi päätellä että tässä olisi paljon kaikkea kivaa tökittävää ja näitä voisi oikeastaan käydä järjestyksessä läpi, mutta huomasin että tuolla on portti 513 auki. Googletan pikaisesti mikä tämä on, ja ilmeisesti tämä on sen verran vanha tekniikka että hakutulokset antaa pelkkiä exploitteja.
Kokeilen rlogin -l root 192.168.56.101

![Screenshot_2020-12-07_21-45-51](https://user-images.githubusercontent.com/54954455/101397673-ac9b9900-38d5-11eb-9f13-23f8dbb591b7.png)

Telnet-palvelu olisi varmaan seuraava low hanging fruit mutta tuollahan on irkkipalvelin niin katsotaan mitä searchsploit sanoo tohon:

![Screenshot_2020-12-07_21-56-27](https://user-images.githubusercontent.com/54954455/101398663-1a949000-38d7-11eb-9c46-6943a2dbb277.png)

ja kokeillaan vaikka tuota ensimmäistä vaihtoehtoa *UnrealIRCd 3.2.8.1 - Backdoor Command Execution (Metasploit).

![Screenshot_2020-12-07_22-00-42](https://user-images.githubusercontent.com/54954455/101399099-ae665c00-38d7-11eb-9591-9f89c6b7a6b0.png)

Hups, jotain taisi unohtua:

![Screenshot_2020-12-07_22-06-44](https://user-images.githubusercontent.com/54954455/101399729-8d523b00-38d8-11eb-8588-e99cc69240f8.png)

valitsin payloadin, mutta metasploit valittaa että en ole asettanut LHOST parametria.

![Screenshot_2020-12-07_22-16-59](https://user-images.githubusercontent.com/54954455/101400695-f1c1ca00-38d9-11eb-9a12-fc5c34f92036.png)

Väittäisin että jossain on nyt vikaa. Kokeilin vielä toista payloadia mutta sain saman tuloksen.
Kokeilen porttia 21. Ensin etsin löytyykö tälle jotain haavoittuvuutta.

![Screenshot_2020-12-07_22-32-12](https://user-images.githubusercontent.com/54954455/101402625-a9f07200-38dc-11eb-81ac-d2472999c9ee.png)

Lataan tuon haavoittuvuuden käyttöön ja asetan RHOST ja RPORTS parametrit.

![Screenshot_2020-12-07_22-35-37](https://user-images.githubusercontent.com/54954455/101402561-934a1b00-38dc-11eb-9f0f-469328b34435.png)

Laitetaan haavoittuvuus ajoon ja maali!

![Screenshot_2020-12-07_22-40-03](https://user-images.githubusercontent.com/54954455/101402990-3438d600-38dd-11eb-961f-171988dde953.png)

# H6

## a) Palauta kaikki. Laita tähän a-kohtaan linkki jokaiseen kotitehtäväraporttiisi.

Helppoa, kaikki on tässä samassa readme-dokumentissa:

https://github.com/Maximum-petteri/pen#h1  
https://github.com/Maximum-petteri/pen#h2  
https://github.com/Maximum-petteri/pen#h3  
https://github.com/Maximum-petteri/pen#h4  
https://github.com/Maximum-petteri/pen#h5  


## c) Tee 5 tiivistettä eri ohjelmilla ja arvaa ne hashcatilla.

Aloitetaan luomalla hashes.txt ja wordlist.txt tiedostot johon keräilen generoitua salasanatiivisteitä ja varsinaisia salasaKoknoja.
Seuraavaksi kokeilen tehdä MD5-tiivisteen md5sum-komennolla `echo "password" -n | md5sum`.

![Screenshot from 2020-12-13 22-43-34](https://user-images.githubusercontent.com/54954455/102024605-bd4b8380-3d9b-11eb-9135-a1af4a418e84.png)

`-n` parametri poistaa echon ulostulosta rivinvaihdon jolloin md5sum on eheä ilman rivinvaihtoa.

Ajan hashcatin komennolla `hashcat -m 0 -a 0 hashes.txt wordlist.txt --force`, jossa parametrit `-m 0` määrittää MD5 tiivisteen ja `-a 0` lukee tekstitiedostosta jokaisen rivin kandidaattina salasanalle. Joudun käyttämään `--force` parametria, koska vanhassa kannettavassa ei ole mitään näytönohjainta, jota hashcat voisi käyttää laskentaan.

![Screenshot from 2020-12-13 22-44-56](https://user-images.githubusercontent.com/54954455/102024891-42836800-3d9d-11eb-9165-8ff1bab02016.png)

Kokeilin perään vielä hitusen monimutkaisemman salasanan:

![Screenshot from 2020-12-13 22-45-44](https://user-images.githubusercontent.com/54954455/102024992-cdfcf900-3d9d-11eb-937e-8cc5c7012880.png)

Ajoin hashcatin samoilla parametreillä kuin ylempänä, ja eihän tuo salasana tuottanut sen enempää tuskaa.

![Screenshot from 2020-12-13 22-47-08](https://user-images.githubusercontent.com/54954455/102025005-dfde9c00-3d9d-11eb-97a9-1e0917fbef14.png)

Kokeillaan seuraavaksi tehdä salasana openssl-komennolla seuraavasti: `openssl passwd -1 -salt xyz password` jossa parametri `-1` tekee MD5 tiivisteen, ja `-salt` "suolaa" salasanatiivisteen lisäämällä `xyz` kirjainyhdistelmän tiivisteeseen, tosin tämä on helposti luettavissa suoraan tiivisteestä.
Tarkistan vielä hashid-komennolla hashes.txt tiedoston että tämä löytää kahta eri tyyppiä tiivisteitä tiedostosta.

![Screenshot from 2020-12-13 22-50-08](https://user-images.githubusercontent.com/54954455/102025202-284a8980-3d9f-11eb-8f8d-feb19d2dfb9f.png)

Uusi tiiviste näyttäisi olevan tyyppiä MD5 Crypt, Cisco-IOS(MD5) tai FreeBSD MD5. Lunttaan sivustolta https://hashcat.net/wiki/doku.php?id=example_hashes mitä numeroa tulisi käyttää tämän tiivisteen purkamiseen ja kokeilen seuraavaa: `hashcat -m 500 -a 0 hashes.txt wordlist.txt --force`.

![Screenshot from 2020-12-13 22-52-48](https://user-images.githubusercontent.com/54954455/102025338-d81ff700-3d9f-11eb-9d54-febd0145cf99.png)

ja pienen hetken päästä tuokin on saatu auki:

![Screenshot from 2020-12-13 22-52-31](https://user-images.githubusercontent.com/54954455/102025359-0c93b300-3da0-11eb-8ecb-c3ddf5f45f41.png)

Seuraavaksi kokeillaan tehdä mkpasswd-komennolla vähän pidempi, 256 merkkiä pitkä salasanatiiviste: `mkpasswd --method=SHA-256 --stdin`.

![Screenshot from 2020-12-13 22-58-16](https://user-images.githubusercontent.com/54954455/102025408-78761b80-3da0-11eb-991c-75f492202e7f.png)

Katson hashid-komennolla että kyseiselle tiivisteelle annetaan tyyppi SHA-256 Crypt. Ajan hashcatin parametreilla `hashcat -m 7400 -a 0 hashes.txt wordlist.txt --force`, mutta tämäkään ei tuota mitään ongelmaa.

![Screenshot from 2020-12-13 23-01-38](https://user-images.githubusercontent.com/54954455/102025494-fcc89e80-3da0-11eb-9779-6e97c91ff0af.png)

Päätän sitten venyttää salasanatiivistettä 512 merkin pituiseksi:

![Screenshot from 2020-12-13 23-02-07](https://user-images.githubusercontent.com/54954455/102025508-1b2e9a00-3da1-11eb-98f8-038ae7f7a3df.png)

ja kun ajoin hashcatin parametrilla `-m 1800`, joka viittaa SHA-512 Crypt -tiivisteeseen sain vihdoin virheen:

![Screenshot from 2020-12-13 23-05-20](https://user-images.githubusercontent.com/54954455/102025545-6052cc00-3da1-11eb-96e3-5d04bbc352e0.png)

Hashcat antoi minulle linkin artikkeliin https://hashcat.net/wiki/doku.php?id=frequently_asked_questions#how_to_create_more_work_for_full_speed jonka nopeasti silmäilemällä päättelin että minulla ei ollut minun salasanakandidaattilistassa tarpeeksi salasanoja, joten kokeilin copy/pastea rockyou.txt -tiedostosta ensimmäiset pari sivua omaan wordlist-tiedostoon mukaan. Tämä auttoi.

![Screenshot from 2020-12-13 23-11-11](https://user-images.githubusercontent.com/54954455/102025612-dd7e4100-3da1-11eb-847e-247ba40205ce.png)

## d) Kokeile hydraa johonkin uuteen maaliin.

Kokeilen ajaa hydraa Metasploitable 2:n palveluihin ftp, ssh ja telnet komennolla: `hyrda -L users.txt -P wordlist.txt [ftp, ssh tai telnet]://192.168.56.101  
-u -V -f`.

Lista parametreista:

`-L` käyttää tekstitiedoston rivejä käyttäjälistana.  
`-P` lista salasanoille.  
`-u` kiertää käyttäjälistan nimet, jotta ei tule peräkkäisiä yrityksiä samalle käyttäjälle.  
`-f` lopettaa yrittämisen ensimmäiseen onnistumiseen.  
`-V` näyttää ruudulla miten ohjelman ajaminen etenee.  

![Screenshot from 2020-12-14 10-18-24](https://user-images.githubusercontent.com/54954455/102056925-d08b3d00-3df5-11eb-8611-f2d45393b6b8.png)

![Screenshot from 2020-12-14 10-18-48](https://user-images.githubusercontent.com/54954455/102056957-dd0f9580-3df5-11eb-8997-793d41f8dcb9.png)

## e) Kokeile hydraa omaan weppilomakkeeseen.



## f) Tee oma sanalista itse tekemästäsi ja keksimästäsi weppisivusta.

Kokeilin ajaa Cewl-komentoa keskeneräiselle weppisivulleni https://badloop.com seuraavanlaisesti: `cewl -a -w cewl_test.txt https://badloop.com`. Parametrit `-a` ottaa mukaan sivun metadatan ja `-w` kirjoittaa tuloksen tekstitiedostoon.
Sivuillani on tällä hetkellä vain ansi-grafiikkaa mutta yksi mielenkiintoinen asia osui silmään käydessä läpi tuloksia:

![Screenshot from 2020-12-14 10-35-19](https://user-images.githubusercontent.com/54954455/102058787-b868ed00-3df8-11eb-94ec-ad34747e5148.png)

![Screenshot from 2020-12-14 10-40-29](https://user-images.githubusercontent.com/54954455/102058873-d5052500-3df8-11eb-8f02-1197f38c6329.png)

Listassa näkyy sana **compiling**, joka ei esiinny kyseisen sivuston koodissa, vaan badloop.com:in alla olevan https://federalaudioreserve.com:in etusivulla,

![Screenshot from 2020-12-14 10-42-31](https://user-images.githubusercontent.com/54954455/102059095-1a295700-3df9-11eb-9cd8-da9d9e8dfe67.png)

Tämä johtunee siitä että cewl-komento oletuksena asettaa `-d` parametrin kakkoseksi, jolloin cewl löytää jollain logiikalla myös samassa osoitteessa asuvan alasivuston sisällön.

![Screenshot from 2020-12-14 10-43-20](https://user-images.githubusercontent.com/54954455/102059367-7db38480-3df9-11eb-9734-73229b3a18ac.png)

