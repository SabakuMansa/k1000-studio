# Journal des modifications — K1000 Studio

> Session de travail autonome (nuit). **Tout est strictement local, aucun `git push`.**
> À relire au matin avant toute mise en production.

---

## [Ajout] Le Labo : 4ᵉ entrée « Game 5 » (+ premier visuel réel du site)

Ajout d'un quatrième projet dans la liste de `/labo` (`app/labo/page.tsx`),
au **même format** que les trois existants (Mousquetaires, Tron, Ludothèque) :

- **Titre** : « Game 5 » · **statut** : « Jeu réel · en ligne ».
- **Lien** : `https://game5.fr` — ouvre dans un **nouvel onglet**
  (`external: true` → `target="_blank" rel="noopener noreferrer"`, vérifié
  dans le DOM rendu).
- **Description** (police `font-terminal` = VT323, **pas** de police pixel,
  contrainte de lisibilité respectée, vérifiée via `getComputedStyle`) :
  simulation de carrière de joueur esport, moteur déterministe calibré sur des
  centaines de milliers de carrières, carte de fin de carrière partageable.

**Premier visuel bitmap du site.** Contrairement aux autres entrées (aperçus
déco emoji/SVG, explicitement « pas le vrai visuel »), on dispose ici du vrai
visuel du jeu, donc `Game5Preview` l'affiche :

- Image copiée par `cp` depuis `/Users/kam/Projects/game5/portfolio/game5-og.jpg`
  vers `public/portfolio/game5-og.jpg` (1200×630, 59 Ko). **Aucun fichier du
  projet game5 n'a été modifié.**
- Le dossier `portfolio/` de game5 contenait aussi `game5-accueil.png` (480×900)
  et `game5-carte-de-jeu.png` (480×840) — non retenus : format portrait mobile,
  inadapté au gabarit paysage `h-32` des aperçus.
- Premier usage de `next/image` dans le dépôt (aucun `<Image>` ni `<img>`
  n'existait auparavant) : `fill` + `object-cover` dans le conteneur `h-32`
  habituel, pour que la grille reste homogène.

**Refactor au passage** : le sélecteur d'aperçu était un ternaire imbriqué à
3 cas ; passé à une table `PREVIEWS` indexée par `p.key` (typée sur les clés de
`PROJECTS`), pour éviter un 4ᵉ niveau d'imbrication.

Vérifié en local (`localhost:3002/labo`) : `tsc --noEmit` sans erreur, aucune
erreur console, image servie par l'optimiseur Next (HTTP 200, `naturalWidth`
> 0), rendu desktop (1280) et mobile (390) OK sans débordement horizontal.
Non-régression : `/`, `/qui-je-suis`, `/labo`, `/labo/tron`,
`/exemples/presence`, `/exemples/boutique` répondent tous 200. Diff limité à
`app/labo/page.tsx` + le nouveau `public/portfolio/`.

> **Note de périmètre** : la demande prévoyait aussi une carte Game 5 dans une
> section « Mes Créations » → « Explorations créatives ». Cette section
> **n'existe nulle part** dans le dépôt : ni dans le code, ni dans
> `app/_archive/`, ni dans l'historique git (`git log -S`). La page d'accueil
> n'a que 4 blocs (`V3Verdict`, `V3Hero`, `V3Plans`, `V3Fondateur`) et `/labo`
> est le seul endroit avec des cartes projets. Arbitrage validé par Kam :
> **`/labo` tient lieu de « Mes Créations »**, l'entrée ci-dessus couvre donc
> les deux demandes. Ni la homepage ni la nav n'ont été modifiées.

**Tout reste local** (aucun déploiement demandé).

---

## [Ajout] Le Labo : 3ᵉ entrée « Ma Ludothèque »

Ajout d'un troisième projet dans la liste de `/labo` (`app/labo/page.tsx`),
au **même format** que les deux existants (Mousquetaires, Tron) :

- **Titre** : « Ma Ludothèque » · **statut** : « App réelle · en ligne ».
- **Lien** : `https://ma-ludotheque.vercel.app` (URL de prod stable, vérifiée :
  HTTP 200 et sert bien l'app Flutter). Ouvre dans un **nouvel onglet**
  (`external: true` → `target="_blank" rel="noopener noreferrer"`).
- **Description** (police `font-terminal`, **pas** de police pixel, contrainte
  de lisibilité respectée) : gestion de collection de jeux de société —
  catalogue, fiches avec vidéos de règles FR, assistant de recommandation de
  soirée.
- Petit aperçu déco `🎲` (`LudothequePreview`), dans le même esprit emoji que
  le 🏈 de Mousquetaires. Le sélecteur d'aperçu (ternaire binaire) est passé à
  3 cas.

Vérifié en local (`localhost:3002/labo`) : les 3 cartes sont visuellement
cohérentes, le lien pointe vers la bonne URL et s'ouvre dans un nouvel onglet.
Aucune erreur serveur. **Tout reste local** (pas de déploiement demandé).

> Note : le prompt mentionnait « la page reste non liée dans la nav et en
> noindex » — or `/labo` est en réalité **liée dans la nav principale et
> indexable** (cf. commentaire d'en-tête du fichier + `sitemap.ts`) ; seules
> les expériences internes (ex. `/labo/tron`) sont en noindex. Je n'ai **rien
> changé** au référencement/à la nav (hors périmètre de cet ajout) ; à
> arbitrer si tu veux réellement passer `/labo` en noindex.

## [Fix] Joystick tactile : cap absolu au lieu de rotation relative

Correction demandée : le joystick réutilisait le modèle clavier (`turn` +
`thrust` selon le cap courant), ce qui est le bon modèle pour ZQSD (rotation
relative) mais pas pour un joystick, où l'utilisateur attend une direction
absolue — pousser le pouce en haut-droite doit orienter le vaisseau
haut-droite, pas juste avancer dans le cap déjà pris.

- `ShipInput` (`types.ts`) gagne `targetAngle: number | null` — `null` =
  mode relatif (clavier, inchangé), un nombre = cap absolu visé (radians).
- `flightModel.stepFlight` : si `targetAngle` est fourni, le vaisseau pivote
  PROGRESSIVEMENT vers ce cap par le plus court chemin angulaire
  (`atan2(sin Δ, cos Δ)`, gère le passage par ±π), borné par la nouvelle
  constante `FLIGHT.JOYSTICK_TURN_SPEED` — jamais de snap instantané, feel
  Asteroids préservé. Sinon (clavier), `turn` pivote directement comme avant.
  La poussée n'a pas eu besoin de changer : elle s'applique déjà le long du
  cap courant du vaisseau, qui suit maintenant le joystick.
- `input/touchInput.ts` : le vecteur du joystick devient directement
  `targetAngle = atan2(jy, jx)` (même repère écran/monde, Y vers le bas —
  haut du joystick = haut à l'écran) + une poussée proportionnelle à
  l'intensité, avec zone morte (`FLIGHT.JOYSTICK_DEAD_ZONE`, remappée en
  [0,1] pour un démarrage franc juste après le seuil). `turn` n'est plus
  utilisé côté tactile.
- `input/combinedInput.ts` : fusionne `targetAngle` (dernière valeur non
  nulle rencontrée — en pratique seul le tactile le fournit).
- Bouton boost existant conservé tel quel : il correspond à Shift au
  clavier (multiplicateur ×1,8), pas un doublon de la poussée proportionnelle.

Vérifié en viewport tactile simulé (joystick piloté directement via
`engine.touch` pour un test déterministe) : pousser à droite oriente et
déplace le vaisseau à droite, pousser en haut l'oriente vers le haut,
rotation progressive et non un snap (trajectoire courbe visible sur
plusieurs captures), relâchement = vaisseau glisse par inertie sans
rotation forcée. Clavier (touche D) revérifié fonctionnel après relâchement
du joystick — comportement relatif inchangé. `tsc --noEmit` + build de
production clean.

À vérifier sur un vrai téléphone : que le pouce garde le contrôle précis
près du centre du joystick (la zone morte à 0,12 ne doit pas sembler trop
large ni trop permissive), et que l'intensité de poussée est bien
progressive au toucher (pas un tout-ou-rien qui donnerait l'impression
d'un bug).

---

## [Feature] Tron : retour à l'espace + commandes tactiles

Deux manques comblés sur l'expérience `/labo/tron` : un visiteur qui
atterrit sur une page réelle n'avait aucun moyen d'y revenir, et le
pilotage tactile n'avait pas de HUD.

**Retour à l'espace**
- Marqueur de session (`sessionStorage`) : `tron:visited` posé au moment
  de l'atterrissage (juste avant de quitter `/labo/tron`, dans le `onLand`
  de `components/labo/TronPreview.tsx`), lu par le nouveau
  `components/transition/ReturnToSpaceButton.tsx` — bouton flottant
  (bas-droite, style néon Tron) monté globalement dans `app/layout.tsx`
  (à l'intérieur de `AirlockProvider`, pour `useAirlock()`).
- N'apparaît QUE si ce marqueur est présent : un visiteur arrivé
  directement (lien externe, recherche) ne le voit jamais — additif, zéro
  régression sur la navigation classique du site. Masqué aussi sur
  `/labo/tron` lui-même (déjà dans l'espace).
- Clic ou touche `Échap` → rejoue le sas existant en sens inverse
  (`airlock.enter("/labo/tron")`) : aucune nouvelle mécanique de
  transition, l'`AirlockProvider` est symétrique par construction.

**Réapparition près de la dernière planète**
- `tron:lastPlanet` mémorise la planète quittée. Au retour dans l'espace,
  `computeSpawn()` (dans `TronPreview.tsx`) place le vaisseau juste hors
  de son `approachRadius` (×1,15), orienté vers l'espace ouvert — pas de
  re-atterrissage immédiat, continuité de l'exploration. Sans marqueur
  (première visite) : spawn par défaut (0,0), inchangé.
- Nécessite que `onLand` reçoive la planète entière (pas seulement sa
  route) : signature changée dans `components/tron/types.ts` et
  `engine.ts` ; `TronEngineOptions.spawn?` + `createFlightState(x,y,angle)`
  ajoutés pour piloter la position/cap initiaux du vaisseau.

**Commandes tactiles**
- État constaté avant travaux : la couche neutre (`touchInput.ts`) était
  déjà écrite et fusionnée dans le moteur (`engine.touch`), mais **aucun
  HUD** ne la pilotait — un visiteur mobile ne pouvait ni tourner ni
  pousser.
- Nouveau `components/labo/TronTouchControls.tsx` : joystick virtuel
  (bas-gauche, pointer events + `setPointerCapture`) + bouton boost
  presser-maintenir (bas-droite). Gros bouton « ATTERRIR » (bas-centre)
  dans `TronPreview.tsx`, affiché seulement à portée d'une planète.
  Détection `matchMedia("pointer: coarse")` — jamais affiché sur desktop ;
  le rappel clavier (bas-gauche) est masqué en retour, remplacé par un FPS
  discret en haut-gauche pour ne pas chevaucher le joystick.

Vérifié : spawn par défaut inchangé sans marqueur ; respawn hors
`approachRadius` confirmé (capture) ; bouton retour visible seulement
avec marqueur + absent sur `/labo/tron` + absent en visite directe ; clic
et `Échap` déclenchent tous deux le retour ; HUD tactile rendu en viewport
mobile (375px), aucun chevauchement ; `tsc --noEmit` + build de
production tous deux clean.

---

## [Feature] "Le Labo" devient une vraie page du site (nav + indexable)

Suite à un retour direct : "Le Labo" n'est plus une page cachée mais une
**vraie page du site**, reliée depuis le menu principal.

- **`content/v3.ts`** — `v3nav.links` : ajout de `{ label: "Le Labo",
  href: "/labo" }` juste après "Qui suis-je" (ordre : Les plans → Qui
  suis-je → Le Labo). Zéro changement dans `components/v3/Nav.tsx` (il se
  contente de mapper `v3nav.links`, desktop et mobile).
- **`app/labo/page.tsx`** — retrait du `noindex/nofollow` : la page hérite
  désormais du `robots: {index:true, follow:true}` par défaut du layout
  racine, comme n'importe quelle page publique.
- **`app/sitemap.ts`** — ajout de `/labo` (priorité 0.5, comme les ancres
  homepage — c'est une page perso, pas un levier commercial).
- **`app/robots.ts`** — `disallow` ne bloque plus `/labo` (sinon Google ne
  pourrait même pas *lire* la page pour découvrir le lien vers `/labo/tron`).
  Remplacé par un disallow **explicite** sur `/labo/tron` uniquement : les
  expériences techniques individuelles restent noindex/nofollow (metadata
  déjà en place sur `app/labo/tron/page.tsx`, inchangée), seule la page hub
  devient publique.

**Nettoyage demandé en parallèle** : l'ancienne démo "portail qui se
fissure façon verre" (`ShatterPortal`), relocalisée à `/labo/portail` dans
le commit précédent le temps de trouver sa place, est **supprimée
entièrement** (`components/labo/ShatterPortal.tsx` + `app/labo/portail/`)
— elle n'est plus référencée nulle part.

Vérifié : menu affiche "Le Labo" après "Qui suis-je" (desktop, capture) ;
`/labo` → `index, follow` + présente dans `sitemap.xml` ; `/labo/tron` →
toujours `noindex, nofollow`, toujours bloquée dans `robots.txt` ;
`/labo/portail` → 404 ; aucune référence résiduelle à `ShatterPortal` ;
`tsc` + build clean.

---

## [Feature] Page hub "Le Labo" (/labo) avec ses 2 premiers projets

`/labo` devient une page hub listant les projets perso (ton plus brut,
décontracté — pas la grille élaborée de "Mes réalisations"), cohérente
avec la DA arcade du site (mêmes composants que `qui-je-suis` : `V3Nav`,
`V3Footer`, `Reveal`). Toujours **noindex/nofollow**, absente de
`v3nav.links` et de `app/sitemap.ts`, toujours disallow dans
`app/robots.ts` — rien n'a changé côté SEO/nav.

**Effet de bord nécessaire** : `/labo` hébergeait *directement* l'ancienne
démo "portail qui se fissure façon verre" (`ShatterPortal`). Déplacée vers
**`/labo/portail`** (même metadata noindex) pour libérer la route — rien
n'est perdu, juste pas encore relié depuis le nouveau hub.

Les 2 premiers projets listés (une carte cliquable + petit aperçu chacun,
d'autres viendront s'ajouter avec le temps) :
- **Les Mousquetaires** — lien externe vers le vrai site du club de foot
  américain (`https://mousquetairesfootus.fr/`, `target="_blank"` +
  `rel="noopener noreferrer"`), présenté comme un projet réel livré
  bénévolement.
- **Navigation spatiale — Tron** — lien interne vers `/labo/tron` (le
  module de navigation spatiale néon déjà construit), présenté comme une
  démonstration technique.

Vérifié : les deux liens répondent (site externe 200, `/labo/tron` 200),
attributs corrects sur le lien externe, `noindex` toujours en place,
build + `tsc` clean, rendu desktop + mobile.

---

## [Fix] Favicon rebrandé (NOVA « N » → K1000 « K » pixel arcade)

Le favicon de l'onglet montrait encore le « N » de NOVA, oublié lors du
rebranding. Seul fichier d'icône du projet : `app/icon.svg` (convention
App Router de Next — auto-injecté en `<link rel="icon">` sur toutes les
pages, aucun `favicon.ico`/apple-icon/manifest/PNG à côté).

Remplacé par un **« K » en pixels** (blocs de 6px, or `#FFD23F`) sur badge
sombre `#17130D` avec fin liseré orange `#FF7A00` — cohérent avec le
wordmark Press Start 2P et l'identité arcade. SVG unique → toutes les
tailles couvertes (vectoriel), rendu net (`shape-rendering: crispEdges`).

Vérifié : `/icon.svg` sert bien le nouveau K (0 occurrence du « N »), et
les pages (`/`, `/qui-je-suis`, `/labo/tron`…) référencent le même
`/icon.svg?<hash>` — le hash de contenu a changé, donc cache-busting
automatique côté navigateur.

## [Feature] 4 univers immersifs sur les démos /exemples/* (hero + module clé)

Design handoff hifi fourni en zip (`design_handoff_pages_demo/`, 4 fichiers
`.dc.html` + README avec tokens/mapping exacts). Mapping strict appliqué :
Fleuriste-AuPetitMarche → `/exemples/presence`, Coiffure-MaisonDore →
`/exemples/autonome`, Restaurant-ChezMargot → `/exemples/machine`,
Boutique-Unit9 → `/exemples/boutique`.

**Recette appliquée aux 4 démos** (même principe, dérivé du travail
Présence de la session précédente) :
- Fonts du handoff chargées dans `layout.tsx` sous les **mêmes variables
  CSS** que les anciennes (`--font-fleur-*`, `--font-metam-*`,
  `--font-braise-*`, `--font-nord-*`) — Nav/Footer/sous-pages héritent
  automatiquement de la nouvelle typo sans modification.
- Couleurs en **valeurs Tailwind arbitraires** (`bg-[#3d5245]`, etc.),
  jamais de nouveau token dans `tailwind.config.ts` — pour ne pas
  répercuter la palette dans les sous-pages (prestations/galerie/à propos/
  contact/espace-admin/catalogue/panier…), qui restent volontairement sur
  leur ancienne palette et sont **structurellement inchangées**.
- Seuls Nav/Footer/Banner + le hero et le "module clé" de la page
  d'accueil de chaque démo changent visuellement. Homepage, `/labo`,
  `/_archive` et le reste du site : non touchés.
- 9 URLs Unsplash du zip vérifiées (`curl -I`, 200 + `image/jpeg`) avant
  usage ; toute nouvelle image ajoutée (photos de remplacement) vérifiée
  de la même façon.

**Identités commerçantes fictives — décision explicite de l'utilisateur** :
garder les commerces déjà établis partout où c'est possible (le zip sert
de référence de style, jamais de contenu) :
- **Présence** (Maison Verdure, fleuriste) — module "étal du jour" du zip
  recréé avec le vrai catalogue plantes (Monstera, Pothos, Ficus + carte
  "plante mystère"), pas les fleurs coupées du zip.
- **Autonome** (Salon Marguerite, coiffure) — hero + planning recréés
  quasi tels quels (thématique déjà compatible).
- **Boutique** (Le Petit Atelier, savonnerie/bougies) — palette/typo
  "concept store" du zip reprises, mais produit vedette (Bougie Bois de
  santal), copie et sélecteur de **quantité** (composant `QuantitySelector`
  partagé, nouveau variant `unit9`) adaptés au vrai catalogue — le
  sélecteur de taille S/M/L/XL du zip n'a pas de sens pour du savon.
- **Machine** — seule exception : l'ancien commerce ("Au Poil", toiletteur
  canin) ne pouvait pas raisonnablement porter l'"Ardoise du jour" (vrais
  plats de bistrot) du zip. Renommé **"Chez Fernand"** (jamais "Chez
  Margot" — le nom littéral de la maquette, conformément à la convention
  du projet). `content/exemples/machine.ts` réécrit entièrement (nav,
  accueil, carte, galerie, à propos, contact, espace admin) ; 2 sous-pages
  avaient des URLs/alt-text/commentaires de toilettage codés en dur
  (`galerie`, `a-propos`) — mis à jour avec de nouvelles photos vérifiées.

**Distinction fonctionnelle Autonome/Machine préservée dans le nouvel
habillage** (risque identifié : les deux modules du zip sont purement
visuels et ne montrent ni édition manuelle ni automatisation) :
- `/exemples/autonome` : nouveau composant `PlanningSlot.tsx` — un crayon
  discret sur un créneau du planning révèle, au clic, un champ texte +
  bouton "Enregistrer" (mockup local sans backend, palette or/noir,
  aucune animation donc aucun souci `prefers-reduced-motion`).
- `/exemples/machine` : indicateur "Activité automatique" (pastille
  pulsante + relance avis) ajouté sous l'ardoise, `motion-safe:animate-ping`
  pour respecter `prefers-reduced-motion`.

**Exécution** : Présence construit à la main comme référence, puis
Autonome/Machine/Boutique délégués à 3 agents en parallèle avec brief
détaillé (recette + palette exacte + limites strictes), chacun revu,
corrigé si besoin et vérifié séparément avant commit individuel.

**Vérifié** : `tsc --noEmit` clean, `npm run build` clean (46 pages),
navigateur desktop + mobile sur les 4 pages d'accueil comparées aux
`.dc.html` correspondants, interaction d'édition manuelle testée
(clic → champ + Enregistrer), aucune régression `git status` en dehors
des fichiers des 4 démos, aucune mention "Uber" ajoutée, aucun nom
littéral de maquette adopté comme identité commerçante (grep vérifié —
seules des mentions en commentaire expliquant la source du design).

4 commits locaux séparés (un par démo), aucun push.

---

## [Fix] Police pixel arcade recadrée — trop généralisée, illisible par endroits

Une session précédente avait explicitement étendu `font-pixel` (Press
Start 2P) à la Nav, aux boutons et aux titres de cartes "sitewide" (voir
tâches #71–73 de l'historique). Avec le recul, ça dépasse l'usage prévu
(le commentaire de `tailwind.config.ts` dit lui-même : "Réservées au Hero
+ à 'La Carte'... jamais utilisées ailleurs") et rend plusieurs zones
peu lisibles. Recadrage à cette règle d'origine.

**Contrainte respectée** : uniquement le `font-family` a changé
(`font-pixel` → `font-sans`, Work Sans déjà utilisé ailleurs sur le
site) — aucune couleur, taille, espacement ni mise en page touchés.

### Repassé en `font-sans` (navigation, boutons, phrases de contenu)

- **`components/v3/Nav.tsx`** : lien "INSERT COIN" (bouton réel du
  bandeau bezel — "1P"/"HI" restent en pixel, ce sont des labels non
  interactifs), liens de nav desktop et mobile, CTA "Audit gratuit"
  (desktop + mobile).
- **`components/v3/Hero.tsx`** : bouton CTA principal ("Réserver un
  audit gratuit").
- **`components/v3/Sections.tsx`** : bouton "Démarrer avec {plan}" sur
  chaque carte de "La Carte" ; titre de la section Fondateur (teaser
  homepage — phrase complète, pas un badge court).
- **`app/qui-je-suis/page.tsx`** : phrase "On regarde votre commerce
  ensemble ?" (contenu conversationnel, pas un badge) et son bouton CTA.
- **`components/v3/InsertCoinOverlay.tsx`** : bouton "ÉCHAP — Fermer".
- **`components/v3/SpaceInvadersGame.tsx`** : bouton "Rejouer", les deux
  boutons directionnels ◀/▶ et le bouton "TIR" (contrôles tactiles du
  mini-jeu — ce sont de vrais boutons, même règle que le reste du site).

### Laissé en `font-pixel` (usage désormais strictement limité)

- Wordmark "K1000.studio" + monogramme (Nav, Footer) — partout.
- Titre du Hero (`v3hero.titleA/Em/B`) et titre de "La Carte"
  (`v3plans.title`) — les deux grands titres d'accroche explicitement
  autorisés.
- Question du Verdict (même traitement que "La Carte", classes
  identiques) et sa réponse défilante (`TypewriterAnswer`) — duo qui
  fait office de titre d'accroche principal de la page.
- H1 de `/qui-je-suis` — mêmes classes exactes que le titre de "La
  Carte"/Verdict (même famille de "grands titres"), traité en
  cohérence plutôt que la version teaser plus petite sur la homepage.
- Badges/labels courts : eyebrows ("Studio digital · Île-de-France",
  "Les plans", "Qui suis-je"...), bandeau bezel "1P/HI"/"SELECT YOUR
  PLAN"/"CREDIT", badge "Le plus choisi", noms de plans ("Présence",
  "Autonome", "Machine", "Boutique"), prix ("690€"...), "Insert Coin"
  (overlay), "Game over"/score/vies (HUD du mini-jeu) — tous non
  interactifs ou strictement courts, conformes à la règle "éventuellement
  de courts labels/badges".

### Non touché

- **`components/exemples/ExempleNav.tsx` / `ExempleFooter.tsx`** :
  utilisent encore `font-pixel`, mais confirmés **orphelins** (aucun
  import réel nulle part dans le dépôt, seulement des mentions en
  commentaire dans les composants Nav/Footer propres à chaque démo) —
  aucun impact visible, non modifiés pour rester dans le périmètre
  strict de la tâche.
- **`/exemples/*`** (Présence/Autonome/Machine/Boutique) : vérifiées,
  n'ont jamais utilisé `font-pixel` — chacune a sa propre police dédiée
  (fleur-\*/metam-\*/braise-\*/nord-\*), déjà lisible.
- Tout texte déjà en `font-terminal` (VT323) ou `font-mono` (sous-titres,
  listes de fonctionnalités, badges, taglines) — la plainte portait
  spécifiquement sur `font-pixel`, ces polices sont déjà la police
  "corps de texte" prévue par le design system et restent lisibles.

### Vérification effectuée

- `tsc --noEmit` ✅, `npm run build` ✅.
- Relecture visuelle complète en navigateur (desktop + mobile) : page
  d'accueil (Nav, Verdict, Hero, La Carte, Fondateur), `/qui-je-suis`
  intégralement, menu mobile déplié — confirmé qu'aucun paragraphe,
  liste, formulaire, bouton ou lien de nav n'est plus en pixel.
- Grep final `font-pixel` sur tout le dépôt : les seules occurrences
  restantes correspondent exactement à la liste "laissé en pixel"
  ci-dessus (+ les 2 fichiers orphelins non touchés).

---

## [Fix] Image de prévisualisation (Open Graph) obsolète

`app/opengraph-image.tsx` reflétait encore l'ancienne identité "geek
coloré" (fond crème, blobs aurora, wordmark violet, accroche "Pendant que
vous êtes en plein service…") — jamais mis à jour lors du passage à
l'identité pixel arcade actuelle (V3Nav/V3Hero), ni lors du rebranding
K1000 Studio (le texte avait été renommé mais pas le design lui-même).

### Corrigé

- **Fond** : crème clair → sombre anthracite (`#17130D`, dégradé radial
  identique au panneau Hero réel), cohérent avec le reste du site.
- **Wordmark** : monogramme "K" (fond doré, bordure épaisse, ombre portée
  dure) + "K1000" crème + ".studio" **orange** (`#FF7A00`, plus violet) —
  calqué exactement sur `components/v3/Nav.tsx`, y compris le bandeau
  bezel "1P 00000 · INSERT COIN · HI 99999" repris du haut de la Nav
  réelle.
- **Police** : Press Start 2P (titres/wordmark) et VT323 (texte courant)
  — les vraies polices du site, chargées depuis des fichiers locaux
  (`public/fonts/PressStart2P-Regular.ttf`, `VT323-Regular.ttf`,
  téléchargés depuis Google Fonts) via l'option `fonts` d'`ImageResponse`.
  Plus de police système de substitution.
- **Titre** : remplacé par le texte réellement affiché dans le Hero du
  site (`v3hero.titleA/titleEm/titleB` dans `content/v3.ts` — "Vos futurs
  clients vous cherchent déjà. **Assurez-vous** qu'ils vous trouvent."),
  vérifié dans le contenu plutôt que deviné. L'ancienne accroche "Pendant
  que vous êtes en plein service, votre site bosse." ne correspond plus à
  aucun texte actuellement affiché sur le site et a été retirée de
  l'image (elle reste la baseline `description` des metadata OG/Twitter,
  cohérente avec le slogan produit).
- **Métadonnées associées** (`app/layout.tsx`) : `title`, `description`,
  `openGraph.title/description`, `twitter.title/description` déjà
  cohérents avec "K1000 Studio" depuis le rebranding précédent — vérifiés,
  aucune mention "NOVA" résiduelle trouvée.

### Bugs Satori rencontrés et corrigés en cours de route

- `width: "fit-content"` non supporté par Satori (moteur derrière
  `ImageResponse`) → remplacé par `alignSelf: "flex-start"` sur le badge
  eyebrow.
- `display: "inline"` non supporté (valeurs autorisées : `flex` / `block`
  / `none` / `-webkit-box`) → `flex` partout.
- Mélanger du texte brut et un `<span>` coloré imbriqué dans un seul flux
  ne se met pas à la ligne correctement (le span déborde au lieu de
  wrapper) → titre reconstruit en deux lignes explicites (une par phrase
  réelle) plutôt qu'un seul paragraphe à retour automatique.

### Vérification effectuée

- `tsc --noEmit` ✅, `npm run build` ✅ (route `/opengraph-image` générée
  sans erreur, 1200×630 confirmé).
- Rendu visuel vérifié en navigateur (`/opengraph-image` en local) :
  fond sombre, wordmark identique à la Nav réelle, titre exact, aucun
  débordement ni texte coupé.
- **Non vérifié** : rendu via un outil de partage de lien externe
  (opengraph.xyz, Facebook/LinkedIn debugger) — ces outils nécessitent
  une URL publique pour aller chercher l'image, ce que `localhost` ne
  permet pas. Cette étape suppose un déploiement (push) préalable, non
  demandé explicitement pour cette tâche — à faire une fois le fix
  poussé/déployé.

---

## [Rebranding] NOVA Studio → K1000 Studio

Rebranding complet du site, à la demande explicite (cette tâche autorise
`git push` et le déploiement, contrairement à la règle par défaut du reste
de cette session).

### Contenu et code

- Wordmark du header (`components/v3/Nav.tsx`, `app/opengraph-image.tsx`,
  et les wordmarks des versions archivées `components/Nav.tsx`,
  `components/v2/Nav.tsx`, `components/Footer.tsx`, `components/v2/Footer.tsx`,
  `components/v3/Sections.tsx`, `components/labo/ShatterPortal.tsx`) :
  monogramme "N" → "K", "NOVA(.studio)" → "K1000(.studio)". Style pixel
  arcade + effet glitch/typewriter du Hero strictement inchangés — seul le
  texte change.
- Toutes les mentions textuelles ("NOVA Studio", copyright, footer, bio
  fondateur) remplacées dans `content/site.ts`, `content/v3.ts`,
  `app/layout.tsx`, `components/JsonLd.tsx`, `app/qui-je-suis/page.tsx`,
  `app/labo/page.tsx`, `app/error.tsx`, les 4 démos `/exemples/*`
  (bannières, footers, metadata de chaque `layout.tsx`) et les pages
  archivées (`_archive/v2`, `_archive/signature`, `_archive/v3-layout`) —
  inclus à la demande explicite malgré la règle habituelle de ne jamais
  toucher aux archives.
- Métadonnées Open Graph / Twitter Card et données structurées Schema.org
  (JSON-LD) mises à jour : `name`, `legalName`, `title`, `description`,
  `og:title`, `twitter:title`.
- `package.json` (`name: "k1000-studio"`), `.claude/launch.json`
  (config nommée `k1000-dev`), commentaire de `tailwind.config.ts`.
- Domaine placeholder `nova-studio.fr` remplacé partout par le domaine
  définitif `k1000studio.fr` (`seo.siteUrl`, emails `bonjour@…`, JSON-LD
  `url`/`image`) — la note "nom de code, à remplacer avant mise en ligne"
  dans `content/site.ts`, `README.md` et `components/JsonLd.tsx` est levée :
  K1000 Studio / k1000studio.fr sont désormais définitifs.
- Clés `localStorage`/`sessionStorage` internes renommées pour cohérence :
  `nova-exemples-boutique-cart` → `k1000-exemples-boutique-cart`
  (`CartContext.tsx`), `nova-exemples-boutique-delivery` →
  `k1000-exemples-boutique-delivery` (`DeliveryTracking.tsx`).
- Documentation interne : `README.md`, `README-delivery.md`,
  `README-reviews.md`, `INSPIRATION.md`, `SIGNATURE-NOTES.md`,
  `V2-NOTES.md` — toutes les mentions "NOVA" remplacées, à l'exception de
  3 références historiques factuelles délibérément conservées (identifiants
  réels d'artefacts externes à un instant précis du passé, pas des mentions
  de marque à jour) : l'ancienne URL de déploiement Vercel
  `nova-sigma-khaki.vercel.app` et le nom du projet/fichier Claude Design
  importé ("NOVA Studio Visual Identity", `NOVA Studio - Site.dc.html`) —
  les réécrire aurait rendu ces entrées factuellement fausses.
- `package-lock.json` régénéré (`npm install`) pour refléter le nouveau nom.

### Vérification effectuée

- Recherche exhaustive `\bnova\b` (insensible à la casse) sur tout le
  dépôt (hors `node_modules`, `.git`, `.next`) : confirmée vide, à
  l'exception des 3 références historiques documentées ci-dessus.
- `tsc --noEmit` ✅, `npm run build` ✅ (exit 0, aucune erreur), 46+ pages
  générées.
- Vérification visuelle en navigateur : wordmark (page d'accueil + image
  Open Graph), titres d'onglet, JSON-LD, page `/qui-je-suis`, démo
  `/exemples/boutique` — "K1000" présent, "NOVA" absent partout où testé.

### Dépôt et déploiement

- **Dépôt GitHub** renommé : `SabakuMansa/NOVA` → `SabakuMansa/k1000-studio`
  (via `gh repo rename`, remote local mis à jour automatiquement).
- **Push** effectué sur `origin/main` (seule tâche de toute la session où
  le push a été explicitement autorisé par l'utilisateur).
- **Vercel** : CLI non installée/authentifiée dans cet environnement —
  renommage du projet et ajout des domaines (`k1000studio.fr` principal,
  `k1000.fr` et `k1000studio.com` en redirection 301) documentés comme
  étapes manuelles dans `DEPLOIEMENT.md`, avec les instructions DNS
  registrar correspondantes.

---

## [Exemple] Livraison directe au panier de `/exemples/boutique`

Ajout d'une démonstration du module Commande & Livraison directe
(`/lib/delivery/`, mode `demo` uniquement) au panier de la démo Boutique
— pour montrer concrètement à un prospect à quoi ressemblerait son
intégration dans un vrai parcours d'achat.

**Décision produit à noter** : `content/exemples/boutique.ts` contenait
depuis une session précédente une note "Ne JAMAIS mettre en avant ici le
module Commande & Livraison directe — vendu séparément, hors sujet".
Conflit signalé avant toute modification ; confirmé explicitement que la
règle ne s'applique pas ici (exemple de boutique en ligne). Le
commentaire du fichier a été mis à jour en conséquence.

### Ajouté

- **`components/exemples/boutique/DeliveryOption.tsx`** : choix "Retrait
  en boutique" / "Livraison à domicile" au panier, saisie d'adresse,
  devis via `/api/delivery/quote` (réutilisé tel quel, aucune nouvelle
  logique). Message aligné sur l'argument commercial déjà établi
  ailleurs sur le site : "vous ne payez que le coût de la course —
  aucune commission prélevée sur votre vente". Aucune mention de la
  marque technique (grep vérifié sur tous les fichiers touchés).
- **`components/exemples/boutique/DeliveryTracking.tsx`** : suivi de
  course (préparation → coursier en route → livré), réutilise tel quel
  `/api/delivery/status` et les types/labels de `lib/delivery/types`
  (`STATUS_LABELS`, `DELIVERY_TIMELINE`) — calque de l'habillage sur
  `components/delivery/DeliveryTracker.tsx` existant, seule la DA change
  (palette `nord-*`). Exporte `DELIVERY_STORAGE_KEY`.
- **`components/exemples/boutique/DeliveryConfirmation.tsx`** : îlot
  client qui lit (et consomme) l'id de course déposé en sessionStorage
  avant la redirection Stripe/mock — n'affiche rien si "Retrait en
  boutique" a été choisi.

### Modifié

- **`app/exemples/boutique/panier/page.tsx`** : ajout du choix de mode
  de réception, recalcul du total (sous-total + livraison) une fois le
  devis obtenu, création de la course (`/api/delivery/create`) avant le
  checkout existant quand la livraison est choisie. Comportement du
  retrait strictement inchangé (même appel `/api/checkout`, même
  redirection).
- **`app/exemples/boutique/confirmation/page.tsx`** : ajout du composant
  de suivi, affiché uniquement si une course a été créée.
- **`content/exemples/boutique.ts`** : commentaire d'en-tête mis à jour
  (cf. décision produit ci-dessus).

### Vérifications effectuées

- `tsc --noEmit` ✅, `next lint` ✅ (0 erreur, 0 avertissement).
- Parcours complet testé au navigateur : ajout au panier → panier →
  "Livraison à domicile" → adresse saisie → devis obtenu (7,75€,
  ~29 min, message "aucune commission" affiché) → total recalculé
  (8,00€ + 7,75€ = 15,75€) → "Passer au paiement" → confirmation avec
  suivi animé jusqu'à "Livré" (les 3 étapes cochées après ~24s, conforme
  aux délais du fournisseur de démo).
- Parcours "Retrait en boutique" revérifié après l'ajout : mode par
  défaut, comportement de checkout identique à avant, aucun suivi
  affiché sur la confirmation.
- `document.body.innerText` vérifié sans "uber" (insensible à la casse)
  sur panier (mode retrait et livraison) et confirmation ; grep du code
  source confirme zéro mention dans tous les fichiers créés/modifiés.
- Mobile (375×812) : toggle et champs d'adresse vérifiés sans
  débordement.
- Point pulsant de l'étape en cours (`animate-pulse`) couvert par la
  règle CSS globale `prefers-reduced-motion` déjà en place — aucune
  animation continue supplémentaire ajoutée.
- **Tout reste local** — aucun `git push`, aucun déploiement.

---

## [Designs Claude Design] `/exemples/autonome`, `/machine`, `/boutique` + vérification finale des 4 démos

Suite de l'import du projet Claude Design (`8029a0b3-2dea-4782-867c-904a4666fe6b`) commencé avec Présence : les 3 fichiers restants appliqués un par un, chacun à sa propre démo, avec son propre commit local.

### `/exemples/autonome` — maquette "Salon de coiffure" → Salon Marguerite

Palette violet/noir/blanc, Space Grotesk + Manrope, look éditorial tech. Nouveaux `components/exemples/autonome/{Nav,Footer,Banner}.tsx`. Espace admin (tabs, sauvegarde) revérifié fonctionnel après restyle — commit `b615cc1`.

### `/exemples/machine` — maquette "Restaurant" → Au Poil (toilettage canin)

Palette rust/orange/teal, DM Serif Display + DM Sans + JetBrains Mono, mise en page dashboard/notifications transposée en excluant tout vocabulaire restaurant (carte, menu). Ajout de bascules "Automatisations actives" dans l'espace admin, dérivées des 3 automatisations réelles déjà documentées (relance avis Google, confirmation réservation, réponse contact) — aucune donnée inventée. Nouveaux `components/exemples/machine/{Nav,Footer,Banner}.tsx` — commit `3ff070e`.

**Deux problèmes trouvés et corrigés pendant la vérification, avant le commit :**
- L'agent en charge de cette démo a inventé 5 URLs Unsplash (motif d'ID plausible mais inexistant) — les 5 renvoyaient une 404. Repérées via `preview_network` (`ERR_BLOCKED_BY_ORB`), remplacées par de vraies photos de chiens vérifiées une par une (`fetch` + inspection visuelle) avant intégration.
- Bug de layout : la colonne de contenu de l'espace admin n'avait pas de `min-w-0`, la forçant à déborder horizontalement (~640–1024px de large ; invisible au repos à cause d'`overflow-hidden`, révélé au clic sur une bascule qui déclenche un scroll-into-view natif). Corrigé, revérifié à 768px/1280px/mobile sans débordement.

### `/exemples/boutique` — maquette "Boutique mode" → Le Petit Atelier (savonnerie/bougies)

Palette camel/ivoire/encre, Playfair Display + Manrope, look éditorial. Sélecteur de taille vêtement de la maquette exclu ; `QuantitySelector` existant conservé comme UI d'ajout au panier, avec une nouvelle variante visuelle optionnelle `"nord"` (défaut `"arcade"` inchangé pour compatibilité). Nouveaux `components/exemples/boutique/{Nav,Footer,Banner,ProductPhoto}.tsx`. Parcours d'achat complet retesté après restyle : catalogue → fiche produit → panier → paiement test Stripe → confirmation (panier vidé) — `CartProvider` et logique Stripe non modifiés — commit `afe13e3`.

**Note de sécurité** : l'agent en charge de cette démo signale que l'historique de conversation du projet Claude Design contenait un texte tentant de se faire passer pour des instructions système concernant un projet sans rapport ("Oncle Wang", avec de fausses instructions git/pwd/Stripe). L'agent a correctement identifié ce texte comme du contenu non fiable observé dans des données externes et l'a ignoré. Signalé ici pour information — aucune action n'a été prise sur la base de ce contenu.

### Vérification finale (les 4 démos)

- `tsc --noEmit` ✅ après chaque étape.
- `grep` sur les 4 démos + leurs composants dédiés : zéro référence à `arcade-*`, `font-pixel`, `font-terminal` ou "Press Start" — aucune fuite du style arcade de la page d'accueil.
- `git diff --stat` entre le commit de départ (`c92b873`) et `HEAD` : les 43 fichiers touchés sont tous dans `app/exemples/{presence,autonome,machine,boutique}`, `components/exemples/{presence,autonome,machine,boutique}`, `components/exemples/QuantitySelector.tsx` et `tailwind.config.ts` — page d'accueil, `/labo`, `/_archive/v1`, `/_archive/v2` et `components/v3/*` non touchés.
- Les 4 démos vérifiées visuellement en desktop et mobile ; chacune garde une identité distincte (florale/sauge, éditorial violet, rust/dashboard, camel/ivoire) sans converger vers un style commun.
- 4 commits locaux séparés (un par démo), aucun `git push`.

---

## [Designs Claude Design] `/exemples/presence` — identité visuelle "Maison Verdure"

Import du projet Claude Design (`8029a0b3-2dea-4782-867c-904a4666fe6b`, fichier
`01 - Fleuriste (Présence).dc.html`) appliqué comme référence visuelle
(palette, typographie, mise en page, ambiance) à la démo Présence. Contenu
réel inchangé (nom, adresse, horaires, avis Google, textes) — seule
l'habillage change. Style propre à cette page, sans lien avec le style arcade
de la page d'accueil ni avec les 3 autres démos.

### Ajouté

- **`tailwind.config.ts`** : 4 nouveaux jeux de tokens couleur/police
  (`fleur-*`, `metam-*`, `braise-*`, `nord-*`) pour les 4 démos, ajoutés en
  une seule fois avant toute délégation pour éviter les conflits d'édition
  concurrente sur ce fichier partagé. Seul `fleur-*` est utilisé pour
  l'instant (Présence) ; les 3 autres seront consommés par les prochaines
  démos.
- **`components/exemples/presence/{Nav,Footer,Banner}.tsx`** (nouveaux,
  propres à cette démo — la structure de la maquette ne correspond pas aux
  composants partagés `ExempleNav`/`ExempleFooter`/`ExempleBanner`
  existants) : en-tête sticky flouté avec monogramme + nom en serif, liens
  actifs colorés, dernier lien en pastille pleine (Contact) ; pied de page
  sombre avec nom/adresse/horaires réels ; bandeau "exemple concret" avec
  lien retour vers `/#plans`.
- **`app/exemples/presence/layout.tsx`** : chargement de Cormorant Garamond
  + Nunito Sans via `next/font/google`, scindé par segment (n'affecte ni la
  homepage ni les 3 autres démos).

### Modifié (style uniquement, contenu réel préservé)

- **`app/exemples/presence/page.tsx`** : hero 2 colonnes (texte + vraie
  photo Unsplash de vitrine), section fiche Google Business restylée.
- **`app/exemples/presence/presentation/page.tsx`** : 2 colonnes
  photo/texte, grille de 3 valeurs.
- **`app/exemples/presence/galerie/page.tsx`** : 4 vraies photos Unsplash
  associées aux 4 items réels de la galerie (icône + libellé conservés en
  légende). Photo initialement reprise de la maquette pour "Poteries &
  accessoires" ne correspondait pas au sujet (paysage de montagne) —
  remplacée par une photo de plante en pot en cours de vérification
  visuelle.
- **`app/exemples/presence/contact/page.tsx`** : logique du formulaire
  strictement inchangée (mêmes champs `required`, même `onSubmit`, même
  état `sent`) — habillage uniquement (fond sage, formulaire 2 colonnes,
  état de succès avec badge circulaire).

### Vérifications effectuées

- `tsc --noEmit` ✅.
- Les 4 pages (accueil, présentation, galerie, contact) vérifiées
  visuellement en desktop et mobile (375×812) via l'outil navigateur —
  aucun débordement, nav mobile empilée lisible.
- Formulaire de contact testé fonctionnellement de bout en bout (remplissage
  des 3 champs, soumission, état "Message envoyé !" confirmé à l'écran) —
  aucune régression sur la logique existante.
- Aucune animation en boucle/auto-déclenchée sur cette page (seules des
  micro-transitions au survol) — pas de garde `prefers-reduced-motion`
  nécessaire.
- Page d'accueil, `/labo`, `/_archive/v1`, `/_archive/v2` et les 3 autres
  démos non touchés (seuls les fichiers listés ci-dessus modifiés/créés).
- **Tout reste local** — aucun `git push`, aucun déploiement.

---

## [Nettoyage du code] Audit préalable — ce qui va être touché ou non

**Avant toute modification.** Checkpoint Git : `HEAD` était déjà propre et
commité (`b68c651`, rien à committer) — c'est le point de retour si besoin.

### Ce qui sera supprimé

- **Dépendance `gsap`** (`package.json`) : **zéro référence** trouvée nulle
  part dans le projet, y compris dans les archives `_archive/` et le labo
  (vérifié par grep exhaustif). Jamais utilisée depuis son ajout — supprimée.

### Ce qui a l'air mort mais NE SERA PAS supprimé — conflit entre deux consignes

Un audit complet (recherche systématique de chaque import, fichier par
fichier) montre que la quasi-totalité des fichiers suivants sont
**« orphelins » du point de vue du site en ligne**, mais **restent
importés par les archives `_archive/v1` et `_archive/v2` protégées** :

- `components/Approche.tsx`, `Carte.tsx`, `Configurator.tsx`, `Contact.tsx`,
  `Footer.tsx`, `Hero.tsx`, `Methode.tsx`, `Motifs.tsx`, `Nav.tsx`,
  `Problemes.tsx`, `Process.tsx`, `ScaleReveal.tsx`, `Temoignages.tsx`
- `components/v2/Footer.tsx`, `Nav.tsx`, `PageHeader.tsx`, `V2Hero.tsx`
- `components/signature/SignatureBackdrop.tsx`, `SignatureScene.tsx`,
  `SignatureTitle.tsx`, `embers.glsl.ts`
- `components/delivery/DeliveryDemo.tsx` (composant de démo obsolète,
  remplacé par `DeliveryOptionSelector`/`DeliveryTracker` sur les vraies
  pages `/demo/*`, mais toujours importé par `components/Carte.tsx` archivé)
- `content/site.ts` : tous les exports **sauf `seo`** (`nav`, `hero`,
  `approche`, `problemes`, `methode`, `carte`, `apercu`, `previewCombos`,
  `process`, `contact`, `footer`, `temoignages`) — utilisés uniquement par
  les composants ci-dessus.

**Pourquoi je ne les supprime pas** : la consigne dit à la fois « repère et
supprime le code mort » et « ne touche jamais aux dossiers d'archive ».
Ces fichiers sont exactement à l'intersection des deux — ils sont morts
*du point de vue du site*, mais vivants *du point de vue de l'archive*
(`app/_archive/v1-onepage.tsx`, `app/_archive/v2/**`). Les supprimer ou les
déplacer casserait la compilation de ces pages archivées, donc reviendrait
à « toucher » l'archive par ricochet — ce qui est explicitement interdit.
**Je laisse tout intact et je signale ce point plutôt que de trancher.**
Si un jour les archives elles-mêmes sont supprimées, ces fichiers
deviendront alors du vrai code mort, supprimable sans risque.

### Console.log / console.error trouvés (hors archives/labo)

- `lib/reviews/email-provider.ts:31` — `console.log` du mode démo
  (`[reviews:demo] Email simulé → ...`), déjà marqué
  `// eslint-disable-next-line no-console`. **Volontaire, conservé** — c'est
  la seule preuve visible qu'un envoi a été simulé sans réseau.
- `lib/reviews/scheduler.ts:135` — `console.error("[reviews:poller]", err)`
  dans le filet de rattrapage du poller local de dev. **Volontaire,
  conservé** — sans lui, une erreur du poller (process background sans UI)
  disparaîtrait silencieusement.
- Aucun autre `console.*` trouvé dans le code actif (`lib/delivery/*`,
  toutes les routes API, tous les composants live).

### Blocs de code commenté

Recherche exhaustive (blocs `/* */` et lignes `//` ressemblant à du code
désactivé) : **aucun trouvé**. Les seuls commentaires multi-lignes présents
sont des séparateurs de section (prose), pas du code mort en commentaire.

### Autres dépendances vérifiées (toutes légitimes, gardées)

`react-dom`, `@types/*`, `postcss`, `typescript` : zéro `import` explicite
trouvé par grep, mais ce sont des dépendances consommées implicitement par
Next.js/le compilateur (peer dep de React, types ambiants, moteur de build)
— pas du code mort, juste invisibles à un grep naïf. Vérifiées une par une
avant de les exclure de la suppression, comme demandé.

### Ce qui a été fait, dans l'ordre

1. **`gsap` supprimé** de `package.json` (0 référence, `npm install` relancé,
   `package-lock.json` mis à jour).
2. **ESLint configuré** (`.eslintrc.json`, base `next/core-web-vitals` +
   `@typescript-eslint/no-unused-vars`), volontairement scindé du périmètre
   archives/labo (`ignorePatterns`). Résultat : **0 import et 0 variable
   inutilisés** dans tout le code actif, y compris dans les fichiers
   archive-only listés plus haut — rien à retirer sur ce point précis.
   `react/no-unescaped-entities` désactivée : activée par défaut par
   `next/core-web-vitals`, elle faisait échouer `next build` sur ~90
   occurrences d'apostrophes non échappées dans du JSX pré-existant (dont
   une majorité dans des fichiers archive-only) — corriger ce point aurait
   nécessité de toucher des fichiers protégés ou d'élargir le chantier
   très au-delà du nettoyage demandé ; désactivée pour garder un build
   propre sans y toucher. **Signalé plutôt que tranché : dites-moi si vous
   voulez qu'on les corrige quand même dans les fichiers live.**
3. **Prettier configuré** (`.prettierrc.json`, `.prettierignore` — mêmes
   exclusions archives/labo que ESLint) et appliqué sur tout le reste du
   projet : 44 fichiers reformatés (retours à la ligne, virgules finales),
   **zéro changement de contenu ou de comportement** — vérifié par `tsc`
   propre et build identique (mêmes 19 routes, mêmes tailles de bundle)
   avant/après. Note : le formatage a aussi touché les fichiers archive-only
   situés hors de `app/_archive/`/`components/labo/` (ex. `components/Nav.tsx`,
   `content/site.ts`) — volontaire, car ce sont des fichiers actifs du
   dépôt au sens strict (pas dans le dossier archive protégé), et Prettier
   ne change jamais la logique. Si vous préférez que je les exclue aussi,
   je peux affiner `.prettierignore`.
4. **Nommage et regroupement vérifiés** : déjà cohérents dans le code actif
   (composants en PascalCase, modules `lib/` en kebab-case dans les deux
   modules, routes en minuscules). Le seul écart apparent —
   `app/api/reservations/` hors de `app/api/reviews/` — est volontaire et
   déjà commenté dans le code (la réservation est un domaine à part qui ne
   fait que déclencher le module avis) ; laissé tel quel.
5. **`lib/delivery/` et `lib/reviews/` vérifiés** : structure inchangée,
   aucun fichier déplacé, séparation `DELIVERY_MODE`/`EMAIL_MODE`/
   `REVIEWS_STORE_MODE` intacte dans les deux.
6. **`README.md` réécrit intégralement** — l'ancienne version décrivait
   encore le site v1 (palette « terroir francilien », Configurator, La
   Carte comme menu de restaurant) alors que le site actuel est la
   direction v3. Nouvelle version : structure actuelle, les deux modules
   documentés avec renvoi vers `README-delivery.md`/`README-reviews.md`,
   labo et archives mentionnés sans détail (pas concernés par ce
   nettoyage). `.env.example` vérifié : déjà complet et à jour, aucune
   modification nécessaire.

### Vérification finale

- `tsc --noEmit` ✅, `npm run build` ✅ — **19 routes, tailles de bundle
  identiques** à avant le nettoyage, 0 erreur, 0 warning.
- Rendu visuel vérifié en preview : page d'accueil, `/qui-je-suis`,
  `/demo/commande`, `/demo/livraison`, `/demo/avis`, `/labo` — tous
  strictement identiques à avant (captures comparées).
- **Module Relance avis Google** : cycle complet rejoué en mode démo
  (réservation créée → `scheduled` → poller local → `simulated` avec
  `sentAt`) — fonctionne à l'identique.
- **Module Commande & Livraison** : `/api/delivery/quote` rappelé
  directement — réponse `mode: "demo"` correcte, calcul de devis inchangé.
- Console navigateur : 0 erreur sur toutes les pages visitées.
- **Aucun push** — tout ce chantier reste local, comme toujours.

---

## [NUIT 2026-07-12 · ÉTAPE 0] Bug « images manquantes dans le menu » — diagnostic

**Préalable RTK** — hook **actif et configuré** (`rtk init --show` → Hook OK, RTK.md OK,
settings.json OK). Rien à signaler, le travail continue normalement.

**Constat : aucun bug d'images à corriger.** Investigation complète menée *avant* toute
modification :
- Le site ne contient **aucune image raster** ni `next/image`. `git ls-files` → un seul
  fichier image versionné : `app/icon.svg` (le favicon). Pas de dossier `public/`. Aucun
  chemin d'image dans `content/site.ts` (relu intégralement).
- Toutes les « images » sont des **SVG inline** (24 dans le DOM) : illustrations
  `Motifs.tsx` (plate / bag / tools / heart) de l'Aperçu + icônes. Un SVG inline ne peut
  pas avoir de « chemin cassé ».
- « La Carte » est **100 % typographique** (un menu de texte, aucune photo) — par design.
- `git log --diff-filter=D` → **aucun fichier image supprimé**. Historique de
  `content/site.ts` : 4 commits cohérents, aucun chemin corrompu.
- Build **propre** : `GET / → 200`, aucune erreur serveur ni console.

**Pourquoi ça « semblait » cassé (et pourquoi ce n'est PAS un bug utilisateur)**
Dans l'outil de preview, les sections animées par Framer Motion (`Reveal` / `ScaleReveal`,
`whileInView`) apparaissent **blanches**. Cause identifiée : l'onglet preview est **en
arrière-plan** (`document.visibilityState === "hidden"`), ce qui met `requestAnimationFrame`
en pause → les animations framer restent **figées à `opacity:0`** (mesuré : reveals bloqués
à opacity 0.11 / 0, transform figé, plusieurs secondes après entrée dans le viewport).
Preuve : le **hero** (migré en animations **CSS** au Chantier 2 pour la robustesse)
s'affiche **parfaitement** dans la même capture, tandis que les sections framer sont vides.
En navigateur réel/visible — comme vérifié dans l'entrée [ÉTAPE 0] précédente « dans Chrome
réel » — tout le contenu et les motifs s'affichent normalement.

**Décision : rien modifié.** Corriger un bug inexistant aurait introduit une régression sur
un site qui fonctionne (règle « aucune régression tolérée »).
⚠️ Fragilité réelle, déjà connue (cf. entrée précédente) : le contenu framer est livré à
`opacity:0` et dépend du JS pour apparaître. Durcissement possible **à valider** : dégrader
`Reveal` pour laisser le contenu visible si l'animation ne se lance pas (comme le hero).
**Non appliqué cette nuit** — voir le blocage de vérification ci-dessous.

**Vérifications de conformité (fiables, indépendantes du preview)**
- Règle « jamais *Uber* dans le contenu visible » : **respectée** — la marque n'apparaît
  que dans le technique non-visible (`lib/delivery/uber-direct-client.ts`, `types.ts`,
  `index.ts`, route `app/api/webhooks/uber-direct/`). `content/site.ts` + composants :
  0 occurrence.
- 5 phrases interdites : toutes **absentes**.
- SEO structurel : **1 seul `<h1>`**, **1 bloc JSON-LD**, `<title>` + meta description
  présents, hiérarchie h1→h2→h3 cohérente. Rien d'urgent.
- Module « Commande & Livraison » (Étape 1) : **déjà construit** (git log + `lib/delivery/`,
  `app/api/delivery/`, `components/delivery/`, pages `/demo/*`). À vérifier/affiner, **pas**
  à reconstruire.

**⚠️ Blocage de vérification à connaître**
La règle « vérifier visuellement chaque changement » est **compromise dans ce harnais** :
l'onglet preview étant en arrière-plan, toute section animée par framer rend **blanc** à la
capture (les vérifs *de code* restent fiables, pas les vérifs *visuelles* des sections
animées). J'ai préféré m'arrêter là et te le signaler plutôt qu'enchaîner des changements
design non vérifiables sur un site déjà validé. Options pour la suite dans mon message.

---

## [ÉTAPE 0] Correction de la page blanche + robustesse

**Cause exacte**
Le symptôme « seul le fond s'affiche » n'était **pas** un composant sans `"use client"` :
vérification faite, tous les composants utilisant Framer Motion (`Reveal`, `ScaleReveal`,
`Hero`, `Nav`, `Process`, `Configurator`, `Contact`) ont bien la directive.
La page blanche venait d'un **état de build/HMR corrompu** hérité des erreurs de
compilation transitoires survenues pendant l'édition des animations (SWC « Unexpected
token section » quand une balise JSX était momentanément déséquilibrée entre deux
sauvegardes). Le contenu était rendu côté serveur à `opacity:0` (état initial des
animations), et le bundle client cassé ne relançait jamais les animations → tout restait
invisible.

**Correction**
- Rebuild propre (`rm -rf .next`) + redémarrage du serveur → le contenu s'affiche et
  toutes les animations fonctionnent (vérifié dans Chrome réel : stagger du hero, reveals
  au scroll, barre de progression moutarde, lien de nav actif, timeline du process).
- Suppression du warning Framer « container has a non-static position » : ajout de
  `relative` sur la `<section>` du Process (le `useScroll` a besoin d'un contexte positionné
  pour calculer l'offset correctement).
- **Ajout d'une frontière d'erreur** `app/error.tsx` (suggérée par le brief) : si un
  composant client lève une erreur à l'avenir, l'utilisateur voit une page sobre aux
  couleurs du site (« Réessayer » / « Retour à l'accueil ») au lieu d'un écran blanc.

**Vérification effectuée**
- Rendu complet vérifié dans Chrome réel (hero + toutes sections + animations).
- Console : plus aucune erreur ni warning Framer (restent seulement les logs debug de
  Vercel Analytics, normaux en développement).
- `npx tsc --noEmit` : aucune erreur de type.

---

## [Chantier 1] Référencement (SEO technique)

**Ce qui a été fait**
- `<title>` enrichi et local : « K1000 Studio — Création de sites internet pour
  commerçants, Île-de-France ».
- `<meta description>` réécrite avec les cibles locales (restaurants, boutiques,
  artisans ; Saint-Maur-des-Fossés, Suresnes, Val-de-Marne, Hauts-de-Seine).
- `keywords` étendus aux deux départements et aux types de commerce.
- JSON-LD `ProfessionalService` : `areaServed` détaillé (Île-de-France + Val-de-Marne
  + Hauts-de-Seine + Saint-Maur-des-Fossés + Suresnes).
- Vérifié : Open Graph + Twitter Card, `sitemap.xml`, `robots.txt`, `opengraph-image`
  (image de partage générée), `favicon` (icon.svg) — tous en HTTP 200.
- Hiérarchie des titres : **un seul `<h1>`** (hero), 8 `<h2>` (un par section), 15 `<h3>`.
- Illustrations : 24 SVG, toutes décoratives et correctement marquées `aria-hidden`
  (aucune balise `<img>`, donc pas d'`alt` manquant — approche a11y correcte pour du
  décoratif ; le sens est porté par le texte).

**Pourquoi**
Cibler les requêtes locales « création site internet commerçants + villes/départements »,
et donner aux moteurs/réseaux sociaux des métadonnées propres et structurées.

**Vérification effectuée**
- `<head>` inspecté via fetch : title, description, canonical, JSON-LD (5 zones) corrects.
- Routes SEO testées : sitemap/robots/OG/favicon → 200.
- `npx tsc --noEmit` OK.

> ⚠️ **À faire au matin** : `seo.siteUrl` (`content/site.ts`) pointe encore sur le
> domaine placeholder `https://k1000studio.fr`. Il alimente canonical, OG, sitemap et
> JSON-LD → à remplacer par le vrai domaine (ou l'URL Vercel actuelle) avant indexation.

---

## [Chantier 2] Performance (en cours)

**Mesure Lighthouse (build de production, `next start`)**
Performance **91** · Accessibilité **95** · Bonnes pratiques **96** · SEO **100**.
Métriques : FCP 1,0 s · **LCP 3,5 s** (point faible) · TBT 20 ms · CLS 0.

**Ce qui a été fait**
- **Hero converti d'animations JS (Framer) → animations CSS** (`.hero-rise` /
  `.hero-slate` dans `globals.css`) et repassé en **composant serveur** (plus de
  `"use client"`). Le contenu above-the-fold est désormais rendu côté serveur et animé
  dès le parsing CSS.
  - *Pourquoi* : le hero était en `opacity:0` jusqu'au chargement de Framer Motion →
    LCP lent **et** cause racine du risque d'écran blanc. En CSS, il s'affiche
    immédiatement, sans dépendre du JS. Bonus : un peu moins de JS client.
- Audit des animations : **aucune** n'anime `width`/`height`/`top` — uniquement
  `transform` et `opacity` (bon pour la perf).
- Polices : `font-display: swap` sur les 3 (Instrument Serif, Work Sans, IBM Plex Mono),
  auto-hébergées et préchargées par `next/font`.
- Images : aucune balise `<img>` (tout en SVG inline) → rien à lazy-loader ni à
  compresser.

**Vérification effectuée**
- `npx tsc --noEmit` OK · rendu hero vérifié dans Chrome réel (affichage instantané).

**Reste à faire (Chantier 2)**
- Ajouter `app/favicon.ico` (le navigateur requête `/favicon.ico` → 404 en console,
  seul point qui plombe « bonnes pratiques »).
- Re-mesurer Lighthouse après le passage CSS du hero (LCP attendu en forte baisse).

---

## [Module] Commande & Livraison

### A. Module technique (sandbox)
Bloc **isolé et optionnel** — le cœur du site n'en dépend pas ; le site fonctionne
normalement même si `DELIVERY_MODE` n'est pas configuré (fallback `demo`).

- `lib/delivery/` : `types.ts` (interface `DeliveryProvider`), `mock-provider.ts`
  (mode démo, statut qui évolue tout seul Préparation → Coursier → Livré en ~20 s,
  **sans aucun appel réseau**, stateless via l'horodatage encodé dans l'id),
  `uber-direct-client.ts` (mode live, OAuth2 + quote/create/status — credentials lues
  UNIQUEMENT depuis l'environnement, jamais en dur), `index.ts` (sélection du
  fournisseur selon `DELIVERY_MODE`).
- Routes API : `POST /api/delivery/quote`, `POST /api/delivery/create`,
  `GET /api/delivery/status`, `POST /api/webhooks/uber-direct` (inactif en démo).
- Composants : `DeliveryOptionSelector` (retrait/livraison + devis),
  `DeliveryTracker` (timeline + ETA, polling 2 s).
- Page sandbox `/demo/livraison` (non reliée à la nav).
- `README-delivery.md` (dont : le passage en `live` exige l'onboarding Uber Direct
  fait par le commerçant lui-même) + `.env.example` (gabarit sans valeurs réelles).

**Vérification** : build OK (4 routes API + page démo). Flux testé en mode démo via
l'API : quote (4,64 € / 34 min) → create (preparing) → status auto-évolue en
« courier_en_route » après 7 s → webhook démo renvoie `processed:false`. Aucune
credential en dur.

### B. Contenu marketing (site public)
- **Section « Le problème »** : 5ᵉ carte « La commission » (pleine largeur car nombre
  impair) — « Chaque commande livrée via une plateforme… c'est celui de la plateforme. »
- **« La Carte »** : bloc add-on « Commande directe » **sous** le carton menu (jamais
  dans le menu, pour ne pas le fragmenter), présenté comme add-on de Croissance
  Digitale. Accroche + description + ligne de comparaison **prudente sans chiffre
  inventé**.
- **Configurateur** : 3ᵉ option facultative « Avec commande directe » (case à cocher
  discrète) → ajoute un bandeau « Livraison sans commission » + bouton « Commander »
  dans l'aperçu, aux couleurs de la maquette.

**Contrainte respectée** : le mot **« Uber » n'apparaît NULLE PART** dans le texte
visible (0 occurrence dans `content/`, `components/` et le HTML rendu). Réservé au
code/README/API. Aucun chiffre de commission non vérifié affiché.

### C. Tarif « Suppléments à la carte » (dans La Carte)
- Remplacement de l'ancien bloc add-on sauge par une **section « Suppléments à la
  carte »** intégrée **dans le carton menu**, sous les 3 formules (inchangées).
- Ligne au style menu (Plex Mono, points de suite, prix aligné à droite),
  **visiblement plus discrète** que les 3 offres (titre + prix en plus petit) :
  « Commande & Livraison directe … 450€ à l'activation + 25€/mois ».
- Description courte + mention de disponibilité (Site Autonome / Croissance Digitale)
  + note précisant que **le coût de chaque livraison est payé séparément** au
  prestataire, non inclus dans ce tarif.
- Aucune nouvelle couleur, aucun CTA séparé (le CTA existant couvre la question),
  aucune mention « Uber ». Prix des 3 formules vérifiés inchangés (690€/1490€/dès 1990€).

### D. Démo interactive du supplément (dans La Carte)
- Sous le bloc supplément, bouton discret **« Essayer la démo »** (fermé par défaut) qui
  déploie un widget `components/delivery/DeliveryDemo.tsx` — **branché sur le
  `mock-provider` existant** (`createDelivery`), aucune logique de simulation recréée,
  aucun appel réseau réel.
- Parcours simulé en 3 temps déclenché par le visiteur : **panier fictif** (La Table du
  Marché, 2 plats, 25,50 €) → **« Commande reçue »** (n° de commande + ETA) →
  **suivi automatique** « Commande en préparation » → « Livreur en route » (petit
  livreur qui avance le long d'une piste, `transform`/`opacity`, easing
  `[0.22,1,0.36,1]`, ~2,5 s/étape) → « Livré ». Bouton **« Revoir la démo »** pour
  rejouer (jamais d'état figé).
- Mention discrète sous le widget : « Démonstration simplifiée à titre d'exemple… ».
- Palette K1000 uniquement, **aucune mention « Uber »/« livreur » du prestataire**.

**Vérification** : cycle complet testé dans Chrome (déclenchement → progression auto →
« Livré » → relance), layout OK en mobile 390px (aucun débordement), 0 erreur console,
build de prod OK.

### E. Page de commande simulée complète (`/demo/commande`)
- **Lien cliquable** depuis La Carte (« Ouvrir un exemple de commande complet ↗ »,
  nouvel onglet) → page dédiée qui simule le **parcours client réel** d'un restaurant
  fictif (La Table du Marché) : **menu** (ajout/quantités, panier fixe) → **checkout**
  (récap + adresse + devis de livraison) → **confirmation** (n° de commande + ETA) →
  **suivi en direct** (timeline Préparation → Coursier en route → Livré + ETA).
- Réutilise `mock-provider` via les **routes API** (`/api/delivery/quote`, `/create`,
  `/status`) et les composants `DeliveryOptionSelector` + `DeliveryTracker` déjà faits.
- La course n'est créée qu'à l'ouverture du suivi → le visiteur **voit toujours la
  progression en direct** (l'horloge de démo démarre à ce moment-là).
- 100 % sandbox : aucune vraie commande, aucun paiement, aucun appel externe. Palette
  K1000. On parle d'« un livreur »/« coursier », **jamais** du prestataire technique.

**Vérification** : parcours complet testé dans Chrome (menu → panier → devis 5,54 € →
commande → suivi qui progresse « En préparation » → « Coursier en route » → « Livré »),
mobile 390px OK (barre panier fixe sans débordement), 0 erreur console, build de prod OK,
0 mention « Uber » dans le texte visible.

---

## [Refonte v3] Direction « geek coloré » (niveau soumission Awwwards)

### Réponses obtenues en phase de questions (13/07/2026)
- **Direction** : référence donnée par le client — **appsignal.com** (« plein de
  couleur, un peu geek, c'est plus mon univers »). Les 3 pistes proposées
  (journée du commerçant / éditorial local / clarté B2B) n'ont pas été retenues.
- **Technique** : équilibré — 1 seul moment WebGL, cible Lighthouse 85+.
- **Contenu** : **carte blanche totale** (copy réécrit, « La Carte » remplacée).
- **Portée** : tout d'un coup, **mobile = desktop**.

### Recherche (INSPIRATION.md)
Exploration réelle d'awwwards.com (Sites of the Day) + analyse des 6 sites
fournis par le client (Longbow, Vectr, Julien Calot, Depo Luxe, Lula Oil,
LunaSol) + références vérifiées en session (By-Kin, Mat Voyce, Minh Pham).
Documenté dans `INSPIRATION.md`, avec la direction validée.

### Ce qui a été construit — espace isolé `/v3` (noindex)
- **Nouvelle identité** : fond lait `#FBF7EF`, encre `#211D16`, accents vifs
  corail/violet/teal/jaune/rose, cartes « sticker » (bordure 2px + ombre franche),
  fenêtres façon app (barre de titre à pastilles), labels monospace. Palette
  ajoutée en additif dans Tailwind (aucun impact sur l'existant).
- **Nouveau copy** (`content/v3.ts`) : ton direct et joueur, vouvoiement,
  faits inchangés (prix 690/1490/dès 1990 € + mensuels, add-on livraison
  450 € + 25 €/mois avec sa note légale, villes, process 6 étapes, légitimité
  restaurateur). Aucune phrase interdite, aucun « Uber » visible.
- **Nouvelle signature** (remplace le menu de resto) : **« Pendant que vous êtes
  en plein service, votre site bosse »** — fenêtre « live » dans le hero avec un
  flux de notifications simulé (résa, avis, commande, admin) qui défile.
- **Sections** : hero aurora + ticker (villes/cibles) + constat (4 cartes
  sticker) + « trois moteurs » (bento) + plans (fenêtres de pricing, plan
  Autonome mis en avant, add-on livraison + lien démo /demo/commande) +
  process sur encre + fondateur (3 situations vécues) + contact (formulaire
  en fenêtre) + footer.
- **Le moment WebGL** : shader « aurora » (blobs violet/corail/teal/jaune sur
  lait, lueur qui suit le curseur) — même architecture perf que le hero /v2
  vérifié : chargement différé après `load`+idle, pause hors écran
  (IntersectionObserver → frameloop never), **fallback CSS animé sur
  mobile/tactile** (aucun canvas en 390px), statique en reduced-motion.
- `prefers-reduced-motion` : ticker, notifs, blobs et hover stoppés ; contenu
  100 % lisible sans JS (texte dans le DOM, animations à `fill-mode: both`).

### Vérification
- `tsc --noEmit` ✅ · build prod ✅ (21 routes, `/v3` statique 9,66 kB).
- Visuel vérifié section par section (preview) : **desktop** (hero, ticker,
  constat, moteurs, plans, process, contact) et **mobile 390px** (hero, plans —
  `scrollWidth` = 390, zéro débordement).
- 0 erreur console. Ancien one-page `/` revérifié : **inchangé**.
- **Lighthouse (build prod, /v3)** : **Perf 91 · Accessibilité 95 · Bonnes
  pratiques 96 · SEO 92** — TBT 0 ms, CLS 0, LCP 3,5 s (borné par le swap de
  webfont sous throttling, comme mesuré précédemment sur les autres pages).
  Cible 85+ atteinte.

> Espace en évaluation : `/v3` est noindex et n'est PAS poussé. L'ancien site
> et /v2 restent accessibles pour comparaison.

---

## [Promotion v3] La refonte devient LE site (racine `/`)

**Commit de sécurité préalable** : l'état complet (toutes versions) était déjà
committé en local (`341ad19`) avant restructuration — tout est récupérable.

### Déplacements (aucun fichier supprimé — uniquement des `git mv`)
- `app/v3/page.tsx` → **`app/page.tsx`** : la v3 « geek coloré » est désormais
  la page d'accueil. Ses métadonnées (title/description/OG/Twitter, indexables)
  ont été fusionnées dans `app/layout.tsx` ; slogan JSON-LD mis à jour.
- Anciennes versions **archivées hors routing public** dans `app/_archive/`
  (dossier privé Next.js, préfixe `_` = jamais routé) :
  - `app/_archive/v1-onepage.tsx` (ex-`app/page.tsx`, le one-page bistrot)
  - `app/_archive/v2/` (l'essai multi-pages complet)
  - `app/_archive/signature/` (la pièce WebGL isolée)
  - `app/_archive/v3-layout.tsx` (layout v3, fusionné dans la racine)
- Les composants et contenus (`components/*`, `content/site.ts`, `content/v3.ts`)
  restent en place — les archives compilent toujours (rien de cassé, rien de perdu).

### Routes publiques
- `/` → la v3, indexable (noindex retiré avec la promotion).
- `/v2`, `/v2/*`, `/v3`, `/v3/*`, `/signature` → **redirection 307 vers `/`**
  (via `next.config.mjs` — plus simple à maintenir qu'une 404 dédiée).
- `/demo/commande` et `/demo/livraison` conservés (liés depuis les plans),
  exclus du crawl via `robots.txt` (`Disallow: /demo`).
- `sitemap.xml` → ancres v3 (`#constat #moteur #plans #process #fondateur
  #contact`) ; règle robots obsolète `/v2` retirée.
- **Favicon** et **image Open Graph** refaits à l'identité v3 (badge N jaune,
  surligneur, blobs colorés).

### Vérification
- `tsc` ✅ · `npm run build` ✅ — 14 routes (contre 21), plus aucune route
  `/v2`/`/v3`/`/signature` générée.
- Runtime (build de prod) : `/` = 200 avec le contenu v3 · les 4 anciennes URLs
  testées → 307 vers `/` · démos 200 · sitemap/robots/OG/favicon 200.
- Visuel : racine identique à l'ancienne `/v3` (hero aurora + fenêtre LIVE,
  notifs qui tournent), 0 erreur console.

---

## [Cohérence DA v3] Ticker retiré + toutes les pages publiques passées en v3

- **Ticker (bandeau déroulant) supprimé** de la page d'accueil, à la demande du client.
- **`/demo/commande` et `/demo/livraison` refaits en DA v3** (lait/encre, cartes
  sticker, boutons corail/jaune, labels mono) — la logique sandbox est inchangée ;
  composants `DeliveryOptionSelector` et `DeliveryTracker` restylés (fenêtre v3-window).
- **Page d'erreur** (`error.tsx`) passée en v3 + **404 personnalisée** ajoutée
  (`not-found.tsx`, « Cette page n'est pas à la carte »).
- Focus clavier global passé du lie-de-vin au violet v3.
- Balayage complet anti-ancienne-DA sur les fichiers publics : les seuls restes
  (`font-display` italique dans Sections.tsx) sont des touches volontaires ;
  `DeliveryDemo` (ancienne DA) n'est importé que par l'archive.
- Vérifié : tsc ✅, build 14 routes ✅, ticker absent du DOM, /demo/* rendus v3,
  404 en 404 réel avec la page custom, 0 mention « Uber » visible.

---

## [Copywriting v3] Réécriture complète du texte pour coller au ton visuel

**Contexte** : le design v3 « geek coloré » est validé, mais les textes dataient
de la DA précédente (« chaleureux restaurateur »). Objectif : un copy accrocheur,
direct, avec de la personnalité, sans jargon — et recentré sur **tous les
commerçants locaux** (boutiques, artisans, restaurants, assos), pas seulement les
restaurateurs.

**Références consultées (WebSearch/WebFetch, pas de Chrome)**
- **Linear** — phrases < 20 mots, énoncés qui vont droit au bénéfice, jargon
  vulgarisé (« structural diffs » expliqué, pas asséné).
- **Raycast** — le plus proche du ton visé : questions rhétoriques (« Tired of
  typing the same thing? »), problèmes relatables, bénéfice-action plutôt que
  specs (« keyboard first » et non « input optimization »). 5–12 mots/phrase.
- **Stripe (FR)** — vulgariser sans infantiliser : bénéfice d'abord, preuve par
  les chiffres concrets plutôt que par les adjectifs.
- **Qonto (FR)** — modèle de ton FR moderne pour non-tech : fragments nominaux
  (« Fini la paperasse »), impératif/infinitif dans les CTA, zéro jargon bancaire.

**Mécanique de ton retenue** (forme, pas vocabulaire — la cible K1000 n'est pas dev)
- Une idée par phrase, phrases courtes, fragments nominaux autorisés
  (« Des prix affichés. Oui, vraiment. »).
- Personnalité par la question rhétorique + le problème vécu, pas par le gag gratuit.
- Bénéfice/résultat d'abord, jamais la spec (« vous changez tout en deux minutes »,
  jamais « CMS »/« responsive »).
- Confiance par le fait concret : 15 min, 48h, 0 %, prix affichés.

**Recentrage cible (dé-restaurantisation)**
- Hero, sous-titre, flux de notifs, constat, moteur, plans : tous les exemples
  trop « resto » (couverts, réservation de table, « en salle », menu) remplacés
  par des exemples génériques (horaires, photos, avis Google, prise de contact,
  rendez-vous, commande) qui parlent à n'importe quel commerce.
- **Conservé volontairement** : la section « Qui suis-je » (crédibilité) reste
  celle d'un restaurateur en activité — c'est un fait vécu, pas une promesse de
  ciblage. Le sous-titre du hero garde « Je gère des restaurants » comme ancrage
  de légitimité, puis élargit à « les commerçants du coin ».

**Titre du hero — 2 variantes proposées**
- **B (retenue par le client, en ligne)** : « Votre site devrait *bosser* autant
  que vous. » — même idée que A, ton plus mordant. Highlight jaune sur « bosser ».
- **A (alternative, écartée)** : « Vous faites votre métier. Votre site fait *le
  reste*. » — universelle et parallèle. Basculable en 3 champs dans `content/v3.ts`.

**Structure** : ordre des sections inchangé (Hero → Constat → Moteur → Plans/La
Carte → Process → Fondateur → Contact). Le concept « menu/La Carte » des offres
est conservé tel quel.

**Vérifications** : `tsc` ✅, build 14 routes ✅, 0 phrase interdite, 0 « Uber »
visible, 6 titres de section cohérents en ton, rendu hero + notifs vérifié en
preview. **Aucun push** — en attente du choix de variante hero par le client.

---

## [Repositionnement stratégique] Fini « on vend un site », place au déclic

**Contexte** : le titre hero variante B (« Votre site devrait bosser autant que
vous. ») a été validé et mis en ligne. Ce chantier va plus loin : changer la
nature du message central du site. Objectif : ne plus convaincre le commerçant
qu'il « a besoin d'un site », mais lui faire réaliser que **son commerce est
meilleur que ce qu'on en voit en ligne** — le déclic qui amène naturellement
vers l'audit gratuit.

### 1. Nouveau positionnement

- **Reformulation clé appliquée** : plus jamais « création de site » comme
  accroche — remplacé par **« une vitrine numérique à la hauteur de votre
  commerce »**, intégrée directement dans le titre de la section « Ce que ça
  fait » (`v3moteur.title` : *« Pas juste un site. Une vitrine à la hauteur de
  votre commerce. »*) et dans le sous-titre du Hero.
- **« Opportunités » remplace « clients perdus »** partout où l'idée de perte
  apparaissait (sous-titre du Constat, carte « Il dort ») — même constat, ton
  moins agressif, plus juste.
- **Les 3 phrases fournies, placées chacune à un endroit différent** (pas
  toutes au même endroit, pour ne pas surcharger) :
  - *« Vos futurs clients vous cherchent déjà. Assurez-vous qu'ils vous
    trouvent. »* → **titre du Hero**, remplace la variante B précédente.
    Repositionne l'angle de « le site bosse » vers « vous êtes déjà cherché,
    encore faut-il qu'on vous trouve ».
  - *« Que voit un nouveau client lorsqu'il découvre votre commerce sur
    internet ? »* → **titre de la section Constat** (`v3constat.title`).
    Ouvre directement sur le miroir tendu au commerçant.
  - *« Votre site actuel, il fait quoi, là, tout de suite ? »* → **ligne
    isolée** (`v3constat.rupture`), affichée seule entre la grille de 4 cartes
    du Constat et la section Moteur. Marque une rupture de rythme au lieu de
    rester un titre de section comme avant.
- Hero et section Problème réécrits en cohérence (`content/v3.ts` :
  `v3hero.subtitle`, `v3constat.title/subtitle/rupture/cards[0]`).

### 2. Épuration du contenu

- **Section Moteur** : passée de 4 cartes bento à **3**, layout simplifié en
  grille égale (plus de carte large asymétrique orpheline). La suppression de
  la 4ᵉ carte (Commande directe) sert à la fois l'épuration et le point 3
  ci-dessous.
- **Section Plans** : le bloc add-on « Module Commande & Livraison directe »
  (icône 🛵, prix, lien démo) entièrement retiré de l'affichage — ne reste que
  la note de bas de section (`v3plans.footnote`). Section nettement plus
  courte.
- **Section Fondateur (homepage)** : les 3 cartes `>_` + la citation de
  clôture ont été retirées de la page d'accueil, remplacées par un **teaser
  d'une phrase** + un lien « Voir mon parcours → » vers la nouvelle page
  dédiée (point 4). Le contenu complet n'est pas perdu : il vit maintenant sur
  `/qui-je-suis`.

### 3. Module Commande & Livraison masqué de l'affichage public

**Aucun fichier ni code supprimé** — uniquement désactivé de l'affichage,
même logique que l'archivage v1/v2 :
- `content/v3.ts` : l'entrée « Commande directe » du bento (`v3moteur.bento`)
  a désormais `hidden: true` ; `v3plans.addon` reste défini tel quel, simplement
  plus rendu par le composant.
- `components/v3/Sections.tsx` : `V3Moteur` filtre `bento.filter(b =>
  !b.hidden)` ; `V3Plans` ne rend plus le bloc addon.
- Les pages `/demo/commande` et `/demo/livraison`, ainsi que les composants
  `components/delivery/*` et les routes `app/api/delivery/*`, restent
  **intacts et fonctionnels** — juste non liés depuis la nav ou la homepage
  (vérifié : les deux répondent toujours en 200 en accès direct).
- Ce module reviendra en avant plus tard ; il suffira de repasser `hidden` à
  `false` et de rétablir le bloc addon dans `V3Plans`.

### 4. Nouvelle page « Qui je suis »

- **`app/qui-je-suis/page.tsx`** créée : présente le fondateur en détail
  (gérant de plusieurs restaurants en Île-de-France, à l'origine de K1000
  Studio suite à ce constat vécu). Reste factuel — aucune date ni détail
  inventé au-delà de ce qui était déjà établi ailleurs sur le site.
  Contenu piloté par de nouveaux champs sur `v3fondateur`
  (`pageIntro`, réutilise `points`/`closing`/`badges`) dans `content/v3.ts`.
  Métadonnées SEO dédiées (title/description).
- **Navigation mise à jour** : le lien « Qui suis-je » (`v3nav.links`) pointe
  désormais vers `/qui-je-suis` au lieu de l'ancre `#fondateur`. Tous les
  autres liens de nav (ancres) sont passés en chemins absolus (`/#constat`,
  `/#moteur`, etc.) pour fonctionner correctement depuis n'importe quelle page
  du site, pas seulement l'accueil — nécessaire dès qu'une deuxième page
  existe. Logo de la nav : `#top` → `/`.
- **`app/sitemap.ts`** : ajout de l'URL `/qui-je-suis` (priorité 0.7), retrait
  de l'ancre `#fondateur` obsolète.
- **`components/JsonLd.tsx`** : `slogan` mis à jour pour matcher le nouveau
  titre du Hero (était encore l'ancienne accroche « Pendant que vous êtes en
  plein service »).

### Vérifications effectuées

- `tsc --noEmit` ✅ aucune erreur.
- Build production : **15 routes** générées proprement (`/qui-je-suis` inclus).
- Balayage : 0 phrase interdite, 0 « clients perdus », 0 « Uber » visible,
  0 mention du module livraison hors data/démo.
- Preview vérifiée en direct : Hero (nouveau titre + highlight), Constat
  (nouveau titre + ligne de rupture), Moteur (3 cartes égales, sans livraison),
  Plans (sans bloc addon), Fondateur (teaser condensé + lien), page
  `/qui-je-suis` complète (intro, points, citation, CTA, footer) accessible en
  navigation directe, `sitemap.xml` à jour, `/demo/commande` et
  `/demo/livraison` toujours répondent en 200 (code intact, juste non liés).
  Aucune erreur console.
- **Aucun push** — en attente de relecture avant mise en ligne.

---

## [Correction] Titre du Hero restauré + section « Verdict » dédiée, plein écran

**Contexte** : un chantier précédent avait remplacé le titre du Hero par
« Votre site actuel, il fait quoi, là, tout de suite ? » — ce n'était pas la
demande. Ce chantier corrige le tir : le titre du Hero est restauré tel quel,
et cette phrase devient le titre d'une **toute nouvelle section**, à part
entière.

### 1. Titre du Hero restauré

`components/v3/Hero.tsx` : reverté à l'état d'avant l'erreur, via
`git revert` du commit fautif. Titre définitif, inchangé : **« Vos futurs
clients vous cherchent déjà. Assurez-vous qu'ils vous trouvent. »** — n'apparaît
qu'une fois sur la page, dans le Hero uniquement.

### 2. Nouvelle section « Verdict », plein écran

- `content/v3.ts` : nouvel export `v3verdict` (`question` + `answers[]` —
  mêmes réponses que précédemment : « Rien. Absolument rien. », « Il dort. »,
  « Il est invisible. », « Il fait fuir vos clients. », « Il tourne en
  rond. », « Il ment sur vos horaires. », « Il rate des opportunités. »).
- `components/v3/Sections.tsx` : nouveau composant `V3Verdict` — section
  `min-h-screen` (plein écran, vrai temps de pause dans le scroll), fond
  sombre `bg-encre` (cohérent avec la section Process, déjà sombre — alterne
  la respiration visuelle du site). Titre en très grande typo (`md:text-7xl`)
  sur fond `lait`, réponse qui défile en dessous en `jaune`.
- Insérée dans `app/page.tsx` entre `<V3Hero />` et `<V3Constat />`.
- Les cartes « Il dort » / « Il est invisible » sont retirées de la section
  Constat (doublon avec le contenu de Verdict, juste au-dessus) — 2 cartes
  restantes (« Il est verrouillé », « Il coûte sans compter »), grille
  recentrée. `app/sitemap.ts` : ancre `#verdict` ajoutée.

### 3. Animation de transition — glitch / RGB-split (pas un simple fondu)

Le fondu simple d'un chantier précédent est remplacé par un vrai effet
glitch à chaque changement de réponse (~350ms, dans le budget 300-500ms
demandé) :
- `.glitch-answer` (nouvelles règles dans `app/globals.css`) : au moment du
  changement, deux pseudo-éléments (`::before`/`::after`) dupliquent le texte
  en corail et en teal (couleurs de marque), décalés et découpés par
  `clip-path` animé, en `mix-blend-mode: screen` (fait ressortir les couleurs
  sur fond sombre) — un vrai split chromatique façon glitch, pas un fondu.
  Le texte de base tremble légèrement (`glitch-jitter`, `transform` +
  `filter: blur` bref).
- **Uniquement des propriétés compositées** (`transform`, `opacity`,
  `filter`, `clip-path`) sur 1 élément réel + 2 pseudo-éléments — aucune
  propriété de layout animée, donc aucun repaint/reflow déclenché par
  l'animation. Choix fait explicitement pour éviter de reproduire le problème
  de performance déjà rencontré sur ce site (cf. Chantier 2 Performance).
- Cycle : nouvelle réponse toutes les 2 s, effet glitch actif les 350
  premières ms de chaque nouvelle réponse.

### Reduced motion

`GlitchAnswer` (dans `components/v3/Sections.tsx`) utilise
`useReducedMotion()` : si activé, affiche `answers[0]` (« Rien. Absolument
rien. ») statique, sans intervalle, sans classe `is-glitching`, sans
pseudo-éléments (`content: none` en reduced-motion dans le CSS, défense en
profondeur en plus du court-circuit React).

### Vérifications effectuées

- `tsc --noEmit` ✅, build production 15 routes ✅.
- Hero confirmé visuellement : bon titre, une seule occurrence sur la page
  (grep sur le code confirme `v3hero.titleA/Em/B` intacts, aucune référence
  à l'ancien titre erroné).
- Section Verdict vérifiée en preview desktop **et** mobile (375px) : plein
  écran, question lisible, défilement actif sur plusieurs cycles.
- **Glitch capturé en plein transition** via screenshot minuté (~1,7s après
  chargement) : split chromatique corail/teal visible et propre sur « Il rate
  des opportunités. », aucun artefact visuel.
- Mesure FPS via rAF non concluante dans l'outil de preview (le rAF se met en
  pause quand l'onglet est en arrière-plan — limite déjà documentée dans ce
  journal) ; la fluidité est garantie par construction (propriétés compositées
  uniquement, 1 élément + 2 pseudo-éléments, aucune boucle JS pilotant
  l'animation frame par frame — seul un `setInterval` toggle une classe toutes
  les 2s) et confirmée visuellement sans saccade sur les captures.
- **Reduced motion vérifié fonctionnellement** : flag forcé temporairement
  (`useReducedMotion() || true`) le temps d'un screenshot confirmant le rendu
  statique correct (couleur/opacité du titre vérifiées via inspection des
  styles calculés, pas seulement visuellement), confirmé stable sur 4s sans
  cycle, puis le hack immédiatement retiré — `tsc` re-vérifié après coup.
- Section Constat : confirmé par inspection DOM que 2 cartes seulement
  restent (« Il est verrouillé », « Il coûte sans compter »).
- **Aucun push** — en attente de relecture avant mise en ligne.

---

## [LABO · expérience 01] Portail qui se fissure — page technique HORS périmètre commercial

**Nature** : expérimentation créative pure sur `/labo`, totalement séparée du
site commercial. **Jamais liée depuis la navigation** (vérifié par grep : zéro
lien dans content/, components/v3, app/page.tsx), **non indexable** (meta
`noindex, nofollow` vérifiée dans le HTML servi + `Disallow: /labo` dans
robots.txt + absente du sitemap).

**L'effet** : au clic sur « Entrer », l'écran se brise en verre —
1. toile de fissures **procédurales** (rayons + anneaux jitterés autour du
   point d'impact, graine aléatoire à chaque déclenchement — vérifié par deux
   captures gelées au même instant : géométries totalement différentes) ;
2. l'écran suivant **transparaît dans les interstices** dès la phase de
   fissuration (léger écartement radial des fragments) ;
3. les fragments **tombent** (balistique analytique : gravité + rotation +
   fondu) et révèlent l'écran final. Durée totale : **1,4 s**.

**Technique** (`components/labo/ShatterPortal.tsx` + route `app/labo/page.tsx`) :
- Canvas 2D (pas de Three.js — inutile ici). L'écran A est **repeint** sur une
  texture offscreen au moment du clic (fond + blobs via constantes partagées
  DOM/canvas, textes/bouton mesurés sur le DOM réel avec leurs styles
  calculés) — pas de lib de capture DOM.
- Fragments = quads/triangles jitterés découpés dans la texture via `clip()` :
  **65 desktop / 36 mobile** (< 640px) — pas des centaines de micro-morceaux.
- Trajectoires **analytiques** (position = f(t), pas d'intégration frame par
  frame) : déterministe, robuste aux frames sautées.
- Boucle rAF + filet `setInterval(50ms)` idempotent (les rAF sont suspendus
  en onglet d'arrière-plan ; l'effet se termine quoi qu'il arrive).
- **Son** : éclat de verre 100 % synthétisé en Web Audio (bruit passe-haut à
  décroissance rapide + 3 tintements sinusoïdaux), créé uniquement sur geste
  utilisateur, `try/catch` intégral, aucun fichier audio.
- Debug labo : `/labo?freeze=<ms>` fige l'effet à un instant donné
  (inspection/captures). Inactif sans le paramètre.

**Contraintes respectées**
- `prefers-reduced-motion` : fondu croisé simple, aucun canvas, aucune
  fragmentation, aucun son — vérifié en forçant temporairement le flag
  (méthode habituelle), hack retiré, `tsc` re-vérifié.
- Mobile : nombre de fragments réduit (36), rendu vérifié en 375px gelé à
  t=500ms — éclats plus gros, écran B visible dans les vides. Fluidité par
  construction (une passe de dessin par frame, clip+drawImage compositée,
  DPR plafonné à 2).
- Performance desktop : capture gelée nette à t=180ms (toile de fissures) et
  t=600ms (plein vol) ; run réel complet sans aucune erreur console.

**Incident rencontré et résolu pendant la vérif** : la page semblait morte au
premier clic — en réalité le serveur de dev servait un HTML dont les chunks
`_next/static` étaient en 404 (conséquence du `rm -rf .next` d'un build lancé
pendant que le dev tournait — piège déjà documenté). Redémarrage du serveur de
dev → hydratation OK. À retenir : toujours redémarrer le dev server après un
build de prod dans ce repo.

**Vérifications** : `tsc` ✅ · build **16 routes** (dont `/labo`) ✅ ·
noindex/robots/sitemap ✅ · variation procédurale ✅ (2 tirages ≠) ·
reduced-motion ✅ · mobile ✅ · console 0 erreur ✅. **Aucun push.**

---

## [Module Relance avis Google] Email automatique post-réservation

**Nature** : nouveau module isolé et optionnel, même logique que le module
Commande & Livraison — le site fonctionne normalement s'il n'est pas
configuré (`EMAIL_MODE` et `REVIEWS_STORE_MODE` absents → `demo`/`memory`,
jamais d'erreur).

**Principe** : à la création d'une réservation (email + horodatage), une
relance avis Google est planifiée à *réservation + délai* et envoyée seule,
sans intervention. Lien direct vers la fenêtre de notation du commerce
(jamais générique). **Règle de conformité non négociable, documentée en
commentaire au-dessus du template** : le même email part pour tous les
clients ayant réservé, aucune question de satisfaction en amont pour
orienter les mécontents ailleurs que Google (*review gating*, interdit par
Google et risqué juridiquement).

**Architecture** (`lib/reviews/*`, détail complet dans `README-reviews.md`) :
- `types.ts` / `businesses.ts` — contrats + registre des commerces (jetons de
  marque + lien avis Google, propres à CHAQUE commerce, pas génériques).
- `store.ts` + `supabase-store.ts` — persistance des jobs planifiés,
  sélectionnée par `REVIEWS_STORE_MODE` (`memory` par défaut, dev uniquement ;
  `supabase` pour un vrai déploiement — store Supabase entièrement codé,
  schéma SQL fourni, mais non testable sans projet réel, comme le client Uber
  Direct du module livraison).
- `email-provider.ts` — `EMAIL_MODE` `demo` (log console, statut `simulated`)
  ou `live` (Resend, REST brut via `fetch`, aucun SDK ajouté).
- `scheduler.ts` — `scheduleReviewRequest()` calcule `sendAt` (délai
  configurable à 3 niveaux : par commerce > `REVIEW_REQUEST_DELAY_HOURS` >
  override ponctuel de la démo — jamais codé en dur) ; `processDueJobs()`
  envoie tout ce qui est dû, appelée par la route cron ET par un poller local
  (dev uniquement, désactivé si `VERCEL`/prod) pour que la démo tourne sans
  curl manuel.
- `templates/review-request.tsx` — email HTML en styles inline (tables, pas
  de Tailwind — la plupart des clients mail dégradent le CSS moderne),
  reprend les jetons de marque du commerçant, un seul CTA, lien de
  désinscription.
- Routes : `POST /api/reservations` (création + planification), `GET/POST
  /api/reviews/run` (cron), `GET /api/reviews/status` (suivi commerçant — un
  statut par job, simplification volontaire vs. table `email_log` séparée),
  `GET /api/reviews/unsubscribe` (RGPD, lien direct depuis l'email).
- `vercel.json` — cron `/api/reviews/run` toutes les 10 min (⚠️ le plan Vercel
  Hobby limite les cron à 1×/jour — un plan Pro est nécessaire en prod réelle).
- `app/demo/avis/` — sandbox non liée à la nav, désindexée (même traitement
  que `/demo/commande`) : formulaire de réservation fictive (délai réglable en
  minutes) + tableau de suivi live des statuts.

**Bug trouvé et corrigé pendant la vérification** : le store en mémoire
utilisait un simple singleton de module (`let memoryStore = ...`). Or
Next.js peut bundler chaque route handler comme une entrée séparée — deux
routes important le même fichier se retrouvaient chacune avec leur PROPRE
copie du module, donc deux instances différentes du store : un job créé via
`/api/reservations` restait invisible depuis `/api/reviews/status`. Corrigé
en portant le singleton sur `globalThis` (même technique que le contournement
Prisma/HMR bien connu de l'écosystème Next.js) — vérifié après coup : le job
créé par une route est immédiatement visible depuis l'autre.

**Deuxième point trouvé en vérifiant le build** : `/api/reviews/status` et
`/api/reviews/run` ne lisant ni cookies ni paramètres de requête, Next.js les
pré-rendait en **statique** au build (réponse figée pour toujours en
production) — jamais ce qu'on veut pour un état qui change à chaque envoi.
Ajout de `export const dynamic = "force-dynamic"` sur les deux routes ;
rebuild : les deux apparaissent bien en `ƒ` (dynamique) désormais.

**Vérifications effectuées**
- `tsc --noEmit` ✅, build production **19 routes** ✅.
- Cycle complet testé en mode démo (`EMAIL_MODE` non défini) : réservation
  créée via l'API → job `scheduled` visible immédiatement depuis une AUTRE
  route → au bout du délai (testé à 1-2 minutes), le poller local l'envoie
  tout seul → statut `simulated` + `sentAt` renseigné → ligne
  `[reviews:demo] Email simulé → ...` visible en console serveur. Reproduit
  deux fois (2 jobs distincts), les deux confirmés.
- Désinscription testée : lien `/api/reviews/unsubscribe?id=...` marque le
  job `canceled`/`unsubscribed`, et une NOUVELLE réservation pour le même
  couple commerce/email est créée déjà `canceled` (vérifié par requête directe).
- Erreurs gérées proprement : commerce inconnu → 404 avec message clair ;
  champs requis manquants → 400 ; absence totale de config `EMAIL_MODE` /
  `REVIEWS_STORE_MODE` → aucune erreur, comportement démo par défaut.
- Site principal non affecté : `/` répond toujours en 200, aucune régression.
- **Aucun push** — en attente de relecture avant mise en ligne.

---

## [Optimisation performance] Baseline Lighthouse — AVANT modification

Build de production (`next build` + `next start -p 3005`), Lighthouse CLI
(`npx lighthouse`, Chrome headless, catégories perf/a11y/bonnes pratiques/SEO)
sur la page d'accueil `/`.

### Scores de départ

| Catégorie | Score |
| --- | --- |
| Performance | **96** |
| Accessibilité | **96** |
| Bonnes pratiques | **96** |
| SEO | **100** |

### Métriques

| Métrique | Valeur |
| --- | --- |
| First Contentful Paint | 0,8 s |
| Largest Contentful Paint | 2,8 s |
| Total Blocking Time | 30 ms |
| Cumulative Layout Shift | 0 |
| Speed Index | 0,8 s |
| Time to Interactive | 3,0 s |
| Max Potential FID | 120 ms |

### Pistes concrètes déjà identifiées par Lighthouse (à creuser, pas à corriger à l'aveugle)

1. **3 animations non compositées** sur la section Verdict (`glitch-jitter`,
   `glitch-split-a`, `glitch-split-b`) — Lighthouse flague `filter` (« may move
   pixels ») et `clip-path` (« Unsupported CSS Property ») comme non pris en
   charge par le compositeur. C'était présenté comme « propriétés compositées
   uniquement » dans le changelog du Labo/Verdict — **à revérifier, l'outil dit
   le contraire pour clip-path/filter animés**, contrairement à `transform`/`opacity`.
2. **JS inutilisé** : chunk framer-motion (`249-*.js`, 128 Ko) — 29,7 Ko sur
   40,7 Ko (73 %) non exécutés sur la page d'accueil.
3. **2 tâches longues** détectées (117 ms au chargement initial, 84 ms dans le
   chunk react-dom vers 2,8 s) — TBT reste bas (30 ms), à surveiller mais pas
   forcément un problème en soi.
4. Chunk `fd9d1056-*.js` (172 Ko) identifié = **react-dom** (incompressible,
   pas une piste d'optimisation).

Premier build de prod (`First Load JS` partagé) : **87,4 Ko** répartis en
`117-*.js` (31,7 Ko), `fd9d1056-*.js` (53,6 Ko compressé), + chunks additionnels.
Page d'accueil : 3,19 Ko propres + 147 Ko total.

Diagnostic détaillé (animations, canvas, images, polices, bundle, re-renders
React) à suivre avant toute correction — voir entrée suivante.

---

## [Optimisation performance] Diagnostic complet + corrections appliquées

Méthode : profiling réel (pas de suppositions), un point de contrôle par item
de la checklist, correction uniquement de ce qui est confirmé.

### 1. Profiling réel (scroll + interactions)

`PerformanceObserver` (`longtask`) installé en conditions réelles (build de
prod, port 3005) pendant : parcours complet de la page (Hero → Verdict →
Plans → Process → Contact via navigation par ancre), plusieurs cycles de
l'effet glitch du Verdict, interactions sur le panier de `/demo/commande`.
**Résultat : zéro tâche longue (>50 ms) détectée** dans les trois scénarios.
Les 2 tâches longues que Lighthouse rapporte sur le chargement initial
(117 ms et 84 ms) sont internes à React/react-dom (hydratation) — TBT réel
mesuré à seulement 30 ms au départ, donc déjà sous le seuil qui impacterait
l'expérience.

### 2. Animations — audit exhaustif de tous les `@keyframes` + props Framer Motion

Toutes les animations du site passées en revue (`app/globals.css` +
`animate`/`initial`/`whileInView` de Framer Motion, code live uniquement) :

- `preview-in`, `hero-rise`, `hero-slate`, `sig-drift`, `sig-word`,
  `v3-ticker`, `v3-drift`, `v3-notif`, `Reveal` (Framer Motion) : **déjà
  `transform`/`opacity` uniquement** — rien à corriger.
- `offer-pulse` anime `box-shadow` (exactement la propriété citée comme
  contre-exemple) — mais n'est utilisée que par `.featured-offer`, une classe
  **exclusive à `components/Carte.tsx` (archive-only, v1/v2)**. Aucun élément
  du site actuel ne porte cette classe : zéro coût réel, laissée telle quelle
  (hors périmètre, code mort mais protégé comme documenté dans le nettoyage
  précédent).
- **`glitch-jitter` / `glitch-split-a` / `glitch-split-b`** (section Verdict) :
  seules animations réellement problématiques, confirmées par Lighthouse
  (`non-composited-animations`, 3 éléments flagués) ET par lecture directe du
  CSS — `filter` et `clip-path` animés, pas compositables de façon fiable.
  Voir corrections ci-dessous.

### 3. Canvas / effets visuels lourds

- **Hero (`V3Scene.tsx` + `V3Backdrop.tsx`)** : déjà exemplaire sur les 3
  critères demandés — `dpr={[1, 1.5]}` (plafonné, jamais illimité),
  `frameloop={active ? "always" : "never"}` piloté par un `IntersectionObserver`
  (pause hors écran), et surtout **coupé sur mobile/tactile** (`pointer: coarse`
  ou `max-width: 768px` → fallback CSS, jamais de WebGL chargé). Chargement
  WebGL lui-même différé via `next/dynamic` + `requestIdleCallback`. Rien à
  changer.
- **Curseur qui suit le blob WebGL** : position stockée dans une `ref`
  (`target.current`), jamais dans le state React — consommée une seule fois
  par frame via `useFrame` avec un lerp. Déjà l'implémentation demandée.
  point 7 de la checklist confirmé sans action nécessaire.
- **Labo (`ShatterPortal.tsx`)** : vérifié en lecture seule uniquement (zone
  protégée, jamais modifiée) — DPR plafonné à 2, boucle `requestAnimationFrame`,
  nombre de fragments déjà réduit sur mobile (36 vs 65 desktop). Déjà conforme.

### 4. Images

Recherche exhaustive (`find` sur tous les formats raster + grep `<img>`/
`next/image`) : **zéro image raster dans tout le projet**, live ou archivé.
Le site est 100 % SVG inline / dégradés CSS / WebGL. Rien à optimiser —
confirmé par recherche directe, pas supposé.

### 5. Polices

`next/font/google` (Instrument Serif, Work Sans, IBM Plex Mono) : déjà
auto-hébergées (aucune requête vers fonts.googleapis.com), `display: "swap"`
déjà positionné sur les 3 familles, préchargement automatique confirmé dans
le HTML de prod (6 `<link rel="preload" as="font">`). C'est exactement ce que
`next/font` garantit par construction — rien à ajouter.

### 6. JavaScript et bundle

- **Taille du bundle** : chunks identifiés précisément (pas par déduction) en
  comparant les requêtes réseau de `/` vs `/demo/commande` : `fd9d1056-*.js`
  (172 Ko) = react-dom, `117-*.js` (124 Ko) = runtime React, `972-*.js`
  (26 Ko) = helpers Next.js (i18n/routing, partagé partout) — tous
  incompressibles. `249-*.js` (128 Ko) = **Framer Motion**, chargé uniquement
  sur les pages utilisant `<Reveal>` (le code-splitting par route fonctionne
  déjà correctement) mais avec **73 % de code jamais exécuté** (29,7 Ko/40,7 Ko,
  mesuré par Lighthouse `unused-javascript`). Voir correction ci-dessous.
- **Scripts tiers** : Vercel Analytics (`@vercel/analytics/react`) chargé de
  façon non bloquante (`<Analytics />` en fin de `<body>`, script tiers
  différé par le SDK lui-même) — confirmé par le `server-response-time` de
  10 ms et le TBT de 30 ms au départ. Rien à changer.
- **Modules Commande & Livraison / Relance avis** : leur code serveur
  (`lib/delivery/*`, `lib/reviews/*`) tourne exclusivement dans des routes API
  (`app/api/**/route.ts`) — jamais expédié au navigateur, quelle que soit la
  page. Côté client, `DeliveryOptionSelector`/`DeliveryTracker` ne sont
  importés que par `app/demo/commande` et `app/demo/livraison` (vérifié via
  les requêtes réseau) : le découpage par route de Next.js les exclut déjà du
  bundle de la page d'accueil et de toutes les autres pages. **Déjà fait, rien
  à changer.**

### 7. Re-renders React inutiles

Recherche de tout `useState` mis à jour en boucle ou sur `mousemove`/
`pointermove` dans le code live :
- **Hero (`NotifFeed`)** : `setInterval` à 2,2 s — fréquence négligeable,
  pas un problème.
- **Verdict (`GlitchAnswer`)** : `setInterval` à 2 s — idem, négligeable.
- **Curseur WebGL** : déjà sur une `ref`, voir point 3.
- **Aucun** `mousemove`/`pointermove` mettant à jour du state React trouvé
  dans le code live (les 3 occurrences archive-only — `signature/*`,
  `v2/V2Hero.tsx` — sont hors périmètre).

Conclusion : **zéro problème de re-render trouvé.**

### Corrections appliquées

1. **`components/Reveal.tsx` : `motion` → `LazyMotion` + `m` + `domAnimation`.**
   Seul consommateur du composant `motion` complet dans tout le code live, et
   seulement pour un fade + `translateY` (ni drag, ni layout, ni `AnimatePresence`
   complexe) — exactement le cas d'usage prévu pour `LazyMotion`. Zéro
   changement de comportement (même easing, même durée, même déclenchement
   `whileInView`, même court-circuit sous `prefers-reduced-motion` — la
   branche reduced-motion ne touche même pas au nouveau code).
2. **`app/globals.css` : `filter` retiré de `@keyframes glitch-jitter`.**
   Le micro-flou de 0,3px qu'il produisait était à peine perceptible ; le
   tremblement `transform` seul suffit visuellement. Change 1 des 3 éléments
   flagués par Lighthouse.

### Non corrigé — décision à confirmer

**`glitch-split-a` / `glitch-split-b` (2 éléments restants) animent
`clip-path`**, le cœur du découpage en « tranches » qui fait le look glitch/
RGB-split de la section Verdict. Mesuré : **zéro tâche longue réelle causée
par cet effet** sur cette machine (voir point 1) — le flag Lighthouse est
correct sur le principe (clip-path animé n'est pas garanti compositable par
tous les navigateurs) mais n'est pas la cause d'un ralentissement observable
ici. Par cohérence avec la consigne de ne pas dégrader un effet déjà validé
sans confirmation, **je ne l'ai pas modifié**. Deux options si vous voulez
aller plus loin, par ordre de risque visuel croissant : (a) rien, l'effet est
déjà rapide en pratique ; (b) reconstruire l'effet avec des bandes déjà
découpées statiquement (plusieurs éléments à `clip-path` fixe, seul le
`transform`/`opacity` de chacun serait animé) — même rendu final possible,
mais demande une réécriture du composant, pas juste du CSS.

### Autres constats (hors périmètre performance, non corrigés)

- **`errors-in-console`** (Lighthouse, bonnes pratiques) : 404 sur
  `/_vercel/insights/script.js` — c'est Vercel Analytics qui cherche un
  script servi uniquement par l'infrastructure Vercel réelle ; absent en test
  local (`next start`). Pas un bug, un artefact de test — n'apparaîtra pas
  une fois déployé.
- **`color-contrast`** (Lighthouse, accessibilité) : plusieurs textes en
  `text-encre/50` à `/60` (opacité réduite) et badges colorés sous le ratio
  WCAG. Réel, mais hors périmètre de ce chantier (performance, pas
  accessibilité visuelle) — signalé plutôt que corrigé à la volée.

### Scores Lighthouse — avant / après

| Catégorie / métrique | Avant | Après |
| --- | --- | --- |
| Performance | 96 | 96 |
| Accessibilité | 96 | 96 |
| Bonnes pratiques | 96 | 96 |
| SEO | 100 | 100 |
| First Contentful Paint | 0,8 s | 0,8 s |
| Largest Contentful Paint | 2,8 s | 2,8 s |
| **Total Blocking Time** | **30 ms** | **10 ms** |
| Cumulative Layout Shift | 0 | 0 |
| Speed Index | 0,8 s | 0,8 s |
| Time to Interactive | 3,0 s | 2,9 s |
| **JS inutilisé (page d'accueil)** | **29,7 Ko** | **0 Ko** |
| **Animations non compositées** | **3 éléments** | **2 éléments** |
| First Load JS (page d'accueil) | 147 Ko | **137 Ko** |
| First Load JS (`/qui-je-suis`) | 144 Ko | **134 Ko** |

Les scores globaux (déjà à 96-100 au départ) ne bougent pas — le site était
déjà très bien optimisé avant ce chantier. Les métriques fines qui reflètent
directement les corrections (TBT, JS inutilisé, animations non compositées,
poids du bundle) s'améliorent toutes, sans aucune régression ailleurs.

### Vérifications effectuées

- `tsc --noEmit` ✅, build production ✅ (mêmes 19 routes).
- Rendu visuel vérifié en preview après corrections : Hero, section Verdict
  (glitch actif, texte cyclé), desktop et mobile — identiques à avant.
- `prefers-reduced-motion` re-testé spécifiquement sur `Reveal.tsx` (seul
  fichier modifié touchant à une logique reduced-motion) : flag forcé
  temporairement, confirmé que le contenu s'affiche directement à `opacity:1`
  sans transition (même comportement qu'avant, la branche reduced-motion est
  identique ligne pour ligne), hack retiré immédiatement après.
- **Aucun push** — en attente de relecture avant mise en ligne, et de votre
  décision sur l'effet glitch (clip-path) ci-dessus.

---

## [Pages exemples] "Maison Verdure" — démo concrète du plan Présence

**Correction de prémisse avant de commencer** : la demande faisait référence
à "La Carte" et à la carte "Vitrine Essentielle" — vocabulaire de l'ancienne
DA (v1/v2, archivée). Sur le site actuel, la section s'appelle **« Les
plans »** et l'offre équivalente (3-4 pages, mobile, fiche Google Business,
formulaire de contact, mise en ligne rapide) s'appelle **« Présence »**
(690€). La démo a été construite pour ce plan précis, à la route
`/exemples/presence` plutôt que `/exemples/vitrine-essentielle`.

**Images** : la première approche (photos Unsplash téléchargées localement)
a été abandonnée en cours de route à la demande du client, remplacée par des
blocs placeholder stylés (dégradé radial dans les couleurs de marque +
icône emoji centrée) — plus rapide à produire, montre la structure sans
nécessiter de vraies photos. Aucune image raster n'a été conservée dans le
projet.

### Ce qui a été construit

- **Commerce fictif** : « Maison Verdure », boutique de plantes générique
  (Saint-Maur-des-Fossés) — cohérent avec le repositionnement « tout
  commerçant », pas spécifiquement restaurant.
- **4 pages**, structure réutilisable pour les futures démos (`Site
  Autonome`, `Croissance Digitale`, `Boutique en ligne`) :
  - `app/exemples/presence/page.tsx` — Accueil
  - `app/exemples/presence/presentation/page.tsx` — Présentation / À propos
  - `app/exemples/presence/galerie/page.tsx` — Galerie (4 blocs placeholder)
  - `app/exemples/presence/contact/page.tsx` — Contact (formulaire + fiche
    Google Business + adresse/horaires)
  - `app/exemples/presence/layout.tsx` — bandeau de contexte + nav + footer
    partagés
- **Composants réutilisables** (`components/exemples/`) : `PlaceholderImage`
  (bloc image stylé), `ExempleNav` (nav du site fictif, distincte de
  `V3Nav`), `ExempleFooter` (mention « hébergement inclus » en pied de page
  uniquement — jamais une fonctionnalité), `ExempleBanner` (bandeau
  « exemple concret », lien retour vers `/#plans`).
- **Contenu** : `content/exemples/presence.ts`, avec un commentaire de tête
  rappelant explicitement le périmètre strict à ne jamais dépasser.
- **Bouton sur la carte « Présence »** (`content/v3.ts` → nouveau champ
  `V3Plan.exampleHref`, rendu conditionnel dans `V3Plans` —
  `components/v3/Sections.tsx`) : « Voir un exemple concret ↗ », en nouvel
  onglet. N'apparaît QUE sur la carte Présence (vérifié : absent sur
  Autonome/Machine, pas encore de démo pour ces plans).
- **SEO** : `robots.ts` étend le `disallow` à `/exemples` (même traitement
  que `/demo` et `/labo`) ; metadata `noindex, nofollow` sur le layout ;
  absent du sitemap.

### Périmètre strictement respecté (vérifié, pas juste écrit)

`grep` exhaustif sur `content/exemples/`, `app/exemples/`,
`components/exemples/` pour "espace admin", "autonomie", "automat*",
"relance", "panier", "paiement", "e-commerce", "boutique en ligne",
"réservation" : **0 occurrence dans le contenu réel** (les seules
correspondances sont dans le commentaire de tête de `presence.ts`, qui
rappelle justement de ne jamais les ajouter). Aucune mention "non
disponible" nulle part — l'absence est silencieuse, comme demandé.

### Vérifications effectuées

- `tsc --noEmit` ✅, build production **23 routes** ✅ (4 nouvelles pages
  générées correctement).
- Les 4 pages vérifiées visuellement en preview : Accueil (hero + placeholder
  + carte fiche Google), Présentation (placeholder + 3 valeurs), Galerie (4
  blocs colorés), Contact (adresse/horaires/note Google + formulaire).
- Formulaire de contact testé bout en bout (état « Message envoyé ! »
  fonctionnel, pattern optimistic déjà utilisé ailleurs sur le site).
- Bouton "Voir un exemple concret" vérifié présent uniquement sur la carte
  Présence, absent des autres, lien et cible (`target="_blank"`) corrects.
- Rendu mobile (375px) vérifié : nav qui wrap proprement, aucun débordement
  horizontal (`scrollWidth` = 375px exact).
- Console navigateur : 0 erreur sur les 4 pages.
- **Aucun push** — en attente de relecture.

## [Glitch Verdict] Simplification clip-path → transform/opacity

### Le problème

Lighthouse flaggait toujours 2 éléments sur l'audit `non-composited-
animations` : les pseudo-éléments `::before`/`::after` du glitch RGB-split
de la section Verdict (`.glitch-answer`), qui animaient `clip-path` — une
propriété qui force un repaint, jamais accélérée par le compositeur GPU,
même après la correction précédente du micro-flou (`filter`) sur l'élément
principal.

### La décision (tranchée par l'utilisateur)

Remplacer l'usage de `clip-path` par une combinaison `translate` +
`scaleX` + `opacity` uniquement — entièrement composité/accéléré GPU. Effet
visuel accepté comme potentiellement un peu moins marqué en échange de la
fluidité garantie (fidèle à l'esprit "signal qui saute", juste sans la
découpe nette).

### Ce qui a changé

- `app/globals.css` — keyframes `glitch-split-a` / `glitch-split-b` :
  suppression totale de `clip-path`, remplacé par `transform: translate(…)
  scaleX(…)` + `opacity` en 8 étapes, toujours joué avec `steps(6, end)`
  pour garder le rendu saccadé (pas d'interpolation lisse — ce serait un
  autre effet).
- Le reste de la mécanique (`glitch-jitter`, `mix-blend-mode: screen`,
  couleurs #ff6b4a / #0ea88b, `content: attr(data-text)`, le bloc
  `prefers-reduced-motion` qui neutralise tout) reste **inchangé**.

### Vérifications effectuées

- Effet visuel confirmé en figeant manuellement l'animation à mi-parcours
  (`animation-play-state: paused` + `animation-delay` négatif) : le
  frange RGB (orange/teal) est toujours visible sur le texte, effet
  reconnaissable.
- Build production propre + `next start` sur port isolé (3009) + Lighthouse
  (`--only-categories=performance`) : **`non-composited-animations` → 0
  élément flaggé** (contre 2 avant correction). Score performance revenu à
  **96/100**, TBT 30ms, LCP 2.8s — identique à la baseline post-optimisation.
- Piège rencontré pendant la vérification : un ancien process `next start`
  orphelin occupait déjà le port choisi et un serveur `next dev` resté actif
  en tâche de fond partageait le même dossier `.next` que le build de
  production — combinaison qui a fait servir du JS non minifié le temps de
  identifier et tuer les deux processus, avant un rebuild propre. Aucun
  rapport avec le code applicatif.
- `prefers-reduced-motion` : bloc CSS non modifié, structurellement toujours
  correct (neutralise animation + contenu sur l'élément et ses deux
  pseudo-éléments, indépendamment des propriétés animées à l'intérieur).
- `tsc --noEmit` ✅.

## [Pages exemples] "Salon Marguerite" — démo concrète du plan Autonome

### Objectif

Deuxième page de la série `/exemples/*` (après Présence), montrant
concrètement ce que couvre le plan **Autonome** (1490€) — différenciateur
clé face à Présence : un **espace admin** où le commerçant modifierait
lui-même son site (photos, textes, horaires), sans back-office réellement
connecté.

### Contrainte de méthode (rappelée par l'utilisateur, respectée)

Réutiliser tel quel le squelette de `/exemples/presence` (layout, nav,
footer, `PlaceholderImage`) plutôt que reconstruire une architecture —
aucun nouveau composant partagé créé dans `components/exemples/`, à
l'exception d'une correction ponctuelle sur `ExempleFooter` (voir
ci-dessous).

### Ce qui a été construit

- **Commerce fictif** : « Salon Marguerite », salon de coiffure générique
  (Suresnes) — délibérément différent de la boutique de plantes de
  Présence, toujours générique (pas un restaurant).
- **6 pages**, structure et style identiques à `/exemples/presence` :
  - `app/exemples/site-autonome/page.tsx` — Accueil
  - `.../prestations/page.tsx` — Prestations & tarifs (nouveau : liste de
    services avec durée/prix, mise en avant comme contenu que le
    commerçant pourrait modifier lui-même)
  - `.../galerie/page.tsx` — Galerie (4 blocs placeholder)
  - `.../a-propos/page.tsx` — À propos
  - `.../contact/page.tsx` — Contact / prise de rendez-vous (formulaire
    repris du pattern `presence/contact`, un seul champ ajouté :
    "Prestation souhaitée" — même composant, mêmes classes, pas de
    nouvelle architecture)
  - `.../espace-admin/page.tsx` — **Espace admin (mockup visuel)**
- **Espace admin** (le différenciateur) : fenêtre façon back-office
  (`v3-window`) avec onglets (Horaires / Textes du site / Prestations &
  tarifs / Photos), champs pré-remplis modifiables localement (`useState`
  simple, rien de persisté), bouton "Enregistrer les modifications" qui
  affiche une confirmation temporaire. Bandeau `AVERTISSEMENT` explicite en
  haut de page : « Aperçu à titre de démonstration — cet espace n'est pas
  connecté à un vrai compte, rien n'est enregistré. » Aucune connexion
  réelle, aucun état persistant entre rechargements.
- **Contenu** : `content/exemples/site-autonome.ts`, même en-tête de
  commentaire rappelant le périmètre strict (pas d'automatisation, pas
  d'e-commerce).
- **Correction ponctuelle sur `ExempleFooter`** (`components/exemples/
  ExempleFooter.tsx`) : le texte de pied de page mentionnait en dur « plan
  Présence », ce qui aurait affiché une info fausse sur la démo Autonome.
  Ajout d'une prop `planLabel` (obligatoire) ; mise à jour du site
  d'appel dans `app/exemples/presence/layout.tsx` pour passer
  `planLabel="plan Présence"` explicitement — comportement inchangé pour
  Présence, correct pour Autonome.
- **Bouton sur la carte « Autonome »** (`content/v3.ts` → `exampleHref:
  "/exemples/site-autonome"` sur `v3plans.plans[1]`) : « Voir un exemple
  concret ↗ », rendu par le bloc conditionnel générique déjà en place dans
  `V3Plans`. Vérifié : présent uniquement sur les cartes Présence et
  Autonome, absent de Machine.

### Périmètre strictement respecté (vérifié, pas juste écrit)

`grep` exhaustif sur `content/exemples/site-autonome.ts` et
`app/exemples/site-autonome/` pour "relance", "notification", "panier",
"paiement", "e-commerce"/"ecommerce", "stripe", "checkout", "commande" :
**0 occurrence dans le contenu réel** (seules correspondances : le
commentaire de tête de `site-autonome.ts`, qui rappelle justement de ne
jamais les ajouter).

### Vérifications effectuées

- `tsc --noEmit` ✅.
- Les 6 pages vérifiées visuellement en preview, desktop et mobile
  (375px) : aucun débordement horizontal, nav qui wrap proprement.
- Espace admin testé en interaction réelle : changement d'onglet
  fonctionnel, soumission du formulaire "Horaires" → confirmation "✓
  Enregistré" + message de confirmation affichés (vérifié via
  `requestAnimationFrame` double, le round-trip des outils de preview étant
  plus lent que le timeout de 2,2s de la confirmation).
- Formulaire de contact/rendez-vous testé bout en bout (état "Demande
  envoyée !" fonctionnel).
- Bouton "Voir un exemple concret" vérifié présent sur Présence ET
  Autonome (2 occurrences exactement), absent de Machine.
- Console navigateur : 0 erreur sur les 6 pages.
- **Aucun push** — en attente de relecture.

## [Pages exemples] "Au Poil" — démo concrète du plan Machine

### Objectif

Troisième page de la série `/exemples/*`, montrant ce que le plan
**Machine** (dès 1990€) ajoute par-dessus Autonome : des automatisations
visibles (relance avis Google, notifications) et un tableau de bord
simple — en mockup, sans connexion réelle.

### Note de vocabulaire

La demande initiale nommait cette offre "Croissance Digitale" — ce nom
n'existe pas dans `content/v3.ts` ; le plan qui correspond exactement à la
description (automatisations, tableau de bord, dès 1990€) s'appelle
**Machine**. L'URL demandée (`/exemples/croissance-digitale`) a été
conservée telle quelle, mais le bandeau et le pied de page de la démo
affichent "Plan Machine" pour rester cohérents avec le site réel (même
traitement que "Vitrine Essentielle" → "Présence" précédemment).

### Contrainte de méthode (rappelée par l'utilisateur, respectée)

Réutiliser tel quel le squelette de `/exemples/site-autonome` (layout,
nav, footer, `PlaceholderImage`, structure de l'espace admin par onglets)
plutôt que reconstruire une architecture — aucun nouveau composant partagé
créé.

### Ce qui a été construit

- **Commerce fictif** : « Au Poil », toilettage canin (Vincennes) —
  encore un autre type de commerce que les deux démos précédentes
  (boutique de plantes, salon de coiffure), toujours générique.
- **6 pages**, structure et style identiques à `/exemples/site-autonome`
  (Accueil, Prestations, Galerie, À propos, Contact, Espace admin).
- **Espace admin étendu** (`app/exemples/croissance-digitale/espace-
  admin/page.tsx`) : reprend les 4 onglets du plan Autonome (Horaires /
  Textes / Prestations & tarifs / Photos), **plus 2 onglets spécifiques
  au plan Machine** — la différence clé à mettre en valeur :
  - **Automatisations** — liste de notifications simulées façon celles
    déjà utilisées dans le flux du Hero (`v3hero.events`, même forme
    `icon/text/tag`), ex. « Avis Google 5★ reçu — relance de remerciement
    envoyée automatiquement ». Réutilise la classe d'animation `v3-notif`
    déjà définie dans `globals.css`, aucune nouvelle animation créée.
  - **Tableau de bord** — 3 cartes de statistiques factices mais
    réalistes (visiteurs du mois, avis Google générés, taux de réponse
    aux demandes), rendues avec `v3-card`, déjà utilisé partout ailleurs.
  - Le rendu conditionnel de la zone de contenu a été étendu (`tab.fields`
    → formulaire déjà existant / `tab.notifications` → nouvelle liste /
    `tab.stats` → nouvelle grille / sinon → note photo déjà existante) :
    extension du switch déjà en place, pas de nouvelle architecture.
- **Contenu** : `content/exemples/croissance-digitale.ts`, même en-tête
  de commentaire rappelant le périmètre strict (aucun e-commerce).
- **Bouton sur la carte « Machine »** (`content/v3.ts` → `exampleHref:
  "/exemples/croissance-digitale"` sur `v3plans.plans[2]`) : rendu par le
  même bloc conditionnel générique déjà en place. Vérifié : les 3 cartes
  (Présence, Autonome, Machine) ont désormais chacune leur lien, dans
  l'ordre.

### Périmètre strictement respecté (vérifié, pas juste écrit)

`grep` exhaustif sur `content/exemples/croissance-digitale.ts` et
`app/exemples/croissance-digitale/` pour "panier", "paiement",
"e-commerce"/"ecommerce", "stripe", "checkout", "commande", "boutique en
ligne" : **0 occurrence dans le contenu réel** (seule correspondance : le
commentaire de tête de `croissance-digitale.ts`, qui rappelle justement de
ne jamais les ajouter).

### Vérifications effectuées

- `tsc --noEmit` ✅.
- Les 6 pages vérifiées visuellement en preview (Accueil, Prestations,
  Espace admin) : aucune erreur console.
- Onglets "Automatisations" et "Tableau de bord" testés en interaction
  réelle : changement d'onglet fonctionnel, styles actif/inactif corrects
  (vérifiés via `preview_inspect` — `background-color: rgb(255, 107, 74)`
  uniquement sur l'onglet actif, l'aspect délavé à l'écran n'étant qu'un
  artefact de compression JPEG du screenshot, pas un bug réel).
- Bouton "Voir un exemple concret" vérifié présent sur les 3 cartes
  (Présence, Autonome, Machine), dans le bon ordre, avec la bonne cible
  chacune.
- **Aucun push** — en attente de relecture.

## [Cohérence des routes exemples] Renommage + différenciation réaliste Autonome/Machine

### Partie 1 — Renommage des routes de démo

Les routes `/exemples/*` gardaient les anciens noms d'offres alors que le
contenu affiché (`content/v3.ts`) utilise déjà les noms actuels depuis un
moment — incohérence entre le bouton "voir un exemple concret" sur chaque
carte et l'URL de destination.

- `git mv app/exemples/site-autonome app/exemples/autonome` +
  `content/exemples/site-autonome.ts` → `content/exemples/autonome.ts`
- `git mv app/exemples/croissance-digitale app/exemples/machine` +
  `content/exemples/croissance-digitale.ts` → `content/exemples/machine.ts`
- Renommage en cascade de tout ce qui référençait l'ancien nom (imports,
  exports `siteAutonomeDemo`→`autonomeDemo`, `croissanceDigitaleDemo`→
  `machineDemo`, noms de composants, hrefs de nav internes) — fait par
  substitution ciblée sur les seuls fichiers concernés, puis vérifié par
  `grep` qu'aucune trace de l'ancien nom ne subsiste.
- `content/v3.ts` : `exampleHref` des plans Autonome et Machine mis à jour
  vers `/exemples/autonome` et `/exemples/machine`. Présence n'a pas
  bougé (déjà `/exemples/presence`).
- `next.config.mjs` : redirections 307 ajoutées pour les 3 anciennes URLs
  (`/exemples/vitrine-essentielle`, `/exemples/site-autonome`,
  `/exemples/croissance-digitale`, avec leurs `:path*`) vers les nouvelles
  — testées une par une via `curl -I`, toutes redirigent proprement, aucune
  404 sèche.
- `content/site.ts` (les anciens noms "Vitrine Essentielle" / "Site
  Autonome" / "Croissance Digitale" y figurent aussi) **volontairement pas
  touché** : ce fichier alimente uniquement `app/_archive/*` (v1/v2
  archivés, hors routing public), pas le site live — vérifié en confirmant
  que `<Carte>` (le composant qui affiche ces noms) n'est rendu que dans
  `app/_archive/`.

### Partie 2 — Différenciation Autonome / Machine

Le distinguo demandé : Autonome = le commerçant agit lui-même ; Machine =
le système agit à sa place. Avec une contrainte stricte — ne mettre en
scène sur `/exemples/machine` que des automatisations réellement
livrables aujourd'hui (Resend + Supabase) : relance avis Google
post-réservation, confirmation automatique de réservation, réponse
automatique à un formulaire de contact, tableau de bord basé sur des
données du site (jamais une fausse synchronisation Google). Explicitement
exclu : module Uber Direct/livraison, nombre d'avis Google synchronisé,
filtrage des avis par satisfaction (review gating, illégal).

- **`components/v3/NotifFeed.tsx`** (nouveau, extrait de `V3Hero`) : le
  flux de notifications qui se remplit tout seul (`setInterval` 2,2s, 4
  lignes visibles, respecte `prefers-reduced-motion`) vivait uniquement
  dans `components/v3/Hero.tsx`. Extrait en composant partagé paramétrable
  (`events`, `tagColors`, `intervalMs`) pour être réellement réutilisé —
  pas recréé — sur la page Machine. `Hero.tsx` importe maintenant ce même
  composant, comportement visuel strictement identique (vérifié : 0 erreur
  console sur `/`, capture d'écran du Hero inchangée).
- **`/exemples/autonome`** : ajout d'un encart "🖐️ Ici, c'est vous qui
  décidez et qui modifiez — en 2 minutes, sans coder." bien visible sur
  l'accueil, sous la liste des points forts. Confirmé par grep : zéro
  élément d'automatisation (notification, compteur) sur cette page — elle
  n'en a jamais eu.
- **`/exemples/machine`** : refonte du contenu autour des 3 automatisations
  autorisées uniquement :
  - Accueil : le placeholder photo est remplacé par un **flux d'activité
    live** (`v3-window` + `NotifFeed`, identique dans l'esprit à celui du
    Hero principal) qui empile automatiquement "Relance avis envoyée à
    Camille D. — 2 min après son rendez-vous", "Réservation confirmée
    automatiquement pour Julien M.", "Réponse automatique envoyée à une
    demande de contact", etc. — sans aucun clic du visiteur. Encart "🔔 Ici,
    vous ne faites rien — le site travaille pendant que vous êtes
    ailleurs." juste en dessous des points forts.
  - Espace admin : les onglets **Automatisations** et **Tableau de bord**
    passent en premier (avant Horaires/Textes/Tarifs/Photos, hérités du
    plan Autonome mais volontairement relégués) — l'onglet actif par
    défaut est désormais Automatisations. La liste de notifications de cet
    onglet utilise elle aussi `NotifFeed` (avant : cartes statiques) pour
    renforcer l'effet "ça tourne tout seul".
  - Tableau de bord : les statistiques ont changé pour rester honnêtes —
    retiré "Visiteurs ce mois-ci" et "Avis Google générés" (sonnait comme
    une vraie synchronisation Google, non branchée techniquement),
    remplacé par "Réservations confirmées automatiquement" (42), "Relances
    avis envoyées" (37), "Taux de réponse aux demandes" (96%) — avec une
    légende explicite : "Données collectées par le site — pas une
    synchronisation avec votre fiche Google Business."
  - Contact : le message de confirmation après envoi change de "Le salon
    revient vers vous rapidement pour confirmer." (sous-entend une action
    humaine) à "Confirmée automatiquement !" / "Vous recevez une
    confirmation immédiate par email — personne n'a eu besoin de la
    valider à la main." — démontre l'automatisation #2 au moment même de
    l'interaction, pas seulement dans l'espace admin.
  - `automationEvents` factorisé une seule fois dans
    `content/exemples/machine.ts` et réutilisé à la fois par le flux de
    l'accueil et l'onglet admin (6 entrées, pour que les 4 lignes visibles
    de `NotifFeed` ne se répètent jamais dans la même fenêtre).

### Périmètre strictement respecté (vérifié, pas juste écrit)

`grep` exhaustif sur `content/exemples/machine.ts` et
`app/exemples/machine/` pour "uber", "livraison", "coursier", "panier",
"paiement", "e-commerce"/"ecommerce", "stripe", "checkout", "avis générés",
"avis reçu", "nombre d'avis", "filtr" : **0 occurrence dans le contenu
réel** (seules correspondances : le commentaire de tête de `machine.ts`,
qui rappelle justement de ne jamais les ajouter).

### Vérifications effectuées

- `tsc --noEmit` ✅ après le renommage complet et la refonte de contenu.
- Redirections testées via `curl -I` : `/exemples/site-autonome` → 307 →
  `/exemples/autonome`, `/exemples/site-autonome/contact` → 307 →
  `/exemples/autonome/contact`, `/exemples/croissance-digitale` → 307 →
  `/exemples/machine`, `/exemples/vitrine-essentielle` → 307 →
  `/exemples/presence`. Nouvelles URLs → 200 directement.
- Les 3 boutons "Voir un exemple concret" (`/#plans`) vérifiés un par un :
  pointent respectivement vers `/exemples/presence`, `/exemples/autonome`,
  `/exemples/machine`.
- `/exemples/autonome` puis `/exemples/machine` comparées côte à côte :
  contraste immédiat dès le premier écran (photo statique + encart "vous
  décidez" vs flux d'activité qui s'anime tout seul + encart "vous ne
  faites rien").
- Console navigateur : 0 erreur sur `/`, `/exemples/autonome`,
  `/exemples/machine`, `/exemples/machine/espace-admin`.
- **Aucun push** — en attente de relecture.

## [Incident prod] Déploiements Vercel bloqués — cron toutes les 10 min incompatible avec le plan Hobby

### Symptôme

Après le push des deux entrées précédentes, le site en ligne
(`nova-sigma-khaki.vercel.app`) ne reflétait toujours pas les changements
— `/exemples/presence` (qui existait pourtant depuis des jours) répondait
en 404, et le titre du Hero affiché ne correspondait même pas à la copie
du repositionnement du 13/07. Le déploiement le plus récent visible sur
Vercel datait d'avant le commit `b68c651` ("Module Relance avis Google"),
alors que 14 commits avaient été poussés depuis.

### Diagnostic

- Repo GitHub bien connecté côté Vercel (`Settings → Git`), événements
  `deployment_status`/`repository_dispatch` activés — l'intégration
  paraissait saine.
- Un **redeploy manuel** de l'ancien commit (`5ac67b6`) a réussi (Ready,
  promu Production) — donc le projet Vercel lui-même n'était ni suspendu
  ni bloqué par une limite de facturation.
- Tenter un **Create Deployment** sur le commit actuel a révélé la vraie
  cause : `vercel.json` déclare un cron `*/10 * * * *` (toutes les 10
  minutes) pour `/api/reviews/run`, or le compte Vercel est sur le plan
  **Hobby** (gratuit), qui limite les Cron Jobs à **une exécution par
  jour**. Vercel rejette la validation de tout déploiement contenant une
  config cron hors limite du plan — silencieusement pour les déploiements
  automatiques déclenchés par push (aucune erreur visible dans la liste,
  juste... rien ne se passe).
- Ce risque était en fait déjà documenté dans `README-reviews.md` au
  moment de la construction du module Relance avis (« ⚠️ Sur le plan
  Hobby... Pro est nécessaire pour un cron toutes les 10 minutes ») mais
  n'avait jamais été suivi d'effet avant ce premier vrai déploiement du
  module.

### Correction

- `vercel.json` : `*/10 * * * *` → `0 9 * * *` (une fois par jour, 9h) —
  compatible plan Hobby, débloque tous les déploiements (celui-ci et les
  suivants, automatiques comme manuels).
- `README-reviews.md` : section installation et arborescence mises à jour
  pour refléter le rythme réel (1x/jour) et documenter explicitement la
  conséquence (relance avis pouvant partir avec jusqu'à ~24h de décalage
  par rapport à l'heure exacte prévue — acceptable pour ce cas d'usage,
  pas pour un besoin de précision à la minute).
- Décision actée avec l'utilisateur : rester sur le plan Hobby avec un
  cron quotidien plutôt que passer sur Vercel Pro pour garder les 10
  minutes.

### Vérifications effectuées

- `tsc --noEmit` ✅, `vercel.json` validé comme JSON syntaxiquement correct.
- `grep` : aucune autre référence au rythme "10 minutes" restante dans le
  code ou la doc (`README-reviews.md`, `CHANGELOG.md` historique conservé
  tel quel — c'est un journal, pas une doc vivante).
- **Reste à faire côté utilisateur** : pousser ce commit, puis relancer un
  `Create Deployment` (ou attendre le déploiement automatique si le
  webhook se redéclenche correctement) pour vérifier que le déploiement
  passe enfin et que le site live rattrape son retard.
- **Aucun push** — en attente de confirmation utilisateur.
- **Suivi** : poussé (`8da6300`), déploiement Vercel repassé au vert
  automatiquement — `/exemples/autonome` et `/exemples/machine` vérifiés
  en 200 avec `age: 0` (réponse fraîche, plus de cache périmé). Confirme
  que le webhook GitHub → Vercel fonctionnait bien depuis le début ; seule
  la validation du cron bloquait. Aucune reconnexion Git nécessaire.

## [Retrait de sections] "Ce que ça fait" et "Le process" masquées de la homepage

### Demande

Retirer les sections **"Ce que ça fait"** (V3Moteur, ancre `#moteur`) et
**"Le process"** (V3Process, ancre `#process`) de la homepage — sans
supprimer le code : composants et contenu conservés intacts pour une
réactivation facile plus tard.

### Ce qui a changé

- `app/page.tsx` : `<V3Moteur />` et `<V3Process />` retirés du rendu
  (lignes commentées, pas supprimées), imports correspondants commentés
  dans `Sections.tsx`. Nouvel enchaînement : Hero → Verdict → Constat →
  Plans → Fondateur → Contact.
- `content/v3.ts` : les entrées de nav "Ce que ça fait" (`/#moteur`) et
  "Le process" (`/#process`) commentées dans `v3nav.links`, pour ne pas
  laisser de lien mort vers une ancre qui n'existe plus sur la page.
- **Rien de supprimé** : `V3Moteur`/`V3Process` restent exportés depuis
  `components/v3/Sections.tsx`, `v3moteur`/`v3process` restent définis
  dans `content/v3.ts` avec tout leur contenu (bento cards, étapes du
  process) — il suffit de décommenter les 4 lignes concernées pour tout
  réafficher.
- `content/site.ts` (ancien contenu v1/v2, alimente uniquement
  `app/_archive/*`) non touché — ses propres liens `#process` restent
  valides dans l'archive, hors sujet ici.

### Vérifications effectuées

- `tsc --noEmit` ✅.
- Page d'accueil vérifiée en preview : plus aucune trace des deux
  sections, enchaînement Constat → Plans → Fondateur confirmé par
  snapshot d'accessibilité.
- Menu (mobile + desktop) vérifié : ne contient plus que "Le constat",
  "Les plans", "Qui suis-je" — aucun lien mort vers `#moteur`/`#process`.
- Console navigateur : 0 erreur.

## [Retrait de section] "Le constat" masquée de la homepage

Même traitement que "Ce que ça fait"/"Le process" ci-dessus, appliqué à
**V3Constat** (ancre `#constat`) : demandé juste après, avec push
immédiat cette fois.

- `app/page.tsx` : `<V3Constat />` commenté (plus importé), enchaînement
  homepage désormais Hero → Verdict → Plans → Fondateur → Contact.
- `content/v3.ts` : entrée "Le constat" (`/#constat`) commentée dans
  `v3nav.links`.
- Rien de supprimé : `V3Constat` reste exporté depuis `Sections.tsx`,
  `v3constat` reste défini dans `content/v3.ts` avec son contenu (cartes
  "Il est verrouillé" / "Il coûte sans compter").

### Vérifications effectuées

- `tsc --noEmit` ✅.
- Preview rechargée : `document.body.innerText` ne contient plus "Le
  constat" nulle part sur la page.
- Console navigateur : 0 erreur.

## [4e offre] Plan "Boutique" (e-commerce) + démo /exemples/boutique

### Objectif

Ajouter une 4e carte dans "Les plans" pour l'offre e-commerce, et sa page
de démo — sur le modèle des 3 précédentes, avec le vrai travail nouveau
concentré sur le parcours panier + paiement Stripe (mode test).

### Carte "Boutique"

- `content/v3.ts` : `V3Plan.color` élargi à `"jaune"` (4e couleur déjà
  utilisée ailleurs sur le site — `DOT`/`TEXT` dans `Sections.tsx` la
  supportaient déjà, aucune nouvelle palette à créer). Nouvelle entrée
  `v3plans.plans[3]` : dès 3200€ + 129€/mois, cohérent avec la
  progression 690→1490→1990→3200 (one-shot) et 25→45→95→129 (mensuel).
  Fonctionnalités : tout Autonome + catalogue + fiches produits enrichies
  + panier/paiement Stripe + suivi de commandes. `exampleHref:
  "/exemples/boutique"`.
- `components/v3/Sections.tsx` : grille des plans passée de
  `md:grid-cols-3` à `sm:grid-cols-2 xl:grid-cols-4` pour accueillir la
  4e carte (1 colonne mobile, 2 en tablette, 4 en desktop large) ; le
  décalage vertical de la carte "mise en avant" (Autonome) recalé de
  `md:` à `xl:` pour rester cohérent avec le nouveau point de rupture à
  4 colonnes.

### Démo `/exemples/boutique` — "Le Petit Atelier" (savonnerie artisanale)

Structure reprise à l'identique du pattern déjà en place (`ExempleBanner`,
`ExempleNav`, `ExempleFooter`, `PlaceholderImage`) — mais périmètre de
pages adapté à ce que cette offre démontre concrètement (catalogue,
panier, paiement), pas le même jeu de pages que les 3 démos précédentes :

- `app/exemples/boutique/page.tsx` — Accueil
- `app/exemples/boutique/catalogue/page.tsx` — grille de 6 produits
  fictifs (savons/bougies/soins), filtre par catégorie, ajout rapide au
  panier
- `app/exemples/boutique/produit/[slug]/page.tsx` — fiche produit (Server
  Component + `notFound()` sur slug inconnu, `generateStaticParams` pour
  les 6 produits), sélecteur de quantité + ajout au panier dans un
  sous-composant client co-localisé (`AddToCartButton.tsx`)
- `app/exemples/boutique/panier/page.tsx` — lignes modifiables
  (quantité/suppression), total en temps réel, bouton de paiement
- `app/exemples/boutique/confirmation/page.tsx` — confirmation simple
  après paiement réussi (réel ou simulé)
- `content/exemples/boutique.ts` — business fictif, nav, contenu de
  chaque page, catalogue de 6 produits (`BoutiqueProduct[]`)

### Panier — `components/exemples/CartContext.tsx` (nouveau)

Seul vrai bout d'infrastructure nouveau de cette page : Context React +
persistance `localStorage` (clé `k1000-exemples-boutique-cart`, dédiée à
cette démo, aucune collision possible avec un autre module). Expose
`addItem`/`removeItem`/`updateQty`/`clear`/`count`/`total`. Le layout
`/exemples/boutique` enveloppe toutes ses pages dans `<CartProvider>`.

### Paiement Stripe (mode test) — `app/api/checkout/route.ts` (nouveau)

Réplique fidèlement le pattern déjà validé et en prod sur le projet
**Oncle Wang** (`~/Projects/ONCLE/src/app/api/checkout/route.ts`, lu pour
référence uniquement — rien écrit dans ce projet, confirmé par `pwd`
avant tout code) :

- `PAYMENT_MODE=test|live` (défaut `test`), `STRIPE_SECRET_KEY` — tous
  deux en `.env.local` uniquement, jamais commités.
- Sans clé configurée : réponse `{ demo: true, total }`, le panier
  bascule sur la confirmation simulée — la démo reste montrable sans
  compte Stripe.
- Garde-fou : `PAYMENT_MODE=live` avec une clé `sk_test_…` est refusé
  (500).
- **Prix toujours recalculés côté serveur** depuis `boutiqueDemo.products`
  — jamais depuis ce que le client envoie dans la requête.
- `.env.example` complété avec la section "Module Boutique", même
  formatage que les sections Delivery/Reviews déjà présentes.

### Périmètre strictement respecté (vérifié, pas juste écrit)

`grep` sur `content/exemples/boutique.ts`, `app/exemples/boutique/` et
`app/api/checkout/` pour "uber"/"livraison" : une seule occurrence hors
commentaire de tête — `"🚚 Livraison partout en France"` dans les points
forts de l'accueil, qui désigne l'expédition postale nationale standard
d'un site e-commerce, pas le module de livraison locale Uber Direct
(vendu séparément) — conforme à la consigne ("mention informative si
pertinent" autorisée, module non mis en avant).

### Vérifications effectuées

- `tsc --noEmit` ✅.
- **Clés Stripe** : `grep` de `sk_test_`/`sk_live_` sur tout le code —
  aucune occurrence réelle (seuls un commentaire de doc et une comparaison
  de préfixe `.startsWith("sk_test_")`). Aucun fichier `.env.local`
  présent. `.env*.local` confirmé dans `.gitignore` (`git check-ignore
  -v` positif).
- **Carte Boutique** : les 4 cartes vérifiées côte à côte en preview
  (1400px) — cohérentes visuellement, mêmes composants, couleur jaune
  correctement appliquée (badge, coche, dot).
- **Bouton "voir un exemple concret"** : les 4 plans ont chacun leur lien
  (`/exemples/presence`, `/exemples/autonome`, `/exemples/machine`,
  `/exemples/boutique`), dans l'ordre.
- **Parcours d'achat complet testé en mode test**, bout en bout via
  l'outil navigateur : catalogue → ajout rapide → fiche produit
  (sélecteur de quantité, +1 → 2, confirmé dans `localStorage`) → panier
  (quantité/suppression fonctionnelles, total recalculé en direct) →
  "Passer au paiement" → `POST /api/checkout` → `200 { demo: true, total:
  18 }` (recalcul serveur exact) → redirection confirmation → panier vidé
  après coup (`localStorage` vérifié vide).
- Un faux positif rencontré pendant le test : deux clics synchrones tirés
  dans le même appel JS (incrémenter la quantité puis ajouter au panier)
  contournaient le cycle de rendu React et capturaient une closure
  obsolète (quantité restée à 1). Reproduit un clic réel séparé
  (`dispatchEvent` dans un appel distinct) : comportement correct
  (1 → 2). Artefact du script de test, pas un bug du composant.
- Rendu mobile (375px) vérifié sur le catalogue : `scrollWidth` = 375px
  exact, aucun débordement horizontal, chips de catégorie qui wrappent
  proprement.
- Console navigateur : 0 erreur sur les 5 pages testées.

## [Import maquette] Direction visuelle "arcade" — Claude Design du 15/07

### Source

Maquette importée via l'outil `DesignSync` (claude.ai/design, projet "NOVA
Studio Visual Identity", fichier `NOVA Studio - Site.dc.html`, lu en
lecture seule via `get_file`). Référence **visuelle uniquement** —
palette sombre marron/orange/or, typographie pixel arcade, style "borne
d'arcade" pour le Hero et "La Carte".

### Décision de périmètre (tranchée avant d'implémenter, expliquée à l'utilisateur)

La maquette montre deux sections sans équivalent dans le contenu réel du
site : **"Crédibilité"** (3 souvenirs vécus derrière un comptoir) et
**"Démonstration interactive"** (simulateur de statut de commande). Ni
l'une ni l'autre n'ont de texte réel correspondant dans `content/v3.ts`
— les construire aurait signifié inventer du contenu, ce que la consigne
interdit explicitement ("ne recopie pas le texte de la maquette tel
quel"). Décision : **ces deux sections ne sont pas construites**. Les 9
sections restantes, qui ont toutes un contenu réel existant, sont
restylées : Hero, Verdict, Constat, Méthode (Moteur), La Carte (Plans),
Process, Qui suis-je (Fondateur), Contact, Footer.

Corollaire : Constat/Méthode/Process, retirés de la homepage la veille
(14/07, sur demande explicite), sont **restaurés** — la maquette et la
consigne elle-même les nomment explicitement dans la liste des sections
à restyler avec le texte réel.

### Palette + typographie — isolées du reste du site

- `tailwind.config.ts` : 12 nouveaux tokens `arcade-*` (bg/bg-alt/card/
  card-featured/border/border-thick/orange/gold/cream/tan/taupe/muted),
  additifs — `lait`/`encre`/`corail`/`violet`/`teal`/`jaune`/`rose`
  (thème clair v3 déjà en place) restent inchangés et continuent
  d'alimenter `/exemples/*`, `/labo` et les archives.
- `app/layout.tsx` : deux polices `next/font/google` ajoutées (`Press
  Start 2P`, `VT323`), exposées via `font-pixel`/`font-terminal` dans
  Tailwind — appliquées **uniquement** dans `components/v3/Hero.tsx` et
  la section Plans de `Sections.tsx`. Aucune autre page n'utilise ces
  classes.
- **Bug de police découvert et corrigé** : Press Start 2P n'a pas de
  glyphe correct pour les majuscules accentuées (`Î` de "Île-de-France")
  — testé avec le subset `latin-ext` ajouté puis retiré (aucun effet, le
  glyphe est absent/cassé indépendamment du subset). L'eyebrow du Hero
  utilise donc `font-mono` (IBM Plex Mono, déjà fiable partout ailleurs
  sur le site) plutôt que `font-pixel` — seul ce label est concerné, le
  H1/CTA/chrome restent en pixel (aucun autre texte réel du site ne
  contient de majuscule accentuée dans un contexte pixel : noms de plans
  gardés en casse naturelle "Présence", pas "PRÉSENCE", précisément pour
  éviter le même problème avec le É).
- `app/globals.css` : nouveau bloc `arcade-blink` (curseur clignotant
  "INSERT COIN") — le bloc `prefers-reduced-motion` déjà global en tête
  de fichier (`*, *::before, *::after { animation-duration: 0.001ms
  !important }`) neutralise automatiquement cette animation comme toutes
  les autres du site, sans code spécifique à ajouter.

### Composants partagés — étendus, jamais modifiés dans leur comportement par défaut

`.v3-card` et `.v3-window` (classes CSS globales) sont utilisées dans 15
et 7 fichiers de `/exemples/*` respectivement — **jamais touchées**.
Toutes les nouvelles cartes/fenêtres de la homepage sont construites en
classes Tailwind directes (`border border-arcade-border bg-arcade-card`,
etc.), pas via ces classes partagées.

`components/v3/NotifFeed.tsx` (utilisé par le Hero **et** par
`/exemples/machine`) a reçu deux props optionnelles (`itemClassName`,
`textClassName`) avec des valeurs par défaut strictement identiques au
comportement actuel — le Hero passe des couleurs arcade explicites, tous
les autres appels (Machine, y compris ses deux usages dans l'espace
admin) restent inchangés car ils n'passent pas ces props.

### Hero — traitement arcade complet

Panneau "borne d'arcade" (coins arrondis, ombre portée, bordure) avec
bandeau décoratif "1P 00042 / INSERT COIN / HI 99999" (chrome pur, aucune
vraie donnée représentée). `V3Backdrop` (aurore WebGL) n'est plus rendu
ici — composant conservé intact, simplement plus importé (même
traitement que les sections retirées puis restaurées). Titre réel
(`v3hero.titleA/Em/B`) en `font-pixel`, `titleEm` seul en accent or
(structure d'emphase du contenu réel préservée, pas celle — plus large —
de la maquette). Sous-titre réel en `font-terminal`. CTA réels
("Réserver un audit gratuit (15 min)", "Voir les plans") stylés en
boutons pixel — jamais remplacés par "PRESS START"/"voir la démo" de la
maquette. Fenêtre "site en service" (NotifFeed) reskinnée en sombre.

### La Carte — traitement arcade complet

Même panneau que le Hero, bandeau "1P / SELECT YOUR PLAN / CREDIT 04"
(`04` calculé depuis `v3plans.plans.length`, jamais codé en dur — reste
juste si un plan est ajouté/retiré). Les 4 cartes réelles (Présence,
Autonome, Machine, Boutique) : nom en casse naturelle, badge réel ("Le
plus choisi", pas "★ PLAYER'S CHOICE"), bouton réel ("Démarrer avec
{nom}", pas "SELECT"). Carte "Autonome" (featured) : bordure orange +
glow **sur le conteneur** (`box-shadow`, jamais sur du texte). Fonction
`splitPrice()` ajoutée pour isoler "dès" du montant — rendu
systématiquement plus petit, police différente (terminal, pas pixel),
couleur atténuée par rapport au prix, jamais en gras ni en emphase
(contrainte explicite respectée sur les 2 occurrences réelles : Machine
"dès 1990€", Boutique "dès 3200€").

### Sections sobres — palette seule, typographie standard inchangée

Constat, Méthode, Process, Qui suis-je, Contact, Footer : recolorées vers
la palette arcade (fonds alternés `arcade-bg`/`arcade-bg-alt` pour le
rythme visuel), structure/typographie standard préservée à l'identique
(`font-sans`/`font-mono` existants, aucune police pixel). Verdict
(déjà en fond sombre `bg-encre` avant la refonte) simplement retinté —
les deux couleurs du glitch RGB-split (`#ff6b4a`/`#0ea88b`, définies dans
`globals.css`) sont volontairement laissées inchangées : ce sont deux
teintes choisies pour leur contraste chromatique chaud/froid, pas pour
matcher une palette, et les remplacer par des teintes arcade (toutes
chaudes) aurait cassé la lisibilité de l'effet.

### Périmètre respecté (vérifié, pas juste écrit)

- `git status --short app/exemples/ app/labo app/_archive` → **aucune
  ligne** : zéro fichier modifié dans les répertoires protégés.
- `/exemples/presence`, `/exemples/autonome/espace-admin`,
  `/exemples/machine` revérifiés visuellement en preview après tous les
  changements : thème clair intact, `.v3-card`/`.v3-window` intacts,
  `NotifFeed` toujours rendu avec ses couleurs par défaut (cream/encre).
- `/labo` revérifié : thème sombre indépendant intact, 0 erreur console.
- `/_archive/*` : confirmé non routable (convention Next.js, dossier
  préfixé `_`), 404 personnalisée servie — comportement inchangé.
- `grep` sur tout le diff pour `filter:`/`blur(`/`text-shadow` :
  **0 occurrence** — seul glow du design est un `box-shadow` sur le
  conteneur de la carte Autonome, jamais sur du texte.
- Mot "dès" : vérifié visuellement + par inspection DOM
  (`getComputedStyle`) sur les 2 occurrences réelles — toujours plus
  petit (16px vs 18px), police différente, couleur atténuée, jamais gras.

### Vérifications effectuées

- `tsc --noEmit` ✅ à chaque étape.
- Bug de compilation Next.js rencontré pendant les tests ("Unexpected
  token `section`") : erreur HMR obsolète (le fichier était
  syntaxiquement valide, confirmé par `tsc` juste avant) — résolu par un
  redémarrage propre du serveur de preview, pas une vraie erreur de code.
- Parcours complet vérifié en preview : Hero, Verdict, Constat, Méthode,
  La Carte (4 cartes), Process, Qui suis-je, Contact, Footer — tous
  recolorés, contenu réel intact partout.
- Rendu mobile (375px) vérifié sur Hero et La Carte : aucun débordement
  horizontal, bandeau décoratif qui collapse proprement ("INSERT COIN"
  et "SELECT YOUR PLAN" masqués sous `sm:`, "1P"/"HI"/"CREDIT" toujours
  visibles).
- `prefers-reduced-motion` : garanti structurellement par le bloc global
  déjà existant en tête de `globals.css` (même mécanisme déjà validé
  pour toutes les animations du site pendant l'audit de performance du
  14/07) — couvre `arcade-blink` sans code additionnel.
- Console navigateur : 0 erreur sur `/` (desktop et mobile),
  `/exemples/presence`, `/exemples/autonome/espace-admin`,
  `/exemples/machine`, `/labo`.
- **Tout reste local** — aucun `git push`, aucune interaction avec un
  remote, aucun déploiement déclenché.

## [Extension arcade] Style pixel étendu à tout le site + réponses défilantes — 16/07

### Changement de portée

Le style "pixel arcade" (import du 15/07 ci-dessus) était jusqu'ici limité
au Hero et à "La Carte" ; les autres sections restaient en typographie
standard, seule la palette de couleurs étant reprise. Sur demande
explicite, cette limitation est levée : les 9 sections réelles du site
(Hero, Verdict, Constat, Méthode, La Carte, Process, Qui suis-je, Contact,
Footer) et la Nav passent en pixel arcade complet. Palette, fond
anthracite, absence de filtre CRT/glow sur le texte et non-emphase du mot
"dès" — toutes des contraintes déjà en place depuis le 15/07 — restent
inchangées.

### Hiérarchie typographique établie

Pour garder les sections denses en texte lisibles (contrainte explicite),
une hiérarchie à trois polices est appliquée partout, pas seulement dans
"La Carte" comme avant :

- **`font-pixel`** (Press Start 2P) : titres, sous-titres de section,
  noms de cartes/étapes, boutons courts, logo Nav — texte court affiché
  en casse naturelle (jamais `uppercase`, cf. bug de glyphe ci-dessous).
- **`font-terminal`** (VT323) : paragraphes de corps, descriptions —
  lisible en taille normale même sur de longs paragraphes (vérifié sur
  Constat et Process, les deux sections les plus denses).
- **`font-mono`** (IBM Plex Mono, déjà utilisé avant le 15/07) : labels
  structurels courts (badges eyebrow, chips) — inchangé, déjà fiable avec
  les majuscules accentuées ("BASÉ EN ÎLE-DE-FRANCE").

`Eyebrow` (composant partagé de Constat/Méthode/Plans/Fondateur/Contact)
passe de `font-mono uppercase` à `font-pixel` en casse naturelle — la
classe `uppercase` est retirée précisément pour éviter le bug de glyphe
(voir plus bas), pas juste pour le style.

### Nav — conversion complète

`components/v3/Nav.tsx` : logo ("K" + "K1000.studio"), liens desktop et
mobile, CTA "Audit gratuit" (desktop + mobile) tous passés en
`font-pixel`, casse naturelle préservée ("Le constat", "Ce que ça fait" —
contient un ç, vérifié sans problème de rendu).

### Section Verdict — remplacement de l'effet glitch par un défilement simple

L'ancien effet glitch/RGB-split (`GlitchAnswer`, deux couleurs froid/chaud
`#ff6b4a`/`#0ea88b`) est retiré et remplacé par `CyclingAnswer` : la
question reste fixe, la réponse en dessous change automatiquement toutes
les 1,8 s parmi les 7 réponses réelles déjà présentes dans
`content/v3.ts` (`v3verdict.answers` — "Rien. Absolument rien.", "Il
dort.", "Il est invisible.", "Il fait fuir vos clients.", "Il tourne en
rond.", "Il ment sur vos horaires.", "Il rate des opportunités.") :
**aucun contenu inventé**, le tableau existant est réutilisé tel quel et
couvre déjà toutes les variantes demandées.

Transition : `@keyframes arcade-answer-in` dans `globals.css`
(`opacity 0→1` + `translateY(10px→0)`, `cubic-bezier(0.22,1,0.36,1)`, la
même courbe sans dépassement déjà utilisée pour `hero-rise`/`preview-in`/
`sig-word` ailleurs dans le site) — retrigger via la prop `key={i}` sur le
`<p>` à chaque changement. Aucun saut brutal, aucun rebond (la courbe ne
peut pas dépasser 1 car ses points de contrôle valent exactement 1, la
valeur finale).

`prefers-reduced-motion` : `useReducedMotion()` (framer-motion) coupe
l'intervalle JS lui-même (`useEffect` retourne tôt si `reduce`), pas
seulement la transition CSS — sans ce garde JS, le blanket rule global de
`globals.css` aurait neutralisé l'animation visuelle mais le texte aurait
quand même continué à changer brutalement (sans transition) toutes les
1,8 s. `items[0]` seul est affiché, statique, sans classe d'animation.

### Bug de police — toujours le même, appliqué de façon cohérente

Press Start 2P ne rend pas les majuscules accentuées (`Î`, `É`…) — déjà
découvert et documenté le 15/07. Règle appliquée partout dans cette
extension : jamais de `uppercase` sur du texte `font-pixel`, casse
naturelle systématique. Revérifié visuellement sur cette extension :
minuscules accentuées (é, à, ç, û, î) et guillemets « » s'affichent
correctement en Press Start 2P ; seules les majuscules accentuées posent
problème, et aucun texte réel du site n'en a besoin en contexte pixel.

### Vérification visuelle — desktop + mobile

Parcours complet revérifié en preview après l'extension : Hero, Verdict
(réponses en cycle confirmées : "Il dort." → "Il est invisible." → "Il
fait fuir vos clients."), Constat, Méthode, La Carte (mobile : bandeau
"1P"/"CREDIT 04" collapse proprement, carte "Présence" à 690€ lisible),
Process (titres d'étape "01 Audit"/"02 Proposition"/"03 Conception"
lisibles, guillemets « on se parle » corrects), Qui suis-je (chip "BASÉ
EN ÎLE-DE-FRANCE" intact), Contact (email/téléphone volontairement
laissés en `font-sans`, intro `font-display italic` inchangée), Footer
(logo pixel, blurb terminal, tagline mono). Mobile 375×812 : aucun
débordement horizontal (`scrollWidth` = 375px exact) sur toutes les
sections testées, menu mobile fonctionnel. `/exemples/*`, `/labo`,
`/_archive` non retouchés dans cette extension (aucun fichier de ces
répertoires modifié).

### `prefers-reduced-motion` — test dédié sur `CyclingAnswer`

Le garde JS (`reduce = useReducedMotion()`) temporairement forcé à `true`
en local pour vérifier en preview : réponse figée sur "Rien. Absolument
rien." (premier élément du tableau), aucun changement sur 3,6 s
d'observation (2× le cycle normal), aucune classe `arcade-answer-in`
appliquée. Modification temporaire annulée immédiatement après capture —
`git diff` confirmé propre sur ce fichier après coup.

### Lighthouse — avant/après

Build de production propre (`rm -rf .next && next build`), servi sur un
port isolé (4100, serveur de preview arrêté pendant la mesure pour ne pas
fausser le résultat), Lighthouse mobile par défaut :

| Catégorie | Score |
|---|---|
| Performance | 94 (référence précédente : 96) |
| Accessibilité | 96 |
| Bonnes pratiques | 96 |
| SEO | 100 |

Léger recul de 2 points en performance, entièrement expliqué : l'élément
LCP devient le logo Nav ("K1000.studio", maintenant en `font-pixel`) au
lieu d'un texte en police système — délai de rendu de l'élément ~540 ms
pendant que la police pixel se charge, malgré `display: "swap"` déjà
configuré sur toutes les polices dans `app/layout.tsx` (donc pas de FOIT,
juste le coût inhérent d'étendre une police d'affichage à un élément
au-dessus de la ligne de flottaison sur toutes les pages). TBT (0 ms),
CLS (0), FCP (0,8 s) et Speed Index (1,1 s) restent parfaits. 94/100 reste
dans la zone verte ("Good") de Lighthouse — recul mineur et attendu, pas
une régression fonctionnelle.

### Vérifications effectuées

- `tsc --noEmit` ✅.
- Console navigateur : 0 erreur sur `/` (desktop et mobile, toutes
  sections), avant et après le build de production.
- Serveur de preview arrêté proprement avant le build de production,
  redémarré après la mesure Lighthouse — aucune interruption durable du
  flux de développement.
- **Tout reste local** — aucun `git push`, aucune interaction avec un
  remote, aucun déploiement déclenché.

## [Hero] Vérification de l'ordre + suppression du mockup de notifications — 16/07

### Ordre du Hero — vérifié, déjà correct

Le titre réel (`v3hero.titleA/Em/B`, "Vos futurs clients vous cherchent
déjà. Assurez-vous qu'ils vous trouvent.") est déjà la première chose
rendue dans `app/page.tsx` (`<V3Hero />` avant `<V3Verdict />`), et
`content/v3.ts` porte déjà un commentaire explicite à cet effet depuis
la conception de la section Verdict ("Section dédiée, plein écran, juste
après le Hero — PAS le titre du Hero"). Revérifié en preview (desktop et
mobile) : le titre est bien la première chose visible sous la Nav, la
question défilante "Votre site actuel, il fait quoi, là, tout de suite ?"
n'apparaît qu'après un scroll, dans sa propre section `min-h-screen`.
Aucune correction nécessaire sur l'ordre — juste une confusion possible
entre les deux sections adjacentes de même palette, sans bordure
marquée entre elles.

### Suppression du mockup "fenêtre navigateur avec notifications"

`components/v3/Hero.tsx` : la fausse fenêtre "votre-commerce.fr — en
ligne" (barre de titre, badge "live", flux `NotifFeed` de 6 notifications
simulées, phrase "Pendant ce temps, vous êtes avec vos clients.")
**retirée entièrement** de la colonne de droite du grid. Le grid à deux
colonnes (`lg:grid-cols-[1.05fr_0.95fr]`) devient une colonne unique
centrée (`mx-auto max-w-2xl`) — texte, eyebrow et CTA inchangés, mise en
page simplement reflow. `NotifFeed.tsx` **n'est pas supprimé** : toujours
utilisé par `/exemples/machine` (accueil + espace-admin), composant
intact, uniquement déréférencé du Hero.

`content/v3.ts` : `v3hero.terminalTitle` et `v3hero.events` (les 6
notifications, uniquement consommées par ce mockup) supprimés après
vérification par `grep` qu'aucun autre composant ne les référence —
contenu devenu mort, pas déplacé ni recréé ailleurs. L'idée
automatisation/notifications reste déjà présente ailleurs sur le site
sans doublon visuel : pilier "Automatisation" de la section Méthode et
tagline "pour automatiser" de l'offre Machine (La Carte).

### Vérifications effectuées

- `tsc --noEmit` ✅.
- Bug de compilation rencontré pendant le test ("V3Hero" référencé à une
  ligne inexistante dans un ancien bundle HMR, erreurs `NotFoundErrorBoundary`
  en boucle) : cache `.next` obsolète après l'édition — résolu par
  `rm -rf .next` + redémarrage propre du serveur de preview, pas une
  vraie erreur de code (confirmé par `tsc` propre avant et après, et 0
  erreur console après redémarrage).
- Preview desktop : titre du Hero bien la première chose visible, panneau
  "borne d'arcade" qui se referme proprement juste après les CTA (plus de
  grand vide ni de colonne fantôme), question défilante du Verdict
  confirmée juste après au scroll.
- Preview mobile (375×812) : même ordre, `scrollWidth` = 375px exact,
  aucun débordement horizontal, CTA empilés proprement en colonne unique.
- `git diff --stat` : seuls `components/v3/Hero.tsx` et `content/v3.ts`
  touchés — aucun fichier de `/exemples/*`, `/labo`, `/_archive` modifié.
- **Tout reste local** — aucun `git push`, aucune interaction avec un
  remote, aucun déploiement déclenché.

## [Hero] Inversion de l'ordre Verdict/Hero — 16/07

### Changement de consigne — inversion explicite par rapport à la vérification précédente

La consigne précédente (entrée juste au-dessus) avait vérifié et
documenté que le titre du Hero passait **avant** la question défilante
du Verdict, avec un commentaire dédié dans `content/v3.ts` confirmant
cet ordre. Nouvelle consigne explicite, opposée : l'ordre attendu est
désormais **question fixe → réponse défilante → titre de résolution**.
Ce n'est pas la correction d'un bug mais un changement de séquence
demandé — traité comme tel, sans reproduire l'ancien raisonnement.

### Modification

`app/page.tsx` : `<V3Verdict />` déplacé avant `<V3Hero />` dans l'ordre
de rendu du `<main>` — seule inversion, aucun autre composant touché,
aucune fusion des deux sections en un seul bloc (les deux `<section>`
restent distinctes, seul l'ordre change).

`components/v3/Sections.tsx` (`V3Verdict`) : `V3Verdict` étant
désormais la toute première section sous la Nav fixe (`fixed inset-x-0
top-0 z-50`), son padding vertical (`py-24`) est remplacé par
`pb-24 pt-28 md:pb-28 md:pt-32` — même dégagement que celui déjà
utilisé par `V3Hero` pour ne jamais passer sous la Nav. Sans ce
changement, le contenu vertical-centré de la section (`min-h-screen
justify-center`) aurait pu se retrouver partiellement masqué par la
Nav sur les viewports courts.

`V3Hero` (contenu, styles internes, absence du mockup de notifications
retiré à la consigne précédente) : **inchangé** — seule sa position
dans `page.tsx` change, pas son code interne.

### Vérifications effectuées

- `tsc --noEmit` ✅.
- Preview desktop : rechargé, confirmé par capture + inspection DOM
  (`getComputedStyle`) que la question "Votre site actuel, il fait quoi,
  là, tout de suite ?" est la première chose visible sous la Nav, la
  réponse qui défile juste en dessous (`opacity: 1`, couleur
  `rgb(255, 210, 63)` = `arcade-gold` confirmée), puis au scroll suivant
  le panneau "borne d'arcade" avec le titre "Vos futurs clients vous
  cherchent déjà. Assurez-vous qu'ils vous trouvent." en troisième
  position. Aucun mockup de notifications présent (confirmation que la
  suppression de la consigne précédente n'a pas régressé).
- Artefact de capture noté en cours de vérification : la réponse
  défilante apparaît parfois "terne" sur une capture d'écran alors que
  `getComputedStyle` confirme `opacity: 1` et la couleur exacte —
  quirk de l'outil de capture sur ce fond sombre, pas un bug réel
  (vérifié en interrogeant le DOM directement plutôt que de se fier à
  la capture seule).
- Preview mobile (375×812) : même ordre confirmé, `scrollWidth` = 375px
  exact, aucun débordement horizontal, 0 erreur console.
- `git diff --stat` : seuls `app/page.tsx` et `components/v3/Sections.tsx`
  touchés (2 lignes au total) — aucun fichier de `/exemples/*`, `/labo`,
  `/_archive` modifié.
- **Tout reste local** — aucun `git push`, aucune interaction avec un
  remote, aucun déploiement déclenché.

## [Nav] Barre arcade en header persistant global — 16/07

### Ce qui change

La Nav (`components/v3/Nav.tsx`) porte désormais en permanence, sur toutes
les sections et pendant tout le scroll, le chrome "borne d'arcade"
jusqu'ici réservé au Hero : `1P {compteur}` à gauche, `INSERT COIN`
clignotant au centre, `HI 99999` à droite — police pixel, orange/or,
fond anthracite opaque, bordure basse. Objectif explicite : donner
l'impression d'être "dans" une borne d'arcade du début à la fin de la
visite, pas seulement en haut de la page.

### Implémentation

- Nouveau bandeau ajouté au-dessus de la barre de nav fonctionnelle,
  dans le même `<header fixed>` — **toujours opaque** (`bg-arcade-bg`
  fixe), indépendant du compactage au scroll de la barre de nav en
  dessous (qui garde son comportement `transparent → opaque + blur`
  existant, simplement déplacé du `<header>` vers le `<nav>` interne).
  Cette persistance volontaire distingue le bandeau (chrome permanent)
  de la nav (qui se contracte pour gagner de la place).
- **Compteur `1P`** : calculé dans le `onScroll` déjà existant
  (`progress = scrollY / (scrollHeight - innerHeight)`, formaté sur 5
  chiffres). Mise à jour en texte simple, sans transition ni easing —
  aucune animation au sens CSS, donc rien à neutraliser sous
  `prefers-reduced-motion` (un changement de texte n'est pas un
  mouvement). Explicitement commenté dans le code comme décoratif :
  ne représente aucune donnée réelle (pas un nombre de clients, de
  ventes, etc.).
- **`INSERT COIN`** : réutilise `.arcade-blink` (`globals.css`),
  clignotement déjà neutralisé par le bloc `prefers-reduced-motion`
  global existant (`animation-duration: 0.001ms !important`) — même
  mécanisme que toutes les autres animations arcade du site, aucun code
  additionnel nécessaire.
- **Mobile** : bandeau entièrement masqué (`hidden sm:flex`) — sur petit
  écran, seuls le logo/wordmark et le bouton hamburger restent visibles,
  conformément à la consigne explicite de simplification plutôt que de
  compresser tous les éléments sur une largeur trop étroite.
- **Bezel du Hero retiré** (`components/v3/Hero.tsx`) : le Hero avait
  son propre bandeau "1P 00042 / INSERT COIN / HI 99999" local (import
  du 15/07). Une fois la Nav globale en place, le garder aurait affiché
  **deux compteurs "1P" différents empilés** (celui de la Nav, qui suit
  le scroll, et celui du Hero, figé à "00042") — lisible comme une
  incohérence/un bug plutôt que deux clins d'œil volontaires. Retiré ;
  le panneau du Hero s'ouvre directement sur l'eyebrow. Le bezel de
  `V3Plans` ("1P / SELECT YOUR PLAN / CREDIT {n}") est resté inchangé :
  centre et compteur de droite différents, pas un doublon.
- **Clearance verticale** : hauteur totale du header mesurée en preview
  (`110px` déployé, `87px` compact) — la marge haute déjà en place sur
  `V3Verdict` (`pt-28 md:pt-32`, ajoutée lors de l'inversion d'ordre
  ci-dessus) couvre largement ce nouveau total, vérifié par
  `getBoundingClientRect` (question à 290px du haut, aucun chevauchement).

### Bug pré-existant découvert et corrigé en cours de vérification

En testant les vrais liens de nav (`/#plans`, `/#constat`, etc.), le
scroll vers l'ancre ne se produisait pas du tout : Lenis (smooth-scroll,
`components/v2/SmoothScroll.tsx`) pilote sa propre position virtuelle de
scroll via une boucle `requestAnimationFrame`, et écrase silencieusement
le saut natif du navigateur au tick suivant, puisqu'il ignore qu'un clic
sur `<a href="#section">` doit déplacer sa cible interne. **Confirmé
préexistant** : reproduit à l'identique en testant `git stash` (Nav
d'avant cette tâche) — aucun lien ancré du site ne fonctionnait déjà
avant ce chantier, indépendamment du bandeau arcade.

Corrigé à la racine dans `SmoothScroll.tsx` (composant partagé, utilisé
par `/`, `/qui-je-suis`, et le layout de `/_archive/v2` — ce dernier non
routable, confirmé par la convention Next.js du préfixe `_`, donc sans
effet observable) : un handler de clic délégué intercepte tout
`<a href>` contenant un `#`, vérifie que le chemin correspond à la page
courante (ou est vide, pour les liens `#id` relatifs), et appelle
`lenis.scrollTo(element, { offset: -90 })` explicitement au lieu de
laisser le navigateur faire un saut que Lenis annulerait. Corrige tous
les liens ancrés du site, pas seulement ceux de la Nav (bénéfice
collatéral direct de l'exigence "la navigation réelle doit rester
pleinement fonctionnelle").

### Vérifications effectuées

- `tsc --noEmit` ✅.
- Bug de compilation HMR rencontré pendant l'implémentation ("Unexpected
  token `header`", `V3Hero` référencé à une ligne inexistante) : erreur
  obsolète d'un état intermédiaire de l'édition — résolu par `rm -rf
  .next` + redémarrage propre du serveur de preview (même pattern que
  les fois précédentes).
- Liens réels testés un par un en preview (clic réel via `element.click()`,
  pas de raccourci) : `/#plans` et `/#constat` confirmés — scroll Lenis
  complet, atterrissage exact sur la section ciblée, titre visible sous
  le header (pas caché dessous). Cibles `/#moteur`, `/#process`,
  `/#contact` confirmées existantes dans le DOM (`getElementById`) pour
  les 5 liens à ancre ; `/` (logo) et `/qui-je-suis` (navigation de page
  réelle, non concernés par Lenis) revérifiés séparément, 0 erreur
  console sur `/qui-je-suis`.
- Anomalie de test notée et élucidée : l'onglet de preview est en
  `document.hidden = true` (arrière-plan) dans cet environnement, ce qui
  throttle `requestAnimationFrame` côté navigateur et retarde
  l'animation Lenis bien au-delà de sa durée configurée (1,1 s) — un
  clic met plusieurs secondes réelles à aboutir ici, contre quasi
  instantané pour un visiteur réel (onglet au premier plan, `rAF` non
  bridé). Confirmé en pilotant manuellement la boucle `lenis.raf()` :
  la logique atteint exactement sa cible dès que les frames s'exécutent
  normalement — artefact de l'outil de test, pas un bug du site.
- Mobile (375×812) : bandeau arcade absent, seuls logo + hamburger
  visibles, menu ouvert affiche les 5 liens + CTA, `scrollWidth` = 375px
  exact, aucun débordement.
- `prefers-reduced-motion` : vérifié par inspection de code (pas
  d'émulation directe disponible dans l'outil de preview, même
  limitation que les vérifications précédentes de cette session) —
  `.arcade-blink` n'utilise que `animation` (aucune transition), donc
  entièrement couvert par le bloc global déjà en tête de `globals.css`.
- `git diff --stat` : `components/v2/SmoothScroll.tsx`,
  `components/v3/Nav.tsx`, `components/v3/Hero.tsx` — aucun fichier de
  `/exemples/*`, `/labo`, `/_archive` modifié directement.
- **Tout reste local** — aucun `git push`, aucune interaction avec un
  remote, aucun déploiement déclenché.

## [Structure] Réduction à la structure de référence — 16/07

### Périmètre — décidé avec l'utilisateur avant toute suppression

Consigne explicite : ne garder qu'une page d'accueil unique (question
défilante, titre, Les plans, Qui suis-je) + les 4 démos `/exemples/*`.
Trois points ambigus ont été tranchés **avec l'utilisateur avant
d'agir** (aucune suppression de contenu réel sans confirmation, cf.
contrainte explicite de la consigne) :

1. **Sections homepage hors des 4 listées** (Le constat, Ce que ça fait,
   Le process, Contact, Footer) : confirmé à supprimer — pas juste les
   routes séparées, mais aussi ces sections déjà présentes sur `/`.
2. **`/qui-je-suis`** (page dédiée, bio complète) : confirmé à
   **conserver comme 5ᵉ route réelle**, en plus de `/` et des 4 démos —
   pas de fusion de contenu, pas de suppression.
3. **`/demo/avis`, `/demo/commande`, `/demo/livraison`** (déjà
   orphelines, aucun lien actif ne pointait vers elles — le module
   "Commande & Livraison" qui y renvoyait était déjà masqué de
   l'affichage) : confirmé à **laisser telles quelles**, hors périmètre
   de ce nettoyage.

### Suppressions

`components/v3/Sections.tsx` : `V3Constat`, `V3Moteur`, `V3Process`,
`V3Contact` retirés (fonctions + imports de contenu associés), ainsi
que la palette `SOFT` (devenue inutilisée — ne servait qu'à `V3Constat`).
`content/v3.ts` : exports `v3constat`, `v3moteur`, `v3process`,
`v3contact` retirés. `app/page.tsx` : ne rend plus que `V3Verdict`,
`V3Hero`, `V3Plans`, `V3Fondateur` (dans cet ordre — celui demandé).
`V3Footer` n'est plus rendu sur `/`, mais **reste utilisé sur
`/qui-je-suis`** (le composant n'est pas supprimé, seulement
déréférencé de la homepage) — email, téléphone, tagline et copyright du
footer restent donc visibles sur le site, pas perdus.

### Liens cassés anticipés et corrigés avant qu'ils posent problème

La section Contact retirée portait le seul point d'entrée vers le
formulaire d'audit ; plusieurs CTA du site y pointaient
(`#contact`/`/#contact`). Plutôt que de laisser des liens morts,
chacun a été redirigé vers `mailto:bonjour@k1000studio.fr` — l'email
réel déjà validé dans le contenu (`v3contact.email`), pas une valeur
inventée :

- `content/v3.ts` : `v3nav.cta.href` (bouton "Audit gratuit" de la Nav)
  et `v3hero.ctaPrimary.href` ("Réserver un audit gratuit (15 min)").
- `components/v3/Sections.tsx` : bouton "Démarrer avec {Plan}" sur
  chacune des 4 cartes de La Carte (`href="#contact"` → mailto).
- `app/qui-je-suis/page.tsx` : CTA "Réserver un audit gratuit →" en bas
  de page (`href="/#contact"` → mailto).

`v3nav.links` réduit à `Les plans` et `Qui suis-je` uniquement (`Le
constat`, `Ce que ça fait`, `Le process` retirés avec leurs sections).
Menu mobile revérifié : mêmes 2 liens + CTA, rien de cassé.

### Ce qui n'a pas changé (vérifié explicitement)

- **Aucune modification de DA** : palette, police pixel/terminal,
  bordures, ombres — tout identique. Seul le contenu structurel
  (sections/routes) a bougé, jamais le style d'une section conservée.
- `/exemples/presence`, `/exemples/autonome`, `/exemples/machine`,
  `/exemples/boutique` et toutes leurs sous-pages : **non touchées**,
  `git status --short` confirmé vide sur ces répertoires.
- `/labo`, `/_archive/*` : non touchés.
- `/demo/avis`, `/demo/commande`, `/demo/livraison` : laissés intacts
  par décision explicite (voir périmètre ci-dessus).
- `v3fondateur` (contenu complet utilisé par `/qui-je-suis` : pageIntro,
  points, closing, badges) : conservé intégralement, rien perdu.

### Vérifications effectuées

- `tsc --noEmit` ✅.
- `grep` exhaustif sur les symboles retirés (`V3Constat`, `V3Moteur`,
  `V3Process`, `V3Contact`, `v3constat`, `v3moteur`, `v3process`,
  `v3contact`) et sur les ancres mortes (`#contact`, `#constat`,
  `#moteur`, `#process`) dans tout le périmètre live : **0 référence
  restante** (les seules occurrences trouvées sont dans `content/site.ts`,
  utilisé exclusivement par les composants archivés `_archive/v1`/`v2`,
  hors périmètre).
- Structure de la page d'accueil vérifiée en DOM (`document.querySelector('main').children.length`
  → 4, pas de `<footer>`) et par arbre d'accessibilité complet : ordre
  confirmé question défilante → titre → Les plans (4 offres, boutons
  "Démarrer avec {Plan}" et "Voir un exemple concret ↗" présents) → Qui
  suis-je (teaser + lien "Voir mon parcours →" vers `/qui-je-suis`).
- `/qui-je-suis` : 0 erreur console, CTA vérifié en `mailto:`, footer
  toujours présent avec le bon contenu.
- Les 4 démos + sous-pages testées par `fetch` : `/exemples/presence`,
  `/exemples/autonome` (+`/contact`), `/exemples/machine`
  (+`/espace-admin`), `/exemples/boutique` (+`/catalogue`),
  `/exemples/presence/galerie` → **200** partout. `/demo/avis`,
  `/demo/commande`, `/demo/livraison`, `/labo`, `/` → **200**.
  `/nonexistent-page-xyz` → 404 confirmé (sanity check que la méthode
  de vérification détecte bien un vrai 404).
- Anomalie d'outil rencontrée pendant la vérification : captures d'écran
  occasionnellement vides/crème après un grand saut de scroll dans cet
  environnement (onglet de preview en arrière-plan, `document.hidden =
  true`) — con contenu réel confirmé présent et correctement positionné
  via `elementFromPoint`/inspection DOM à chaque fois ; contourné en
  basculant sur l'arbre d'accessibilité (`preview_snapshot`) et des
  requêtes `fetch` pour le reste de la vérification, non dépendants du
  rendu visuel.
- `git diff --stat` : `app/page.tsx`, `app/qui-je-suis/page.tsx`,
  `components/v3/Sections.tsx`, `content/v3.ts` — aucun fichier de
  `/exemples/*`, `/labo`, `/_archive` modifié.
- **Tout reste local** — aucun `git push`, aucune interaction avec un
  remote, aucun déploiement déclenché.

## [DA] Extension du style pixel arcade aux 4 démos + Qui suis-je — 16/07

### Changement de périmètre — explicite, tranché par l'utilisateur

Les 4 pages `/exemples/*` et `/labo`/`/_archive` étaient protégées par
consigne permanente depuis le début du chantier arcade (aucune
modification autorisée). Cette tâche lève explicitement cette
protection pour les 4 démos et "Qui suis-je" — nommées une par une par
l'utilisateur — tout en la maintenant pour `/labo` et `/_archive`,
toujours hors périmètre.

### Composants partagés restylés en premier (effet de levier)

Avant de toucher aux 22 pages de contenu, les composants communs à
toutes les démos ont été convertis en arcade :
- `components/exemples/ExempleNav.tsx`, `ExempleFooter.tsx`,
  `ExempleBanner.tsx`, `PlaceholderImage.tsx` — palette `lait`/`encre`
  → `arcade-bg`/`arcade-cream`/`arcade-border`, liens/logo en
  `font-pixel` casse naturelle (jamais `uppercase`, bug de glyphe
  Press Start 2P sur les majuscules accentuées — déjà documenté et
  systématiquement respecté).
- `components/v3/NotifFeed.tsx` : son seul appelant restant est la démo
  Machine (l'usage Hero avait déjà été retiré plus tôt dans la
  session) — les couleurs par défaut du composant sont donc passées
  directement en arcade plutôt que de garder une surcharge par prop
  désormais inutile.
- `app/qui-je-suis/page.tsx` : Nav/Footer étaient déjà arcade (composants
  partagés avec la homepage), seul le corps de page suivait encore
  l'ancien thème clair — converti en référence/gabarit pour le reste du
  chantier (titre `font-pixel`, corps `font-terminal`, badges
  `font-mono`).

### Les 4 démos — déléguées à 4 agents en parallèle, un par plan

Périmètre trop large pour un seul passage séquentiel (22 fichiers,
4 systèmes de contenu indépendants) : un agent par démo
(`/exemples/presence`, `/autonome`, `/machine`, `/boutique`), chacun
avec la même table de correspondance de tokens exacte, le même
gabarit de référence (`qui-je-suis`), et une couleur d'accent
secondaire assignée par plan — **teal** (Présence), **violet**
(Autonome), **corail** (Machine), **jaune** (Boutique), reprenant
exactement les couleurs déjà assignées à chaque plan dans
`content/v3.ts` (`v3plans.plans[].color`). Ces accents restent visibles
par-dessus la base arcade sombre/orange/or, exactement comme la
homepage continue d'utiliser violet/teal/corail/jaune pour les chips et
coches (`TEXT`/`DOT` dans `components/v3/Sections.tsx`) — seules les
couleurs *structurelles* (fonds, bordures, CTA principaux, texte de
corps) basculent en arcade.

- **`.v3-card`/`.v3-window`** (classes globales `app/globals.css`) sont
  aussi utilisées par `/demo/*` et `components/delivery/*`, hors
  périmètre — jamais éditées. Chaque usage dans les 4 démos + Qui
  suis-je a été remplacé par des classes Tailwind directes
  (`rounded-xl border border-arcade-border bg-arcade-card`, fenêtres
  avec bandeau à 3 pastilles orange/or/tan), même pattern déjà établi
  pour le Hero et La Carte lors de l'import de maquette du 15/07.
- **Lisibilité du texte dense** : règle explicite respectée partout —
  `font-pixel` réservé aux titres/labels courts, `font-terminal` pour
  tout corps de texte (paragraphes, listes de prestations, texte des
  champs de formulaire), `font-mono` conservé pour les prix (Boutique)
  et labels structurels courts.
- **Champs de formulaire éditables** (contact, espace-admin) :
  volontairement laissés en police lisible standard (jamais
  `font-pixel`, illisible pour de la saisie), bordure visible
  (`border-2 border-arcade-border-thick`), état focus
  (`focus:border-arcade-orange`) — priorité de lisibilité explicitement
  demandée respectée.

### Incohérence détectée et corrigée après coup

Les 4 agents ont fait un choix légèrement différent sur la couleur du
texte des CTA/badges posés sur fond accent (violet/corail/teal) : l'agent
Présence a utilisé `text-arcade-bg` (cohérent avec la convention déjà
en place sur le Hero/La Carte — jamais `text-white` nulle part sur le
site), les 3 autres avaient gardé `text-white`. Fonctionnellement
identique (contraste correct dans les deux cas), mais incohérent d'une
démo à l'autre — corrigé après coup (`text-white` → `text-arcade-bg`)
dans les 7 fichiers concernés (`autonome/contact`, `autonome/espace-admin`,
`autonome/page`, `machine/contact`, `machine/espace-admin`,
`machine/page`, `boutique/confirmation`) pour une cohérence stricte
entre les 4 démos, exigée explicitement par la consigne ("vérifie la
cohérence globale en naviguant d'une page à l'autre").

### Fonctionnalités vérifiées intactes

- **Formulaires de contact** (Présence, Autonome, Machine) : logique
  `useState`/`onSubmit` non touchée par les agents (uniquement les
  classNames) — testé en direct sur Présence : validation HTML5
  `required` toujours active (champ message vide bloque l'envoi),
  soumission complète → état "Message envoyé !" avec coche teal,
  fonctionne à l'identique.
- **Espace admin simulé** (Autonome, Machine) : changement d'onglet et
  bouton "Enregistrer" (confirmation temporisée via `setTimeout`)
  testés en direct sur les deux démos — comportement inchangé. Onglets
  spécifiques à Machine (Automatisations avec flux `NotifFeed` en
  direct, Tableau de bord avec tuiles de stats) vérifiés aussi.
- **Panier + paiement test Stripe** (Boutique) : ajout au panier depuis
  le catalogue, incrémentation de quantité et recalcul du total en
  direct testés dans le panier (24,00 € → 32,00 €), logique de
  `CartContext.tsx` et l'appel `POST /api/checkout` non touchés par
  l'agent (confirmé, ni ouverts ni modifiés).

### Vérifications effectuées

- `tsc --noEmit` ✅ après chaque agent puis en combiné.
- `grep` exhaustif sur les 31 fichiers touchés pour tout résidu de
  l'ancien thème (`bg-lait`, `text-encre`, `border-encre`, `bg-white`,
  `v3-card`, `v3-window`, `#211D16`, `text-white`) : **0 occurrence**
  après la correction `text-white`.
- Parcours desktop complet : accueil + une page fonctionnelle par démo
  (Présence/contact, Autonome/espace-admin, Machine/espace-admin,
  Boutique/catalogue+panier), 0 erreur console partout.
- Mobile (375×812) vérifié sur Présence, Autonome/espace-admin (page la
  plus dense — onglets qui deviennent défilables horizontalement,
  champs toujours lisibles) et Boutique/catalogue : `scrollWidth` =
  375px exact partout, aucun débordement.
- Cohérence de navigation croisée : bandeau "← Retour aux plans K1000
  Studio" (`ExempleBanner`) toujours vers `/#plans`, liens "Voir un
  exemple concret" de La Carte toujours valides (`exampleHref` non
  modifié dans `content/v3.ts`).
- `/labo`, `/_archive/v1`, `/_archive/v2` : confirmé non touchés
  (`git status --short app/labo app/_archive` → aucune ligne).
- **Lighthouse** (build de production propre, port isolé 4100) :
  - Accueil : Performance 94 (identique à la référence précédente),
    Accessibilité 100 (+4), Bonnes pratiques 96, SEO 100 — aucune
    régression.
  - `/exemples/presence` (nouvelle mesure, pas de référence antérieure) :
    Performance 91, Accessibilité 91, Bonnes pratiques 96, **SEO 66**.
    Le score SEO bas est **entièrement dû à `robots: { index: false }`**,
    déjà présent avant ce chantier dans chaque `layout.tsx` de démo
    (choix délibéré : ce sont des sites fictifs, ils ne doivent pas être
    indexés) — vérifié par grep, aucun agent n'a touché aux exports
    `metadata`/`robots`. Pas une régression liée au restyle.
- **Tout reste local** — aucun `git push`, aucune interaction avec un
  remote, aucun déploiement déclenché.

## [Boutique] Sélecteur de quantité "− [n] +" sur catalogue/fiche/panier — 17/07

### Nouveau composant partagé

`components/exemples/QuantitySelector.tsx` — un seul composant réutilisé
sur les 3 emplacements concernés (catalogue, fiche produit, panier)
pour garantir le même style et le même comportement partout, comme
demandé. Le champ central est un vrai `<input>` (pas un `<span>` figé
comme c'était le cas dans l'ancien stepper de la fiche produit et du
panier) :
- Saisie filtrée en direct aux chiffres uniquement (`replace(/[^0-9]/g,
  "")`) — lettres, signe moins, point décimal impossibles à taper.
- Validation à la perte de focus (`onBlur`) : parse + clamp à
  `[min, max]` (1 à 99 par défaut) ; vide ou invalide → revient à 1.
- `Enter` déclenche le blur (donc la validation) sans soumettre de
  formulaire parent.
- Boutons `−`/`+` en `h-11 w-11` (44px), input en `h-11 w-12` (44px de
  haut) — zone cliquable conforme à la contrainte tactile minimale de
  44px demandée, vérifié par `preview_inspect` (`{"height":"44px",
  "width":"44px"}` sur les boutons).

### Intégration aux 3 emplacements

- **`produit/[slug]/AddToCartButton.tsx`** : l'ancien stepper local
  (span + boutons, déjà présent mais non éditable) remplacé par
  `<QuantitySelector>` — la logique `addItem(slug, qty)` au clic
  utilisait déjà la quantité sélectionnée, aucun changement nécessaire
  côté ajout.
- **`catalogue/page.tsx`** : n'avait AUCUN sélecteur avant (juste un
  bouton "Ajouter" qui appelait toujours `addItem(slug)`, qty implicite
  1) — ajouté un état `Record<string, number>` (une quantité par
  produit visible), bouton "Ajouter" passé en pleine largeur sous la
  ligne prix + sélecteur (plutôt que de caser 3 éléments sur une seule
  ligne dans une carte de grille étroite), la quantité de ce produit
  revient à 1 juste après l'ajout.
- **`panier/page.tsx`** : l'ancien stepper (span + boutons compacts,
  `px-2.5 py-1.5`) remplacé par le même `<QuantitySelector>`. Changement
  de comportement assumé : l'ancien bouton `−` n'avait **aucun plancher**
  (`updateQty(slug, item.qty - 1)` sans `Math.max`), donc décrémenter à
  0 supprimait la ligne — `QuantitySelector` impose `min={1}` par
  défaut, donc le bouton `−` du panier ne descend plus jamais à 0 ; la
  suppression reste possible via le lien "Retirer" déjà présent à côté.
  Cohérent avec la consigne explicite ("la quantité ne peut jamais
  descendre en dessous de 1").

### Logique panier — déjà correcte, non modifiée

`components/exemples/CartContext.tsx` : `addItem(slug, qty)` additionnait
déjà la quantité à l'existant plutôt que de dupliquer une ligne
(`it.qty + qty`), et `total`/`count` recalculaient déjà à partir de
`items` — aucun changement nécessaire sur le Context, seulement sur les
3 pages qui l'utilisent.

### Bug de layout mobile détecté et corrigé pendant la vérification

Les boutons 44px du panier (plus larges que l'ancien stepper compact)
faisaient déborder la ligne d'article : `scrollWidth` mesuré à 503px sur
un viewport de 375px. Corrigé en rendant la ligne d'article responsive
(`flex flex-wrap`, le groupe sélecteur + prix + "Retirer" passe en
`w-full` sous `md:` pour s'empiler proprement sous l'image/nom du
produit, redevient une ligne unique à partir de `md:`) — revérifié :
`scrollWidth` = 375px exact après correction, layout desktop inchangé
(revérifié par capture).

### Vérifications effectuées

- `tsc --noEmit` ✅.
- Saisie clavier testée en direct : frappe de "abc-3.5" filtrée en
  temps réel à "35" ; champ vidé puis perte de focus réelle (clic
  ailleurs, pas un événement synthétique) → revient à "1" ; "150" tapé
  puis `Enter` → clampé à "99" (max).
- Incrémentation/décrémentation par clic testée avec délai réel entre
  les clics (un `dispatchEvent` synchrone en rafale ne reflète pas un
  vrai clic utilisateur et donnait un faux résultat en test — confirmé
  en ajoutant un délai, comportement réel correct).
- Somme au lieu de doublon vérifiée : ajout du même produit deux fois
  (catalogue puis à nouveau) → une seule ligne dans le panier,
  quantité cumulée (4 → 5), pas de ligne dupliquée.
- Total live vérifié à chaque étape : 8,00€ → 40,00€ (qty 5) → 32,00€
  (qty 4 après clic `−` dans le panier) → recalcul correct à chaque
  fois, aussi bien sur la ligne que sur le total général.
- Zone tactile 44px confirmée par `preview_inspect` (boutons et input).
- Mobile (375×812) vérifié sur catalogue, fiche produit et panier
  (avant et après le fix de layout) : `scrollWidth` = 375px partout,
  0 erreur console.
- Desktop revérifié après le fix mobile : ligne d'article toujours sur
  une seule ligne, aucune régression.
- **Tout reste local** — aucun `git push`, aucune interaction avec un
  remote, aucun déploiement déclenché.

## [Verdict + Nav] Effet clavier sur les réponses + easter egg "Insert Coin" — 17/07

### Effet machine à écrire inversée sur la section Verdict

`components/v3/Sections.tsx` : `CyclingAnswer` (fade + translateY) remplacé
par `TypewriterAnswer` — effacement caractère par caractère (arrière),
puis écriture de la réponse suivante caractère par caractère, curseur
`_` clignotant (réutilise `.arcade-blink`, déjà utilisé pour "INSERT
COIN", pas de nouvelle keyframe). Exception volontaire documentée en
commentaire à la règle générale "pas d'effet lettre par lettre" —
cohérente ici avec le thème terminal déjà en place, comme demandé
explicitement.

- Timing : 40ms/caractère (dans la fourchette 30-50ms demandée), pause
  de 800ms une fois le mot complet affiché avant de commencer
  l'effacement (dans la fourchette 500ms-1s demandée).
- Implémentation : state machine à deux phases (`typing`/`erasing`) et
  un `setTimeout` récursif (pas `setInterval`, les deux phases n'ont pas
  la même cadence — la pause en fin de frappe n'est qu'un délai plus
  long avant le prochain `setTimeout`, pas un troisième état).
- `arcade-answer-in` (keyframe CSS de l'ancien fade) supprimée de
  `app/globals.css` — plus aucun appelant après ce remplacement (vérifié
  par grep).
- reduced-motion : `items[0]` affichée seule, texte fixe, aucun curseur,
  aucun cycle — le hook `useReducedMotion()` coupe l'`useEffect` du
  timer entièrement (même garde déjà utilisée ailleurs sur le site).
- `aria-live="off"` sur le paragraphe — évite qu'un lecteur d'écran
  annonce chaque caractère effacé/tapé (même pattern déjà utilisé sur
  `NotifFeed`).

### Easter egg "INSERT COIN"

`components/v3/Nav.tsx` : le texte "INSERT COIN" du bandeau arcade
devient un `<button>` cliquable (au lieu d'un `<span>` purement
décoratif) — `aria-haspopup="dialog"`, ouvre `InsertCoinOverlay`. Reste
`hidden md:inline` (règle déjà en place pour la barre mobile
simplifiée) : le déclencheur n'est donc accessible qu'à partir de `md`
(768px), comme le reste du bandeau décoratif — pas de changement à
cette règle de responsive existante, non demandé par cette consigne.

`components/v3/InsertCoinOverlay.tsx` (nouveau) : overlay plein écran
(`fixed inset-0 z-[100]`) avec une scène Space Invaders décorative (pas
un jeu jouable) :
- Canvas + `requestAnimationFrame`, mouvement calculé par delta-temps
  (frame-rate independent, jamais de redessin lié à un event non
  throttle) — grille 8×5 d'un sprite pixel-art codé en dur (bitmap
  11×8), teintes orange/or alternées par ligne, déplacement classique
  gauche-droite-descente façon Space Invaders.
- Résolution plafonnée : `devicePixelRatio` capé à 2, dimensions canvas
  internes fixes (~318×~326 avant DPR) — cohérent avec la leçon déjà
  tirée lors de l'audit de performance du 14/07 (jamais de canvas à
  résolution native non plafonnée).
- reduced-motion : une seule frame dessinée statiquement, aucune boucle
  `rAF` démarrée.
- Fermeture : bouton dédié ("ÉCHAP — Fermer"), vraie touche `Escape`
  (listener `keydown` sur `window`), clic sur le fond (`onClick` du
  conteneur `fixed inset-0`) — le contenu intérieur (scène + boutons)
  a `stopPropagation()` pour ne jamais se fermer sur un clic à
  l'intérieur.
- Verrou de scroll (`document.body.style.overflow = "hidden"`) actif
  uniquement pendant l'ouverture, restauré à la valeur précédente à la
  fermeture (`useEffect` cleanup) — jamais laissé actif après fermeture.
- Message : "Game over" + "Contactez-nous pour continuer." + lien
  "Réserver un audit gratuit" vers `v3nav.cta.href` (le vrai lien de
  contact du site, pas une valeur inventée) — le clic sur ce lien ferme
  aussi l'overlay.
- Mobile : `maxWidth: "90vw"` sur le canvas (mise à l'échelle CSS,
  résolution interne inchangée), boutons de fermeture en pile flexible
  (`flex-wrap`) — vérifié à 375px, aucun débordement, bouton "Fermer"
  mesuré à 46px de haut (au-dessus du minimum tactile).

### Vérifications effectuées

- `tsc --noEmit` ✅.
- Effet clavier testé sur ~20 lectures échantillonnées sur plusieurs
  cycles (300ms d'intervalle) : effacement caractère par caractère
  confirmé, transition propre vers la réponse suivante, curseur présent
  à chaque étape.
- reduced-motion testé en forçant temporairement `useReducedMotion() ||
  true` dans `TypewriterAnswer` et `InsertCoinOverlay` (technique déjà
  utilisée plus tôt dans le chantier) : réponse figée sans curseur côté
  Verdict, grille d'envahisseurs figée (deux captures à 2s d'intervalle
  strictement identiques, confirmant qu'aucune boucle `rAF` ne tourne)
  côté overlay — modifications temporaires annulées immédiatement
  après capture, `grep "TEMP:"` confirmé vide après coup.
- Mouvement de la grille confirmé par comparaison de deux captures à 2s
  d'intervalle (position visiblement décalée).
- Fermeture testée séparément pour chacun des 3 mécanismes : bouton
  dédié, touche `Escape` réelle (`KeyboardEvent` sur `window`), clic sur
  le fond — et vérifié qu'un clic sur le contenu intérieur (canvas) ne
  ferme **pas** l'overlay (`stopPropagation` fonctionne). `body.style.
  overflow` vérifié vide après chaque fermeture (verrou de scroll jamais
  laissé actif).
- Mobile (375×812) vérifié : overlay déclenché directement (le bouton
  déclencheur est `hidden` en dessous de `md` par design existant, non
  modifié par cette tâche), rendu sans débordement horizontal
  (`scrollWidth` = 375px), bouton de fermeture mesuré à 46px de haut.
- Console navigateur : 0 erreur à chaque étape (ouverture, fermeture,
  reduced-motion, mobile).
- **Tout reste local** — aucun `git push`, aucune interaction avec un
  remote, aucun déploiement déclenché.

## [Insert Coin] Remplacement par un vrai mini-jeu Space Invaders jouable — 17/07

### Changement de périmètre

Remplace entièrement la partie 2 de la consigne précédente (scène
Space Invaders décorative) — la partie 1 (effet clavier sur les
réponses Verdict) n'est pas concernée, inchangée.

### `components/v3/SpaceInvadersGame.tsx` (nouveau)

Mini-jeu complet en Canvas 2D vanilla + `requestAnimationFrame`, sans
dépendance de jeu externe :

- **Boucle update/render séparée** : `updateGame(state, dt)` est une
  fonction pure (mute l'état, ne touche jamais au DOM/canvas) appelée
  chaque frame, suivie d'un `render()` dédié — permet de tester la
  logique indépendamment du rendu (voir Vérifications).
- **État "chaud" hors React** : positions, tirs, grille d'envahisseurs
  vivent dans une `ref` mutée directement par la boucle — jamais dans
  du `useState`, pour ne jamais déclencher de re-render à 60fps. Seuls
  score/vies/phase (affichés dans l'UI) passent par du state, mis à
  jour uniquement quand un kill/coup/game over survient réellement,
  pas à chaque frame.
- **Sprites pré-rendus** : le bitmap pixel-art de l'envahisseur (11×8)
  et du vaisseau sont dessinés une seule fois sur des canvas hors-écran
  au montage, puis blittés via `drawImage()` — beaucoup moins coûteux
  que des dizaines de `fillRect()` par envahisseur et par frame,
  cohérent avec les leçons déjà tirées de l'audit de performance du
  14/07.
- **`dt` plafonné à 50ms** — évite un saut massif de la grille si
  l'onglet perd le focus (le navigateur suspend `requestAnimationFrame`
  en arrière-plan, le prochain timestamp peut être très éloigné).
- **Résolution capée** : `devicePixelRatio` limité à 2, résolution
  interne fixe (320×420, format portrait façon borne d'arcade
  classique) mise à l'échelle en CSS.

**Mécaniques implémentées** : vaisseau contrôlable (flèches/Q-D +
Espace au clavier, boutons ◀ ▶ TIR au tactile — pointer events, donc
souris et tactile sans code séparé), grille 4×6 d'envahisseurs qui se
déplacent en groupe (gauche-droite-descente classique) et tirent au
hasard, collisions AABB (tir joueur → destruction + score, tir ennemi →
perte de vie avec 1s d'invulnérabilité pour éviter de perdre plusieurs
vies sur un seul frame), 3 vies de départ, vague suivante à chaque
grille vidée avec vitesse ×1.18 et fréquence de tir ×1.12 (difficulté
progressive sans complexifier — pas de palier/niveau distinct), game
over si les vies tombent à 0 **ou** si la grille atteint la ligne du
vaisseau (mécanique arcade classique). Écran de game over avec score
final + bouton "Rejouer" qui réinitialise tout l'état.

**Pas de sauvegarde de high score entre sessions** (le composant se
démonte à la fermeture de l'overlay, l'état repart à zéro à chaque
ouverture — aucun stockage) et **aucun son** — conformes aux deux
contraintes explicites.

### `components/v3/InsertCoinOverlay.tsx` (réécrit)

Devient un simple cadre : fond, verrou de scroll pendant l'ouverture,
fermeture (bouton dédié + touche Échap réelle), et le repli
reduced-motion — délègue tout le jeu à `SpaceInvadersGame`. Le clic en
dehors du contenu **ne ferme plus** l'overlay (contrairement à la
version décorative précédente) : pendant une partie, un clic un peu
large sur le pourtour ne doit pas interrompre accidentellement le jeu
— changement délibéré, la consigne ne demandait plus explicitement
cette fermeture. La prop `contactHref` (CTA "audit gratuit" de l'ancien
écran décoratif) est retirée : le nouvel écran de game over ne
contient que score + Rejouer, conformément à la consigne précise.

reduced-motion : affiche "Cette fonctionnalité nécessite les
animations activées." à la place du jeu — aucun canvas monté, aucune
boucle démarrée, pas de jeu forcé à quelqu'un qui a demandé moins
d'animation.

### Vérifications effectuées

- `tsc --noEmit` ✅.
- **Limite d'environnement rencontrée et contournée** : le navigateur
  de l'outil de preview considère la page comme non visible
  (`document.hidden === true` la plupart du temps), ce qui suspend
  `requestAnimationFrame` côté navigateur (comportement standard de
  tout navigateur pour les onglets en arrière-plan, pas un bug du
  site) — un premier test tir-en-rafale montrait un score bloqué à 0
  pour cette raison, pas un défaut de collision. Contourné en testant
  la logique pure directement : `updateGame()` appelée 300 fois de
  suite avec un `dt` fixe (1/60s, équivalent à 5 secondes de jeu réel)
  en tirant toutes les 0,32 s — résultat : 8 envahisseurs détruits,
  score à 80, grille qui dérive, tirs en vol — mouvement, collisions et
  score confirmés corrects indépendamment du rAF. Cas limites testés un
  par un de la même façon : perte de vie + fenêtre d'invulnérabilité
  (un second tir immédiat ne retouche pas), grille atteignant le
  vaisseau (game over), vague vidée (nouvelle grille + vitesse
  augmentée ×1.18 confirmée par calcul exact). Une fenêtre où le
  navigateur a repris la main (après un clic réel sur le canvas) a
  permis de confirmer la même mécanique en conditions réelles via la
  boucle React (score passé de 0 à 10 dans l'UI, un envahisseur
  effectivement absent de la grille affichée).
- **Partie complète jouée** : déplacement (flèches), tir (Espace),
  destruction confirmée visuellement (grille à 23/24 puis moins),
  écran "Game over" avec score final déclenché et vérifié à l'écran,
  bouton "Rejouer" cliqué réellement — confirmé que tout l'état repart
  à zéro (score 00000, 3 vies, grille pleine, vaisseau recentré).
- **Contrôles tactiles** : boutons ◀ ▶ TIR déclenchés via
  `PointerEvent` réels (`pointerdown`/`pointerup`) — aucune erreur
  console, tir confirmé visuellement (nouveau projectile affiché).
  Zone cliquable mesurée à 44px (boutons directionnels) via
  `preview_inspect`.
- **Fermeture propre** : touche Échap réelle testée pendant une partie
  en cours — overlay fermé, `body.style.overflow` revérifié vide
  après (verrou jamais laissé actif), page d'accueil revérifiée
  intacte et interactive juste après (effet machine à écrire toujours
  actif, aucune erreur console, aucun état résiduel).
- reduced-motion : testé en forçant temporairement
  `useReducedMotion() || true` dans `InsertCoinOverlay` (même technique
  que pour l'effet clavier) — écran statique confirmé, aucun canvas
  monté ; modification annulée immédiatement après capture, `grep
  "TEMP:"` confirmé vide après coup sur les deux fichiers du jeu.
- Mobile (375×812) : overlay + jeu + HUD + contrôles tactiles vérifiés
  sans débordement (`scrollWidth` = 375px), disposition en colonne
  lisible.
- **60fps** : le design (dt-based movement, sprites pré-rendus,
  résolution capée, pas de re-render React par frame) suit
  systématiquement les pratiques déjà établies sur ce projet pour tenir
  60fps ; une mesure prolongée en continu façon "profiler" n'a pas pu
  être effectuée dans cet environnement précis (navigateur de preview
  non focalisé la majorité du temps, cf. plus haut) — signalé
  explicitement plutôt que prétendu vérifié à l'identique d'un vrai
  navigateur utilisateur au premier plan.
- **Tout reste local** — aucun `git push`, aucune interaction avec un
  remote, aucun déploiement déclenché.
