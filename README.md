# H2:

A2

Authentication bypasses: Tutkitaan sivuston HTML-koodia inspectorilla ja kokeillaan manuaalisesti vaihtaa kysymysten muuttujat:
    
    "secQuestion0" ja "secQuestion1" => "secQuestion2" ja "secQuestion3"

Näitä todennäköisesti ei ole olemassa, ja näiden arvoiksi "test".

Toimii. 
En kyllä osaa hahmottaa miksi, kyllähän sivuston pitäisi osata tarkistaa invalidit muuttujat myös?

Secure passwords:

käytän salasanan sijaan "salalausetta" ja yleensä suomeksi.

    Estimated guesses needed to crack your password: 15000000000000002000000000000000
    Score: 4/4
    Estimated cracking time: 292471208677 years 195 days 15 hours 30 minutes 7 secondsScore: 4/5 

A3

Insecure Login:

Käynnistetään Wireshark loopback-asetuksella.
Painoin login-nappulaa ja menen wiresharkkiin etsimään HTTP-protokollalla POST-viestiä.
Wiresharkista löytyy kaksi POST-riviä ja avaan aikajanalla ensimmäisen näistä. Skrollailen ja luen viestin sisältöä ja 
viimeinen rivi näyttää epäilyttävältä:
    
    {"username":"CaptainJack","password":"BlackPearl"}.

A7

Try It! Reflected XSS:

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

A8

Cross-Site Request Forgeries:

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
