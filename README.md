# Baseline

Desktopová aplikácia pre Windows 11 x64 v C# / WinUI 3. Skutočná inventarizácia WMI, kontrola runtime balíkov, oficiálne sťahovanie, overovanie podpisov a inštalácia kompatibilných ovládačov z Microsoft Update Catalog. Neobsahuje simulované výsledky.

**Verzia 1.0:** distribúcia teraz používa jediný **offline setup.exe približne 66 MB**, pod limitom 100 MB. Hosting ani účet nie sú potrebné. Obsahuje aplikáciu a jej knižnice; runtime balíky a ovládače sa sťahujú až pri výbere v aplikácii. Zostavenie, spustenie, témy a 43 kontrol vrátane skutočného rozbalenia vloženého obsahu boli overené. Systémové inštalácie, WUA pomocník a celý cyklus registrácie/odinštalovania ešte vyžadujú VM testy. Podrobnosti: [overenie](docs/OVERENIE.md).

## Spustenie teraz

Spustite `artifacts/setup.exe`. Aplikácia sa nainštaluje pod `%LOCALAPPDATA%/Programs/Baseline` aktuálneho používateľa bez internetu. Kontrola počítača sa spustí automaticky pri otvorení aplikácie. Pri neprítomnom internete funguje lokálna detekcia; najnovšie .NET metadáta nie sú overené.

Alternatívne rozbaľte celý `artifacts/Baseline-win-x64.zip` a spustite `Baseline.exe`; pri ručnom rozbalení nevzniká záznam odinštalovania. Vždy zachovajte všetky súbory priečinka. Python ani SDK nepotrebujete.

## Zostavenie

Na Windows 11 x64 nainštalujte .NET SDK 8.0.425 alebo novší SDK podporujúci .NET 8. NuGet obnoví WinUI aj build nástroje; Visual Studio nie je povinné. Použite PowerShell 7. Windows obsahuje .NET Framework kompilátor pre malý zavádzač a pomocník.

```powershell
./scripts/Build.ps1
dotnet run --project tests/Baseline.Checks.csproj -c Release
```

V tomto pracovnom prostredí je SDK už dostupné:

```powershell
./scripts/Build.ps1 -DotNet "$env:LOCALAPPDATA/BaselineBuild/dotnet/dotnet.exe"
```

Skript vytvorí `artifacts/app`, `artifacts/Baseline-win-x64.zip` a offline `artifacts/setup.exe` s vloženým ZIP a pripnutým SHA-256. Automaticky odmietne výsledok väčší než 100 000 000 bajtov. Odinštalátor je samostatný malý EXE; po inštalácii nevzniká druhá kópia veľkého setup.exe. Pri zostavení sa Visual C++ stiahne dočasne iba na vytvorenie overených metadát katalógu a zmaže sa. Do distribúcie sa nepribalí jeho inštalátor. Pred zostavením zatvorte aplikáciu z výstupného priečinka.

Voliteľný online variant je stále zostaviteľný. Len ak chcete vlastnú HTTPS distribúciu, po publikovaní presného ZIP použite:

```powershell
./scripts/Package-Setup.ps1 -ReleaseUrl 'SEM_VLOZTE_SKUTOCNU_HTTPS_ADRESU_ZIP'
```

Tento voliteľný krok **neprebaľuje ZIP** a nahradí výstupný setup.exe malým online zavádzačom. Pre aktuálny offline setup ho nepoužívajte: žiadny hosting nie je potrebný.

## Rozsah

| Oblasť | Implementácia |
|---|---|
| Hardvér | Windows zostava/architektúra, CPU, GPU, doska, SMBIOS revízia, BIOS, model PC, PNP ID a hardvérové ID |
| Visual C++ | v14 x86/x64; obidva pohľady registra, cieľ a hash z oficiálneho balíka pri zostavení |
| .NET Desktop | rady 8 a 10, x86/x64, aktuálne oficiálne release metadáta a SHA-512 |
| .NET Framework | 4.8.1 z oficiálneho inštalátora a 3.5 cez DISM / Windows Update |
| DirectX | x86/x64, indikátory June 2010, oficiálny redist, rozbalenie a podpísaný DXSETUP |
| Ovládače | WUA vyhľadávanie, inštalácia jednotlivého/vybraných alebo všetkých ponúkaných kompatibilných balíkov; licenčné podmienky, skutočný WUA priebeh; firmware vylúčený |
| BIOS | verzia/model/revízia, vysvetlenie neistoty, potvrdenie samostatného postupu a overené všeobecné stránky výrobcov; automatické vyhľadanie najnovšej verzie pre konkrétne dosky zatiaľ nie je implementované |
| Rozhranie | Slovenské WinUI, Mica/fallback, tri témy, stabilná poloha ikon menu, automatická kontrola, farebné a zoradené súčasti, prehľad s počítadlami, zariadenia podľa kategórie, vypnutý výber textu, decentný copyright |
| Inštalátor | offline EXE pod 100 MB, vložený ZIP + SHA-256, skutočný bajtový priebeh prípravy/rozbaľovania, staging, HKCU odinštalovanie, ponuka Štart, žiadna služba/autostart/tray |

Staršie Visual C++ rady, každá historická DirectX DLL, automatický výber konkrétnych vendor GPU/chipset inštalátorov mimo Windows Update a ARM64 distribúcia nie sú súčasťou aktuálnej zostavy. .NET Framework 4.x sa spravuje ako jedna rodina; 4.8.1 nahrádza staršie 4.x. Natívne DirectX indikátory negarantujú kompletnosť Managed DirectX/XACT alebo opravu ľubovoľnej chyby DLL. Visual C++ snímka vyjadruje verziu overenú pri zostavení, nie živé overenie najnovšej verzie pri každom spustení.

## Úložisko a oprávnenia

`%LOCALAPPDATA%/Programs/Baseline` obsahuje nainštalovanú aplikáciu; Nastavenia, logy, cache a malý odinštalátor sú v `%APPDATA%/Baseline`. Staging a záloha programu sú pri inštalácii v `%LOCALAPPDATA%/Programs`, aby premenovanie zostalo na rovnakom disku. GUI a zavádzač bežia bez zvýšenia práv. Oficiálne systémové inštalátory, DISM a pomocník WUA žiadajú UAC až pri príslušnej operácii. WUA pomocník neprijíma ľubovoľnú cestu ani príkaz: prijíma UUID aktualizácie, revíziu a náhodný identifikátor pomenovaného kanála. GUI ukladá výsledky v pôvodnom profile aj pri použití iného administrátorského účtu.

Po úspechu sa dočasné stiahnuté balíky odstránia. Po neúspešnom spustení môže zostať už overený balík v `downloads` na diagnostiku; nezaznamenáva sa úspech. Pri opakovaní sa stiahne nový súbor. WUA spravuje vlastnú systémovú cache. Odinštalovanie zachová nastavenia/logy a neodstraňuje zdieľané runtimes ani ovládače.

Distribučné EXE zatiaľ nie sú podpísané vydavateľským certifikátom. Integrita vloženého archívu sa overuje proti SHA-256; pred verejnou distribúciou podpíšte finálny `setup.exe`, aplikáciu aj pomocník a vykonajte VM protokol. Závislosti .NET a WinUI sú pribalené v offline obsahu. Overenie spustenia na čistom Windows bez už nainštalovaného Visual C++ zostáva osobitným VM testom. Aktuálny základ používa .NET 8; pred dlhodobým vydaním treba plánovať prechod na novší podporovaný LTS.

Podrobnejšia [architektúra a rozšírenie katalógu](docs/ARCHITEKTURA.md).

## Ovládače a komponenty výrobcu vo verzii 1.0

Microsoft Update Catalog sa kontroluje automaticky pri štarte a po úspešnej inštalácii ovládačov. Prehľad počíta problémy ovládačov aj novšie ponuky pre zariadenia; zhodné zariadenia nepočíta dvakrát. Nedokončená kontrola sa nezobrazuje ako úspešná nula.

Inventár číta všetky zistené grafiky vrátane integrovanej, PCI vendor ID, vydavateľa a verziu ovládača. AMD/Intel systémové zariadenia (chipset) a NVIDIA/AMD/Intel grafiky majú vlastné karty. Inštalácia kompatibilných ovládačov prebieha z Microsoft Update Catalog cez podpísané CAB/INF a PnPUtil; ak balík nie je ponúkaný, karta obsahuje overený postup výrobcu. Samostatné výrobné balíky mimo WUA sa nevyberajú naslepo podľa CPU. Vypnutá grafika, ktorú firmware nesprístupní Windowsu, nemusí byť zistiteľná.

NVIDIA PhysX 9.23.1019 je samostatná voliteľná inštalácia pri NVIDIA grafike; sťahuje sa až na výber, má pripnutý SHA-512, overenie podpisu NVIDIA Corporation a kontrolu verzie v oboch pohľadoch odinštalovacích registrov. Otvorí sa oficiálny inštalátor. Reálna inštalácia PhysX na Windows 11 zostáva VM testom.

CUDA Runtime a Intel oneAPI/Intel Runtime sa detegujú podľa evidovaných komponentov. Aplikačné privátne kópie nemusia byť evidované. Automatická inštalácia ľubovoľnej CUDA/oneAPI verzie nie je implementovaná: používateľ dostane oficiálny postup a vyberie knižnicu/verziu vyžadovanú aplikáciou. AMD grafické runtime komponenty sa riešia kompatibilným Radeon ovládačom; univerzálny samostatný „AMD runtime“ balík sa nevymýšľa. Voliteľné komponenty nie sú súčasťou Nainštalovať všetky ovládače a nezvyšujú počet chýbajúcich súčastí.

Overené primárne zdroje: [AMD Radeon a Ryzen chipset](https://www.amd.com/en/resources/support-articles/faqs/GPU-Driver-Autodetect.html), [NVIDIA PhysX](https://www.nvidia.com/en-us/drivers/physx/physx-9-23-1019-driver/), [CUDA Windows](https://docs.nvidia.com/cuda/cuda-installation-guide-microsoft-windows/index.html), [Intel runtime verzie](https://www.intel.com/content/www/us/en/developer/tools/oneapi/runtime-versions-download.html).

## Nový inštalátor

Setup aj odinštalátor teraz používajú WPF z .NET Framework 4.8 vstavaného vo Windows 11. Nepotrebujú už nainštalovaný WinUI runtime. Rozhranie zodpovedá aplikácii: Segoe UI, zaoblené karty/tlačidlá, systémová svetlá/tmavá téma a zvýrazňovacia farba, vysoký kontrast, natívne zaoblenie okna, Mica podľa podpory a nastavenia priehľadnosti alebo nepriesvitné pozadie. Jemné animácie rešpektujú systémové nastavenia.

Obrazovky: príprava, skutočný priebeh kopírovania/sťahovania a rozbaľovania, samostatné overovanie SHA-256, chyba s opakovaním a dokončenie. Po úspechu sú k dispozícii Spustiť Baseline a Ukončiť. Aplikácia sa spustí iba na výslovné kliknutie a inštalátor sa potom zavrie. Odinštalovanie má zladený vzhľad. Súbežné spustenie setup/odinštalátora pre rovnakého používateľa blokuje pomenovaný mutex.

Vývojový náhľad vzhľadu bez inštalovania: `./scripts/Preview-Setup.ps1 -Page complete` (ďalej welcome, error, progress, removed a svetlé varianty). Náhľad je osobitný EXE so symbolom PREVIEW; tieto testovacie obrazovky nie sú súčasťou distribučného setup.exe. Úplný cyklus inštalácie/registrácie/odinštalovania a spustenie po dokončení ešte vyžadujú VM overenie.

Nové umiestnenie: %LOCALAPPDATA%/Programs/Baseline pre aplikáciu a %APPDATA%/Baseline pre používateľské dáta. Pri rozpoznanej starej inštalácii v PCReady nový setup po úspechu vyradí starý odinštalátor a program; pôvodné nastavenia a logy zachová a nastavenia prevezme do Baseline. Tento migračný cyklus ešte vyžaduje VM test.

Rozhranie Baseline 1.0: slovenský copyright, vlastná vektorovo kreslená ikona, úvodná voľba zástupcu na plochu a animovaný inštalátor bez zobrazenia cieľovej cesty. Súčasti ponúkajú Nainštalovať vybrané aj Nainštalovať všetky; druhé tlačidlo vyberá len podporované chýbajúce alebo zastarané balíky. Ikonu možno znovu vytvoriť cez scripts/Create-Icon.ps1.

.NET Framework 3.5: na Windows 11 26H1 (build 28000) a novšom Baseline sťahuje samostatný DotNet35Setup.exe z oficiálneho Microsoft odkazu, overuje pevný SHA-512 a Authenticode a spúšťa /quiet /norestart. Na starších Windows používa NetFx3 cez DISM. Overovanie podpisov používa absolútnu cestu vstavaného Windows PowerShellu a jeho systémové moduly, aby nezdedilo nekompatibilné moduly PowerShellu 7.

## Úpravy rozhrania z 6. októbra 2026

Aplikácia, setup aj odinštalátor ponúkajú slovenčinu a angličtinu. Výber je spoločný pre aktuálneho používateľa a ukladá sa do `HKCU\Software\Baseline\Language`. Jemné pohyblivé svetlo na pozadí rešpektuje priehľadnosť, animácie a vysoký kontrast Windowsu. Príloha k požadovanému efektu nebola dostupná; použitý je jemný modrofialový efekt.

Prehľad obsahuje celkové zhodnotenie, chýbajúce súčasti a ovládače, hardvér a zariadenia. Súhrn rodín súčastí bol odstránený; výsledok neúplnej kontroly sa nevydáva za úspech. BIOS je z celkového zhodnotenia výslovne vyňatý. Neoveriteľná dostupnosť aktualizácie BIOS-u má červené upozornenie. Automatické vyhľadávanie ani flashovanie BIOS-u nie sú implementované.

Ovládače majú oddelené dostupné aktualizácie, hardvér výrobcu a voliteľné komponenty. V súčastiach je prvá hlavná akcia Nainštalovať všetky. Dokončená kontrola nezanecháva stavový text v ľavom dolnom rohu. Nastavenia používajú jednotné karty s nadpisom, opisom a ovládaním; odinštalovanie spúšťa priamo `%APPDATA%\Baseline\uninstall.exe` a zavrie aplikáciu.

Optimalizácie obsahujú 12 kategórií s ikonami Segoe Fluent v mriežke kariet s priamymi prepínačmi, krátkymi opismi a štítkami Odporúčané, Pokročilé, Vyžaduje reštart a Vyžaduje správcu. Žiadna položka neotvára Nastavenia Windowsu. Zmeny používajú pevný katalóg registrov, služieb, powercfg, DISM a DNS PowerShell API. Prekrývajúce sa požiadavky sú zlúčené; ovládanie rovnakej voľby v dvoch kategóriách zdieľa zálohu.

Pred prvou zmenou sa trvalo uloží pôvodná hodnota vrátane typu a neprítomnosti. Vypnutie prepínača a Obnoviť predvolené obnovujú tento pôvodný stav, nie továrenské nastavenia Windowsu. Používateľské zálohy sú v `%APPDATA%\Baseline\optimization-user-v2.json`, systémové v `%ProgramData%\BaselineOptimizations\machine-v2.json` s chráneným zápisom. Staré zálohy dvoch pôvodných volieb sa migrujú. Pri chybe sa vykoná návrat; ak aj návrat zlyhá, záloha zostáva dostupná. Otvorenie stránky len číta nastavenia.

Systémové zmeny vykonáva pribalený OptimizationWorker.exe po UAC. Prijíma iba identifikátor z pevného katalógu a operáciu, nie ľubovoľný príkaz. Odporúčaný profil súkromia zahŕňa aj minimalizáciu diagnostiky na povinné údaje (s UAC na podporovaných edíciách), históriu vyhľadávania, návrhy účtu a sledovanie spúšťania aplikácií. Odporúčaný profil má 36 volieb vrátane zastavenia služby DiagTrack, obmedzenia voliteľnej diagnostiky, reklamných návrhov, histórie a Game Mode. Zastavenie telemetrie môže obmedziť diagnostiku a negarantuje odstránenie všetkej komunikácie. Pokročilé zásahy, GPU, napájanie a sieť sa hromadne nezapínajú. Podpora sa overuje podľa buildu, edície, miestnej politiky a dostupnosti konkrétneho nastavenia. Stav prepínača označuje zmenu spravovanú Baseline; spätné čítanie konfigurácie nepotvrdzuje každé správanie Windowsu.

Odstránené a neoveriteľné funkcie majú nedostupný prepínač s dôvodom. Recall sa nevypína, pretože jeho politiky odstraňujú uložené snímky a nespĺňajú úplnú reverzibilitu. VRR sa nevynucuje bez spoľahlivého overenia hardvéru. Voľba Game Bar obmedzuje aktiváciu ovládačom a úvodný panel; nevypína Win+G ani neodinštaluje aplikáciu. DoH používa iba overenú šablónu aktuálneho resolvera a nemení adresu DNS servera. Hibernácia obnovuje konfiguráciu, nie už zmazaný obsah predchádzajúceho hibernačného súboru.

Automatické testy overujú katalóg, bezpečný odporúčaný výber, trvalé zálohy, návrat pri chybe, ochranu pred cudzími cestami a presnú obnovu typov registra v izolovanej testovacej vetve HKCU. Rozhranie bolo skontrolované v spustenej aplikácii. Skutočné privilegované zmeny služieb, SMB1, napájania a DoH a ich účinnosť naprieč edíciami Windowsu ešte vyžadujú samostatné VM testy.

Postupy nastavení sú podľa [dokumentácie Microsoftu](https://learn.microsoft.com/en-us/windows/privacy/manage-connections-from-windows-operating-system-components-to-microsoft-services); inšpirácia rozhraním podľa [O&O](https://manuals.oo-software.com/ooshutup10/en/docs/features/overview/). Presné fungovanie volieb súkromia naprieč všetkými edíciami Windowsu a celý inštalačný/odinštalačný cyklus na čistom systéme zostávajú osobitnými VM testami.

Karta kategórie uvádza počet nastavení, názov a počet odporúčaných. Otvorí samostatný pohľad v tom istom okne so šípkou späť; bočné menu používa hamburger.

Úvodné karty používajú veľké ikony vpravo so zvislým vycentrovaním a vnútorným odstupom. Karty kategórií Optimalizácií majú jednotnú výšku, jeden rám a mriežku s 1–3 stĺpcami podľa šírky okna. Rozšírený odporúčaný profil obsahuje aj vzhľadové a používateľské preferencie; potvrdenie pred aplikáciou vypíše všetky zmeny.

## Úpravy vzhľadu a výkonu

Nové modrofialové logo používa monogram B, zhodný PNG/ICO v aplikácii, setup a odinštalátore. Úvodné karty majú zvislo vycentrované ikony vpravo. Scrollbar je pri pravom kraji; obsah má 28 px spodný odstup, priebeh operácií a stav majú samostatný vnútorný odstup. Svetlá téma má tlmené sivofialové pozadie a kontrastnejší copyright. Menu opäť používa natívnu animáciu NavigationView.

Prehľad ponúka opravu chýbajúcich podporovaných súčastí s existujúcim potvrdením inštalácie a tlačidlo na riešenie chýbajúcich ovládačov. Optimalizácie ukazujú počet aktívnych odporúčaných nastavení podľa skutočnej konfigurácie; nepodporované položky sa nezapočítavajú a chyby overenia sa nepovažujú za úspech. Aktívne nastavenie mohlo byť zapnuté už pred Baseline.

Štyri nové výkonové voľby obmedzujú priehľadný výber súborov, tiene názvov ikon, miniatúry médií a preview handlers. Obnova zachová pôvodný typ a prítomnosť hodnoty. Náhľady a tiene sú pokročilé, pretože menia funkčnosť alebo čitateľnosť; tieto položky sa automaticky neaplikujú. Zvýšenie herného FPS sa negarantuje. Microsoft opisuje [vizuálne efekty a výkon](https://support.microsoft.com/en-gb/windows/experience/performance-optimization/tips-to-improve-pc-performance-in-windows) a mapovanie [IconsOnly/ShowPreviewHandlers](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-gppref/a6ca3a17-1971-4b22-bf3b-e1a5d5c50fca).


### Úpravy výkonu a distribúcie (2026-10-08)

- Kontrola optimalizácií načíta každý z dvoch zálohovacích denníkov iba raz na refresh,
  namiesto opakovaného čítania pre každú položku. Mutácie naďalej načítavajú aktuálnu
  zálohu pod mutexom; pôvodný stav sa nikdy neprepisuje cache údajmi.
- Hromadné odporúčania preskočia nastavenia, ktoré už v systéme zodpovedajú voľbe,
  aj keď neboli aplikované Baseline. Platí aj pre elevated worker.
- V kategóriách je okamžité vyhľadávanie v názvoch a popisoch bez opätovného čítania
  Windowsu a bez zmeny prepínačov. Prázdny výsledok má vysvetlenie.
- Hlavná stránka optimalizácií má tlačidlo Obnoviť stav. Počty majú správne slovenské tvary.
- Release ZIP používa SmallestSize, kde je dostupné, inak Optimal. PDB ostávajú
  v artifacts/symbols mimo inštalátora. Runtime závislosti sa neodstraňujú a trimming
  sa nepoužíva. setup.exe klesol z 67 301 376 na 64 891 392 bajtov pri tejto zostave.
- Zostava a 74 kontrol prešli. Na PC vizuálne overené načítanie prehľadu,
  hlavná stránka optimalizácií a filtrovanie kategórie Výkon a napájanie.
  Nejde o potvrdenie funkčnosti všetkých systémových zásahov na všetkých verziách OS.

### Priame ovládače z Microsoft Update Catalog (2026-10-08)

- V sekcii Ovládače je jediný zdroj Microsoft Update Catalog.
  Katalóg používa priame HTTPS vyhľadávanie hardvérových ID, štyri súbežné kontroly
  a časový limit 90 sekúnd. Vyhľadávanie a inštalácia cez službu Windows Update boli odstránené.
- Ponuky vyžadujú presnú zhodu silných ID (pri PCI vrátane SUBSYS), x64,
  explicitnú podporu Windows 11 a jeho vydania, novšiu verziu a nie starší dátum.
  Pre chýbajúci ovládač je možné ponúknuť kompatibilnú prvú inštaláciu.
  Nejde o úplné pokrytie všetkých zariadení ani náhradu OEM pravidiel Windows Update.
- Jedna dávka katalógových ovládačov používa jeden elevated worker a jedno UAC.
  Worker prijíma iba GUID balíkov, opätovne overí metadáta aj lokálny hardvér,
  stiahne iba CAB z presného Microsoft servera, overí SHA-256 a Microsoft podpis
  CAT, verziu a ID v INF. PnPUtil preverí členstvo súborov v podpísanom katalógu
  a výber necháva na Windows; ovládače sa nevynucujú a PC sa nereštartuje.
- Pred zmenou exportuje aktuálne OEM ovládače do chráneného úložiska
  `%ProgramData%\BaselineDrivers\<session>\<update>\backup`. CAB a rozbalené kópie
  sa po spracovaní odstránia. Potvrdenie úspechu vyžaduje následnú detekciu
  verzie zariadenia alebo oznámený požadovaný reštart.
- BIOS, firmware, extension INF a SoftwareComponent INF sa neinštalujú touto cestou.
  Niektoré balíky tak môžu byť odmietnuté; Baseline zobrazí chybu alebo vynechá neoverenú ponuku.
- Katalóg nemá podporované verejné API. Zmena webového rozhrania alebo neurčitá
  kompatibilita musí viesť k chybe alebo vynechaniu, nikdy k odhadu balíka.
- Read-only kontrola na vývojovom PC: 61 ID, jedna ponuka, žiadna chyba, 10,4 sekundy.
  Automatizované kontroly pokrývajú výber, odmietnutie nesprávnej platformy,
  firmvéru, downgradu, zdroja bez HTTPS, cudzieho servera a chýbajúceho SHA-256.
  Skutočná systémová inštalácia nebola na vývojovom PC vykonaná; pred distribúciou
  treba overiť UAC, podpisy, export a aktualizáciu na testovacom PC/VM.


### Prehľad a jediný zdroj ovládačov

Prehľad započítava chýbajúce aj zastarané súčasti, ponuky novších ovládačov a PNP problémy. Rovnaké zariadenie s chybou a ponukou sa počíta iba raz. Horný stavový blok s písmom 22 a ikonou napravo kontroluje aj dostupné odporúčané optimalizácie. Fajka vyžaduje úspešné kontroly; chyba alebo nedokončená kontrola ju nezobrazí. Pokročilé/nepodporované optimalizácie sa nevyžadujú. Zdroje ovládačov aj elevated inštalácia používajú výhradne Microsoft Update Catalog. Na PC vizuálne overená ponuka Realtek a počet 1 v prehľade; 132 automatických kontrol prešlo.

### Tichá inštalácia

`setup.exe /silent` nainštaluje aplikáciu bez zobrazenia okna, otázok, nového odkazu na ploche a spustenia Baseline. Vzhľad sa nastaví podľa systému a jazyk podľa podporovaného jazyka systému (inak angličtina). `setup.exe --uninstall /silent` podporuje aj tiché odinštalovanie. Návratové kódy: 0 úspech, 87 neplatné argumenty, 1603 chyba operácie, 1618 súbežný inštalátor. Chyby operácie sa zapisujú do `%APPDATA%/Baseline/logs/setup.log`. Bežiaca aplikácia sa nezatvára násilne; v takom prípade inštalácia skončí chybou. Pre Microsoft Store zadajte parameter `/silent`. Overené boli odmietnutie súbežnej operácie a neplatných argumentov; celý úspešný tichý cyklus treba overiť na čistom testovacom Windows.
