# Architecture — Système de Gestion des Incidents Aéroportuaires

## Vue d'ensemble

```
projet-aeroport/
├── backend/                        # Spring Boot 3 / Java 21
├── frontend/                       # React + Vite + Tailwind CSS
├── docker-compose.yml              # Production (3 services)
├── docker-compose.dev.yml          # Développement (PostgreSQL seul)
└── ARCHITECTURE.md
```

## Architecture backend (Spring Boot)

```
backend/src/main/java/com/aeroport/incidents/
├── config/
│   ├── SecurityConfig.java         # Configuration Spring Security
│   ├── CorsConfig.java             # Configuration CORS
│   └── ApplicationConfig.java      # Beans généraux (PasswordEncoder, etc.)
├── securite/
│   ├── jwt/
│   │   ├── JwtService.java         # Génération et validation des tokens
│   │   └── JwtProperties.java      # Propriétés JWT (secret, expiration)
│   └── filtre/
│       └── FiltreAuthentificationJwt.java  # Filtre JWT par requête
├── entite/
│   ├── Utilisateur.java            # Table users
│   ├── Equipe.java                 # Table equipes
│   ├── Incident.java               # Table incidents
│   ├── HistoriqueIncident.java     # Table historique_incidents
│   └── Notification.java           # Table notifications
├── dto/
│   ├── requete/
│   │   ├── ConnexionRequete.java
│   │   ├── IncidentRequete.java
│   │   ├── EquipeRequete.java
│   │   └── UtilisateurRequete.java
│   └── reponse/
│       ├── ConnexionReponse.java
│       ├── IncidentReponse.java
│       ├── EquipeReponse.java
│       ├── UtilisateurReponse.java
│       └── NotificationReponse.java
├── repository/
│   ├── UtilisateurRepository.java
│   ├── EquipeRepository.java
│   ├── IncidentRepository.java
│   ├── HistoriqueRepository.java
│   └── NotificationRepository.java
├── service/
│   ├── AuthentificationService.java
│   ├── IncidentService.java
│   ├── EquipeService.java
│   ├── UtilisateurService.java
│   ├── NotificationService.java
│   └── HistoriqueService.java
├── controller/
│   ├── AuthentificationController.java
│   ├── IncidentController.java
│   ├── EquipeController.java
│   ├── UtilisateurController.java
│   ├── NotificationController.java
│   └── SseController.java
└── exception/
    ├── GestionnaireExceptionsGlobal.java
    ├── RessourceNonTrouveeException.java
    └── AccesInterditException.java
```

## Architecture frontend (React)

```
frontend/src/
├── composants/
│   ├── commun/
│   │   ├── Bouton.jsx
│   │   ├── Champ.jsx
│   │   ├── Tableau.jsx
│   │   ├── Badge.jsx
│   │   ├── Modal.jsx
│   │   ├── Filtre.jsx
│   │   ├── Pagination.jsx
│   │   └── ChargementSpinner.jsx
│   ├── incidents/
│   │   ├── FormulaireIncident.jsx
│   │   ├── CarteIncident.jsx
│   │   ├── TableauIncidents.jsx
│   │   └── HistoriqueIncident.jsx
│   ├── equipes/
│   │   └── FormulaireEquipe.jsx
│   ├── utilisateurs/
│   │   └── FormulaireUtilisateur.jsx
│   ├── notifications/
│   │   ├── ClochNotification.jsx
│   │   └── ListeNotifications.jsx
│   └── tableau-bord/
│       ├── CarteKpi.jsx
│       ├── GraphiqueCategories.jsx
│       ├── GraphiqueGravites.jsx
│       ├── GraphiqueEvolution.jsx
│       └── GraphiqueLocalisations.jsx
├── pages/
│   ├── Connexion.jsx
│   ├── TableauDeBord.jsx
│   ├── Incidents.jsx
│   ├── DetailIncident.jsx
│   ├── MesIncidents.jsx            # Vue technicien
│   ├── DeclarerIncident.jsx        # Vue personnel
│   ├── MesSignalements.jsx         # Vue personnel
│   ├── Equipes.jsx
│   ├── Utilisateurs.jsx
│   └── Notifications.jsx
├── layouts/
│   ├── LayoutPrincipal.jsx         # Sidebar + header + outlet
│   └── LayoutAuth.jsx              # Centré, sans sidebar
├── services/
│   ├── axios.js                    # Instance Axios + intercepteurs
│   ├── authService.js
│   ├── incidentService.js
│   ├── equipeService.js
│   ├── utilisateurService.js
│   └── notificationService.js
├── hooks/
│   ├── useAuth.js
│   ├── useNotificationsSse.js
│   └── usePagination.js
├── contexte/
│   └── AuthContexte.jsx
├── routes/
│   └── RoutesProtegees.jsx
└── utilitaires/
    ├── constantes.js
    ├── formatDate.js
    └── rolesPermissions.js
```

## Flux de données

```
Client React
    │
    ├── Axios (JWT dans header Authorization)
    │       │
    │       ▼
    │   Spring Security Filter Chain
    │       │
    │       ├── FiltreAuthentificationJwt  →  JwtService
    │       │
    │       ▼
    │   Controllers REST  →  Services  →  Repositories  →  PostgreSQL
    │
    └── SSE (EventSource)
            │
            ▼
        SseController  →  NotificationService
```

## Modèle relationnel

```
users (1) ──────────────< incidents (déclaré_par)
users (1) ──────────────< incidents (resolu_par)
equipes (1) ─────────────< incidents (equipe_assignee)
incidents (1) ────────────< historique_incidents
incidents (1) ────────────< notifications
users (1) ──────────────< historique_incidents (modifie_par)
```
