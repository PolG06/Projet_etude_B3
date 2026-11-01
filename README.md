# Inventaire infra Althea

Arborescence de l'infrastructure d'Althea Systems, classée **par site** (Paris, Genève, OVH Site 1, OVH Site 2). Chaque machine a sa fiche : ce qu'elle fait, la technologie choisie, ses ressources et leur justification, et où elle est redondée.

La page a quatre onglets :

- **Inventaire** : les serveurs par site, avec leur fiche.
- **Schéma** : le schéma d'architecture dessiné dans draw.io.
- **Postes clients** : les logiciels **ajoutés** par Althea sur les postes d'accès (portables de Paris, clients légers de Genève) et dans les sessions VDI, avec pour chacun à quoi il sert, comment il est installé, configuré et mis à jour. Deux rubriques à part : les **applications web** (tout ce qui s'utilise par le navigateur, pour les utilisateurs et pour les équipes techniques, avec où y accéder), et ce qui est **retiré de Windows** dans le modèle VDI, avec la raison.
- **Serveurs physiques** : les machines physiques de chaque site (Paris, Genève…) avec leurs ressources, leurs disques, leur réseau, le cluster dont elles font partie et un commentaire sur ce qu'elles font. La fiche liste aussi ce qui tourne dessus, d'après `infra.csv`.

Le dépôt contient ces fichiers :

| Fichier | Rôle |
|---|---|
| `index.html` | La page. Vous n'avez pas besoin d'y toucher. |
| `infra.csv` | **Les machines** : une ligne par serveur, VM, conteneur ou service. |
| `vlans.csv` | **Les VLAN** de chaque site : numéro, nom, réseau, passerelle, DHCP, usage. |
| `infra-shema.drawio.svg` | **Le schéma d'architecture**, modifiable dans draw.io. |
| `postes.csv` | **Les logiciels des postes clients** : une ligne par agent ou logiciel. |
| `physiques.csv` | **Les serveurs et équipements physiques** : une ligne par machine. |
| `README.md` | Ce mode d'emploi. |

Chaque onglet a sa propre adresse : `…/#inventaire`, `…/#schema`, `…/#postes`, `…/#physiques`. Pratique pour envoyer directement le schéma à quelqu'un.

## Publier sur GitHub Pages

1. Créez un dépôt sur GitHub et déposez-y tous les fichiers (**Add file > Upload files**, puis **Commit changes**).
2. **Settings > Pages** : choisissez **Deploy from a branch**, la branche `main`, le dossier `/ (root)`, puis **Save**.
3. Une à deux minutes plus tard, la page est en ligne sur `https://<votre-compte>.github.io/<nom-du-dépôt>/`.

## Modifier les données

Ouvrez `infra.csv` dans Excel, LibreOffice Calc, ou directement dans l'éditeur de GitHub (icône crayon). **Une ligne = un élément.**

| Colonne | Contenu |
|---|---|
| `id` | Identifiant court et unique, sans espace (ex. `dc-dns`). Sert à rattacher les éléments entre eux. |
| `parent` | `id` de l'élément qui le contient. Exemple : `awx` a pour parent `k8s-paris`, le cluster qui le fait tourner. Vide pour un élément placé directement dans un site. |
| `site` | Site de l'élément : `Paris`, `Genève`, `OVH Site 1`, `OVH Site 2`… À remplir seulement pour les éléments sans parent (les autres prennent le site de leur parent). |
| `nom` | Nom affiché. |
| `type` | Un de ces mots : `physique`, `hyperviseur`, `vm`, `cluster`, `conteneur`, `service`, `groupe`. |
| `fonction` | Ce que fait la machine. |
| `technologie` | Technologie retenue et pourquoi. |
| `vlan` | Numéro du ou des VLAN de la machine. Plusieurs VLAN : séparez-les par `\|` (le premier est affiché dans l'arbre). Le nom et le réseau du VLAN viennent de `vlans.csv`. |
| `ip` | Adresse IP. Plusieurs interfaces : `admin: 10.10.10.21 \| cluster: 10.10.71.21 \| idrac: 10.10.10.121` (la première IP est affichée dans l'arbre). |
| `vcpu`, `ram`, `stockage` | Ressources en nombres : vCPU (threads pour un serveur physique), RAM en Go, stockage total en Go. |
| `justif_ressources` | Pourquoi ces quantités de ressources. Affiché sous les ressources dans la fiche. |
| `reseau` | Informations réseau. |
| `replique_de` | Pour une machine de secours : `id` de la machine qu'elle redonde. La page affiche alors « Réplique de… » d'un côté et « Redondé sur… » de l'autre, avec un lien entre les deux. |
| `redondance` | Comment l'élément est redondé (réplication, fréquence…). |
| `note` | Remarque libre. |
| `cluster` | Facultatif. `id` du cluster dont la VM fait partie quand ses VM sont sur plusieurs hyperviseurs (ex. `wazuh-paris`). La ligne du cluster n'apparaît alors plus dans l'arbre (ses VM sont sous leurs hyperviseurs) : chaque VM porte l'étiquette « Membre de… », et sa fiche donne le lien vers le cluster (composition, conteneurs, réplique OVH) et la liste des autres membres. |

### Le fichier vlans.csv

Une ligne par VLAN et par site : `site`, `vlan` (numéro), `nom`, `reseau` (ex. `10.10.30.0/24`), `passerelle`, `dhcp`, `usage`. Un clic sur un site affiche la liste de ses VLAN.

> **Les numéros de VLAN et les adresses IP sont vides pour l'instant** : les colonnes `vlan` et `ip` d'infra.csv, et `vlan`, `reseau` et `passerelle` de vlans.csv, sont à remplir quand le plan d'adressage sera décidé. Les réseaux prévus (nom, DHCP, usage) restent listés dans vlans.csv. Les adresses données en exemple dans ce README ne servent qu'à montrer le format.

Les totaux d'un site (sous son nom, et au clic) comptent tous les **serveurs** : VM, clusters Kubernetes, serveurs physiques et hyperviseurs. Les ressources additionnent les VM, les clusters et les serveurs physiques ; celles des hyperviseurs sont affichées à part (« Capacité des hyperviseurs ») car elles sont déjà utilisées par leurs VM. Les conteneurs, groupes et services managés ne sont pas comptés.

**Un cluster Kubernetes se traite comme une VM** : sa ligne porte directement les ressources de tous ses nœuds (vCPU, RAM, stockage additionnés), leurs adresses dans la colonne `ip` (`API : 10.10.31.20 | nœud1: 10.10.31.21 | …`), et le détail par nœud dans `justif_ressources`. Les conteneurs ont pour `parent` l'`id` du cluster. Pas de ligne VM entre le cluster et ses conteneurs.

**Exception : un cluster dont les VM tournent sur des hyperviseurs différents** (K3s commun, K3s site web). Sa ligne `cluster` garde les conteneurs mais **n'a pas de ressources** ; chaque VM a sa propre ligne, rangée sous son hyperviseur (`parent` = `hyperviseur-infra-1`…). La page ne compte pas un cluster sans ressources, pour éviter un double comptage.

**Rangement par hyperviseur** : chaque VM du cluster Proxmox a pour `parent` son nœud préféré (`hyperviseur-infra-1` à `4`). C'est la répartition en temps normal : la haute disponibilité peut la déplacer après une panne.

**Ajouter un serveur** : ajoutez une ligne. **Ajouter un site** (une future filiale) : écrivez simplement son nom dans la colonne `site`, il apparaît automatiquement.

### Le schéma (infra-shema.drawio.svg)

Le schéma est enregistré en **SVG modifiable** : une image que la page affiche directement, mais qui contient aussi le schéma draw.io complet. Vous pouvez donc continuer à le modifier dans draw.io.

**Créer le fichier** (une seule fois) :

1. Ouvrez votre schéma dans draw.io (application ou app.diagrams.net).
2. **Fichier > Exporter en > SVG**.
3. Cochez **Inclure une copie de mon diagramme** (c'est ce qui le rend modifiable), puis **Exporter**.
4. Nommez le fichier exactement `infra-shema.drawio.svg` et déposez-le dans le dépôt, à côté de `index.html`.

**Le modifier ensuite** : ouvrez directement `infra-shema.drawio.svg` dans draw.io (**Fichier > Ouvrir**), modifiez, enregistrez (**Ctrl+S**) : draw.io réécrit le même fichier SVG. Déposez-le à nouveau sur GitHub. Plus besoin de garder un `.drawio` à part.

L'onglet **Schéma** propose zoom (−, +, 100 %, Ajuster) et un bouton de téléchargement. Il fonctionne aussi quand `index.html` est ouvert par un double-clic, sans Internet.

Ordre de recherche de la page : `infra-shema.drawio.svg`, puis `infra-shema.drawio.png` (PNG modifiable, même principe), puis `infra-shema.drawio` (affiché avec le visualiseur en ligne de diagrams.net, qui demande Internet). Si le fichier est vide, la page l'indique.

### Le fichier postes.csv

Une ligne par agent ou logiciel. Séparateur point-virgule, UTF-8.

| Colonne | Contenu |
|---|---|
| `id` | Identifiant court et unique (ex. `vdi-fortiedr`). |
| `cible` | `Poste d'accès` (portables de Paris et clients légers de Genève), `Session VDI`, ou les deux séparés par `\|`. Sert aux filtres. |
| `categorie` | Rubrique d'affichage (Système, Sécurité, Applications…). Les rubriques s'affichent dans l'ordre du fichier. |
| `nom` | Nom affiché. |
| `statut` | `Installé`, `Pilote uniquement`, `Option`, `À préciser`, `Web`, `Retiré` ou `Non installé`. Chaque statut a sa couleur ; `Web` alimente le filtre « Applications web » et `Retiré` le filtre « Retirés de Windows ». Pour une application web, `installation` indique où y accéder et `configuration` comment on s'y connecte. |
| `role` | À quoi ça sert (pour un logiciel retiré : ce que c'est). |
| `technologie` | Logiciel retenu et pourquoi (pour un logiciel retiré : par quoi il est remplacé). |
| `installation` | Comment il est installé : WAPT sur les PC, image maître des clients légers, modèle VDI (pour un logiciel retiré : comment il est retiré). |
| `configuration` | Comment il est configuré (GPO, Ansible, console…). |
| `mises_a_jour` | Comment il est mis à jour. |
| `serveur` | `id` du ou des serveurs de `infra.csv` concernés, séparés par `\|`. La fiche affiche un lien qui ouvre ce serveur dans l'onglet Inventaire. |
| `licence` | Licence ou coût. |
| `note` | Remarque libre. Pour un logiciel retiré ou non installé, c'est la raison affichée en tête de fiche. |

### Le fichier physiques.csv

Une ligne par serveur ou équipement physique. Séparateur point-virgule, UTF-8. L'onglet regroupe les machines par site, avec un résumé par site (nombre de serveurs, threads, RAM et stockage utile additionnés).

| Colonne | Contenu |
|---|---|
| `id` | Identifiant court et unique (ex. `phy-infra1`). |
| `site` | Site physique : `Paris`, `Genève`… Un nouveau site apparaît automatiquement, avec son filtre. |
| `emplacement` | Où se trouve la machine (salle serveur, local technique, baie…). |
| `nom` | Nom affiché. |
| `modele` | Référence du matériel (ex. `Lenovo ThinkSystem SR655 V3`). Vide si elle n'est pas encore choisie. |
| `categorie` | Rôle : hyperviseur d'infrastructure, hyperviseur VDI, serveur de sauvegarde, pare-feu… |
| `cluster` | Cluster dont la machine fait partie (ex. « Cluster Proxmox principal (nœud VDI) »), ou `Aucun`. |
| `processeur` | Processeurs (ex. `2 × 16 cœurs`). |
| `threads` | Nombre de threads (nombre). Vide pour un équipement réseau : il est alors compté comme équipement et non comme serveur. |
| `ram` | RAM en Go (nombre). |
| `stockage_detail` | Disques (ex. `2 × SSD 480 Go en miroir + 6 × NVMe 7,68 To`). |
| `stockage_utile` | Stockage utile en Go (nombre). |
| `reseau` | Cartes et liens réseau. |
| `inventaire` | `id` de la même machine dans `infra.csv`. La fiche affiche alors ce qui tourne dessus et un lien vers l'onglet Inventaire. |
| `justif_config` | Pourquoi ce modèle et cette configuration (besoin calculé, choix du processeur, de la mémoire, des disques). Affiché en tête de fiche. |
| `inclus` | Ce qui est livré avec le serveur de base. Éléments séparés par ` · ` ; dans chaque élément, ` — ` sépare l'élément de ce qu'il apporte. |
| `composants` | Ce qu'il faut commander en plus du serveur de base, même format : `Composant — pourquoi il est ajouté · Composant — pourquoi…`. |
| `commentaire` | Ce que fait la machine. Affiché en tête de fiche et sous le nom dans la liste. |

### Enregistrer depuis Excel

Choisissez **Fichier > Enregistrer sous > CSV UTF-8 (délimité par des virgules)**. Excel français utilise le point-virgule, la page accepte les deux. Les fichiers enregistrés en CSV « classique » sont aussi lus, mais le format UTF-8 est le plus sûr pour les accents.

## Voir la page sur son ordinateur

Si vous ouvrez `index.html` par un double-clic, le navigateur interdit à la page de lire les fichiers toute seule. La page vous propose alors de **choisir les fichiers** (`infra.csv`, `vlans.csv`, `postes.csv`, `physiques.csv`, `infra-shema.drawio.svg`), un par un ou tous ensemble, ou de les glisser sur la page. Elle reconnaît chaque fichier et l'affiche dans le bon onglet. Sur GitHub Pages, tout se charge automatiquement.

Autre solution en local : dans le dossier, lancez `python -m http.server`, puis ouvrez `http://localhost:8000`.

Le bouton **Ouvrir un fichier** permet aussi de prévisualiser une version modifiée avant de la déposer sur GitHub.
