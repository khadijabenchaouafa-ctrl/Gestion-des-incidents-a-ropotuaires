# Gestion-des-incidents-a-ropotuaires
# ✈️ GIA — Gestion des Incidents Aéroportuaires

Plateforme web complète de déclaration, suivi et résolution des incidents aéroportuaires en temps réel.

---

## Technologies

| Couche      | Stack                                      |
|-------------|---------------------------------------------|
| Backend     | Java 21, Spring Boot 3.2, Spring Security, JWT, JPA/Hibernate |
| Frontend    | React 18, Vite, Tailwind CSS, Recharts, Axios |
| Base de données | PostgreSQL 16                          |
| Temps réel  | Server-Sent Events (SSE)                   |
| Conteneurs  | Docker, Docker Compose                     |

---

## Rôles et accès

| Rôle        | Tableau de bord | Incidents | Équipes | Utilisateurs | Notifications SSE |
|-------------|:-:|:-:|:-:|:-:|:-:|
| ADMIN       | ✅ | ✅ complet | ✅ | ✅ complet | ✅ |
| SUPERVISEUR | ✅ | ✅ (sans suppression) | ✅ | ✅ (sans suppression) | ✅ |
| TECHNICIEN  | ❌ | Vue équipe + statut | ❌ | ❌ | ❌ |
| PERSONNEL   | ❌ | Déclarer + mes signalements | ❌ | ❌ | ❌ |

---

## Prérequis

- Java 21+
- Maven 3.9+
- Node.js 20+
- PostgreSQL 16+ **ou** Docker Desktop

---

## Démarrage rapide (développement)

### 1. Démarrer PostgreSQL avec Docker

```bash
docker-compose -f docker-compose.dev.yml up -d
```

### 2. Démarrer le backend

```bash
cd backend
mvn spring-boot:run
```

Le backend démarre sur **http://localhost:8080/api**

Les tables sont créées automatiquement via `schema.sql` et les données chargées via `data.sql`.

### 3. Démarrer le frontend

```bash
cd frontend
npm install
npm run dev
```

Le frontend démarre sur **http://localhost:5173**

---

## Démarrage complet avec Docker Compose (production)

```bash
docker-compose up --build
```

- Frontend : **http://localhost:3000**
- Backend  : **http://localhost:8080/api**
- PostgreSQL : port **5432**

Pour arrêter :

```bash
docker-compose down
```

Pour supprimer les volumes (données) :

```bash
docker-compose down -v
```

---

## Comptes de démonstration

| Rôle          | Email                         | Mot de passe   |
|---------------|-------------------------------|----------------|
| Admin         | admin@aeroport.ma             | Aeroport2024!  |
| Superviseur 1 | superviseur1@aeroport.ma      | Aeroport2024!  |
| Superviseur 2 | superviseur2@aeroport.ma      | Aeroport2024!  |
| SSLIA         | tech.sslia@aeroport.ma        | Aeroport2024!  |
| Médical       | tech.medical@aeroport.ma      | Aeroport2024!  |
| IT / Réseaux  | tech.it@aeroport.ma           | Aeroport2024!  |
| GTA           | tech.gta@aeroport.ma          | Aeroport2024!  |
| Agent d'escale| agent.escale1@aeroport.ma     | Aeroport2024!  |
| Contrôleur ATC| atc1@aeroport.ma              | Aeroport2024!  |

---

## Structure du projet

```
projet-aeroport/
├── backend/
│   ├── src/main/java/com/aeroport/incidents/
│   │   ├── config/           # Spring Security, CORS, ApplicationConfig
│   │   ├── controller/       # REST Controllers
│   │   ├── dto/              # DTO requêtes et réponses
│   │   ├── entite/           # Entités JPA
│   │   ├── exception/        # Exceptions personnalisées + handler global
│   │   ├── repository/       # Spring Data JPA Repositories
│   │   ├── securite/         # JWT Service + Filtre
│   │   └── service/          # Logique métier
│   ├── src/main/resources/
│   │   ├── application.yml
│   │   ├── schema.sql        # Schéma PostgreSQL
│   │   └── data.sql          # Jeu de données (21 users, 10 équipes, 50 incidents)
│   ├── Dockerfile
│   └── pom.xml
├── frontend/
│   ├── src/
│   │   ├── composants/       # Composants réutilisables
│   │   ├── contexte/         # AuthContexte React
│   │   ├── layouts/          # LayoutPrincipal avec sidebar dynamique
│   │   ├── pages/            # Pages par rôle
│   │   ├── routes/           # Routes protégées
│   │   ├── services/         # Axios + services API
│   │   └── utilitaires/      # Constantes, formatage
│   ├── Dockerfile
│   ├── nginx.conf
│   └── package.json
├── docker-compose.yml        # Production (3 services)
├── docker-compose.dev.yml    # Développement (PostgreSQL seul)
└── ARCHITECTURE.md
```

---

## API REST — Endpoints principaux

### Authentification
| Méthode | Endpoint              | Description                  |
|---------|----------------------|------------------------------|
| POST    | /auth/connexion       | Connexion — retourne JWT      |
| POST    | /auth/rafraichir      | Rafraîchir le token d'accès  |

### Incidents
| Méthode | Endpoint                        | Description                     |
|---------|--------------------------------|---------------------------------|
| GET     | /incidents                      | Liste paginée avec filtres      |
| GET     | /incidents/{id}                 | Détail d'un incident            |
| POST    | /incidents                      | Créer un incident               |
| PUT     | /incidents/{id}                 | Modifier un incident            |
| DELETE  | /incidents/{id}                 | Supprimer (ADMIN/SUPERVISEUR)   |
| PATCH   | /incidents/{id}/equipe          | Affecter une équipe             |
| PATCH   | /incidents/{id}/statut          | Changer le statut               |
| GET     | /incidents/{id}/historique      | Historique des actions          |
| GET     | /incidents/equipe/{equipeId}    | Incidents d'une équipe          |
| GET     | /incidents/mes-signalements     | Incidents de l'utilisateur      |
| GET     | /incidents/statistiques         | Stats pour le tableau de bord   |

### Équipes
| Méthode | Endpoint        | Description       |
|---------|----------------|-------------------|
| GET     | /equipes        | Lister les équipes |
| POST    | /equipes        | Créer une équipe   |
| PUT     | /equipes/{id}   | Modifier           |
| DELETE  | /equipes/{id}   | Désactiver         |

### Notifications
| Méthode | Endpoint                    | Description              |
|---------|----------------------------|--------------------------|
| GET     | /notifications              | Liste paginée            |
| GET     | /notifications/non-lues     | Compteur non lues        |
| PATCH   | /notifications/{id}/lue     | Marquer une lue          |
| PATCH   | /notifications/tout-lire    | Tout marquer lu          |
| GET     | /notifications/sse          | Flux SSE temps réel      |

---

## Tests fonctionnels — plan de validation

### Authentification
- [ ] Connexion avec email/mot de passe valides → reçoit access token + refresh token
- [ ] Connexion avec mauvais identifiants → erreur 401 avec message clair
- [ ] Accès à une route protégée sans token → 403
- [ ] Rafraîchissement automatique du token expiré

### Gestion des incidents
- [ ] Déclaration d'un incident critique → notification SSE reçue côté superviseur
- [ ] Filtrage par statut / gravité / catégorie / localisation / recherche
- [ ] Affectation d'une équipe → statut passe automatiquement à EN_COURS
- [ ] Changement de statut → entrée créée dans l'historique
- [ ] Suppression d'un incident par ADMIN → disparaît de la liste
- [ ] Technicien ne peut modifier que les incidents de son équipe

### Tableau de bord
- [ ] KPIs affichent les bons chiffres (total, ouverts, critiques, temps moyen)
- [ ] Graphiques Recharts s'affichent correctement avec les données
- [ ] Évolution mensuelle visible sur 12 mois

### Notifications SSE
- [ ] Connexion SSE établie pour ADMIN et SUPERVISEUR
- [ ] Toast apparaît lors d'un incident critique
- [ ] Compteur cloche mis à jour en temps réel
- [ ] Marquer tout comme lu → compteur revient à 0

### Contrôle d'accès
- [ ] PERSONNEL ne voit que Déclarer + Mes signalements
- [ ] TECHNICIEN ne voit pas le tableau de bord
- [ ] PERSONNEL ne peut pas accéder à /tableau-de-bord → redirigé
- [ ] Seul ADMIN peut supprimer un utilisateur

---

## Variables d'environnement

| Variable                        | Valeur par défaut              | Description              |
|---------------------------------|-------------------------------|--------------------------|
| SPRING_DATASOURCE_URL           | jdbc:postgresql://localhost:5432/aeroport_incidents | URL PostgreSQL |
| SPRING_DATASOURCE_USERNAME      | aeroport_user                 | Utilisateur BDD          |
| SPRING_DATASOURCE_PASSWORD      | aeroport_pass                 | Mot de passe BDD         |
| APPLICATION_JWT_SECRET          | (voir application.yml)        | Clé secrète JWT 64 chars |
| APPLICATION_JWT_EXPIRATION      | 86400000                      | Expiration access (24h)  |
| APPLICATION_JWT_REFRESH_EXPIRATION | 604800000                  | Expiration refresh (7j)  |
