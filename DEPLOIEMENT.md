# Mettre l'EMP en ligne sur ton propre domaine (ex: emp-intranet.fr)

Ce guide t'emmène du zéro jusqu'à un vrai site accessible à cette adresse.
Compte environ 30-45 minutes la première fois. Trois grandes étapes :

1. Acheter le nom de domaine
2. Héberger le backend (l'API + la base de données, dans `emp-bot`)
3. Héberger le site (ce dossier, `emp-website`) et le relier au domaine

---

## 1. Acheter le nom de domaine

Va chez un registrar sérieux — pour un `.fr`, **OVH** ou **Gandi** sont de bons choix
(français, pas chers, ~10€/an). Cherche `emp-intranet.fr`, ajoute-le au panier, paye.
Tu n'as rien d'autre à faire ici pour l'instant — pas besoin de configurer quoi que ce
soit sur le domaine tant que les étapes 2 et 3 ne sont pas prêtes.

## 2. Héberger le backend (API + base de données)

On utilise **Railway** (gratuit pour démarrer, simple à configurer). Tu peux aussi
prendre Render si tu préfères, la logique est identique.

1. Va sur https://railway.app et crée un compte (tu peux te connecter avec GitHub)
2. Mets d'abord le dossier `emp-bot` sur GitHub :
   - Crée un compte GitHub si tu n'en as pas (https://github.com)
   - Crée un nouveau repository (ex: "emp-bot"), privé si tu veux
   - Upload le contenu du dossier `emp-bot` dedans (GitHub te propose un glisser-déposer
     depuis l'interface web si tu n'es pas à l'aise avec `git`)
3. Sur Railway : **New Project** > **Deploy from GitHub repo** > sélectionne ton repo `emp-bot`
4. Railway détecte le `package.json`. Dans les paramètres du service :
   - **Start Command** : `node src/server.js`
   - Ajoute les variables d'environnement (onglet **Variables**) : les mêmes que ton `.env`
     (`DISCORD_TOKEN`, `CLIENT_ID`, `GUILD_ID`, `PORT`, `ROBLOX_GROUP_ID` si tu l'utilises)
5. Une fois déployé, Railway te donne une URL du style `emp-bot-production.up.railway.app`
   — note-la, c'est ton `VITE_API_BASE` pour l'étape suivante.
6. **Important** : la base SQLite vit sur le disque du service. Sur Railway, ajoute un
   **Volume** (onglet Settings > Volumes) monté sur `/app` pour que les données ne soient
   pas effacées à chaque redéploiement.

Si tu veux que le bot Discord tourne aussi en continu (pas juste l'API), répète l'opération
avec un second service sur le même projet Railway, avec **Start Command** : `node src/bot.js`.

## 3. Héberger le site et le relier au domaine

On utilise **Vercel** (gratuit, fait exactement pour ça).

1. Mets aussi le dossier `emp-website` sur GitHub (même méthode qu'à l'étape 2)
2. Va sur https://vercel.com, connecte-toi avec GitHub
3. **Add New Project** > sélectionne ton repo `emp-website`
4. Dans **Environment Variables**, ajoute :
   ```
   VITE_API_BASE = https://emp-bot-production.up.railway.app
   ```
   (remplace par l'URL Railway obtenue à l'étape 2 — sans `/` à la fin)
5. Clique **Deploy**. Après une minute, Vercel te donne un lien du style
   `emp-website.vercel.app` — vérifie que le site fonctionne dessus d'abord.
6. Une fois que ça marche : va dans **Settings > Domains** du projet Vercel, tape
   `emp-intranet.fr`, clique **Add**.
7. Vercel t'affiche 1 ou 2 enregistrements DNS à créer (un `A` et/ou un `CNAME`).
   Retourne chez ton registrar (OVH/Gandi), dans la zone DNS de `emp-intranet.fr`,
   et ajoute exactement ces enregistrements.
8. Le certificat HTTPS se configure tout seul (jusqu'à 24h pour se propager, souvent
   beaucoup moins). Une fois fait, `https://emp-intranet.fr` affiche ton intranet.

## 4. Sécuriser un minimum l'API (recommandé)

Par défaut, `server.js` accepte les requêtes de n'importe où (CORS ouvert). Une fois
que tu connais ton domaine final, dis-le moi et je restreindrai l'API pour qu'elle
n'accepte que les requêtes venant de `emp-intranet.fr` — ça évite que d'autres sites
viennent piocher dans tes données.

## Résumé du flux de données

```
emp-intranet.fr  (Vercel, le site React)
        │  fetch() vers VITE_API_BASE
        ▼
emp-bot-production.up.railway.app  (Railway, l'API Express)
        │
        ▼
emp.sqlite  (Railway, sur le Volume)
```

Le bot Discord (s'il tourne aussi sur Railway) lit et écrit dans **la même base**,
donc les deux sont enfin vraiment synchronisés — c'est la pièce qui manquait depuis
le début.

## Si tu bloques quelque part

Dis-moi précisément à quelle étape et ce que tu vois à l'écran (capture d'écran si
possible) — je débloquerai avec toi.
