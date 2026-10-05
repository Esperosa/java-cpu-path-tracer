# 3D Render Physics

Desktopový 3D editor a CPU renderer v čisté Javě 17 (Swing/AWT, bez externích knihoven): rasterový viewport, ray tracer a path tracer se SAH BVH, node editor materiálů a export obrázků i animací.

Autor: [Jiří Pelikán](https://jirkapelikan.cz/projekty/renderer/). Vzniklo jako semestrální projekt z počítačové grafiky (PGRF2).

<table>
  <tr>
    <td width="50%"><img src="docs/readme-assets/still.png" alt="Čtyři materiály ve studiovém osvětlení, path tracer" width="100%"></td>
    <td width="50%"><img src="docs/readme-assets/showcase-model.gif" alt="Průlet scénou v režimu Model" width="100%"></td>
  </tr>
  <tr>
    <td width="50%"><img src="docs/readme-assets/showcase-raytrace.gif" alt="Animace vykreslená ray tracerem" width="100%"></td>
    <td width="50%"><img src="docs/readme-assets/showcase-pathtrace.gif" alt="Animace vykreslená path tracerem" width="100%"></td>
  </tr>
</table>

## Funkce

**Renderery**

- Rasterový viewport (režimy Model, Basic, Phong) s paralelním rasterem, frustum a backface cullingem a adaptivním interním rozlišením.
- CPU ray tracer: stíny, odrazy, lom a průchod světla, dlaždicové paralelní vykreslování.
- CPU path tracer: globální iluminace, sklo s IOR a disperzí, homogenní objem (Beer-Lambert), russian roulette.
- Akcelerační struktura BVH dělená podle SAH (surface area heuristic).
- Adaptivní vzorkování a progresivní zpřesňování obrazu ve viewportu.
- Denoisery: joint bilateral filtr a temporální reprojekce.
- HDRI prostředí pro osvětlení a pozadí (pět přibalených map).

**Stylizované režimy**

- Wireframe se skrytými hranami a zvýrazněním siluety.
- Dithering (blue noise, pattern) a ASCII režim, který vybírá znak podle podobnosti s blokem obrazu.
- Temporal Noise: tvar objektů je vidět jen z pohybu šumu.
- Hex Mosaic.

**Editor**

- Rozložení podobné Blenderu: toolbar, viewport, panel vlastností, spodní dock s časovou osou.
- Node editor materiálů (25 typů uzlů) s náhledem; jeden graf se vyhodnocuje ve všech rendererech.
- Import OBJ (včetně MTL a textur), STL, glTF a GLB; import PBR sady textur sestaví graf automaticky.
- Primitiva (krychle, koule, válec, torus, torus knot a další), transformace, undo/redo.
- Časová osa s klíčováním.
- Export: statický snímek, sekvence obrázků, GIF a AVI (MJPEG), každý běh do vlastní složky s `manifest.json`.

Experimentální části: částicový spray/splash emitter a základ systému galaxie.

## Požadavky

- JDK 17 nebo novější (doporučeno 21) v `PATH` nebo v `JAVA_HOME`.
- Žádné další závislosti. `pom.xml` slouží jen jako metadata pro IDE, build běží přes skripty níže.

## Build a spuštění

Windows (PowerShell):

```powershell
.\build.ps1                          # kompilace do build/classes
.\build.ps1 -Run -Profile balanced   # kompilace a spuštění
```

Linux, macOS, Git Bash:

```bash
./build.sh
./build.sh --run --profile=balanced
```

Profily spuštění:

| Profil | Nastavení JVM | Použití |
| --- | --- | --- |
| `safe` (výchozí) | `-Xint` | nejstabilnější, nejpomalejší |
| `balanced` | C1 JIT, vybrané metody tracerů vyloučené z kompilace | běžná práce |
| `fast` | plný JIT | nejvyšší výkon |

Po buildu lze aplikaci spustit i přímo:

```bash
java -cp build/classes Main
java -cp build/classes Main --fullscreen
java -cp build/classes Main --internal-render=960x540
```

Windows installer s přibaleným Java runtime vytvoří `.\package.ps1 -Version v1.0.0` (výstup v `build/package/`).

## Testy

Testy jsou samostatné spustitelné třídy bez testovacího frameworku; seznam je v `tests/test-class-list.txt` (59 sad: renderery, materiály, import, editor).

```powershell
.\tests\run-tests.ps1
```

```bash
./tests/run-tests.sh
```

Benchmarky rendererů a exportu (`quick`, `standard`, `full`):

```powershell
.\tests\run-project-metrics.ps1 -BenchmarkMode quick
```

```bash
./tests/run-project-metrics.sh quick
```

## Ovládání (výběr)

| Klávesa | Funkce |
| --- | --- |
| `G`, `1`–`9`, `0` | přepnutí render režimu (`7` ray tracing, `8` path tracing) |
| `Z` | cyklus render režimů |
| `PgUp` / `PgDown` | vzorky na snímek v ray/path režimu |
| `Q` / `E` | navigace ve stylu FPS / Blender |
| `MMB`, `Shift+MMB`, kolečko | orbit, posun, zoom |
| `Shift+A` | menu pro přidání objektu |
| `Alt+G` / `Alt+R` / `Alt+S` | posun / rotace / měřítko |
| `Ctrl+Z` / `Ctrl+Y` | zpět / znovu |
| `F` | zaměřit výběr |
| `H` | nápověda |

Úplný seznam zkratek je v [docs/technical-reference.md](docs/technical-reference.md#ovládání-a-zkratky).

## Struktura projektu

```text
src/
  Main.java
  engine/
    core/        editor, UI controllery, výstup, historie, zkratky
    render/      raster, stylizované režimy
      ray/bvh/   BVH a SAH dělení
      ray/core/  ray tracer, path tracer, vzorkování, denoisery
    material/    materiály, node graf, náhled, import sad textur
    scene/       entity, světla, scéna
    io/          import OBJ, STL, glTF/GLB
    sim/         experimentální simulace
    camera/, math/, geometry/, physics/, ui/, util/
tests/           regresní a smoke testy, benchmarky
assets/          HDRI, výchozí model, ikony
docs/            technická dokumentace a obrázky do README
installer/       skripty Windows installeru
```

## Dokumentace

- [docs/architecture.md](docs/architecture.md): členění balíků a odpovědnosti.
- [docs/rendering.md](docs/rendering.md): renderery a pipeline.
- [docs/materials.md](docs/materials.md): materiálový graf.
- [docs/output.md](docs/output.md): export a session složky.
- [docs/technical-reference.md](docs/technical-reference.md): podrobný popis světelného modelu, matematiky rendererů, Temporal Noise, UI, všech zkratek a naměřené benchmarky.

## Omezení

- Vše běží na CPU; path tracer je vhodný spíš pro menší rozlišení a statické snímky.
- Rasterový viewport není fyzikálně referenční, přesnou odezvu materiálů dávají až ray a path režimy.
- Objemy jsou jen homogenní.
- FBX je v dialogu pro import, ale importer ho nepodporuje.
- Simulace (spray, galaxie) jsou experimentální.

## Zdroje assetů

- HDRI mapy v `assets/environments/` jsou z [Poly Haven](https://polyhaven.com/), licence CC0. Seznam je v [assets/environments/README.md](assets/environments/README.md).
- Výchozí model `assets/models/StartModel.glb` (plyšový jednorožec) pochází ze Sketchfabu; autor a licence zatím nejsou v repozitáři uvedené.

## Licence

Repozitář zatím neobsahuje licenční soubor. Bez něj zůstávají všechna práva autorovi.
