# 🎓 CampusHub — Backend

API REST de **CampusHub**, une plateforme de gestion universitaire pour l'**ENICarthage**.
Elle permet aux **enseignants** de réserver des salles, aux **étudiants** de demander des documents administratifs, à tous de suivre les **événements** du campus, et aux **administrateurs** de valider les demandes depuis un tableau de bord.

Développée avec **Spring Boot 3** et **PostgreSQL**. Elle est consommée par un frontend **Angular**.

> 🔗 **Frontend :** [jaz56/campushub-frontend](https://github.com/jaz56/campushub-frontend)

---

## ✨ Fonctionnalités

### 👨‍🏫 Enseignant
- Consulter les salles, leur capacité, leurs équipements et leur disponibilité
- Rechercher les salles disponibles et les filtrer par bâtiment
- Créer, modifier et annuler une **réservation de salle**
- Suivre ses réservations : `EN_ATTENTE`, `APPROUVEE` ou `REJETEE`
- Créer et gérer des **événements**

### 🎓 Étudiant
- Consulter son profil
- Voir les événements du campus
- Demander un **document administratif** (certificat de scolarité, relevé de notes, attestation de présence, carte étudiant…)
- Suivre l'état de ses demandes

### 🛡️ Administrateur
- **Tableau de bord** avec les statistiques
- **Approuver ou rejeter** les réservations en attente
- Mettre à jour le statut des demandes de documents
- Gérer les utilisateurs (liste, suppression)

### 🔐 Authentification
- Inscription et connexion
- Rôles : `ETUDIANT`, `ENSEIGNANT`, `ADMIN`, `RESPONSABLE_CLUB`

---

## 🛠️ Technologies

| Catégorie | Outils |
|---|---|
| Langage | Java 17 |
| Framework | Spring Boot 3.2 (Web, Data JPA, Security, Validation) |
| Base de données | PostgreSQL (hébergée sur Neon) |
| ORM | Hibernate / JPA |
| Utilitaires | Lombok, Spring DevTools |
| Build | Maven (wrapper `mvnw` inclus) |
| Frontend associé | Angular 21, [campushub-frontend](https://github.com/jaz56/campushub-frontend) (`http://localhost:4200`) |

---

## 🏗️ Architecture

Le code est organisé **par module fonctionnel**. Chaque module a ses propres couches Controller → Service → Repository → Model, et utilise des DTO.

```
campushub/src/main/java/tn/enicarthage/campushub/
├── CampushubApplication.java
├── config/          # SecurityConfig, WebConfig (CORS), DataInitializer (données de test)
├── auth/            # Inscription / connexion
├── admin/           # Tableau de bord et validation des demandes
├── enseignant/      # Salles, réservations, événements, demandes de documents
├── student/         # Profil étudiant, événements, demandes
└── shared/          # Entité User, UserRepository, UserDto
```

---

## 🗃️ Modèle de données

```
  User ──< Reservation >── Salle
   │
   ├──< Evenement        (organisateur)
   └──< DemandeDocument  (demandeur)
```

| Entité | Table | Champs principaux |
|---|---|---|
| **User** | `users` | nom, prénom, email, mot de passe, rôle, département, avatar |
| **Salle** | `salles` | nom, capacité, équipements, bâtiment, étage, disponible |
| **Reservation** | `reservations` | salle, enseignant, début, fin, motif, participants, statut, commentaire admin |
| **Evenement** | `evenements` | titre, description, début, fin, lieu, places max, statut (`OUVERT`, `PLEIN`, `ANNULE`, `TERMINE`), organisateur |
| **DemandeDocument** | `demandes_documents` | type, demandeur, date, statut (`EN_ATTENTE`, `EN_COURS`, `PRET`, `REJETE`, `RECUPERE`), commentaire admin |

---

## 🔌 Endpoints de l'API

URL de base : `http://localhost:8082`

### Authentification — `/api/v1/auth`
| Méthode | Route | Description |
|---|---|---|
| POST | `/login` | Connexion |
| POST | `/register` | Inscription |

### Salles — `/api/v1/salles`
| Méthode | Route | Description |
|---|---|---|
| GET | `/` | Liste des salles |
| GET | `/{id}` | Détail d'une salle |
| GET | `/disponibles` | Salles disponibles |
| GET | `/batiments` | Liste des bâtiments |

### Réservations — `/api/v1/reservations`
| Méthode | Route | Description |
|---|---|---|
| GET | `/me` | Mes réservations |
| GET | `/{id}` | Détail d'une réservation |
| POST | `/` | Créer une réservation |
| PUT | `/{id}` | Modifier une réservation |
| DELETE | `/{id}` | Annuler une réservation |

### Événements — `/api/v1/evenements`
| Méthode | Route | Description |
|---|---|---|
| GET | `/` | Tous les événements |
| GET | `/a-venir` | Événements à venir |
| GET | `/{id}` | Détail d'un événement |
| POST | `/` | Créer un événement |
| PUT | `/{id}` | Modifier un événement |
| DELETE | `/{id}` | Supprimer un événement |

### Demandes de documents — `/api/v1/demandes-documents`
| Méthode | Route | Description |
|---|---|---|
| GET | `/` | Toutes les demandes |
| GET | `/me` | Mes demandes |
| POST | `/` | Créer une demande |
| PATCH | `/{id}/statut` | Changer le statut d'une demande |

### Étudiant — `/api/v1/student`
| Méthode | Route | Description |
|---|---|---|
| GET | `/profile/{id}` | Profil de l'étudiant |
| GET | `/evenements` | Événements |
| POST | `/demandes` | Nouvelle demande |
| GET | `/demandes/{studentId}` | Demandes d'un étudiant |

### Administration — `/api/v1/admin`
| Méthode | Route | Description |
|---|---|---|
| GET | `/dashboard` | Statistiques |
| GET | `/reservations/pending` | Réservations en attente |
| PUT | `/reservations/{id}/approve` | Approuver une réservation |
| PUT | `/reservations/{id}/reject` | Rejeter une réservation |
| GET | `/users` | Liste des utilisateurs |
| DELETE | `/users/{id}` | Supprimer un utilisateur |

---

## 🚀 Installation et lancement

### Prérequis
- **Java 17**
- Une base **PostgreSQL** (locale ou sur [Neon](https://neon.tech))

### 1. Cloner le dépôt

```bash
git clone https://github.com/jaz56/campushub-backend.git
cd campushub-backend/campushub
```

### 2. Configurer la base de données

Dans `src/main/resources/application.properties` :

```properties
server.port=8082
spring.datasource.url=jdbc:postgresql://<HOST>/<DATABASE>?sslmode=require
spring.datasource.username=<USERNAME>
spring.datasource.password=<PASSWORD>
spring.jpa.hibernate.ddl-auto=update
spring.web.cors.allowed-origins=http://localhost:4200
```

> Avec `ddl-auto=create`, toutes les tables sont **recréées à chaque démarrage**, donc les données sont perdues. Utilisez `update` pour les conserver.

### 3. Lancer l'application

```bash
./mvnw spring-boot:run        # Linux / macOS
mvnw.cmd spring-boot:run      # Windows
```

L'API est alors accessible sur **http://localhost:8082**.
Pour vérifier qu'elle tourne : `GET http://localhost:8082/api/test/hello`.

---

## 🧪 Données de test

Au démarrage, `DataInitializer` crée automatiquement :

- **Des salles** du Bâtiment Annexe : Salle MAC, LABO Informatique, Salle Polyvalente (150 places), et les salles des 1er et 2e étages
- **Des comptes de démonstration** :

| Rôle | Email | Mot de passe |
|---|---|---|
| Admin | `admin@enicarthage.tn` | `admin123` |
| Enseignant | `jean.dupont@enicarthage.tn` | `enseignant123` |
| Étudiant | `ahmed.benali@enicarthage.tn` | `etudiant123` |

---

## 🗺️ Améliorations prévues

- [ ] Authentification **JWT** (le token est actuellement factice)
- [ ] Hachage des mots de passe avec **BCrypt**
- [ ] Contrôle d'accès par rôle (toutes les routes sont actuellement ouvertes)
- [ ] Configuration par variables d'environnement
- [ ] Documentation Swagger / OpenAPI
- [ ] Tests unitaires et d'intégration

---

## 🔗 Dépôts du projet

| Partie | Dépôt |
|---|---|
| Backend (Spring Boot) | [jaz56/campushub-backend](https://github.com/jaz56/campushub-backend) |
| Frontend (Angular) | [jaz56/campushub-frontend](https://github.com/jaz56/campushub-frontend) |

---

## 👩‍💻 Équipe

Projet réalisé par des étudiants en génie logiciel de l'**ENICarthage**.
