# Todo App avec Laravel et Vue.js

Une application de gestion de tâches moderne développée avec Laravel 10 et Vue.js 3.

## Fonctionnalités

- ✅ Création, modification et suppression de tâches
- ✅ Organisation des tâches par catégories
- ✅ Personnalisation des catégories avec des couleurs
- ✅ Filtrage des tâches (toutes, actives, complétées)
- ✅ Interface utilisateur moderne et intuitive
- ✅ Persistance des données dans une base de données MySQL

## Prérequis

Avant de commencer, assurez-vous d'avoir installé :

- PHP 8.1 ou supérieur
- Composer (gestionnaire de paquets PHP)
- Node.js et npm
- MySQL ou MariaDB
- Git (optionnel)

## Installation

1. **Cloner le projet**
```bash
git clone [URL_DU_PROJET]
cd todoapp_vue-laravel
```

2. **Installer les dépendances PHP**
```bash
composer install
```

3. **Installer les dépendances JavaScript**
```bash
npm install
```

4. **Configurer l'environnement**
- Copier le fichier `.env.example` en `.env`
```bash
cp .env.example .env
```
- Générer une clé d'application
```bash
php artisan key:generate
```
- Configurer la base de données dans le fichier `.env` :
```
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=todoapp
DB_USERNAME=root
DB_PASSWORD=
```

5. **Créer la base de données**
- Créer une base de données MySQL nommée `todoapp`
- Exécuter les migrations
```bash
php artisan migrate
```

## Démarrage de l'application

1. **Démarrer le serveur Laravel**
```bash
php artisan serve
```

2. **Démarrer le serveur de développement Vite**
```bash
npm run dev
```

3. **Accéder à l'application**
Ouvrez votre navigateur et accédez à :
```
http://localhost:8000
```

## Structure du projet

```
todoapp_vue-laravel/
├── app/
│   ├── Http/
│   │   └── Controllers/
│   │       ├── TodoController.php
│   │       └── CategoryController.php
│   └── Models/
│       ├── Todo.php
│       └── Category.php
├── database/
│   └── migrations/
│       ├── create_todos_table.php
│       └── create_categories_table.php
├── resources/
│   ├── js/
│   │   ├── App.vue
│   │   └── app.js
│   └── views/
│       └── welcome.blade.php
├── routes/
│   ├── api.php
│   └── web.php
└── public/
```

## Utilisation

### Gestion des tâches
- **Ajouter une tâche** : Saisissez le texte dans le champ et appuyez sur Entrée
- **Modifier une tâche** : Double-cliquez sur le texte de la tâche
- **Supprimer une tâche** : Cliquez sur l'icône de corbeille
- **Marquer comme complétée** : Cochez la case à gauche de la tâche

### Gestion des catégories
- **Ajouter une catégorie** : Utilisez le formulaire en haut de l'application
- **Modifier une catégorie** : Cliquez sur l'icône de crayon
- **Supprimer une catégorie** : Cliquez sur l'icône de corbeille
- **Associer une tâche à une catégorie** : Utilisez le menu déroulant à droite de la tâche

## API Endpoints

### Tâches
- `GET /api/todos` - Liste toutes les tâches
- `POST /api/todos` - Crée une nouvelle tâche
- `GET /api/todos/{id}` - Affiche une tâche spécifique
- `PUT /api/todos/{id}` - Met à jour une tâche
- `DELETE /api/todos/{id}` - Supprime une tâche

### Catégories
- `GET /api/categories` - Liste toutes les catégories
- `POST /api/categories` - Crée une nouvelle catégorie
- `GET /api/categories/{id}` - Affiche une catégorie spécifique
- `PUT /api/categories/{id}` - Met à jour une catégorie
- `DELETE /api/categories/{id}` - Supprime une catégorie


## Ressources utiles

- [Documentation Laravel](https://laravel.com/docs)
- [Documentation Vue.js](https://vuejs.org/guide)
- [Tutoriel Laravel pour débutants](https://laravel.com/docs/10.x/installation)
- [Tutoriel Vue.js pour débutants](https://vuejs.org/guide/introduction.html)

## Support

Si vous rencontrez des problèmes ou avez des questions :
1. Consultez la documentation
2. Recherchez sur Stack Overflow
3. Créez une issue sur GitHub

