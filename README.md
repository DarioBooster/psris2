# P@SRIS2 — Rapport Mensuel IT (Version Multi-Utilisateurs)

Plateforme web de gestion des rapports d'activité IT pour la Zone Sanitaire Parakou-N'Dali.  
**Marché BEN23006-10042** | Enabel / BELMAG SARL & POLYGONE+

---

## Architecture

```
psris2_web/
├── src/
│   ├── server.js          ← Serveur Express (point d'entrée)
│   ├── db.js              ← Base SQLite (better-sqlite3)
│   ├── auth.js            ← JWT + bcrypt
│   └── routes/
│       ├── auth.js        ← POST /register, /login, /logout
│       ├── rapports.js    ← GET/POST/DELETE /rapports
│       └── admin.js       ← Routes admin (users, rapports)
├── public/
│   ├── login.html         ← Page connexion/inscription
│   ├── app.html           ← Application rapport (protégée)
│   ├── admin.html         ← Tableau de bord admin
│   ├── app.js             ← Logique formulaire + génération .docx
│   ├── style.css
│   ├── docx.iife.js       ← Librairie docx (bundlée, offline)
│   └── FileSaver.js
├── db/                    ← Créé automatiquement
│   └── psris2.db          ← Base SQLite (créée au 1er démarrage)
├── package.json
├── .env.example
├── railway.toml
└── README.md
```

**Stack :** Node.js + Express + SQLite (better-sqlite3) + JWT  
**Hébergement recommandé :** Railway.app (gratuit, simple)

---

## Installation locale (PC/serveur)

### Prérequis
- Node.js 18+ ([nodejs.org](https://nodejs.org))
- Git ([git-scm.com](https://git-scm.com))

### Étapes

```bash
# 1. Cloner le dépôt
git clone https://github.com/VOTRE_COMPTE/psris2-rapport.git
cd psris2-rapport

# 2. Installer les dépendances
npm install

# 3. Configurer l'environnement
cp .env.example .env
# Éditez .env et changez JWT_SECRET

# 4. Démarrer le serveur
npm start
# → http://localhost:3000
```

### Compte admin par défaut
- **Email :** `admin@psris2.bj`
- **Mot de passe :** `Admin@PSRIS2!`
- ⚠️ Changez ce mot de passe immédiatement après connexion !

---

## Déploiement en ligne — Railway.app (Recommandé)

Railway.app est **gratuit** (500h/mois), supporte Node.js + SQLite, et le déploiement se fait en 3 minutes.

### Étape 1 — Préparer le code sur GitHub

```bash
# Dans le dossier psris2_web/
git init
git add .
git commit -m "Initial commit P@SRIS2"

# Créer un repo sur github.com puis :
git remote add origin https://github.com/VOTRE_COMPTE/psris2-rapport.git
git push -u origin main
```

### Étape 2 — Déployer sur Railway

1. Aller sur [railway.app](https://railway.app) → **Start a New Project**
2. Choisir **Deploy from GitHub repo**
3. Sélectionner votre repo `psris2-rapport`
4. Railway détecte automatiquement Node.js et lance `npm start`

### Étape 3 — Configurer les variables d'environnement

Dans Railway → votre projet → **Variables** → ajouter :

| Variable | Valeur |
|----------|--------|
| `NODE_ENV` | `production` |
| `JWT_SECRET` | *(une chaîne longue et aléatoire)* |
| `DB_PATH` | `/app/db/psris2.db` |

Pour générer un JWT_SECRET solide :
```bash
node -e "console.log(require('crypto').randomBytes(64).toString('hex'))"
```

### Étape 4 — Obtenir l'URL

Railway génère une URL du type : `https://psris2-rapport-production.up.railway.app`

Vous pouvez aussi configurer un **domaine personnalisé** dans les paramètres Railway.

---

## Déploiement alternatif — VPS / Serveur dédié

Si vous avez un serveur Ubuntu/Debian (même un VPS à 5$/mois) :

```bash
# Sur le serveur
sudo apt update && sudo apt install -y nodejs npm git

git clone https://github.com/VOTRE_COMPTE/psris2-rapport.git
cd psris2-rapport
npm install
cp .env.example .env && nano .env  # configurer

# Lancer en production avec PM2
sudo npm install -g pm2
pm2 start src/server.js --name psris2
pm2 startup  # démarrage automatique au reboot
pm2 save

# Nginx comme reverse proxy (optionnel)
sudo apt install nginx
# config dans /etc/nginx/sites-available/psris2
```

---

## Fonctionnalités multi-utilisateurs

### Pour les techniciens
- ✅ Créer un compte (nom, prénom, email, mot de passe)
- ✅ Se connecter / déconnecter
- ✅ Remplir et sauvegarder des rapports (auto-save cloud toutes les 60s)
- ✅ Accéder à tous ses rapports précédents depuis le panneau "Mes rapports"
- ✅ Charger un rapport précédent pour le modifier
- ✅ Soumettre un rapport pour validation
- ✅ Générer le .docx localement (aucun internet requis)

### Pour les administrateurs
- ✅ Tableau de bord avec statistiques globales
- ✅ Voir tous les rapports de tous les techniciens
- ✅ Valider les rapports soumis
- ✅ Gérer les comptes utilisateurs (activer/désactiver, promouvoir admin)

---

## API REST

| Méthode | Route | Description |
|---------|-------|-------------|
| POST | `/api/auth/register` | Créer un compte |
| POST | `/api/auth/login` | Se connecter |
| POST | `/api/auth/logout` | Se déconnecter |
| GET  | `/api/auth/me` | Infos utilisateur connecté |
| PUT  | `/api/auth/profile` | Modifier profil/mot de passe |
| GET  | `/api/rapports` | Liste de mes rapports |
| GET  | `/api/rapports/:mois/:annee` | Récupérer un rapport |
| POST | `/api/rapports/save` | Sauvegarder un rapport |
| PUT  | `/api/rapports/:id/statut` | Changer le statut |
| DELETE | `/api/rapports/:id` | Supprimer un rapport |
| GET  | `/api/admin/users` | [Admin] Liste des users |
| GET  | `/api/admin/rapports` | [Admin] Tous les rapports |
| PUT  | `/api/admin/rapports/:id/statut` | [Admin] Valider |

---

## Sauvegarde de la base de données

La base SQLite est un **simple fichier** : `db/psris2.db`

```bash
# Sauvegarder
cp db/psris2.db db/psris2_backup_$(date +%Y%m%d).db

# Sur Railway, télécharger via Railway CLI
railway run cp /app/db/psris2.db /tmp/backup.db
```

---

## Mise à jour de la liste des structures

Dans `public/app.js`, modifiez la variable `CONFIG.structures` :

```javascript
const CONFIG = {
  structures: [
    "Bureau de Zone Sanitaire",
    "CHUD Borgou",
    // ... vos 38 structures
  ]
};
```

---

*P@SRIS2 — Programme d'Appui à la Santé Sexuelle et Reproductive et à l'Information Sanitaire*
