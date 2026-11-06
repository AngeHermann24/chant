# Chants CTF — affichage des paroles pour OBS Studio

Système en 2 pages web statiques pour afficher les paroles de chants pendant
le culte, sur le modèle du *Bible Plugin for OBS Studio* :

- **`browser.html`** : la page d'affichage, à ajouter comme **source Navigateur**
  dans la scène OBS (texte blanc sur fond transparent).
- **`dock.html`** : la page de contrôle, à ajouter comme **dock de navigateur
  personnalisé** dans l'interface OBS.
- **`chants.json`** : la base de données des chants. C'est le **seul fichier à
  modifier** pour ajouter ou corriger des chants.

Les deux pages communiquent par `BroadcastChannel` (canal `chants-obs`), avec
un repli automatique sur `localStorage` pour les anciennes versions d'OBS.
Aucun framework, aucun serveur, aucune étape de build.

---

## 1. Mise en ligne (hébergement gratuit)

Les pages doivent être servies en `http(s)` : ouvrir `dock.html` en `file://`
ne fonctionne pas (le navigateur bloque la lecture de `chants.json`).

### Option A — GitHub Pages

1. Créez un compte sur <https://github.com> puis un dépôt nommé par exemple
   `chants-ctf`.
2. Téléversez `dock.html`, `browser.html`, `chants.json` (et ce README)
   à la racine du dépôt (*Add file → Upload files*).
3. Dans **Settings → Pages**, choisissez *Deploy from a branch*,
   branche `main`, dossier `/ (root)`, puis **Save**.
4. Après une minute, vos pages sont en ligne :
   - `https://<votre-compte>.github.io/chants-ctf/dock.html`
   - `https://<votre-compte>.github.io/chants-ctf/browser.html`

Pour mettre à jour les chants plus tard : modifiez `chants.json` directement
sur GitHub (crayon *Edit*) — le bouton **Recharger les chants** du dock
relit le fichier sans recharger la page.

### Option B — Netlify

1. Créez un compte sur <https://www.netlify.com>.
2. Glissez-déposez le dossier contenant les 3 fichiers dans
   **Sites → Add new site → Deploy manually**.
3. Netlify donne une adresse du type `https://<nom>.netlify.app` :
   - `https://<nom>.netlify.app/dock.html`
   - `https://<nom>.netlify.app/browser.html`

Pour mettre à jour : redéployez le dossier avec le `chants.json` modifié.

---

## 2. Configuration dans OBS Studio

### La source Navigateur (affichage)

1. Dans la scène de culte, **Sources → + → Navigateur**.
2. URL : `https://<votre-adresse>/browser.html?pos=bas&taille=64&fond=0.55`
3. Largeur : `1920`, Hauteur : `1080` (comme votre canvas).
4. Décochez « Ralentir l'audio… » si présent, laissez le CSS personnalisé vide.
5. Placez/redimensionnez la source à l'écran (le texte est centré en bas).

**Options d'URL de la page d'affichage :**

| Paramètre | Valeurs | Défaut | Effet |
|---|---|---|---|
| `pos`    | `bas`, `centre`, `haut` | `bas` | Position verticale du texte |
| `taille` | nombre de pixels | `64` | Taille de départ des caractères |
| `fond`   | `0` à `1` | `0.55` | Opacité du voile noir derrière le texte (`0` = totalement transparent) |

Exemples :
- `browser.html?pos=bas&taille=72&fond=0` — texte en bas, sans voile.
- `browser.html?pos=centre&taille=48&fond=0.7`

La taille se **réduit automatiquement** si une ligne est trop longue pour
tenir sur la largeur de l'écran.

### Le dock de contrôle

1. Menu **Affichage → Docks → Docks de navigateur personnalisés…**
   (nom selon la version : *Custom Browser Docks*).
2. Nom : `Chants` — URL : `https://<votre-adresse>/dock.html`.
3. Le panneau « Chants » apparaît ; ancrez-le où vous voulez.

> Les deux pages doivent être sur **le même site** (même adresse de base)
> pour communiquer entre elles.

---

## 3. Utilisation pendant le culte

- **Rechercher** : tapez le titre ou le numéro — les accents et les
  majuscules sont ignorés (« ete » trouve « Été »). Les suggestions
  s'affichent sous le champ : `↑`/`↓` pour naviguer, `Entrée` pour choisir.
- **Afficher une strophe** : cliquez dessus — elle passe en surbrillance.
  **Re-cliquez** pour la masquer.
- **Boutons** : `◀ Précédent`, `Suivant ▶`, `Refrain` (va au refrain suivant),
  `Masquer`.
- **Clavier** (hors champ de recherche) : `←`/`→` = strophe précédente /
  suivante, `Échap` = masquer.
- La barre de statut indique le **chant sélectionné** et ce qui est
  **réellement à l'écran**.
- Le dernier chant ouvert est **mémorisé** : il se rouvre tout seul au
  prochain lancement du dock.
- **Recharger les chants** : relit `chants.json` sans recharger la page
  (utile après une modification du fichier).

---

## 4. Ajouter ou modifier un chant

Éditez uniquement `chants.json`. Format :

```json
{
  "chants": [
    {
      "numero": "7",
      "titre": "Titre du chant",
      "strophes": [
        { "type": "strophe",   "lignes": ["Première ligne", "Deuxième ligne"] },
        { "type": "refrain",   "lignes": ["Ligne du refrain", "Suite"] },
        { "type": "interlude", "lignes": ["( instrumental )"] },
        { "type": "pont",      "lignes": ["Ligne du pont", "Suite du pont"] }
      ]
    }
  ]
}
```

Règles :

- `numero` : texte libre (`"12"`, `"A-03"`…), affiché tel quel.
- `type` : `strophe`, `refrain`, `interlude` ou `pont` (étiquette + couleur
  dans le dock). Tout autre type est accepté mais affiché en gris.
- `lignes` : **2 lignes maximum** par strophe (la page d'affichage n'en
  montre que 2). Une seule ligne est possible.
- Entre chaque chant : une virgule. Pas de virgule après le dernier.

Astuce : validez le fichier sur <https://jsonlint.com> si le dock n'affiche
plus rien après une modification.

---

## 5. Dépannage

| Problème | Solution |
|---|---|
| « Impossible de lire chants.json » | Les fichiers doivent être servis en `http(s)` ; vérifiez que `chants.json` est dans le même dossier et valide (jsonlint). |
| Le texte n'apparaît pas à l'écran | Vérifiez que le dock et la source Navigateur pointent vers **le même site** ; rechargez la source Navigateur (clic droit → *Rafraîchir*). |
| Le dock est blanc/vide | Rechargez le dock : menu *Affichage → Docks*, ou redémarrez OBS. |
| Accents cassés dans le JSON | Enregistrez `chants.json` en **UTF-8**. |
