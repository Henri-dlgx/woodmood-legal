# woodmood-legal

Conditions générales d'utilisation de l'application WoodMood, plus le fichier de
version qui déclenche une nouvelle acceptation quand elles changent.

Publié par GitHub Pages :

| Fichier | URL publique | Rôle |
|---|---|---|
| `conditions.html` | https://henri-dlgx.github.io/woodmood-legal/conditions.html | Le texte, affiché dans l'application |
| `version.json` | https://henri-dlgx.github.io/woodmood-legal/version.json | Ce que l'application interroge au démarrage |
| `remise.json` | https://henri-dlgx.github.io/woodmood-legal/remise.json | La liste de remise installateur, lue par l'application |
| `remise.html` | https://henri-dlgx.github.io/woodmood-legal/remise.html | La même liste, lisible dans un navigateur |

L'application (`Henri-dlgx/WM_App-French`) ne code en dur que l'adresse de
`version.json`. Tout le reste — le texte **et** l'adresse de la page — se pilote
depuis ce dépôt, sans jamais republier l'application.

## Les deux façons de modifier les conditions

**Correction mineure** (faute de frappe, reformulation, mise en page) :
modifiez `conditions.html`, poussez. **Ne touchez pas** à `version.json`.
Personne n'est re-sollicité, tout le monde voit le texte corrigé.

**Modification substantielle** (une clause change de sens, on ajoute une
obligation, on modifie la responsabilité…) : modifiez `conditions.html`, **puis**
le champ `version` de `version.json`. À la prochaine ouverture de l'application,
chaque utilisateur retrouve le panneau de connexion, la case décochée et un
message « les conditions ont changé ». Tant qu'il n'a pas relu et recoché, il ne
se reconnecte pas.

C'est cette séparation qui compte : c'est **vous** qui décidez ce qui mérite de
réveiller tous les utilisateurs, pas la date du fichier.

Pensez à mettre à jour, en même temps que `version`, la date affichée en tête de
`conditions.html` (et le champ `updated`) — ce sont les seules choses qui
permettent à un utilisateur de vérifier quelle version il a sous les yeux.

## `version.json`

```json
{
  "version": "2026-09-07",
  "url": "https://henri-dlgx.github.io/woodmood-legal/conditions.html",
  "updated": "2026-09-07",
  "note": "commentaire libre, ignoré par l'application"
}
```

- **`version`** — chaîne libre, comparée telle quelle à ce que le téléphone a
  mémorisé. Une date ISO se lit bien dans un journal ; `1.0`, `1.1` marcherait
  aussi. Seule règle : ne jamais réutiliser une valeur déjà publiée.
- **`url`** — où l'application va chercher le texte. Permet de déplacer les
  conditions ailleurs plus tard (autre hébergeur, domaine dédié) sans republier
  l'application. Si le champ manque, l'application retombe sur l'adresse par
  défaut ci-dessus.

Le JSON doit rester valide : une virgule en trop et l'application ne peut plus
lire la version. Elle est écrite pour survivre à ce cas — les utilisateurs déjà
inscrits continuent de se connecter sur la dernière version acceptée — mais plus
aucune nouvelle acceptation ne sera demandée tant que le fichier est cassé.
Vérification rapide avant de pousser :

```bash
python3 -m json.tool version.json
```

## Délai de propagation

Le CDN de GitHub Pages peut servir l'ancien `version.json` pendant une dizaine de
minutes après le `git push`. Un changement de version n'est donc pas instantané
pour tout le parc — c'est sans conséquence, mais inutile de s'inquiéter si votre
propre téléphone ne demande pas l'acceptation dans la minute.

## Ce que l'application fait exactement

1. Au démarrage, elle lit `version.json` (sans cache, 4 s de délai maximum).
2. Elle compare le champ `version` à la version mémorisée sur le téléphone.
3. Identiques → connexion normale.
4. Différentes, ou aucune version mémorisée → panneau de connexion, case à
   cocher désactivée tant que les conditions n'ont pas été ouvertes, bouton
   « Connecter » désactivé tant que la case n'est pas cochée.
5. **`version.json` injoignable** (hors ligne, Pages en panne) → un utilisateur
   déjà inscrit se connecte normalement sur sa dernière version acceptée. Une
   panne de réseau ne doit jamais empêcher quelqu'un de piloter son poêle.

## La liste de remise

L'installateur doit présenter l'appareil au client avant de partir. `remise.json`
est la liste des points à couvrir ; le client la confirme dans l'application, et
**c'est le poêle qui retient la confirmation**, pas le téléphone.

Conséquence directe : un client qui change de téléphone, réinstalle
l'application ou la partage avec son conjoint n'est jamais re-sollicité. Seule
une réinitialisation d'usine du poêle remet le compteur à zéro — ce qui est
correct, puisqu'un poêle réinitialisé est un poêle réinstallé.

### Une seule liste, deux affichages

`remise.json` est l'unique source. `remise.html` la lit au chargement,
l'application la lit de son côté. **Ne recopiez jamais les points en dur** dans
l'une ou l'autre : la première divergence donnerait à un client une liste
différente de celle qu'il confirme.

```bash
python3 -m json.tool remise.json    # avant chaque push
```

### Modifier la liste

| Ce que vous faites | Effet |
|---|---|
| Reformuler le texte d'un point | Immédiat, personne n'est re-sollicité |
| Ajouter ou retirer un point | Immédiat pour les remises à venir ; les remises déjà confirmées ne bougent pas |
| Changer un `id` | À éviter — voir ci-dessous |

Les `id` (`logs`, `fault`, `transmit`…) sont enregistrés dans le poêle avec la
confirmation. Ils constituent la trace de ce qui a été présenté, des années après.
Réutiliser un `id` pour un point qui veut dire autre chose rend cette trace
mensongère : préférez toujours un nouvel `id` et laissez l'ancien disparaître.

### Ce qui ne se fait PAS ici

Changer le champ `version` de `remise.json` **ne re-sollicite personne**, et c'est
délibéré. C'est la différence de fond avec `version.json` :

- les conditions sont un accord avec une **personne** — elles peuvent changer, et
  la personne doit alors les ré-accepter ;
- la remise est un fait à propos d'une **installation** — elle a eu lieu ou non,
  et redemander deux ans plus tard « votre installateur vous a-t-il montré ? »
  n'aurait aucun sens.

Le champ `version` de `remise.json` sert uniquement à savoir, en relisant une
confirmation ancienne, quelle liste le client avait sous les yeux ce jour-là.

### Ce que l'application fait exactement

1. Au démarrage, elle lit `version.json` et y trouve l'adresse de `remise.json`
   (champ `remise`). Seule l'adresse de `version.json` est codée en dur dans
   l'application.
2. Une fois connectée, elle reçoit du poêle l'état de la remise.
3. Confirmée → rien ne s'affiche, jamais.
4. Non confirmée → un bandeau demande si l'installateur a fait la démonstration.
   « Oui » ouvre la liste ; le client coche, valide, et le poêle enregistre.
   « Pas encore » écarte le bandeau pour cette session seulement, et propose la
   page `remise.html` — un client à qui personne n'a rien montré doit pouvoir
   apprendre par lui-même, pas seulement se faire relancer.
5. **Poêle injoignable, ou poêle d'une génération antérieure qui n'annonce rien**
   → aucun bandeau. Le bandeau ne s'affiche que sur une réponse explicite
   « non confirmé » venue du poêle, jamais sur une absence de réponse.

### La clause de transmission

Le dernier point de la liste (`transmit`) engage le client à transmettre les
mêmes consignes à toute personne qui se sert du poêle chez lui. Il double la
section 4 des conditions : la clause donne la portée juridique, la case rend
l'engagement explicite au moment de la remise. Si vous modifiez l'une, relisez
l'autre.
