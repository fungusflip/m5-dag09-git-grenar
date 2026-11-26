# Git-logg

Det här är min dokumentation av dagens arbete (dag 9, grenar, merge och konflikter). Alla utskrifter är riktiga, kopierade från terminalen.

## 1. Utgångsläget
Utskrift direkt efter att jag klonat min fork:

```
$ git log --oneline --graph --all
* 5a5841d Modul 5 i TypeScript med Vite
* 3804533 Add files via upload
* f7a13a4 Add files via upload
* c9a0075 Initial commit

$ git branch -a
* main
  remotes/origin/HEAD -> origin/main
  remotes/origin/main

$ git status
On branch main
Your branch is up to date with 'origin/main'.

nothing to commit, working tree clean
```

Det finns bara en gren (main) och historiken är en rak linje.

Steg 2 (grenen `meny-nyheter`): jag la till `- Kanelbulle, 33 kr` sist i `kafe-info/meny.md` och committade. `git branch` visade `* meny-nyheter`. På `main` fanns inte raden (grep gav 0 träffar), och när jag bytte tillbaka till grenen fanns den där igen. Det visar att grenen är ett eget spår som inte rör main.

## 2. Merge utan konflikt
Jag ändrade `kontakt.md` på main (telefonnummer) och mergade sedan `meny-nyheter`:

```
$ git merge --no-edit meny-nyheter
Merge made by the 'ort' strategy.
 kafe-info/meny.md | 1 +
 1 file changed, 1 insertion(+)

$ git log --oneline --graph --all
*   a2bcd40 Merge branch 'meny-nyheter'
|\
| * f5f8c91 Lägg till kanelbulle på menyn
* | 7b2dc72 Lägg till telefonnummer
|/
* 5a5841d Modul 5 i TypeScript med Vite
* 3804533 Add files via upload
* f7a13a4 Add files via upload
* c9a0075 Initial commit
```

Fråga: Vad är skillnaden mellan en *fast-forward*-merge och en merge som skapar en egen merge-commit?

Svar: Vid fast-forward har main inte fått några nya commits sedan grenen skapades, så Git flyttar bara main-pekaren framåt till grenens sista commit. Historiken blir en rak linje och ingen ny commit skapas. Här hade jag gjort en egen commit på main (telefonnumret) medan grenen också hade en ny commit. Då har historiken delat sig och Git måste göra en särskild merge-commit med två föräldrar som knyter ihop spåren. Det syns som ruten i grafen. Senare i steg 4 gick `git merge oppettider-helg` som fast-forward (`Fast-forward` i utskriften) eftersom main inte hade ändrats.

## 3. Konflikten
Jag skapade `oppettider-helg` (Helger: 10–17) och `oppettider-sommar` (Helger: 09–18) från samma main-commit. Första mergen gick som fast-forward:

```
$ git merge --no-edit oppettider-helg
Updating a2bcd40..6a3bd45
Fast-forward
 kafe-info/oppettider.md | 2 +-
 1 file changed, 1 insertion(+), 1 deletion(-)

$ git merge --no-edit oppettider-sommar
Auto-merging kafe-info/oppettider.md
CONFLICT (content): Merge conflict in kafe-info/oppettider.md
Automatic merge failed; fix conflicts and then commit the result.
```

Så här såg `kafe-info/oppettider.md` ut innan jag löste den:

```
# Öppettider

Vardagar: 07–18
<<<<<<< HEAD
Helger: 10–17
=======
Helger: 09–18
>>>>>>> oppettider-sommar
Röda dagar: stängt
```

Fråga: Vad betyder `<<<<<<< HEAD`, `=======` och `>>>>>>> grennamn`?

Svar: `<<<<<<< HEAD` startar konflikten och det som står under är min version, alltså den gren jag står på (main, som redan hade 10–17). `=======` är bara skiljelinjen mellan de två versionerna. Under den kommer den andra versionen, från grenen jag slår ihop, och `>>>>>>> oppettider-sommar` markerar slutet och visar vilken gren den kommer från. Markörerna är text som Git skriver in i filen, de ska inte vara kvar efteråt.

Hur löste du den, och varför blev det så?

Svar: Båda grenarna hade ändrat exakt samma rad, så Git kunde inte gissa vilken som skulle gälla. Jag öppnade filen, valde att varken ta 10–17 eller 09–18 rakt av utan skrev en mellanlösning, `Helger: 10–18` (öppnar senare på morgonen som i helgversionen men stänger senare som sommarversionen). Sedan tog jag bort alla tre markörrader, sparade, körde `git add kafe-info/oppettider.md` och `git commit` med standardmeddelandet. Kontroll:

```
$ git status
On branch main
nothing to commit, working tree clean

$ git grep "<<<<<<<" -- kafe-info
(inga träffar)
```

Notera: `git grep "<<<<<<<"` över hela repot ger träffar, men bara i textfiler som pratar om markörerna (README.md, `docs/git-fusklapp.md` och den här loggen). Där är det förklarande text, inte en riktig konflikt. I själva kafé-filerna finns inga markörer kvar.

Steg 6, avbryta en merge: jag skapade `roda-dagar-a` ("Röda dagar: stängt, utom julafton 10–14") och `roda-dagar-b` ("Röda dagar: helt stängt"). `roda-dagar-a` mergades först (fast-forward) och sedan kom konflikten med b, som jag avbröt:

```
$ git merge roda-dagar-b
Auto-merging kafe-info/oppettider.md
CONFLICT (content): Merge conflict in kafe-info/oppettider.md
Automatic merge failed; fix conflicts and then commit the result.

$ git status
On branch main
You have unmerged paths.
  (fix conflicts and run "git commit")
  (use "git merge --abort" to abort the merge)

Unmerged paths:
  (use "git add <file>..." to mark resolution)
	both modified:   kafe-info/oppettider.md

no changes added to commit (use "git add" and/or "git commit -a")

$ git merge --abort

$ git status
On branch main
nothing to commit, working tree clean
```

Efter `--abort` var allt som innan mergen. Grenen `roda-dagar-b` ligger kvar oslagen på sidan, vilket syns i grafen i avsnitt 5.

## 4. Ångra
Steg 7 i terminalen:

```
$ git status --short
 M kafe-info/meny.md

$ git restore kafe-info/meny.md

$ git status
On branch main
nothing to commit, working tree clean
```

Sedan gjorde jag en onödig commit (`45d8df3`) och körde `git revert`:

```
$ git revert --no-edit 45d8df3
[main] Revert "Onödig ändring i menyn"
 1 file changed, 1 deletion(-)

$ git log --oneline --graph -6
* a57efd4 Revert "Onödig ändring i menyn"
* 45d8df3 Onödig ändring i menyn
* 379cfc4 Röda dagar: öppet julafton
*   c46a969 Merge branch 'oppettider-sommar'
|\
| * ed54214 Sommartider: helger 09–18
* | 6a3bd45 Helger öppet 10–17
|/
```

Fråga: Vad är skillnaden mellan `git restore`, `git revert` och `git reset`? Vilken är säkrast när ändringen redan är pushad?

Svar:
- `git restore fil` kastar ändringar som inte är committade och de går inte att få tillbaka. Det rör bara min egen arbetskopia, inte historiken.
- `git revert hash` skapar en NY commit som gör motsatsen till en gammal commit. Den gamla commiten finns kvar, så historiken bevaras och ingen annan påverkas.
- `git reset --hard hash` flyttar grenen bakåt och kastar bort commits och ändringar. Det skriver om historiken och kan radera arbete.

Säkrast när något redan är pushat är `git revert`. Andra kan ha hämtat den gamla historiken. Med revert lägger jag bara till en commit ovanpå, så alla kan hämta den som vanligt. Med reset (och sedan `push --force`) skulle min historik inte längre stämma med deras och jag kan radera deras arbete. `restore` hjälper inte alls för något som redan är committat.

Steg 8, `.gitignore`. Jag skapade `.env`, `debug.log` och `node_modules/x.txt`. Före:

```
Untracked files:
	.env
	debug.log
	node_modules/
```

Jag skapade `.gitignore` med raderna `.env`, `*.log` och `node_modules/`. Efter:

```
Untracked files:
	.gitignore
```

Sedan committade jag bara `.gitignore`.

Varför ska man aldrig committa en `.env`? Den innehåller hemligheter som lösenord och API-nycklar. Allt som pushas till GitHub finns kvar i historiken även om jag tar bort filen senare, och om repot är publikt kan vem som helst läsa nycklarna (och missbruka dem, ofta inom minuter). Då måste nycklarna bytas ut. Man lägger `.env` i `.gitignore` så att den stannar på den egna datorn.

## 5. Pull request
Länk till min pull request (på mitt eget repo, inte Lexicon-RC):

https://github.com/fungusflip/m5-dag09-git-grenar/pull/1

Jag skapade grenen `info-uppdatering`, la till Instagram i `kafe-info/kontakt.md`, och körde `git push -u origin info-uppdatering`. PR:en öppnades med base `fungusflip/m5-dag09-git-grenar` och mergades med en merge-commit. Lokalt körde jag `git switch main`, `git pull` och `git branch -d info-uppdatering`. Historiken efteråt:

```
$ git log --oneline --graph --all
*   5db449d Merge pull request #1 from fungusflip/info-uppdatering
|\
| * 42eff58 Lägg till Instagram i kontaktinfo
| * b856b92 Lägg till .gitignore för .env, *.log och node_modules/
| * a57efd4 Revert "Onödig ändring i menyn"
| * 45d8df3 Onödig ändring i menyn
| * 379cfc4 Röda dagar: öppet julafton
| | * bb786db Röda dagar: helt stängt
| |/
| *   c46a969 Merge branch 'oppettider-sommar'
| |\
| | * ed54214 Sommartider: helger 09–18
| * | 6a3bd45 Helger öppet 10–17
| |/
| *   a2bcd40 Merge branch 'meny-nyheter'
| |\
| | * f5f8c91 Lägg till kanelbulle på menyn
| |/
|/|
| * 7b2dc72 Lägg till telefonnummer
|/
* 5a5841d Modul 5 i TypeScript med Vite
* 3804533 Add files via upload
* f7a13a4 Add files via upload
* c9a0075 Initial commit
```

Här syns två merge-commits från steg 3 och 5, PR-mergen på toppen, och `roda-dagar-b` som aldrig mergades.

## 6. Reflektion
Vad är det värsta som kan hända om man kör `git push --force` på en gren som andra använder?

Svar: Force-push skriver över historiken på GitHub med min version. Om andra hade pushat commits som jag inte hade hämtat försvinner deras arbete från GitHub. De som redan har hämtat grenen får sedan en historik som inte stämmer med servern, vilket ger rörig merge och risk att gamla, borttagna commits dyker upp igen. Därför gör man revert i stället och använder aldrig force på delade grenar.
