# Quelques commandes

* Supprimer toute la BDD et la recharger avec les tables + les seeders : ``php artisan migrate:refresh --seed``

* Migrer vers la BDD : ``php artisan migrate``

* Créer un seeder : ``php artisan make:seeder "nom seeder"``

* Rajouter un seeder à la bdd : ``php artisan db:seed nom_seeder``

* Lister les commandes php artisan : ``php artisan list``

* Créer un controller :  ``php artisan make:controller "nom contrôleur"``

# Projet laravel

## Création du projet Laravel

* Installer Laravel : ``composer global require laravel/installer``

* Se mettre dans le dossier dans lequel on va réaliser le projet et faire : ``composer create-project --prefer-dist laravel/laravel nom_projet``

* Aller ensuite dans le fichier ".env" et mettre les informations concernant la bdd à laquelle se connecter.

## Ajouter un système d'authentification

* Installer le système d'authentification avec la commande ``php artisan ui vue --auth`` et créer les routes et les vues avec ``composer require laravel/ui --dev`` et ``php artisan ui vue --auth``.

* Créer la UserTableSeeder en faisant ``php artisan make:seeder UsersTableSeeder`` et écrire le texte suivant dans la fonction run afin de créer 100 utilisateurs : 

```
factory(App\User::class, 100)->create()->each(function ($user) {
        $user->posts()->save(factory(App\Post::class)->make());
    });
```

Ensuite écrire dans le "UserFactory.php" ceci afin que chaque utilisateurs aient un nom, email et mot de passe :

```
$factory->define(User::class, function (Faker $faker) {
    return [
        'name' => $faker->name,
        'email' => $faker->unique()->safeEmail,
        'email_verified_at' => now(),
        'password' => '$2y$10$TKh8H1.PfQx37YgCzwiKb.KjNyWgaHb9cbcoQgdIVFlYg7B77UdFm',
        'remember_token' => Str::random(10),
    ];
});
```

## Ajout des rôles

* Installer la librairie Spatie/permission :
    * ``composer require spatie/laravel-permission``
    * Ajouter ceci dans le dossier config/app.php :
        ```
        'providers' => [
        // ...
        Spatie\Permission\PermissionServiceProvider::class,
        ];
        ```
    * Faire ``php artisan vendor:publish --provider="Spatie\Permission\PermissionServiceProvider" --tag="migrations"``, ``php artisan migrate`` et ``php artisan vendor:publish --provider="Spatie\Permission\PermissionServiceProvider" --tag="config"``.

* Créer un seeder Roles en faisant ``php artisan make:seeder RolesTableSeeder``.

* Ajouter ``RolesTableSeeder::class,`` en haut de la liste dans le ``$this->call([`` du fichier Databaseseeder.php.

* Créer un rôle en mettant ``Role::create(['name' => 'nom_du_role']);`` dans la fonction run de RolesTableSeeder.

* Mettre ensuite 
```
factory(App\User::class, 50)->create()->each(function ($u) {
    $u->assignRole('nom_du_role');
    $u->save();
});
```

dans le UsersTableSeeder.php afin d'assigner un rôle aux users quand ils seront créés.

## Utiliser les middleware

* Dans un premier temps ajouter ceci au fichier "app/Http/Kernel.php" :
```
protected $routeMiddleware = [
    // ...
    'role' => \Spatie\Permission\Middlewares\RoleMiddleware::class,
    'permission' => \Spatie\Permission\Middlewares\PermissionMiddleware::class,
    'role_or_permission' => \Spatie\Permission\Middlewares\RoleOrPermissionMiddleware::class,
];
```

* Ensuite créer 3 trois controller en faisant ``php artistan make:controller "'nom_du_role'Controller"

* Créer 3 vues pour les 3 rôles

* Mettre "return view('nom_du_role'); dans la fonction index du controller de chacun des rôles.

* Et écrire pour chacun des rôles dans les fichier "routes/web.php" ceci :
```
Route::group(['middleware' => ['role:'nom_du_role']], function () {
    Route::get('/'nom_du_role'', ''nom_du_role'Controller@index')->name('nom_du_role');
});
```

## Créer un profile

* Faire les commandes suivantes :
    * ``php artisan make:controller ProfileController``
    * ``php artisan make:ProfileFactory``
    * ``php artisan make:migration create_profile_table``

* Ajouter ceci au model de l'user (user.php) :
```
public function profile() {
    return $this->hasOne('App\Profile');
   }
```

et ceci au model du profile :
```
public function user() {
    return $this->belongsTo('App\User');
}
```

* Mettre la chose suivante dans le profilefactory :
```
'firstName' => $faker->firstName,
'lastName' => $faker->lastName,
'phoneNumber' => $faker->phoneNumber,
'address' => $faker->address,
```
