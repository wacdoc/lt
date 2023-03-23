# Sukurkite savo SMTP pašto siuntimo serverį

## preambulė

SMTP gali tiesiogiai pirkti paslaugas iš debesies tiekėjų, pavyzdžiui:

* [Amazon SES SMTP](https://docs.aws.amazon.com/ses/latest/dg/send-email-smtp.html)
* [Ali debesies el. pašto siuntimas](https://www.alibabacloud.com/help/directmail/latest/three-mail-sending-methods)

Taip pat galite sukurti savo pašto serverį – neribotas siuntimas, mažos bendros išlaidos.

Žemiau žingsnis po žingsnio parodome, kaip sukurti savo pašto serverį.

## Serverio pasirinkimas

Savarankiškai priglobtam SMTP serveriui reikalingas viešas IP su atvirais 25, 456 ir 587 prievadais.

Dažniausiai naudojami viešieji debesys užblokavo šiuos prievadus pagal numatytuosius nustatymus ir gali būti įmanoma juos atidaryti išduodant darbo užsakymą, tačiau tai labai varginanti.

Rekomenduoju pirkti iš pagrindinio kompiuterio, kuriame šie prievadai yra atidaryti ir kuris palaiko atvirkštinių domenų vardų nustatymą.

Čia aš rekomenduoju [Contabo](https://contabo.com) .

„Contabo“ yra prieglobos paslaugų teikėjas, įsikūręs Miunchene, Vokietijoje, įkurtas 2003 m. su labai konkurencingomis kainomis.

Pirkimo valiuta pasirinkus eurą, kaina bus pigesnė (serveris su 8GB atmintimi ir 4 CPU per metus kainuoja apie 529 juanius, o pradinis diegimo mokestis vienerius metus nemokamas).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/UoAQkwY.webp)

Pateikdami užsakymą pažymėkite, kad `prefer AMD` , o serveris su AMD CPU veiks geriau.

Toliau kaip pavyzdį paimsiu Contabo VPS, kad parodyčiau, kaip sukurti savo pašto serverį.

## Ubuntu sistemos konfigūracija

Operacinė sistema čia yra Ubuntu 22.04

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/smpIu1F.webp)

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/m7Mwjwr.webp)

Jei ssh serveris rodo `Welcome to TinyCore 13!` (kaip parodyta paveikslėlyje žemiau), tai reiškia, kad sistema dar neįdiegta. Atjunkite ssh ir palaukite kelias minutes, kad vėl prisijungtumėte.

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/-qKACz9.webp)

Kai pasirodys `Welcome to Ubuntu 22.04.1 LTS` , inicijavimas baigtas ir galite tęsti toliau nurodytus veiksmus.

### [Pasirenkama] Inicijuoti kūrimo aplinką

Šis veiksmas yra neprivalomas.

Patogumo dėlei ubuntu programinės įrangos diegimą ir sistemos konfigūraciją įdėjau į [github.com/wactax/ops.os/tree/main/ubuntu](https://github.com/wactax/ops.os/tree/main/ubuntu) .

Vykdykite šią komandą, kad įdiegtumėte vienu paspaudimu.

```
bash <(curl -s https://raw.githubusercontent.com/wactax/ops.os/main/ubuntu/boot.sh)
```

Kinijos vartotojai, naudokite šią komandą ir kalba, laiko juosta ir kt. bus automatiškai nustatyta.

```
CN=1 bash <(curl -s https://ghproxy.com/https://raw.githubusercontent.com/wactax/ops.os/main/ubuntu/boot.sh)
```

### „Contabo“ įgalina IPV6

Įgalinkite IPV6, kad SMTP taip pat galėtų siųsti el. laiškus su IPV6 adresais.

redaguoti `/etc/sysctl.conf`

Pakeiskite arba pridėkite šias eilutes

```
net.ipv6.conf.all.disable_ipv6 = 0
net.ipv6.conf.default.disable_ipv6 = 0
net.ipv6.conf.lo.disable_ipv6 = 0
```

Vykdykite [kontabo mokymo programą: IPv6 ryšio pridėjimas prie serverio](https://contabo.com/blog/adding-ipv6-connectivity-to-your-server/)

Redaguokite `/etc/netplan/01-netcfg.yaml` , pridėkite kelias eilutes, kaip parodyta paveikslėlyje žemiau (Contabo VPS numatytasis konfigūracijos failas jau turi šias eilutes, tiesiog panaikinkite jas komentarus).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/5MEi41I.webp)

Tada `netplan apply` , kad pakeista konfigūracija įsigaliotų.

Sėkmingai sukonfigūravus galite naudoti `curl 6.ipw.cn` , kad peržiūrėtumėte išorinio tinklo IPv6 adresą.

## Klonuoti konfigūracijos saugyklos operacijas

```
git clone https://github.com/wactax/ops.soft.git
```

## Sugeneruokite nemokamą domeno vardo SSL sertifikatą

Norint siųsti laiškus, reikalingas SSL sertifikatas šifravimui ir pasirašymui.

Sertifikatams generuoti naudojame [acme.sh.](https://github.com/acmesh-official/acme.sh)

acme.sh yra atvirojo kodo automatinis sertifikatų pasirašymo įrankis,

Įveskite konfigūracijos sandėlį ops.soft, paleiskite `./ssl.sh` ir **viršutiniame kataloge** bus sukurtas `conf` aplankas.

Raskite savo DNS teikėją iš [acme.sh dnsapi](https://github.com/acmesh-official/acme.sh/wiki/dnsapi) , redaguokite `conf/conf.sh` .

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/Qjq1C1i.webp)

Tada paleiskite `./ssl.sh 123.com` , kad sugeneruotumėte `123.com` ir `*.123.com` sertifikatus savo domeno vardui.

Pirmą kartą paleidus bus automatiškai įdiegtas [acme.sh](https://github.com/acmesh-official/acme.sh) ir pridėta suplanuota automatinio atnaujinimo užduotis. Galite pamatyti `crontab -l` , yra tokia eilutė.

```
52 0 * * * "/mnt/www/.acme.sh"/acme.sh --cron --home "/mnt/www/.acme.sh" > /dev/null
```

Sugeneruoto sertifikato kelias yra panašus į `/mnt/www/.acme.sh/123.com_ecc。`

Atnaujinus sertifikatą bus iškviestas `conf/reload/123.com.sh` scenarijus, redaguokite šį scenarijų, galėsite pridėti komandas, pvz., `nginx -s reload` , kad atnaujintumėte susijusių programų sertifikato talpyklą.

## Sukurkite SMTP serverį su chasquid

[chasquid](https://github.com/albertito/chasquid) yra atvirojo kodo SMTP serveris, parašytas Go kalba.

Kaip senųjų pašto serverių programų, tokių kaip Postfix ir Sendmail, pakaitalas, chasquid yra paprastesnis ir lengviau naudojamas, be to, jį lengviau plėtoti.

Paleiskite `./chasquid/init.sh 123.com` bus automatiškai įdiegtas vienu paspaudimu (pakeiskite 123.com savo siuntimo domeno pavadinimu).

## Konfigūruokite el. pašto parašo DKIM

DKIM naudojamas el. pašto parašams siųsti, kad laiškai nebūtų traktuojami kaip šlamštas.

Sėkmingai paleidus komandą, būsite paraginti nustatyti DKIM įrašą (kaip parodyta toliau).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/LJWGsmI.webp)

Tiesiog pridėkite TXT įrašą prie savo DNS (kaip parodyta toliau).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/0szKWqV.webp)

## Peržiūrėkite paslaugos būseną ir žurnalus

 `systemctl status chasquid` Peržiūrėkite paslaugos būseną.

Įprasto veikimo būsena yra tokia, kaip parodyta paveikslėlyje žemiau

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/CAwyY4E.webp)

 `grep chasquid /var/log/syslog` arba `journalctl -xeu chasquid` gali peržiūrėti klaidų žurnalą.

## Atvirkštinė domeno vardo konfigūracija

Atvirkštinis domeno vardas yra skirtas leisti IP adresą pakeisti į atitinkamą domeno vardą.

Nustačius atvirkštinį domeno pavadinimą, el. laiškai nebus identifikuojami kaip šlamštas.

Kai gaunamas laiškas, gaunantis serveris atliks atvirkštinio domeno vardo analizę siunčiančiojo serverio IP adresu, kad patvirtintų, ar siunčiantis serveris turi galiojantį atvirkštinį domeno pavadinimą.

Jei siunčiantis serveris neturi atvirkštinio domeno pavadinimo arba jei atvirkštinis domeno pavadinimas neatitinka siunčiančio serverio IP adreso, priimantis serveris gali atpažinti el. laišką kaip el. laišką arba jį atmesti.

Apsilankykite [https://my.contabo.com/rdns](https://my.contabo.com/rdns) ir sukonfigūruokite, kaip parodyta toliau

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/IIPdBk_.webp)

Nustatę atvirkštinį domeno pavadinimą, nepamirškite sukonfigūruoti domeno vardo ipv4 ir ipv6 nukreipimo į serverį.

## Redaguokite chasquid.conf pagrindinio kompiuterio pavadinimą

Pakeiskite `conf/chasquid/chasquid.conf` į atvirkštinio domeno vardo reikšmę.

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/6Fw4SQi.webp)

Tada paleiskite `systemctl restart chasquid` , kad paleistumėte paslaugą iš naujo.

## Atsarginė conf kopija į git saugyklą

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/Fier9uv.webp)

Pavyzdžiui, aš sukuriu atsarginę conf aplanko kopiją savo „github“ procese taip

Pirmiausia sukurkite privatų sandėlį

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/ZSCT1t1.webp)

Įveskite conf katalogą ir pateikite į sandėlį

```
git init
git add .
git commit -m "init"
git branch -M main
git remote add origin git@github.com:wac-tax-key/conf.git
git push -u origin main
```

## Pridėti siuntėją

paleisti

```
chasquid-util user-add i@wac.tax
```

Galiu pridėti siuntėją

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/khHjLof.webp)

### Patikrinkite, ar slaptažodis nustatytas teisingai

```
chasquid-util authenticate i@wac.tax --password=xxxxxxx
```

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/e92JHXq.webp)

Pridėjus vartotoją, `chasquid/domains/wac.tax/users` bus atnaujintas, nepamirškite pateikti į sandėlį.

## DNS pridėti SPF įrašą

SPF (Siuntėjo politikos sistema) yra el. pašto patvirtinimo technologija, naudojama siekiant užkirsti kelią sukčiavimui el.

Ji patikrina pašto siuntėjo tapatybę tikrindama, ar siuntėjo IP adresas sutampa su tariamo domeno vardo DNS įrašais, todėl sukčiai negali siųsti netikrų el. laiškų.

Pridėjus SPF įrašus, el. laiškai nebus identifikuojami kaip šlamštas.

Jei jūsų domeno vardų serveris nepalaiko SPF tipo, tiesiog pridėkite TXT tipo įrašą.

Pavyzdžiui, `wac.tax` SPF yra toks

`v=spf1 a mx include:_spf.wac.tax include:_spf.google.com ~all`

SPF už `_spf.wac.tax`

`v=spf1 a:smtp.wac.tax ~all`

Atminkite, kad aš čia `include:_spf.google.com` , nes vėliau sukonfigūruosiu `i@wac.tax` kaip siuntimo adresą Google pašto dėžutėje.

## DNS konfigūracija DMARC

DMARC yra santrumpa (domenu pagrįstas pranešimų autentifikavimas, ataskaitų teikimas ir atitiktis).

Jis naudojamas fiksuoti SPF atšokimus (galbūt dėl ​​konfigūracijos klaidų arba kas nors kitas apsimeta jumis, kad siųstų šlamštą).

Pridėti TXT įrašą `_dmarc` ,

Turinys yra toks

```
v=DMARC1; p=quarantine; fo=1; ruf=mailto:ruf@wac.tax; rua=mailto:rua@wac.tax
```

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/k44P7O3.webp)

Kiekvieno parametro reikšmė yra tokia

### p (Politika)

Nurodo, kaip tvarkyti el. laiškus, kurių nepavyksta patvirtinti SPF (Siuntėjo politikos sistema) arba DKIM (DomainKeys Identified Mail). Parametras p gali būti nustatytas į vieną iš trijų reikšmių:

* jokio: nesiimama jokių veiksmų, tik patikrinimo rezultatas grąžinamas siuntėjui per ataskaitų teikimo el. paštu mechanizmą.
* Karantinas: nepatvirtintą paštą įdėkite į šlamšto aplanką, tačiau laiškas tiesiogiai neatmes.
* atmesti: tiesiogiai atmeskite el. laiškus, kurių nepavyko patvirtinti.

### fo (gedimų parinktys)

Nurodo ataskaitų teikimo mechanizmo grąžinamos informacijos kiekį. Jis gali būti nustatytas į vieną iš šių reikšmių:

* 0: Pranešti apie visų pranešimų patvirtinimo rezultatus
* 1: Praneškite tik apie pranešimus, kurių patvirtinimas nepavyko
* d: Praneškite tik apie domeno vardo patvirtinimo klaidas
* s: praneškite tik apie SPF patvirtinimo klaidas
* l: praneškite tik apie DKIM patvirtinimo klaidas

### rua ir ruf

* rua (Ataskaitų URI suvestinėms ataskaitoms): el. pašto adresas, skirtas apibendrintoms ataskaitoms gauti
* ruf (kriminalistinių ataskaitų ataskaitų teikimo URI): el. pašto adresas, skirtas gauti išsamias ataskaitas

## Pridėkite MX įrašų, kad peradresuotumėte el. laiškus į „Google Mail“.

Kadangi neradau nemokamos įmonės pašto dėžutės, palaikančios universalius adresus (Catch-All, gali gauti bet kokius šiuo domeno vardu siunčiamus el. laiškus, be apribojimų priešdams), naudoju chasquid, kad visus laiškus persiųsčiau į savo Gmail pašto dėžutę.

**Jei turite savo mokamą verslo pašto dėžutę, nekeiskite MX ir praleiskite šį veiksmą.**

Redaguoti `conf/chasquid/domains/wac.tax/aliases` , nustatyti persiuntimo pašto dėžutę

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/OBDl2gw.webp)

`*` nurodo visus el. laiškus, `i` yra aukščiau sukurtas siunčiančio vartotojo el. pašto adreso priešdėlis. Norėdami persiųsti laišką, kiekvienas vartotojas turi pridėti eilutę.

Tada pridėkite MX įrašą (čia nukreipiu tiesiai į atvirkštinio domeno vardo adresą, kaip parodyta pirmoje žemiau esančio paveikslo eilutėje).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/7__KrU8.webp)

Baigę konfigūraciją, galite naudoti kitus el. pašto adresus el. laiškams siųsti į `i@wac.tax` ir `any123@wac.tax` , kad sužinotumėte, ar galite gauti el. laiškus sistemoje „Gmail“.

Jei ne, patikrinkite chasquid žurnalą ( `grep chasquid /var/log/syslog` ).

## Išsiųskite el. laišką adresu i@wac.tax naudodami „Google Mail“.

Kai „Google Mail“ gavo laišką, aš, žinoma, tikėjausi atsakyti `i@wac.tax` , o ne i.wac.tax@gmail.com.

Apsilankykite [https://mail.google.com/mail/u/1/#settings/accounts](https://mail.google.com/mail/u/1/#settings/accounts) ir spustelėkite „Pridėti kitą el. pašto adresą“.

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/PAvyE3C.webp)

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/_OgLsPT.webp)

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/XIUf6Dc.webp)

Tada įveskite patvirtinimo kodą, gautą el. paštu, į kurį buvo persiųstas.

Galiausiai jį galima nustatyti kaip numatytąjį siuntėjo adresą (kartu su galimybe atsakyti tuo pačiu adresu).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/a95dO60.webp)

Tokiu būdu užbaigėme SMTP pašto serverio sukūrimą ir tuo pat metu naudojame Google Mail el. laiškams siųsti ir gauti.

## Išsiųskite bandomąjį el. laišką, kad patikrintumėte, ar konfigūracija sėkminga

Įveskite `ops/chasquid`

Vykdykite `direnv allow` įdiegti priklausomybes (direnv buvo įdiegtas ankstesniame vieno klavišo inicijavimo procese ir prie apvalkalo buvo pridėtas kabliukas)

tada bėk

```
user=i@wac.tax pass=xxxx to=iuser.link@gmail.com ./sendmail.coffee
```

Parametrų reikšmė yra tokia

* vartotojas: SMTP vartotojo vardas
* leidimas: SMTP slaptažodis
* kam: gavėjas

Galite išsiųsti bandomąjį el.

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/ae1iWyM.webp)

Rekomenduojama naudoti „Gmail“ bandomiesiems el. laiškams gauti ir patikrinti, ar konfigūracijos buvo sėkmingos.

### TLS standartinis šifravimas

Kaip parodyta paveikslėlyje žemiau, yra šis mažas užraktas, o tai reiškia, kad SSL sertifikatas buvo sėkmingai įjungtas.

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/SrdbAwh.webp)

Tada spustelėkite „Rodyti originalų el.

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/qQQsdxg.webp)

### DKIM

Kaip parodyta paveikslėlyje žemiau, „Gmail“ pradinio pašto puslapyje rodomas DKIM, o tai reiškia, kad DKIM konfigūracija buvo sėkminga.

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/an6aXK6.webp)

Pradinio el. laiško antraštėje patikrinkite Gauta ir pamatysite, kad siuntėjo adresas yra IPV6, o tai reiškia, kad IPV6 taip pat sėkmingai sukonfigūruotas.
