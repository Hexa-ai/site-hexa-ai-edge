# Publier le site Edge sur GitHub Pages

Ce dépôt contient le site produit **HAI-P200-4G**, destiné à être publié sur `https://edge.hexa-ai.fr`.
Il suit les mêmes conventions que le site principal [`Hexa-ai/site-hexa-ai`](https://github.com/Hexa-ai/site-hexa-ai) : site statique, hébergement GitHub Pages, domaine personnalisé via fichier `CNAME`.

---

## ⚠️ Ce qui se passe pour Odoo

Aujourd'hui, `edge.hexa-ai.fr` est un **CNAME vers `hexa-ai.odoo.com`** : c'est un Odoo Online (SaaS) qui répond sur ce nom, avec la documentation (`/knowledge`), le service client (`/helpdesk`), le portail (`/my`, `/web`) et l'ancien formulaire de contact (`/contactus`).

Un nom de domaine ne pouvant pointer que vers un seul hébergeur, brancher GitHub Pages sur `edge.hexa-ai.fr` retire ce nom à Odoo. **Odoo continue de fonctionner normalement sur `https://hexa-ai.odoo.com`** — c'est son adresse d'origine, elle répond déjà aujourd'hui (vérifié : `https://hexa-ai.odoo.com/knowledge/article/65` renvoie la même documentation, sans redirection forcée vers le domaine personnalisé).

Deux dispositions ont donc été prises dans le dépôt :

1. **Les boutons « Documentation » pointent désormais vers `https://hexa-ai.odoo.com/knowledge/article/65`** (8 liens sur les 4 pages). Ils resteront valides avant comme après la bascule.
2. **Le fichier `404.html` rattrape les anciennes URLs Odoo.** GitHub Pages sert cette page pour tout chemin inconnu ; un script y détecte les chemins Odoo (`/knowledge`, `/helpdesk`, `/my`, `/web`, `/contactus`, `/odoo`, `/blog`, `/shop`, `/event`, `/forum`, `/slides`, `/survey`, `/appointment`) et renvoie le visiteur vers le même chemin sur `hexa-ai.odoo.com`. Un lien `edge.hexa-ai.fr/helpdesk/12` déjà parti dans un mail client continuera donc d'aboutir. Les autres chemins inconnus affichent une vraie page 404 aux couleurs de la charte.

> Cette reprise se fait en JavaScript, avec un code HTTP 404 : parfait pour un humain qui clique un vieux lien, mais invisible pour les moteurs de recherche. Si des URLs Odoo sont bien référencées dans Google, la solution propre reste de les faire disparaître de l'index ou de les republier sous `hexa-ai.odoo.com`.

**Deux points à traiter côté Odoo, avant ou juste après la bascule :**

- Dans *Paramètres → Technique → Paramètres système*, passer `web.base.url` à `https://hexa-ai.odoo.com` pour que les futurs mails (tickets, invitations portail, partages Knowledge) contiennent des liens valides. Vérifier au passage que `web.base.url.freeze` est bien à `True`, sinon la valeur se réécrit à la première connexion.
- Retirer `edge.hexa-ai.fr` de la configuration de domaine du site web Odoo, pour éviter qu'Odoo continue de générer des URLs sur un nom qu'il ne sert plus.

Si tu préfères ne rien risquer dans un premier temps, publie d'abord sur l'URL GitHub par défaut (`https://hexa-ai.github.io/site-edge-hexa-ai/`) et ne branche le domaine qu'ensuite : voir l'étape 5.

---

## 1. Contenu du dépôt

```
.
├── index.html              → https://edge.hexa-ai.fr/            (FR)
├── 404.html                page 404 + reprise des anciennes URLs Odoo
├── securite.html           → /securite.html                      (FR)
├── en/
│   ├── index.html          → /en/                                (EN)
│   └── security.html       → /en/security.html                   (EN)
├── img/                    images et logos de protocoles
├── favicon.ico
├── support.js              moteur de rendu des pages (obligatoire)
├── robots.txt              même allow-list de crawlers que le site principal
├── sitemap.xml             4 URLs + alternates hreflang
├── llms.txt                fiche produit pour les moteurs génératifs
├── CNAME                   edge.hexa-ai.fr
└── .gitignore
```

Ne sont **pas** publiés (exclus par `.gitignore`) : `uploads/`, `doc-page.js`, `image-slot.js`, `.thumbnail`, `.claude/` — ce sont des artefacts de l'outil de design, non utilisés par les pages.

---

## 2. Créer le dépôt

Sur GitHub, dans l'organisation **Hexa-ai** : *New repository* → nom `site-edge-hexa-ai` → **Public** (Pages est gratuit sur les dépôts publics ; sur un dépôt privé il faut un plan Pro/Team).

Ne coche ni README, ni .gitignore, ni licence : le dépôt doit rester vide pour le premier push.

En ligne de commande, si tu as `gh` installé :

```bash
gh repo create Hexa-ai/site-edge-hexa-ai --public --description "Site produit HAI-P200-4G — Hexa-AI Edge"
```

---

## 3. Premier envoi

Depuis le dossier du site :

```bash
git init -b main
```

```bash
git add .
```

```bash
git commit -m "Site produit HAI-P200-4G : FR/EN, SEO, robots et sitemap"
```

```bash
git remote add origin https://github.com/Hexa-ai/site-edge-hexa-ai.git
```

```bash
git push -u origin main
```

Vérifie au passage que `git status` ne liste pas `uploads/` ni `doc-page.js` — sinon le `.gitignore` n'a pas été pris en compte.

---

## 4. Activer GitHub Pages

Dans le dépôt : **Settings → Pages**.

- *Source* : **Deploy from a branch**
- *Branch* : `main`, dossier `/ (root)`
- **Save**

Le déploiement prend une à deux minutes. Le site est alors visible sur `https://hexa-ai.github.io/site-edge-hexa-ai/`.

> À ce stade, les liens internes fonctionnent (ils sont tous relatifs), mais les balises `canonical` et le `sitemap.xml` annoncent déjà `edge.hexa-ai.fr`. C'est normal et sans conséquence tant que le domaine n'est pas branché — les moteurs ne verront cette version qu'après la bascule.

---

## 5. Brancher le domaine `edge.hexa-ai.fr`

**a. Côté GitHub** — le fichier `CNAME` est déjà dans le dépôt, GitHub le lit automatiquement. Vérifie dans *Settings → Pages → Custom domain* que `edge.hexa-ai.fr` s'affiche bien.

**b. Côté DNS** — chez le registrar du domaine `hexa-ai.fr`, remplace l'enregistrement existant de `edge` par :

| Type  | Nom    | Valeur               | TTL     |
|-------|--------|----------------------|---------|
| CNAME | `edge` | `hexa-ai.github.io.` | défaut  |

Le TTL par défaut d'OVH (3600 s, soit 1 h) convient très bien — c'est celui utilisé par `www.hexa-ai.fr`. Il ne détermine que la durée de mise en cache de la réponse chez les résolveurs, sans effet sur les performances ni sur le référencement. Un TTL court (300 s) n'a d'intérêt que s'il est posé **avant** une bascule, pour raccourcir la traîne de propagation et pouvoir revenir en arrière rapidement.

S'il existe déjà un enregistrement `A`, `AAAA` ou `CNAME` sur `edge`, il faut le **supprimer** : un nom ne peut pas avoir à la fois un CNAME et d'autres enregistrements.

**c. Attendre la propagation**, puis contrôler :

```bash
nslookup edge.hexa-ai.fr
```

**d. Certificat HTTPS** — une fois le DNS propagé, GitHub génère un certificat Let's Encrypt (quelques minutes). Coche ensuite **Enforce HTTPS** dans *Settings → Pages*.

Si l'option reste grisée avec « unavailable for your site », c'est que le DNS n'est pas encore vu par GitHub : retire puis remets le domaine dans le champ *Custom domain* pour relancer la vérification.

---

## 6. Contrôles après mise en ligne

```bash
curl -sI https://edge.hexa-ai.fr/ | head -1
```

- [ ] `https://edge.hexa-ai.fr/` affiche la page FR, `https://edge.hexa-ai.fr/en/` la page EN
- [ ] `https://edge.hexa-ai.fr/robots.txt` et `/sitemap.xml` répondent
- [ ] Un navigateur configuré en anglais arrive automatiquement sur `/en/` (voir §7)
- [ ] Le sélecteur FR/EN bascule dans les deux sens et le choix est retenu au retour
- [ ] Les boutons de contact ouvrent bien la fenêtre d'envoi de mail
- [ ] Le header reste collé en haut au défilement
- [ ] `https://edge.hexa-ai.fr/helpdesk` renvoie bien vers `https://hexa-ai.odoo.com/helpdesk`
- [ ] `https://edge.hexa-ai.fr/nimportequoi` affiche la page 404 de la charte
- [ ] Sur téléphone : header sur une ou deux lignes, aucune barre de défilement horizontale, contenus empilés en une colonne

Puis déclarer le site dans la Search Console :
`https://search.google.com/search-console` → *Ajouter une propriété* → préfixe d'URL `https://edge.hexa-ai.fr/` → soumettre `https://edge.hexa-ai.fr/sitemap.xml`.

> Le fichier de vérification `google8300810fcf42f1a2.html` du dépôt principal ne vaut **que** pour `www.hexa-ai.fr`. Il faut une vérification propre à ce sous-domaine (fichier HTML à déposer à la racine, ou enregistrement DNS TXT).

---

## 7. Comment fonctionne la redirection de langue

Un script placé dans le `<head>` des deux pages françaises (`index.html`, `securite.html`) applique la règle suivante, avant tout affichage :

| Situation | Résultat |
|---|---|
| Langue principale du navigateur en `fr*` | reste en français |
| Toute autre langue (`en`, `de`, `es`…) | redirection vers la page anglaise |
| Choix déjà fait via le sélecteur FR/EN | respecté, la redirection ne s'applique plus |
| URL suffixée `?lang=fr` | force le français |

Le choix est mémorisé dans `localStorage` sous la clé `hai-lang`, écrite au clic sur le sélecteur du header (attribut `data-hai-lang`). Les liens « FR » des pages anglaises portent en plus `?lang=fr`, ce qui garantit le retour au français même si `localStorage` est bloqué (navigation privée stricte).

Les pages anglaises ne redirigent **jamais** : c'est ce qui rend une boucle impossible.

Pour tester sans changer la langue de ton navigateur, ouvre la console sur `https://edge.hexa-ai.fr/` :

```bash
localStorage.removeItem('hai-lang')
```

puis recharge avec un profil ou un navigateur configuré en anglais.

Côté référencement, chaque page déclare ses `hreflang` (`fr`, `en`, `x-default` → français), ce qui permet à Google d'indexer les deux versions et de servir la bonne selon l'utilisateur, indépendamment du script.

---

## 8. Mettre à jour le site plus tard

L'outil de design réexporte les pages sous leurs noms d'origine (`HAI-P200 EN.dc.html`, `Securite.dc.html`…). Après un nouvel export, il faut refaire la correspondance :

| Export de l'outil | Fichier du dépôt |
|---|---|
| `index.html` | `index.html` |
| `Securite.dc.html` | `securite.html` |
| `HAI-P200 EN.dc.html` | `en/index.html` |
| `Securite EN.dc.html` | `en/security.html` |

Attention, les pages du dossier `en/` ont leurs chemins de ressources préfixés par `../` (`../img/…`, `../support.js`), et chaque page contient dans son `<head>` trois ajouts absents de l'export brut : le bloc SEO, le script de langue et la feuille `<style id="hai-responsive">` qui rend la page utilisable sur mobile. Le plus sûr est de ne réimporter que le corps de page (`<x-dc>…</x-dc>`) et de conserver le `<head>` du dépôt.

Ensuite :

```bash
git add . && git commit -m "Mise à jour du contenu" && git push
```

Le déploiement est automatique, comptez une à deux minutes.

---

## 9. Reste à faire

- **Image de partage** — les balises `og:image` pointent vers `img/hai-p200-4g.webp`, qui n'est pas au format attendu. Le site principal utilise un `img/og-image.jpg` de 1200 × 630. Il en faudrait un équivalent ici, puis remplacer les quatre `og:image` / `twitter:image`.
- **`web.base.url` côté Odoo** — voir la section Odoo en haut de ce document.
