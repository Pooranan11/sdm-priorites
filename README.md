# Priorités SDM

Petit site statique qui transforme le fichier de suivi d'actions `Suivi_actions_SDM.xlsx` en **matrice d'Eisenhower** et en **Kanban**, pour savoir d'un coup d'œil quoi traiter en priorité.

- Un seul fichier : `index.html` (HTML, CSS et JavaScript vanilla, sans build).
- Le fichier Excel est lu **uniquement dans le navigateur** avec [SheetJS](https://sheetjs.com/) 0.18.5, et modifié avec [JSZip](https://stuk.github.io/jszip/) 3.10.1 (CDN jsDelivr, versions figées, avec contrôle d'intégrité SRI). Aucune donnée n'est envoyée sur un serveur : la page interdit même toute requête réseau sortante via sa Content-Security-Policy (`connect-src 'none'`).
- Responsive (téléphone) et compatible mode clair / mode sombre (suit le réglage du système).

## Utilisation

1. Ouvrir `index.html` (double-clic) ou l'URL GitHub Pages.
2. Cliquer sur **Charger mon Excel**, ou glisser-déposer le fichier n'importe où sur la page.
3. Basculer entre **Matrice** et **Kanban**, filtrer par **famille**, cocher **Afficher les terminées** au besoin.

### Changer le statut d'une action

Sur chaque carte :
- le **bouton rond** marque l'action « Fait » (un second clic la rouvre en « À faire ») ;
- le **menu de statut** permet de choisir À faire, En cours, En attente fournisseur ou Fait ;
- dans la vue **Kanban**, on peut aussi **glisser une carte** vers une autre colonne.

Une notification permet d'**annuler** le dernier changement.

**Écriture dans le fichier Excel**

- **Chrome ou Edge sur ordinateur** : le changement est écrit **immédiatement dans le fichier** `.xlsx` sur votre disque. Au premier changement, le navigateur demande l'autorisation de modifier le fichier (choisir « Autoriser à chaque visite » évite de la redonner). Seule la cellule Statut de la ligne concernée est modifiée : mise en forme, listes déroulantes, formules, tableaux et autres onglets sont conservés, et Excel recalcule les formules (Urgent, Important, Quadrant) à l'ouverture. Avant chaque écriture, le fichier est relu, pour ne jamais écraser une modification faite entre-temps dans Excel. Quand vous revenez sur l'onglet, le site relit le fichier s'il a changé.
- **Safari, Firefox, téléphone** : ces navigateurs ne permettent pas à une page web d'écrire dans un fichier. Les changements sont appliqués à une copie, et un bouton **Télécharger l'Excel mis à jour** apparaît dans l'en-tête.

**Fermez le fichier dans Excel** pendant que vous modifiez les statuts depuis le site. Sous Windows, Excel verrouille le fichier ouvert (le site affiche alors un message et annule le changement). Sur Mac, enregistrer dans Excel écraserait les changements faits depuis le site.

Le format `.xls` (ancien) est lisible mais pas modifiable : enregistrez-le en `.xlsx`.

### Mémorisation

Le dernier fichier chargé (ses actions, son nom et la date de chargement) ainsi que la vue choisie sont mémorisés dans le `localStorage` du navigateur : à la réouverture, les priorités sont recalculées à la date du jour. **Effacer les données** vide cette mémoire.

Tant qu'aucun fichier n'est chargé, des données d'exemple fictives sont affichées, et signalées par un bandeau.

> Une connexion Internet est nécessaire au premier chargement de la page pour récupérer SheetJS depuis le CDN. Les données, elles, ne quittent jamais le poste.

## Format attendu du fichier Excel

Onglet **Actions**, ligne 1 = en-têtes. Les colonnes sont retrouvées par leur nom (sans tenir compte de la casse ni des accents), dans n'importe quel ordre :

| Colonne | Valeurs |
|---|---|
| ID | libre (numéro de ligne si vide) |
| Action | texte (lignes vides ignorées, ainsi que la ligne qui commence par « (Exemple) ») |
| Famille | Run quotidien, Incidents, Problem, Changes, Fournisseurs, Sécurité / CVE, Gouvernance, Amélioration, Administratif |
| Interlocuteur | texte |
| Échéance | date Excel, ou texte `jj/mm/aaaa` |
| Priorité | P1 à P5, ou `-` |
| Bloquant, Impact client, Visible manager | Oui / Non |
| Statut | À faire, En cours, En attente fournisseur, Fait |
| Commentaire | texte (facultatif) |

Les colonnes Urgent, Important et Quadrant peuvent exister dans le fichier, mais elles sont **ignorées** : tout est recalculé par le site. Un message d'erreur explicite s'affiche si l'onglet ou une colonne obligatoire manque.

Les dates sont lues comme numéros de série Excel et converties avec `XLSX.SSF.parse_date_code`, ce qui évite les décalages de fuseau horaire.

## Règles de calcul

- **Urgent** : échéance dans 2 jours ou moins (ou dépassée), ou priorité P1/P2, ou Bloquant = Oui.
- **Important** : Impact client = Oui, ou Visible manager = Oui, ou priorité P1/P2, ou famille Incidents / Problem / Fournisseurs / Sécurité / CVE.

| | Urgent | Pas urgent |
|---|---|---|
| **Important** | 1. Faire maintenant (rouge) | 2. Planifier (ambre) |
| **Pas important** | 3. Déléguer / regrouper (bleu) | 4. Reporter / abandonner (gris) |

Les actions au statut « Fait » sortent de la matrice. Dans la matrice, les actions sont triées par échéance la plus proche ; dans le Kanban, par quadrant puis par échéance.

## Déploiement sur GitHub Pages

1. Pousser `index.html` et `README.md` sur un dépôt GitHub (le `.gitignore` exclut les fichiers Excel : **ne jamais versionner le fichier de suivi**, il contient des données professionnelles).
2. Dans le dépôt : **Settings → Pages → Build and deployment → Source : Deploy from a branch**, branche `main`, dossier `/ (root)`, puis **Save**.
3. Le site est disponible après une ou deux minutes à `https://<utilisateur>.github.io/<depot>/`.

Sur un compte GitHub gratuit, Pages exige un dépôt public : seul le code est publié, et le fichier Excel reste sur votre poste.

## Note de sécurité

La version 0.18.5 de SheetJS (dernière version publiée sur npm) a des vulnérabilités connues, déclenchées par des fichiers piégés (CVE-2023-30533, CVE-2024-22363). Le risque est faible ici, puisque le site ne lit que vos propres fichiers, localement. Pour l'éliminer, remplacez la balise `<script>` par une version plus récente distribuée sur `cdn.sheetjs.com` (pensez alors à ajuster la Content-Security-Policy et le hash SRI).
