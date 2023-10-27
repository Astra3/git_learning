# VŠB Informatika Git lekce

Přehled a cheatsheet z git lekce.

## Příprava

Na nastavení gitu je potřeba nějaká příprava :pensive:

### Instalace nástrojů

Instalace Gitu probíhá z [webové stránky](https://git-scm.com/downloads) anebo package manageru vaši linux distribuce (na Ubuntu třeba `sudo apt install git`).

Při instalaci na Windows vyberte, že chcete i nainstalovat OpenSSL, na linuxu konzultujte váš package manager.

### Nastavení VS Code

Vyžadované extensions:

- [Git Graph](https://marketplace.visualstudio.com/items?itemName=mhutchie.git-graph)
- [GitLens](https://marketplace.visualstudio.com/items?itemName=eamodio.gitlens)

### SSH klíče

Git poskytuje dva způsoby jak nahrát commity na remote server: SSH a HTTPS. Mezi těmito dvěma způsoby existuje dlouhodobá válka, nicméně v dnešní době je SSH způsob preferovaný.

SSH klíč se dá vytvořit pomocí `ssh-keygen` příkazu. Na váš private key lze nastavit (měnitelné) heslo, public key nemá cenu šifrovat. Ve výchozím nastavení se vytvoří ve vaší user složce složka `.ssh` a v ní budou dva soubory - `id_rsa` s vašim private key (strážit jako oko v hlavě) a `id_rsa.pub` s vaším pulic key, který nahrajete na remote server (na GitHubu je to v user settings).

## Práce s gitem

Kapitoly v této sekci se věnují terminologii v gitu.

### Commit

```
git commit -m <message>
```

Git funguje jako taková řada commitů, kde nový commit vždy navazuje na předchozí. Jedná se o takový záznam vaší práce a progressu. Vždy, když se cítíte, že jste něco "udělali," tak je dobré udělat commit. Taky je dobré udělat commit, pokud je potřeba provést nějaký rychlý fix.

Mezi příklady kdy commitnout patří třeba dokončení nové featury, rychlé fixnutí jednoduchého bugu anebo jenom čistá potřeba sdílet progress na něčem s kolegy.

Commit by měl obsahovat nějakou stručnou zprávu, která shrnuje změny v něm provedené. Při delší zprávě je lepší napsat souhrn na první řádek a pak v novém odstavci více informací. Každý commit poslaný přes SSH/GPG je podepsaný SSH/GPG klíčem. Každý commit má také hash (většinou SHA256), který se používá jako jeho identifikátor.

#### `.gitignore` file

Jedná se o soubor, ve kterém jsou vypsané soubory a složky, které se nebudou nikdy stageovat a commitovat. Je v něm možné použít wildcards na výběr více souborů.

```git
# ignoruje všechny soubory s koncovkou .pyc
*.pyc

# ignoruje soubor nebo složku s názvem venv
venv

# ignoruje složku s názvem venv
venv/

# ignoruje všechny soubory začínající na "file" ve složce "ahoj"
ahoj/file*
```

`.gitignore` soubory je možné dávat v repozitáři i do podsložek, kdy tyhle změny nebublají do složek nahoru, např. pokud by ve složce `slozka` bylo nastavené pravidlo na ignorování `ggt`, tohle pravidlo by platilo jenom v podložkách složky `slozka`.

### Remote
```
git remote add <name> <URL>
git remote remove <name>
```

Pomocí "remote" v repozitáří říkáme, že jde o místo na síti, kde chceme ukládat, sdílet naše commity s ostatními a přijímat commity od jiných. Remote má vždy nějaké jméno (ve výchozím stavu `origin`) a nějakou URL remote git serveru.

#### Fetch

```
git fetch
```

Fetch operace stáhne současný stav commitů na remote serveru.

#### Pull

```
git pull (--rebase)
```

Pull operace provede fetch a poté nastaví současný commit na ten nejnovější. Při použití `--rebase` flag zahájí rebase při snaze o nastavení na nejnovější commit.

#### Push

```
git push (--force)
```

Nahraje lokální commity do remote. Když se v remote nachází novější commit než ten, který má být nahrán, operace selže. Při použití `--force` flagu umožní mazat commity na serveru (dá se vypnout na serveru a většinou je).

### Stash

Stash je plně lokální...stash necommitovaných změn. Víceméně stash vezme změny v souborech, které nebyly commitovýny a zabalí je do jednoho obědového balíčku, který můžeme kdykoliv rozbalit (a sníst). V případě konfliktů mezi obsahem ve stashi a současnám commit tree dochází k [merge conflict](#merge-conflict).

### Branches

Branches jsou doslova větve do commit tree. Vytváří kopie repozitáře v určitém bodu a fungují samostatně od hlavního branche (**main**/master). Proces přepnutí do branche se nazývá checkout.

Může se jedna o jakýkoliv název bez mezer, nicméně je běžné psát branche ve stylu `kategorie/název`, kde kategorií může být třeba `bugfix`, `feature` a název pak cokoliv, k čemu je branch určený - třeba `feature/vulkan_rewrite`.

V softwarových firmách se branche používají na retroaktivní bugfixy. V teoretickém scénáři může existovat případ, kdy v main branchi se provádí vývoj na verzi 2.0, ale ozve se zákazník s long-term support verzí 1.0 a řekne, že vyžaduje bugfix. V tom případě se vytvoří branch z [tagu](#tags) ve kterém se daný bugfix provede a tento branch se pak pošle zákazníkovi.

Nepsaná konvence je, že v **main**/master branchi je funkční kód, mezitím co nefunkční kód existuje v branches.

### Tags

Tags slouží k označení nějakých milníku v commit tree. Většinou se tedy jedná o nějaký release atd.

Existují dva typy tags:

- **Lightweight tag**

  Lightweight tag slouží slouží jako takový pointer na určitý commit; nemůže obsahovat žádné dodatečné informace.

- **Annotated tag**

  Annotated tag vytváři read-only kopii (branch) z daného commitu. Jak návez napovídá, lze je označit nějakou zprávou, datem, jménem a případně i public key.

Git tagy je potřeba samostatně pushovat do repozitáře pomocí příkazu `git push <remote> <tag_name>`.

### Merge

Merge spojí (zamerguje) jeden branch do druhého.

#### Merge conflict

Konflikt při mergování nastává, když v obou branchích byly provedeny nezávislé změny na jednom souboru.

Příklad: Vytvoříme si branch z main branch, který nazveme `dvojka`. Checkoutneme se do branche `dvojka` a provedeme v něm změny v souboru `main.py`, které zacommitujeme. Poté se přesuneme zpátky do main branche a provedeme jiné změny v souboru `main.py`. Při pokusu o zamergování branche `dvojka` do main narazíme na merge conflict, jelikož změny v souborech se neshodují.

V případě merge conflict máme čtyři možnosti.

1. Accept theirs - Přijmeme verzi z merge, do kterého mergujeme
2. Accept yours - Přijmeme verzi z merge, ze kterého mergujeme
3. Accept combination - Přijememe kombinaci obou verzí
4. Custom - Konflikt vyřešíme vlastními úpravami výsledku.

Výsledkem merge je vždy commit, kterému se běžně říká merge commit.

### Rebase

Rebase je přebalení commitů nad jinými commity. Dalo by se říct, že se jedná o takový přesun. Běžně se používá při [pull](#pull) operaci z remote. V případě neshod při přebalování commitů mezi změnami v souborech vzniká [conflict](#merge-conflict).

Výsledkem operace je, že přebalované commity jsou v commit tree vždy nahoře.


## Pull request

Pull request je featura mnoha git serverů. Je to jenom prachobyčejná žádost o merge. Maintainers mohou psát do PR komentáře jak k příspěvku, tak ke kódu. Mohou vyžádat změny, mohou PR schválit a mohou jej zamergovat. Lze i vyžadovat určité procento souhlasů od maintainers, než bude merge možný.

PR lze tvořit i z branches nacházejících se ve forks, což jsou kopie repozitářu u jiného uživatele. Běžný důvod ke tvorbě forks je, že hlavní repozitář nedovoluje přímý push.
