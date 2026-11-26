# VIctory sur une RTX 4060 8 Go

Une variante de [avatarmanz/VIctory-7900XTX](https://github.com/avatarmanz/VIctory-7900XTX),
elle-même un fork de [VIctory](https://github.com/Marvisak/VIctory), une couche de
compatibilité qui fait tourner un titre Xbox Series nativement sous Windows, à partir des
fichiers que l'utilisateur possède.

Cette variante vise **une carte NVIDIA de 8 Go** : celle d'origine est réglée pour une
Radeon RX 7900 XTX de 24 Go et déborde de la mémoire d'une 8 Go.

**Ce dépôt ne contient aucun fichier, code ou ressource du jeu.** Il vous faut votre propre
copie. Il n'est ni affilié au développeur ou à l'éditeur du jeu, ni approuvé par eux.

## La machine de test

| | |
|---|---|
| Carte graphique | NVIDIA GeForce RTX 4060, 8 Go (groupes de 32 lanes) |
| Processeur | AMD Ryzen 7 7800X3D |
| Mémoire | 32 Go |
| Système | Windows 11 |
| Profil | Series X, rendu en 1920x1080 |

## Où on en est (2026-11-26)

Mesuré sur 13 minutes par partie, menus passés automatiquement, même passage du jeu.

| Version | fps dans le monde | VRAM utilisée / budget | Textures envoyées | Plantage |
|---|---|---|---|---|
| Build 7900 XTX d'origine | 5,8 (pointes 10) | 7,35 / 6,65 Go | 77 000 | non |
| **Ce dépôt, `play-4060.ps1`** | **5,6 (pointes 9)** | **6,75 / 6,67 Go** | **166 000** | **non** |
| Ce dépôt, `play-4060.ps1 -Safe` | 4,9 (pointes 7) | 6,45 / 6,36 Go | — | non |

Ce qui marche : menus, vidéos, chargement, la prison, la mission en bateau.

Ce qui ne va pas encore :

- **Des blocs rectangulaires d'image** montrent le mauvais contenu (une tuile d'écran
  dessinée avec une autre texture). Cause : sur NVIDIA, le tas de descripteurs est
  plafonné à 1 000 000 de vues, et les 262 144 slots que VIctory se réserve pour les tables
  de chaque dessin s'épuisent ; environ 950 tables par relevé sont refusées et les dessins
  indirects qui en dépendent sont abandonnés. C'est
  [l'issue #35](https://github.com/avatarmanz/VIctory-7900XTX/issues/35) du dépôt d'origine.
  `-Safe` en divise le nombre par 2,5 environ, au prix d'environ 13 % de fps.
- Les pull requests #47, #48, #42, #10, #18, #4, #23 et #24 du dépôt d'origine suppriment
  ces refus, mais avec elles le jeu plante au bout de quelques minutes (lecture à une
  adresse nulle dans son propre code) et tourne 35 à 60 % moins vite sur cette machine :
  elles ne sont **pas** incluses ici.
- 5 à 8 passes de calcul écrites pour des groupes de 64 lanes sont refusées par le pilote
  NVIDIA (eau, transmission alpha, particules, effets de triangles).
  `VICTORY_WAVE_WIDE=1` en fait accepter davantage, mais a fait perdre la carte : à éviter.
- Les intérieurs peu éclairés restent sombres sans le ray tracing du jeu.
- La première partie après une nouvelle version saccade le temps que le pilote compile les
  pipelines.

## Ce qui est ajouté par rapport au build 7900 XTX

- **`VICTORY_TEXTURE_DROP_MIPS=N`** : les grosses textures 2D placées par le jeu
  (5 niveaux ou plus, 512 texels ou plus de côté) sont gardées sur la carte sans leurs N
  niveaux les plus détaillés. Les vues de ces textures montrent la copie à partir de son
  premier niveau. Fait sur la mécanique de « premiers niveaux » déjà présente dans ce
  build, d'après [fxceless/victory-fork#610](https://github.com/fxceless/victory-fork).
- Les **copies de textures** du jeu (`CopyResource`, `CopyTextureRegion`, relectures) qui
  touchent une telle texture se font niveau par niveau ; un niveau absent de la copie est
  sauté. Sans cela : « appel non valide » et perte de la carte à l'entrée dans le monde.
- Les **vues de textures en tableau** sont décalées comme les vues 2D, et une vue ne
  demande jamais plus de niveaux que la copie n'en a.

## Comment le lancer

Il faut Windows 10/11 x64, un processeur avec AVX2, une carte Direct3D 12 avec shader
model 6.6 et mesh shaders, Rust (la version est fixée par le dépôt), les outils de
compilation MSVC, et votre propre copie du jeu.

```powershell
git clone https://github.com/Cosatal/VIctory-4060
cd VIctory-4060
copy victory.example.toml victory.toml      # puis indiquez vos chemins, hors de ce dossier
cargo build --locked --profile diag -p victory --features diagnostics
scripts\play-4060.ps1 -Exe <chemin de l'exécutable de votre copie>
```

- `-Safe` : moins de blocs d'image, un peu plus lent.
- `-Native` : la résolution du jeu au lieu de 1920x1080 (plus lent, plus de VRAM).
- `-DropMips 1` : textures plus nettes si votre carte a de la marge (surveillez la VRAM).

Lancez-le depuis une fenêtre normale, pas en administrateur.

## Crédits

Tout ceci repose sur le travail d'autres personnes :

- **[VIctory](https://github.com/Marvisak/VIctory)** par Marvisak et ses contributeurs :
  le chargeur, les services du runtime, le back end Direct3D 12 et le recompilateur de
  shaders.
- **[avatarmanz/VIctory-7900XTX](https://github.com/avatarmanz/VIctory-7900XTX)** : la base
  de ce dépôt (son README est dans [docs/README-7900XTX.md](docs/README-7900XTX.md), avec
  ses propres crédits : fxceless, DexNom, Belobug, FQY55kai, myokex et d'autres).
- **[fxceless/victory-fork](https://github.com/fxceless/victory-fork)** : l'idée de
  `TEXTURE_DROP_MIPS` (pull request #610).
- Les changements de cette variante ont été faits avec
  [Claude Code](https://claude.com/claude-code).

L'historique git d'origine n'est pas repris : ce dépôt part d'un seul commit qui contient
l'arbre complet du build 7900 XTX (commit `0a43566`) plus les changements ci-dessus.

## Licence

GPL-3.0-or-later, comme VIctory. Voir [LICENSE](LICENSE) ; le seul fichier non couvert est
nommé dans [docs/asset-policy.md](docs/asset-policy.md).
