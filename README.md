# 🏢 Gestion des Réservations Avancée

Système de gestion de réservations de salles de réunion avec recherche avancée, filtres multi-critères et pagination — développé en Java avec **JPA/Hibernate** et une base de données **H2 en mémoire**.

---

## 📁 Structure du projet

```
gestion-reservations-avancee/
└── src/
    └── main/
        ├── java/
        │   └── com/example/
        │       ├── model/
        │       │   ├── Equipement.java
        │       │   ├── Reservation.java
        │       │   ├── Salle.java
        │       │   └── Utilisateur.java
        │       ├── repository/
        │       │   ├── SalleRepository.java        (interface)
        │       │   └── SalleRepositoryImpl.java    (implémentation JPA)
        │       ├── service/
        │       │   ├── SalleService.java           (interface)
        │       │   └── SalleServiceImpl.java
        │       ├── util/
        │       │   └── PaginationResult.java
        │       └── App.java                        (point d'entrée)
        └── resources/
            └── META-INF/
                └── persistence.xml
```

---

## 🗄️ Modèle de données

### Tables générées automatiquement par Hibernate

| Table               | Description                                      |
|---------------------|--------------------------------------------------|
| `utilisateurs`      | Utilisateurs (id, nom, prenom, email unique)     |
| `salles`            | Salles de réunion (nom, capacite, batiment, etage, description) |
| `reservations`      | Réservations (date_debut, date_fin, motif, FK salle + utilisateur) |
| `equipements`       | Équipements disponibles (nom, description)       |
| `salle_equipement`  | Table de jointure Salle ↔ Équipement (Many-to-Many) |

### Relations
- `Reservation` → `Salle` (Many-to-One)
- `Reservation` → `Utilisateur` (Many-to-One)
- `Salle` ↔ `Equipement` (Many-to-Many via `salle_equipement`)

---

## ⚙️ Configuration

### `persistence.xml`

    <!-- Entités -->
    <class>com.example.model.Utilisateur</class>
    <class>com.example.model.Salle</class>
    <class>com.example.model.Reservation</class>
    <class>com.example.model.Equipement</class>

    <properties>
        <!-- Base H2 en mémoire -->
        <property name="javax.persistence.jdbc.driver" value="org.h2.Driver"/>
        <property name="javax.persistence.jdbc.url" value="jdbc:h2:mem:testdb;DB_CLOSE_DELAY=-1"/>
        <property name="javax.persistence.jdbc.user" value="sa"/>
        <property name="javax.persistence.jdbc.password" value=""/>

        <!-- Hibernate -->
        <property name="hibernate.dialect" value="org.hibernate.dialect.H2Dialect"/>
        <property name="hibernate.hbm2ddl.auto" value="create-drop"/>
        <property name="hibernate.show_sql" value="true"/>
        <property name="hibernate.format_sql" value="true"/>
    </properties>
</persistence-unit> 
```


---

## 🚀 Fonctionnalités

### ✅ Test 1 — Recherche par créneau horaire

Trouve toutes les salles **non réservées** sur un créneau donné (date_debut / date_fin) via une sous-requête d'exclusion :

```sql
SELECT DISTINCT s.*
FROM salles s
WHERE s.id NOT IN (
    SELECT r.salle_id FROM reservations r
    WHERE r.date_debut <= :fin AND r.date_fin >= :debut
)
```

**Exemple de résultat :**
```
Créneau 2026-03-09 09:00 → 11:00 :
  - Salle B202 (capacité: 15)
  - Salle C303 (capacité: 50)
  - Salle A202 (capacité: 20)
  - Salle B303 (capacité: 40)

Créneau 2026-03-13 14:00 → 16:00 :
  - Salle A101 (capacité: 30)
  - Salle B202 (capacité: 15)
  - Salle C303 (capacité: 50)
  - Salle A202 (capacité: 20)
  - Salle B303 (capacité: 40)
```

---

### ✅ Test 2 — Recherche multi-critères

Filtres disponibles sur les salles :

| Critère                    | Exemple de résultat                        |
|----------------------------|--------------------------------------------|
| Capacité ≥ 30              | A101 (30), C303 (50), B303 (40)            |
| Bâtiment = "Bâtiment A"   | A101, A202                                 |
| Capacité entre 20 et 40, étage 2 | A202 (capacité: 20, étage: 2)       |

---

### ✅ Test 3 — Pagination

Pagination des salles (2 par page) triées par `id` :

```
Total : 5 salles → 3 pages

Page 1 : Salle A101, Salle B202
Page 2 : Salle C303, Salle A202
Page 3 : Salle B303
```

**Métadonnées renvoyées par `PaginationResult` :**

| Propriété               | Valeur  |
|-------------------------|---------|
| Page courante           | 1       |
| Taille de page          | 2       |
| Nombre total de pages   | 3       |
| Nombre total d'éléments | 5       |
| Page suivante           | ✅ disponible |
| Page précédente         | ❌ non disponible |

---

## 🎬 Démonstration Vidéo

> 📹 vidéo de démonstration  montrant l'exécution complète du projet :

https://github.com/user-attachments/assets/b98750ef-3144-4ab5-a904-4cdea4f6290c



