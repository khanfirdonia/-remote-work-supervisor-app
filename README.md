# remote-work-supervisor-app

Application de supervision du travail a distance composee de :

- `frontend-app/` : SPA Angular 22 pour les espaces Employe et RH
- `backend/` : API Strapi 5 et logique metier
- `docs/` : assets et notes projet
- `scripts/` : automatisation locale, dont la creation du ZIP de livraison

La configuration Docker existante est volontairement hors du present document.

## Prerequis

- Node.js `24.16.0` ou une version compatible avec :
  - backend : `>=20.0.0 <=26.x.x`
  - frontend : Angular CLI 22, teste ici avec Node 24
- npm `11.13.0`
- PowerShell 7+ pour le script de ZIP fourni dans ce depot

## Architecture

- Frontend Angular :
  - authentification, guards et interceptors dans `frontend-app/src/app/core/`
  - services API dans `frontend-app/src/app/services/`
  - fonctionnalites Employe et RH dans `frontend-app/src/app/features/`
- Backend Strapi :
  - APIs metier dans `backend/src/api/`
  - policies d autorisation dans `backend/src/policies/`
  - helpers `documentId` dans `backend/src/utils/document-service.ts`
  - tests backend cibles dans `backend/tests/`

## Variables d environnement backend

Le backend requiert un fichier local `backend/.env` derive de `backend/.env.example`.

Variables obligatoires :

- `APP_KEYS`
- `API_TOKEN_SALT`
- `ADMIN_JWT_SECRET`
- `TRANSFER_TOKEN_SALT`
- `JWT_SECRET`
- `ENCRYPTION_KEY`
- `APP_TIMEZONE`

Variables base de donnees :

- mode local par defaut :
  - `DATABASE_CLIENT=sqlite`
  - `DATABASE_FILENAME=.tmp/data.db`
- mode PostgreSQL :
  - `DATABASE_CLIENT=postgres`
  - `DATABASE_HOST`
  - `DATABASE_PORT`
  - `DATABASE_NAME`
  - `DATABASE_USERNAME`
  - `DATABASE_PASSWORD`
  - `DATABASE_SCHEMA`
  - `DATABASE_SSL`

Important :

- ne pas versionner `backend/.env`
- ne pas inclure de `.env` reel dans le livrable
- renouveler tous les secrets qui auraient deja ete partages

## Configuration API frontend

- developpement local : `frontend-app/src/environments/environment.ts`
  - `apiUrl = http://localhost:1337/api`
- build de production : `frontend-app/src/environments/environment.production.ts`
  - `apiUrl = /api`

Le code applicatif Angular consomme la configuration centralisee via `environment.apiUrl`.

## Installation

### Backend

```bash
cd backend
npm ci
```

Puis creer `backend/.env` a partir de `backend/.env.example` et renseigner les valeurs locales.

### Frontend

```bash
cd frontend-app
npm ci
```

## Commandes de developpement

### Backend

```bash
cd backend
npm run develop
```

### Frontend

```bash
cd frontend-app
npm start
```

## Commandes de verification

### Frontend

```bash
cd frontend-app
npm run typecheck
npm run test:ci
npm run build:prod
```

### Backend

```bash
cd backend
npm run typecheck
npm test
npm run build
```

## Build de production

### Frontend

```bash
cd frontend-app
npm run build:prod
```

### Backend

```bash
cd backend
npm run build
```

## Creation du ZIP propre

Le script ci-dessous produit une archive sans `node_modules`, `.git`, `dist`, `.angular`, `.idea`, `.agents`, caches, logs ni fichiers `.env` reels.

```powershell
powershell -ExecutionPolicy Bypass -File .\scripts\create-clean-release.ps1
```

Archive de sortie par defaut :

```text
release/remote-work-supervisor-app.zip
```

Exemple avec chemin explicite :

```powershell
powershell -ExecutionPolicy Bypass -File .\scripts\create-clean-release.ps1 -OutputPath .\release\remote-work-supervisor-app-review.zip
```

## Tests backend ajoutes

Les tests cibles couvrent notamment :

- policies proprietaire ou RH
- filtrage force des collections
- refus d acces a un document d un autre employe
- liste blanche des champs de creation et de mise a jour
- restrictions de statuts sur les workflows critiques
- verification statique des routes reservees RH

## Comptes et donnees de demonstration

Aucun compte par defaut avec secret reel n est documente dans ce depot. Creer les utilisateurs locaux requis via Strapi et l interface RH selon l environnement cible.
