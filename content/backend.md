# Backend

<p class="author">Emil Folino</p>

Vi börjar denna veckan med att skaffa oss en droplet, en server i molnet. På vår server installerar vi programvara och konfigurerar servern för att säkra upp servern och för att vi kan köra nodejs applikationer. Vi tittar sedan på hur vi kan skapa ett API som svarar med JSON med hjälp av Express och en SQLite databas. Som avslutning på denna veckan driftsätter vi både vår backend och den frontend applikation vi skapade tidigare i kursen.

Vi vänder oss nu till dokumentationen för [Node](https://nodejs.org/en/docs/) och [Express](http://expressjs.com/) för att ytterligare se vad man kan göra med Express. Låt oss komma igång med grunderna i Express och hur man sätter upp en applikationsserver som även kan fungera som en vanlig webbserver.



## Titta

Vi ska denna veckan skriva en del asynkron kod och det kan vara bra att ha lite extra bra koll på hur "Event-loop" fungerar i JavaScript. Denna video ger en bra introduktion till hur det fungerar både för frontend och backend.

<div class='embed-container'><iframe width="560" height="315" src="https://www.youtube.com/embed/8aGhZQkoFbQ" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe></div>



## Material

Vi ska i följande stycken först titta på hur vi med hjälp av GitHub Education Pack och DigitalOcean skapar en droplet. I slutet av veckan har vi driftsatt både vår frontend applikation och en simpel backend applikation.



#### En server i molnet

Se till att ha din student e-postadress nära till hands då den behövs för att få tillgång till GitHub Education Pack.



#### GitHub Education Pack

För att få tillgång till rabatter och rabattkoder som erbjuds i [GitHub Education Pack](https://education.github.com/pack) behöver du GitHub veta att du är student. Gå till den länkade sidan och tryck på den blåa knappen "Get your Pack". Viktigt att du använder din student mail när du registrerar dig då mailen måste vara kopplat till en undervisningsinstitution.



#### Digital Ocean

När du är verifierad via GitHub får du tillgång till en rabattkod för Digital Ocean. Efter det går du till [Digital Ocean Sign Up](https://cloud.digitalocean.com/registrations/new) och skapar ett konto. Du behöver ange ett kreditkort, men vi kommer sedan använda rabattkoden så det kommer inte kosta något.

När du har skapat kontot gå till Account längst upp till höger under din användare logga. Gå sedan till Billing fliken och scrolla ner till Promo Code. Här lägger du in rabattkoden du fick från Github Education Pack när du tryckte på länken 'request your offer code'.

Gå sedan till första sidan och tryck 'Get started with a Droplet'. Instruktioner i kommande stycken och resten av kursen kommer utgå från en Debian Stretch (9.x) droplet, så en stark rekommendation är att välja en sån droplet. Jag rekommenderar att ni kör en 10$/månad droplet, då man får bra prestanda och samtidigt inte använder hela rabatten under kursens gång. Välj Frankfurt eller London som region och lägg till din `id_rsa.pub` SSH nyckel så du kan logga in på servern.



#### Första 10 minuter på en server

Med utgångspunkt i artiklar som [My First 5 Minutes On A Server; Or, Essential Security for Linux Servers](https://plusbryan.com/my-first-5-minutes-on-a-server-or-essential-security-for-linux-servers) och [My First 10 Minutes On a Server - Primer for Securing Ubuntu](https://www.codelitt.com/blog/my-first-10-minutes-on-a-server-primer-for-securing-ubuntu/) ska vi i följande stycke titta på hur vi säkrar upp en Linux-baserad server av Ubuntu eller Debian variant.



#### Logga in på servern

Vi loggar in på servern genom att använda SSH via terminalen med kommandot. `ssh root@[IP]` ersätt din [IP] med den IP som visas för din droplet.



#### Lösenord

Än så länge har vi inte ens ett lösenord till vår `root` användare så låt oss se till att sätter ett lösenord. Välj ett säkert lösenord och med säkert lösenord menas ett slumpat och komplext lösenord. Jag rekommenderar starkt att använda en Password Manager och skapa lösenordet med hjälp av denna Password Manager inställt på den mest komplexa inställningen du kan hitta. För Mac och Linux rekommenderas [pass](https://www.passwordstore.org) och för Windows verkar [LastPass](https://www.lastpass.com) vara det mest använda gratis programmet som finns.

När du har skapat ett slumpmässigt och komplext lösenord skriv följande kommando och följ instruktionerna.

```shell
$passwd
```



#### Uppdatera servern

Nästa steg är att uppdatera serverns programvara till senaste version genom att använda verktyget `apt-get`.

```shell
$apt-get update
$apt-get upgrade
```



#### Skapa din egen användare

Vi vill aldrig logga in som `root` då `root` har tillgång till för mycket. Så vi skapar en egen användare `deploy` med följande kommandon. Du kan byta ut `deploy` mot vad som helst, men då ska du göra det i alla följande kommandon. De två första kommandon är för att rensa bort en befintlig användare Digital Ocean lägger till när debian installeras.

```shell
$apt-get remove --purge unscd
$userdel -r debian
$useradd deploy
$mkdir /home/deploy
$mkdir /home/deploy/.ssh
$chmod 700 /home/deploy/.ssh
```

Vi passar på att i samma veva ställa in vilken förvald terminal vår nya använda ska använda, vi väljer `bash` då vi är vana vid den.

```shell
$usermod -s /bin/bash deploy
```



#### Stänga av inloggning med lösenord

Lösenord kan knäckas.

Därför använder vi istället SSH nycklar för att autentisera oss mot servern. Skapa och öppna filen med kommandot `nano /home/deploy/.ssh/authorized_keys` och lägg innehållet av din lokala `.ssh/id_rsa.pub` nyckel i den filen på en rad.

När du har lagt till nyckeln kör du följande två kommandon för att sätta korrekta rättigheter på katalogen och filen.

```shell
$chmod 400 /home/deploy/.ssh/authorized_keys
$chown deploy:deploy /home/deploy -R
```

Testa nu att logga in i ett nytt terminalfönster med kommandot `ssh deploy@[IP]`. Vi har kvar terminal fönstret där vi loggade in som root om något skulle gå fel.

Vi skapar sedan ett lösenord för `deploy` användaren från root terminalfönstret, `passwd deploy`, använd igen ett långt och slumpmässigt lösenord. Och vi lägger till `deploy` som sudo användare med kommandot `usermod -aG sudo deploy`.

Som vi sagt tidigare vill vi bara kunna logga in med SSH nycklar. Vi gör detta genom att ändra tre rader i filen `/etc/ssh/sshd_config`. Öppna filen med din texteditor på din server till exempel nano med kommandot `nano /etc/ssh/sshd_config`.

Hitta raderna nedan och se till att ändra från yes till no. Raderna ligger inte på samma ställe i filer, så ibland får man leta en liten stund. Den sista raden nedan får du skriva in själv.

```shell
$PermitRootLogin no
$PasswordAuthentication no
$AllowUsers deploy
```

Spara filen och starta om SSH med hjälp av kommandot `service ssh restart`. Testa nu att logga ut och in i ditt andra terminal fönster där du tidigare var inloggat som `deploy`.



#### Brandvägg

Först stänger vi ner uppkopplingen för användaren root genom att skriva exit i terminalfönstret. Alla kommandon vi kommer köra från och med nu körs som användaren deploy.

Vi använder oss av brandväggen `ufw` för att stänga och öppna portar till vår server. Installera med kommandot `sudo apt-get install ufw`.

Vi vill nu öppna upp för trafik på 3 portar 22 för SSH, 80 för HTTP och 443 för HTTPS. Vi gör det med hjälp av följande kommandon.

```shell
$sudo ufw allow 22
$sudo ufw allow 80
$sudo ufw allow 443
$sudo ufw disable
$sudo ufw enable
```



#### Automagiska uppdateringar

Vi vill inte hålla på att manuellt uppdatera vår server, men vi vill inte heller sakna en patch när de kommer så vi kommer använda oss av verktyget unattended-upgrades. Vi installerar med `sudo apt-get install unattended-upgrades`.

Uppdatera filen `/etc/apt/apt.conf.d/10periodic` så den innehåller nedanstående.

```shell
APT::Periodic::Update-Package-Lists "1";
APT::Periodic::Download-Upgradeable-Packages "1";
APT::Periodic::AutocleanInterval "7";
APT::Periodic::Unattended-Upgrade "1";
```

Uppdatera även filen `/etc/apt/apt.conf.d/50unattended-upgrades` så den ser ut som nedan.

```shell
Unattended-Upgrade::Allowed-Origins {
    "${distro_id}:${distro_codename}";
    "${distro_id}:${distro_codename}-security";
    "${distro_id}ESM:${distro_codename}";
    //"${distro_id}:${distro_codename}-updates";
};
```



#### fail2ban

Vårt sista steg är att installera verktyget fail2ban som används för att automatiskt kolla logfiler och stoppa aktivitet som vi inte vill ha på vår server. Vi installerar och låter ursprungsinställningarna göra sitt jobb. Vi installerar med `sudo apt-get install fail2ban`.



#### En domän till din server

Som en del av Github Education Pack får du som student även ett domän-namn på top-domänen .me från registratorn namecheap gratis under ett år. Om du vill använda en annan registrator är det fritt fram.

För att använda namecheap tryck på länken "Get access by connecting your GitHub account on Namecheap" och knyta ihop ditt GitHub konto med namecheap och skapa en användare.

När du har kopplat din användare kommer du till en sida där du skapar ditt domännamn. Skriv in din text i kommande bilder har jag använt det domännamn jag valde 'jsramverk.me'.

![Fyll i nameservers hos namecheap.](https://dbwebb.se/image/ramverk2/namecheap-nameservers.png?w=w3)

Gå sedan till Digital Ocean och välj Networking>Domains. Här Väljer du att skapa den valda domänen.

![Skapa domän på Digital Ocean.](https://dbwebb.se/image/ramverk2/do-domains.png?w=w3)

Vi vill sedan peka domänen till vår droplet och för att komma åt root-domänen anger vi @. Vill vi ange en subdomän anger vi subdomänen.

![Peka domän till droplet på Digital Ocean.](https://dbwebb.se/image/ramverk2/do-domain-names.png?w=w3)



#### Installera programvara

Vi ska i denna del installera programvara för att vi kan köra både frontend och backend applikationer på vår server.



#### Installera nginx

Vi installerar webbservern nginx med hjälp av kommandot `sudo apt-get install nginx`. Du ska nu kunna gå till din domän-adress och där se Welcome to nginx! Ibland kan det ta en liten stund innan alla ändringar slå igenom, så nu är att bra tillfälle att hämta kaffe eller gå en runda om det inte fungerar direkt.



#### Installera nodejs och npm

Vi vill ha nodejs och npm installerat så vi kan köra en backend på vår server. Vi installerar LTS (Long Term Support) versionen då detta är vår produktionsserver. Vi installerar nodejs och npm med följande kommandon.

```shell
$sudo apt update
$sudo apt install curl
$cd ~
$curl -sL https://deb.nodesource.com/setup_10.x -o nodesource_setup.sh
$sudo bash nodesource_setup.sh
$sudo apt install nodejs
$nodejs -v
```

Här ska du gärna se en utskrift med ett versionsnummer som inledas med `v10.`.

Om du kör kommandot `npm -v` ser du att du även har Node Package Manager npm installerat med ett versionsnummer över 6. För att vissa program ska kunna installeras via npm behöver vi även installera build-essentials. Vi gör det med kommandot `sudo apt install build-essential`.



#### Installera tmux


tmux är ett oerhört trevligt verktyg att använda om man vill komma tillbaka till samma vy när man loggar in på en server från ett antal olika servrar. Installera med kommandot `sudo apt-get install tmux`.

Du öppnar en tmux session genom att skriva `tmux` i terminalen. I sitt grundutförande är Ctrl-b kommandotangenten, du trycker alltså in Ctrl-b släpper och en knapp till för att utföra kommandot. Du kan skapa nya fönster med Ctrl-b följd av c, du kopplar ner från sessionen med Ctrl-b d och vill du tillbaka till sessionen kan du skriva `tmux a -t 0`. Bra och smidigt när man vill logga in från flera olika datorer, men ändå se samma bild.



#### Installera git


För att lättare kunna driftsätta våra git-repon installerar vi även git med kommandot `sudo apt-get install git`.



### Ett Express API

**Från detta steget utvecklar du lokalt på din dator.**

Modulen [Express finns på npm](https://www.npmjs.com/package/express). Express är en del av [MEAN](http://mean.io/) som är en samling moduler för att bygga webbapplikationer med Node.js. I denna artikeln kommer vi att använda Express (E) och Node.js (N) i MEAN.

Innan vi börjar så skapar vi en `package.json` som kan spara information om de moduler vi nu skall använda.

```shell
# Ställ dig i katalogen du vill jobba
$npm init
```

När du ombeds döpa paketet så ange "me-api" eller något liknande (det spelar ingen roll). Använd bara inte "express" eftersom det paketnamnet redan finns och du får problem i nästa steg. Du kan köra om `npm init` om du vill ändra namn, eller redigera namnet direkt i filen `package.json`.

Nu kan vi installera paketen vi skall använda `express`, `cors` och `morgan`. Vi väljer att spara dem i vår `package.json`.

```shell
$npm install express cors morgan --save
```

Vi använder oss av `cors` för att hantera Cross-Origin Sharing problematik och `morgan` för loggning av händelser i API:t.

Då vi inte vill ha `node_modules` katalogen versionshanterad i git skapar vi filen `.gitignore` och lägger "node_modules/" som första rad i den filen.



#### Verifiera att Express fungerar


Låt oss starta upp en server för att se att installationen gick bra.

Jag börjar med kod som startar upp servern tillsammans med en route för `/` och sparar i en fil du själv skapar `app.js`.

```javascript
const express = require("express");
const app = express();

const port = 1337;

// Add a route
app.get("/", (req, res) => {
    res.send("Hello World");
});

// Start up server
app.listen(port, () => console.log(`Example API listening on port ${port}!`));
```

Sedan startar jag servern.

```shell
$node app.js
Example API listening on port 1337!
```

Nu kan jag skicka requester till servern via curl.

```shell
$curl localhost:1337
Hello World
```

Om jag använder en route som inte finns så får jag en 404 tillsammans med ett svar som säger att routen inte finns.

```shell
$curl -i localhost:1337/asd
HTTP/1.1 404 Not Found
X-Powered-By: Express
Content-Security-Policy: default-src 'self'
X-Content-Type-Options: nosniff
Content-Type: text/html; charset=utf-8
Content-Length: 134
Date: Wed, 15 Mar 2017 08:47:43 GMT
Connection: keep-alive

<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="utf-8">
<title>Error</title>
</head>
<body>
<pre>Cannot GET /asd</pre>
</body>
```

Pröva nu samma routes via din webbläsare. Du bör få motsvarande svar även i din webbläsare.

Det verkar som allt gick bra och Express är uppe och snurrar och svarar på tilltal.



#### Låt npm köra dina skript

I filen `package.json` kan du lägga in skript och köra dem via `npm`. Du kan till exempel lägga till skriptet för att starta servern så här.

```json
{
    "scripts": {
        "start": "node app.js"
    }
}
```

Nu kan du starta servern via `npm start`. Det blir ett sätt att samla enklare skript in i din `package.json`.



#### Svara med JSON

I de allra flesta fall vill vi att vårt API svarar med ett JSON svar. För det använder vi `response` objektets inbyggda funktion `json` istället för `send` som vi såg ovan.

```javascript
const express = require("express");
const app = express();

const port = 1337;

// Add a route
app.get("/", (req, res) => {
    const data = {
        data: {
            msg: "Hello World"
        }
    };

    res.json(data);
});

// Start up server
app.listen(port, () => console.log(`Example API listening on port ${port}!`));
```

I exemplet ovan skickar vi ett JSON objekt när vi skickar en förfrågan till `/`. Vi startar om servern och vi får följande svar om vi testar med curl i terminalen.

```shell
$curl localhost:1337
{"data":{"msg":"Hello World"}}
```



#### Automatisk omstart av node-appen

Vi det har laget har du nog redan börjat tröttna på att starta om din server varje gång du har ändrat, så låt oss göra nått åt detta. Vi använder oss av npm modulen `nodemon` ([Dokumentation](https://www.npmjs.com/package/nodemon)) för att starta om vår node applikation varje gång vi sparar. Vi installerar `nodemon` som ett globalt paket, så vi kan använda det för alla vår node applikationer.

```shell
$npm install -g nodemon
```

För att starta vår applikation i nodemon kontext ändrar vi vårt `npm start` skript.

```json
{
    "scripts": {
        "start": "nodemon app.js"
    }
}
```



#### Routing mot olika request metoder

En route sätts upp för att svara mot en speciell request metod såsom GET, POST, PUT, DELETE. Det är på det sättet man bygger upp en RESTful tjänst.

Här är fyra routes som har samma url, men skiftar i requestens metod.

```javascript
// Testing routes with method
app.get("/user", (req, res) => {
    res.json({
        data: {
            msg: "Got a GET request"
        }
    });
});

app.post("/user", (req, res) => {
    res.json({
        data: {
            msg: "Got a POST request"
        }
    });
});

app.put("/user", (req, res) => {
    res.json({
        data: {
            msg: "Got a PUT request"
        }
    });
});

app.delete("/user", (req, res) => {
    res.json({
        data: {
            msg: "Got a DELETE request"
        }
    });
});
```

Om du testar med din webbläsare så blir det en GET request.

För att testa de andra metoderna så använder jag verktygen Postman eller RESTClient som är ett plugin in till Firefox. Med de verktygen kan jag välja om jag skall skicka en GET, POST, PUT, DELETE eller någon annan av de HTTP-metoder som finns. En sådan REST-klient är ett värdefullt utvecklingsverktyg.

Så här ser det ut när jag skickar en request med en annan metod än GET.

![En DELETE request skickas tll servern som svarar från rätt route.](https://dbwebb.se/image/snapvt17/express-rest-client.png?w=w2)

Det var routes och stöd för olika metoder det. Se till att du installerar en klient motsvarande RESTClient och testa din egen server.

Man vill ofta skicka en annan statuskod än 200 när man gör andra typer av requests än GET. Det kan vi göra med `response` objektets inbyggda funktion `status`.

```javascript
// Testing routes with method
app.get("/user", (req, res) => {
    res.json({
        data: {
            msg: "Got a GET request, sending back default 200"
        }
    });
});

app.post("/user", (req, res) => {
    res.status(201).json({
        data: {
            msg: "Got a POST request, sending back 201 Created");
        }
    });
});

app.put("/user", (req, res) => {
    // PUT requests should return 204 No Content
    res.status(204).send();
});

app.delete("/user", (req, res) => {
    // DELETE requests should return 204 No Content
    res.status(204).send();
});
```

Vi skickar alltså tillbaka statusen 201 när vi skapar objekt med POST anrop och 204 när vi uppdaterar eller tar bort. Det är enkelt gjort med `status` funktion. För se innebörden av alla HTTP status koder finns [följande lista](https://en.wikipedia.org/wiki/List_of_HTTP_status_codes).



#### Route med dynamiskt innehåll

Vi skapar nya routes för att se hur routern hanterar dynamiskt innehåll i form av parametrar.

```javascript
const express = require("express");
const app = express();

const port = 1337;

// Add a route
app.get("/", (req, res) => {
    const data = {
        data: {
            msg: "Hello World"
        }
    };

    res.json(data);
});

app.get("/hello/:msg", (req, res) => {
    const data = {
        data: {
            msg: req.params.msg
        }
    };

    res.json(data);
});

// Start up server
app.listen(port, () => console.log(`Example API listening on port ${port}!`));
```

Vi kan nu använda följande routes och se vad som händer.

```shell
/
/hello/Hello-World
/hello/Hello World
/hello/Jag kan svenska ÅÄÖ
```

Vi ser att parametern hanteras och kan nås i routen via `req.params`. Vi ser också att mellanslag och svenska tecken hanteras och översätts med `encodeURIComponent()`.

I webbsidan ser det ut som det ska.

![I webbläsaren ser det bra ut, ungefär som man tänkte.](https://dbwebb.se/image/ramverk2/dynamic-routing.png?w=w3)

I terminalen där servern kör ser det ut så här.

```shell
GET
/hello/Jag%20kan%20svenska%20%C3%85%C3%84%C3%96
```

Det vi ser är exempel på hur webbläsaren och servern hanterar encodning av udda tecken.

Webbläsaren konverterar länken, urlencodar, så att mellanslagen byts ut mot `%20`. När länken tas emot som en route och översätts till parametrar, så gör Express en urldecode på innehållet. Detta är sättet som används för att hantera udda tecken i en webblänk och det sker automatiskt av webbläsaren och Express.

Det fungerar så här, om man översätter det till ren JavaScript.

```shell
$node
> a = encodeURIComponent("Jag kan svenska åäö")
'Jag%20kan%20svenska%20%C3%A5%C3%A4%C3%B6'
> b = decodeURIComponent(a)
'Jag kan svenska åäö'
>
```

Det är bra att veta om att det finns en hantering av udda tecken som sker i bakgrunden.

Om man vill använda sig av parametrar tillsammans med HTTP metoderna POST, PUT och DELETE används `body-parser`. Vi importerar modulen längst upp i `app.js`. Och lägger sen till att vi vill göra en parse på bodyn genom följande rader kod.

```javascript
const bodyParser = require("body-parser");

...

app.use(bodyParser.json()); // for parsing application/json
app.use(bodyParser.urlencoded({ extended: true })); // for parsing application/x-www-form-urlencoded
```



#### Middleware - CORS och loggning

I express finns termen "middleware" som benämning på callbacks som anropas innan själva routens hanterare anropas. En middleware kan också vara en hanterare som alltid anropas för alla routes.

Låt oss skapa en sådan middleware, som alltid anropas, oavsett route. Den skall skriva ut vilken route som accessades och med vilken metod.

Vi lägger till middleware via metoden `app.use()`. Vi kan lägga till dem för en specifik route, eller för alla routes.

```javascript
// This is middleware called for all routes.
// Middleware takes three parameters.
app.use((req, res, next) => {
    console.log(req.method);
    console.log(req.path);
    next();
});
```

Middleware anropas i den ordningen de är definierade, när de matchar en route. Använd ett anrop till `next()` när du är klar och vill skicka vidare kontrollen till nästa middleware och slutligen till routens hanterare.

Om du vill att denna middleware alltid skall anropas så behöver du lägga den högst upp i din kod.

På serversidan ser du nu delar av innehållet i request-objektet som visar metoden och pathen som anropats, samt eventuellt inkommande parametrar.



#### Loggning med tredjepartsmodul

Vi väljer i vårt API att använda en tredje parts modul `morgan` för loggning. Vi har redan installerad `morgan` som en del av `node_modules` och vi lägger till modulen i `app.js` enligt nedan och använder den inbyggda middleware för att skriva ut loggen. Vi lägger in anropet till `morgan` innan vi anropar några routes då vi vill att loggningen sker för alla routes.

```javascript
const express = require('express');
const morgan = require('morgan');

const app = express();
const port = 1337;

// don't show the log when it is test
if (process.env.NODE_ENV !== 'test') {
    // use morgan to log at command line
    app.use(morgan('combined')); // 'combined' outputs the Apache style LOGs
}
```



#### Cross-Origin Resource Sharing (CORS)

Då vi vill att vårt API ska kunna konsumeras av många olika klienter vill vi tillåta att klienter från andra domäner kan hämta information från vårt API. Vi gör även detta med en tredjepartsmodul `cors`, som vi installerade i början av artikeln. På samma sätt som för `morgan` använder vi den inbyggda middleware och använder funktionen `use`.

```javascript
const express = require('express');
const cors = require('cors');
const morgan = require('morgan');

const app = express();
const port = 1337;

app.use(cors());

// don't show the log when it is test
if (process.env.NODE_ENV !== 'test') {
    // use morgan to log at command line
    app.use(morgan('combined')); // 'combined' outputs the Apache style LOGs
}
```



#### 404 med routes

När användaren försöker nå en route som inte finns så blir det ett svar med statuskod 404.

![Ett standard felmeddelande när routen saknas.](image/snapvt17/express-default-404.png?w=w2)

Man kan lägga till en egen route som blir en "catch all" och agerar kontrollerad hantering av 404.

```javascript
// Add routes for 404 and error handling
// Catch 404 and forward to error handler
// Put this last
app.use((req, res, next) => {
    var err = new Error("Not Found");
    err.status = 404;
    next(err);
});
```

Ovan så använder min hanterare för 404 den inbyggda felhanteraren. Det sker via anropet `next(err)` där `err` är ett objekt av typen `Error`. Min variant är alltså att säga att nu är det felkod 404 och jag överlämnar till den inbyggda felhanteraren att skriva ut felmeddelandet.

![Den inbyggda felhanteraren ger ett fel och en stacktrace.](https://dbwebb.se/image/snapvt17/express-default-error-handler.png?w=w2)

Det finns alltså en inbyggd felhanterare som visar upp information om felet, tillsammans med en stacktrace. Det är användbart när man utvecklar.

När node startar upp Express så är det default i utvecklingsläge. Du kan testa att starta upp i produktionsläge, det ger mindre information i felmeddelandena.

```shell
$NODE_ENV="production" node app.js
```

Nu försvann stacktracen från klienten, men den syns fortfarande i terminalen där servern körs.

![I produktion så visas inte stacktrace för klienten.](image/snapvt17/express-error-handling-production.png?w=w2)

Vi ser till att även skapa ett npm skript för att köra i produktion som vi sedan kan använda på servern. Vi kan då köra `npm run production` för att starta i i produktion.

```json
{
    "scripts": {
        "start": "node app.js",
        "production": "NODE_ENV='production' node app.js"
    }
}
```

När vi utvecklar så blir det enklast att köra development läge (standard). Men när man sätter en server i produktion så får man se till att det också är produktionsläge för felmeddelandena, vilket innebär att visa så lite information som möjligt.



#### En egen hanterare för felutskrift

Vi kan skapa vår egen felhanterare och skicka felmeddelandet som JSON.

En egen felhanterare i Express kan se ut som det `app.use` funktionsanrop längst ner. Vi kombinerar det med vår hanterare för 404 felmeddelande och använder `next(err);` för att skicka vidare felmeddelandet till vår egen hanterare.

```javascript
app.use((req, res, next) => {
    var err = new Error("Not Found");
    err.status = 404;
    next(err);
});

app.use((err, req, res, next) => {
    if (res.headersSent) {
        return next(err);
    }

    res.status(err.status || 500).json({
        "errors": [
            {
                "status": err.status,
                "title":  err.message,
                "detail": err.message
            }
        ]
    });
});
```

Kom ihåg att en sådan här felhanterare är som all annan middleware och det är viktigt i vilken ordning de ligger. De kan anropas i den ordningen som de definieras.



#### Uppdelning av routes

Med tanke på de få routes vi kommer ha tillgängliga i våra API:er hade det inte varit helt orimligt att ha al hantering i `app.js`, men vi väljer ändå att dela upp våra routes då vi gillar bra struktur inför framtida uppskalningar.

Vi skapar katalogen `routes` och i den katalogen skapar vi två stycken filer `index.js` och `hello.js`. Här skapar vi och returnerar ett objekt av typen `express.Router()`.

```javascript
var express = require('express');
var router = express.Router();

router.get('/', function(req, res, next) {
    const data = {
        data: {
            msg: "Hello World"
        }
    };

    res.json(data);
});

module.exports = router;
```

Vi importerar sedan dessa filer i `app.js` och använder de som routehanterare med ett funktionsanrop till `use`.

```javascript
...

const index = require('./routes/index');
const hello = require('./routes/hello');

...

app.use('/', index);
app.use('/hello', hello);
```

På det sättet håller vi `app.js` liten i storlek och var sak har sin plats.



### En databas

Vi vill koppla vårt API mot en databas för att vi ska kunna hämta och spara data där istället för att bara ha statisk data. I denna del av kursen väljer vi att använda den filbaserade relationsdatabasen SQLite. Senare i kursen kommer vi bekanta oss med [Dokument-orienterade databaser](nosql).

Om du inte har SQLite installerat på din utvecklingsdator installera det via XAMPP eller pakethanteraren i ditt operativsystem.

För att kunna spara användare och så småningom redovisningstexter installerar vi npm modulen node-sqlite3 i vårt me-api repo med följande kommando. [Dokumentationen för modulen](https://www.npmjs.com/package/sqlite3) är som alltid vår bästa vän.

```shell
$npm install sqlite3 --save
```

Vi skapar sedan katalogen `db` i vårt repo och i den katalogen filen `texts.sqlite`. Vi ville inte att denna och andra sqlite filer är under versionshantering då de isåfall skriver över vår produktions databas när vi driftsätter så vi lägger till `*.sqlite` i `.gitignore`.

Ett smart drag i detta skedet är att skapa en migrations-fil `db/migrate.sql` som du kan använda för att skapa tabeller. Min migrate-fil innehåller än så länge följande SQL.

```shell
CREATE TABLE IF NOT EXISTS users (
    email VARCHAR(255) NOT NULL,
    password VARCHAR(60) NOT NULL,
    UNIQUE(email)
);
```

Vi har alltså två kolumner `email` och `password` och vi vill att `email` är unik. Vi kan nu med hjälp av följande kommandon skapa tabellen i vår `texts.sqlite` databas.

```shell
$cd db
$sqlite3 texts.sqlite
sqlite> .read migrate.sql
sqlite> .exit
```

Vi kan nu använda `sqlite3` modulen för att lägga till en användare i vår `texts.sqlite` databas på följande sätt.

```javascript
const sqlite3 = require('sqlite3').verbose();
const db = new sqlite3.Database('./db/texts.sqlite');

db.run("INSERT INTO users (email, password) VALUES (?, ?)",
    "user@example.com",
    "superlonghashedpasswordthatwewillseehowtohashinthenextsection", (err) => {
    if (err) {
        // returnera error
    }

    // returnera korrekt svar
});
```



#### sqlite3 på servern

För att detta ska fungera på din droplet måste vi installera `sqlite3` innan vi kör `npm install`. Vi gör detta med `sudo apt-get install sqlite3` som vår `deploy` användare. Vi kan nu hämta senaste versionen av vårt API med `git pull` och köra `npm install` för att installera det nya paketet. Vi behöver även skapa databas filen `db/texts.sqlite` och köra migrations filen.



#### Säker hantering av lösenord

När vi sparar lösenord i en databas vill göra det så säkert som möjligt. Därför använder vi [bcrypt](https://codahale.com/how-to-safely-store-a-password/).

Ibland kan kombinationen av Windows och npm modulen bcrypt ställa till med stora problem. Ett tips hämtat från [installationsmanualen för bcrypt](https://github.com/kelektiv/node.bcrypt.js/wiki/Installation-Instructions#microsoft-windows) är att installare npm paketet `windows-build-tools` med kommandot nedan. Installera det i kommandotolken (cmd) eller Powershell så Windows har tillgång till det.

```shell
$npm install --global --production windows-build-tools
```

Vi installerar bcrypt paketet med npm med hjälp av kommandot `npm install bcryptjs --save`. [Dokumentationen för modulen](https://www.npmjs.com/package/bcryptjs) är som alltid vår bästa vän.



För att hasha ett lösenord med bcrypt modulen importerar vi först modulen och sedan använder vi `bcrypt.hash` funktionen. Antal `saltRounds` definierar hur svåra lösenord vi vill skapa. Ju fler `saltRounds` är svårare att knäcka, men tar också längre tid att skapa och jämföra.

```javascript
const bcrypt = require('bcryptjs');
const saltRounds = 10;
const myPlaintextPassword = 'longandhardP4$$w0rD';

bcrypt.hash(myPlaintextPassword, saltRounds, function(err, hash) {
    // spara lösenord i databasen.
});
```

Det finns även en promise version av biblioteket om man gillar promise eller async/await teknikerna. Läs mer om det i dokumentationen.

För att jämföra ett sparad lösenord med det användaren skrivit in använder vi `bcrypt.compare`.

```javascript
const bcrypt = require('bcryptjs');
const myPlaintextPassword = 'longandhardP4$$w0rD';
const hash = 'superlonghashedpasswordfetchedfromthedatabase';

bcrypt.compare(myPlaintextPassword, hash, function(err, res) {
    // res innehåller nu true eller false beroende på om det är rätt lösenord.
});
```

<div class="under-construction">
    <p>Tidigare användes npm-modulen <code>bcrypt</code>, men verkar finnas installationsproblem med modulen i produktion. Därför används nu bcryptjs istället.</p>
</div>



#### JSON Web Tokens

Vi har i tidigare kurser använt både sessioner och tokens för att autentisera klienter mot en server. Vi ska i detta stycke titta på hur vi implementerar logiken bakom att skicka JSON Web Tokens från servern till en klient. Vi använder modulen jsonwebtoken som vi installerar med kommandot `npm install jsonwebtoken --save` och [dokumentationen finns på npm](https://www.npmjs.com/package/jsonwebtoken).


Vi använder här de två funktioner `sign` och `verify`.

```javascript
const jwt = require('jsonwebtoken');

const payload = { email: "user@example.com" };
const secret = process.env.JWT_SECRET;

const token = jwt.sign(payload, secret, { expiresIn: '1h'});
```

I ovanstående exempel skapar vi `payload` som i detta fallet enbart innehåller klientens e-post. Vi hämtar sedan ut vår `JWT_SECRET` från environment variablerna. En environment variabel sätts i terminalen, både lokalt på din dator och på servern med kommandot `export JWT_SECRET='longsecret'`, där du byter 'longsecret' mot nått långt och slumpmässigt. Se till att denna secret är lång och slumpmässig, gärna 64 karaktärer. `payload` och `secret` blir sedan tillsammans med ett konfigurationsobjekt argument till funktionen `jwt.sign` och returvärdet är vår `token`.

När vi sen vill verifiera en token använder vi funktionen `jwt.verify`. Här skickar vi med token och vår secret som argument. Om token kan verifieras får vi dekrypterat payload och annars ett felmeddelande.

```javascript
jwt.verify(token, process.env.JWT_SECRET, function(err, decoded) {
    if (err) {
        // not a valid token
    }

    // valid token
});
```



#### JWT middleware

Vi såg i guiden [Node.js API med Express](kunskap/nodejs-api-med-express) hur vi kan skapa routes som tar emot POST anrop och hur vi kan använda middleware för att köra en funktion varje gång vi har ett anrop till specifika routes. Om vi skapar nedanstående route i vår me-api ser vi hur middleware funktionen `checkToken` ligger som första funktion på routen. Den anropas först och beroende på om `next()` anropas funktionen efter middleware. Vi observerar även hur vi från klientens sida har skickat med token som en del av headers och hur vi hämtar ut det från request-objektet `req`.

```javascript
router.post("/reports",
    (req, res, next) => checkToken(req, res, next),
    (req, res) => reports.addReport(res, req.body));

function checkToken(req, res, next) {
    const token = req.headers['x-access-token'];

    jwt.verify(token, process.env.JWT_SECRET, function(err, decoded) {
        if (err) {
            // send error response
        }

        // Valid token send on the request
        next();
    });
}
```

Vi ser i kodexemplet ovan att vi använder `req.body` när vi tar emot en POST request från en klient och skickar med det in till modulen/modellen vi använder för att skapa rapporten. För att kunna använda `req.body` har vi dessa två rader längst upp i vår `app.js`. Vi har även sett detta i artikeln [Node.js API med Express](kunskap/nodejs-api-med-express#dynamiskt).

```javascript
app.use(bodyParser.json()); // for parsing application/json
app.use(bodyParser.urlencoded({ extended: true })); // for parsing application/x-www-form-urlencoded
```

I Postman väljer vi att fylla i body fliken istället för params fliken.

Vi såg i artikeln [Login med JWT](https://dbwebb.se/kunskap/login-med-jwt) kursen webapp hur man kan skicka lösenord med [postman](https://www.getpostman.com/). postman är ett utmärkt verktyg för att manuellt testa ett API. I postman kan man även sätta headers under headers fliken för varje request.

![Postman](https://dbwebb.se/image/ramverk2/postman-headers.png?w=c18)



#### Exempelkod

Om ni vill titta på ett fullständigt exempelprogram som använder alla dessa tekniker är [Lager API:t](https://github.com/emilfolino/order_api) från webapp kursen ett bra exempel.



### Driftsättning

Vi börjar med att klona vårt repo till servern. Använd https länken när du klonar för enklast hantering. Jag har skapat en katalog `~/git` där jag klonar mitt repo till. När du har klonat repot kan du göra `npm install` så alla moduler är installerat.

För att våra klienter ska komma åt API:t ser vi till att driftsätta det på vår server. Vi ska använda oss av det som kallas en nginx reverse proxy för att trafiken utifrån på port 80 eller 443 (vanliga portarna för HTTP och HTTPS) ska skickas till vårt API som ligger och lyssnar på en annan port.

När vi installerade nginx fick vi med oss ett antal olika kataloger och konfigurationsfiler. I katalogen `/var/www` kommer vi skapa kataloger för de webbplatser vi vill skapa på vår server. Vi börjar med att logga in på servern som `deploy` och skapar en katalog för vårt API.

Jag kommer i följande exempel utgå ifrån min konfiguration på servern [jsramverk.me](https://jsramverk.me) där mitt API ligger på subdomänen [me-api.jsramverk.me](https://me-api.jsramverk.me).

Jag skapar alltså katalogen `/var/www/me-api.jsramverk.me/html` enklast med kommandot `sudo mkdir -p /var/www/me-api.jsramverk.me/html`. Denna katalog kommer inte användas för filer, men vi kommer använda den i ett senare skede när vi vill spara ett certifikat för HTTPS trafik till vårt API.

Jag har satt i gång API:t med kommandot `npm run production` och API:t ligger och lyssnar på port 8333. Den reverse proxy som vi skapar i följande stycke lyssnar i första skedet på port 80 och skickar vidare förfrågningarna till 8333.

I katalogen `/etc/nginx/sites-available` skapar vi en konfigurationsfil `me-api.jsramverk.me` genom att kopiera standard konfiguration från filen `default` och öppna upp filen i text editorn nano. Vi gör det med följande kommandon.

```shell
$cd /etc/nginx/sites-available
$sudo cp default me-api.jsramverk.me
$sudo nano me-api.jsramverk.me
```

I filen klistrar vi in följande konfiguration. Först skapar vi en server med namnet me-api.jsramverk.me. Vi skapar därefter två stycken `location`. Det är routes där vi vill att nått speciellt ska hända. Den första är för en fil relaterad till det certifikat vi ska installera om ett ögonblick för att fixa HTTPS till vår server. Den andra `location /` är alla andra routes som ska skickas till `http://localhost:8333` där vårt API ligger och lyssnar. Detta kallas en reverse proxy och användas i många sammanhang för att kopplat förfrågningar på port 80 till en annan port. En reverse proxy används då man inte vill öppna portarna utåt, men vill låta nginx ta hand om detta.

```shell
server {
    server_name me-api.jsramverk.me;

    location /.well-known {
        alias /var/www/me-api.jsramverk.me/html/.well-known;
    }

    location / {
        proxy_pass http://localhost:8333;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }

    listen 80;
}
```

Vi sparar filen genom att trycka `Ctrl-X` och skriva in ett y + Enter. Vi skapar sedan en symbolisk länk i katalogen `/etc/nginx/sites-enabled` till vår konfigurations fil för att sidan blir tillgänglig.

```shell
$cd /etc/nginx/sites-enabled
$sudo ln -s /etc/nginx/sites-available/me-api.jsramverk.me
```

Vi vill sedan testa om konfigurationen är korrekt och sedan starta om nginx och det gör vi med följande kommandon.

```shell
$sudo nginx -t
$sudo service nginx restart
```

För att internet ska veta att vi har en server som ligger här och vill svara på förfrågningar skapar vi en subdomän i Digital Ocean gränssnittet. Gå till Networking och välj din domän skriv sedan in din subdomän välj din droplet och skapa subdomänen.

![Digital Ocean subdomän](https://dbwebb.se/image/ramverk2/do-subdomain.png?w=w3)

Det ska nu gå att se ett JSON svar från API:t om vi går till vår subdomän. Ibland kan det ta en liten stund innan subdomäner kommer på plats, så avvakta lite grann om det inte syns direkt.



#### Process manager

Vi vill i mångt och mycket automatisera hur vi startar, uppdaterar och startar om våra nodejs applikationer. För detta ändamålet använder vi en process manager. Det finns ett antal olika [process managers för express applikationer](https://expressjs.com/en/advanced/pm.html), men jag har valt att använda [PM2](http://pm2.keymetrics.io/).

Vi installerar PM2 med kommandot:

```shell
$npm install -g pm2
```

Vi går sedan till katalogen där vi startade vårt me-api och stänger av den node-process vi startade med `npm start` eller `node app.js`. Vi startar istället processen som en pm2 kontext så vi får automatisk omstart och kan göra uppdateringar utan neretid. Vi startar appen i pm2 kontext med följande kommando.

```shell
$pm2 start app.js --name me-api
```

Flaggan --name me-api används för att ge processen ett namn. Kan vara bra inför framtiden när vi vill ha flera olika processer igång samtidigt.



#### HTTPS

Då vi är medvetna om våra användares privatliv vill vi att alla anslutningar till våra tjänster och services sker över HTTPS, som krypterar den data som skickas. Vi behöver därför installera ett certifikat. Vi väljer att använda ett certifikat från [Let's Encrypt](https://letsencrypt.org/) och vi installerar det med tjänsten [Certbot](https://certbot.eff.org/) då vi har tillgång till serverns CLI.

Vi behöver först öppna upp så vi kan installera paket från det som heter APT backports. Vi öppnar upp filen `/etc/apt/sources.list` och letar reda på följande två rader som vi avkommenterar. Raderne brukar finnas längst ner i filen.

```shell
$deb http://mirrors.digitalocean.com/debian stretch-backports main contrib non-free
$deb-src http://mirrors.digitalocean.com/debian stretch-backports main contrib non-free
```

Uppdatera apt-get med `sudo apt-get update`. Vi kan nu installera verktyget certbot med kommandot `sudo apt-get install python-certbot-nginx -t stretch-backports`.

Vi startar verktyget genom att köra kommandot `sudo certbot --nginx`. Vi får då välja för vilka domäner och subdomäner vi vill installera certifikat. Efter att vi har vald domänerna får vi frågan om vi vill omdirigera all trafik till HTTPS istället för HTTP och det svarar vi ja till (i certbot gränssnittet motsvarar det en tvåa).

Vi ska nu se en hänglås i adressfältet om vi uppdaterar i webbläsaren.



## Kravspecifikation

Denna veckan är uppgiften uppdelat i två delar. En del handlar om backend och en del om hur din frontend applikation ska konsumera backend API:t.



#### Del 1: Backend

1. Skapa ett Me-API med nedanstående router.

1. Se till att det finns en `package.json` i katalogen. Filen skall innehålla alla beroenden som krävs.

1. Skapa routen `GET /` där du ger en presentation av dig själv.

1. Skapa routerna `GET /reports/week/1`, `GET /reports/week/2` och `GET /reports/week/3`, som innehåller data för att fylla motsvarande sidor i din Me-applikation.

1. Skapa routerna `POST /register` och `POST /login` för att registrera en användare och logga in. Data du sparar om användare ska vara samma om du hade i registreringsformuläret förra veckan.

1. Skapa routen `POST /reports` för att lägga till data. För att kunna använda denna route ska klienten vara autentiserad med hjälp av JWT.

1. Committa alla filer och lägg till en tagg (1.0.0) med hjälp av `npm version 1.0.0`. Det skapas automatiskt en motsvarande tagg i ditt GitHub repo. Lägg till fler taggar efterhand som det behövs. Var noga med din committ-historik.

1. Pusha upp repot till GitHub, inklusive taggarna.

1. Publicera ditt API publikt och lägg den publika adressen i din inlämning på Canvas.



#### Del 2: Frontend

1. Din frontend Me-applikation ska hämta innehåll från Me-API:t.

1. Koppla registreringsformuläret från förra vecka till Me-API:t.

1. Skapa ett inloggningsformulär för att kunna autentisera användare mot API:t.

1. När en användare är inloggat ska det gå att skapa nya texter för kommande veckor och redigera befintliga texter.

1. Committa alla filer och lägg till en tagg (3.0.0) med hjälp av `npm version 3.0.0`. Det skapas automatiskt en motsvarande tagg i ditt GitHub repo. Lägg till fler taggar efterhand som det behövs. Var noga med din commit-historik.

1. Pusha upp repot till GitHub, inklusive taggarna.

1. Publicera din Me-applikation publikt och lägg den publika adressen i din inlämning på Canvas. I artikeln [Driftsätta din Me-applikation](deploy-frontend) finns information om hur du driftsätter en Me-applikation i ditt valda frontend ramverk.



## Skriva

Vi fortsätter iterativt med att förbättra vårt akademiska skrivande. Använd den återkopplingen du fick på första och andra veckans text och förbättra din inledning.

Gå tillbaka till skrivguiden och titta under [Inledning](http://skrivguiden.se/skriva/uppsatsens_delar/#inledning) för bra tips.

**Lämna in texten som PDF bilaga till din inlämning på Canvas.**
