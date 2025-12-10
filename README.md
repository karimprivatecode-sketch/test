# Exercice 11 – API Laravel Kids  
README – Solution Complète

---

## 1. Mise en place du projet

### Étapes d’installation

```bash
composer install
copy .env.example .env
php artisan key:generate
php artisan migrate
php artisan serve
Base de données
SQLite → créer database/database.sqlite

MySQL → configurer les champs DB_* dans .env

2. Règles du modèle Kid
Champ	Type	Règles
name	string	1–250
birthDate	date	obligatoire
address	string	1–250
zipCode	string	4 chiffres
city	string	1–250
wishList	string	optionnel
wiseLevel	int	1–4

Important
Seul wiseLevel peut être modifié dans PATCH.

3. Permissions (abilities Sanctum)
Méthode	Route	Action	Permission
POST	/kids	créer un enfant	aucune
GET	/kids	liste des enfants	kids:list ou *
GET	/kids/{id}	voir un enfant	* ou kids:read:unwise si wiseLevel=4
PATCH	/kids/{id}	modifier wiseLevel	kids:update ou *
DELETE	/kids/{id}	supprimer un enfant	*

4. Cas spécial : Père Fouettard 👹
Un token avec l’ability :

arduino
Copier le code
kids:read:unwise
Peut uniquement lire les enfants dont :

ini
Copier le code
wiseLevel = 4
Sinon : 403 Forbidden.

5. Codes HTTP importants
Action	Code
Création	201 Created
Lecture / update	200 OK
Suppression	204 No Content
Permission refusée	403 Forbidden
Introuvable	404 Not Found

6. Contrôleur KidsController complet
php
Copier le code
<?php

namespace App\Http\Controllers;

use App\Models\Kid;
use Illuminate\Http\Request;

class KidsController extends Controller
{
    public function index(Request $request)
    {
        $token = $request->user()->currentAccessToken();
        if (!$token->can('kids:list') && !$token->can('*')) {
            return response()->json(['error' => 'Forbidden'], 403);
        }
        return Kid::all();
    }

    public function store(Request $request)
    {
        $validated = $request->validate([
            'name'      => 'required|string|min:1|max:250',
            'birthDate' => 'required|date',
            'address'   => 'required|string|min:1|max:250',
            'zipCode'   => 'required|digits:4',
            'city'      => 'required|string|min:1|max:250',
            'wishList'  => 'nullable|string|max:2000',
        ]);

        $kid = Kid::create($validated);
        return response()->json($kid, 201);
    }

    public function show(Request $request, $id)
    {
        $kid = Kid::findOrFail($id);
        $token = $request->user()->currentAccessToken();

        if ($token->can('*')) {
            return response()->json($kid);
        }

        if ($token->can('kids:read:unwise') && $kid->wiseLevel === Kid::WISE_LEVEL_4) {
            return response()->json($kid);
        }

        return response()->json(['error' => 'Forbidden'], 403);
    }

    public function update(Request $request, $id)
    {
        $token = $request->user()->currentAccessToken();
        if (!$token->can('kids:update') && !$token->can('*')) {
            return response()->json(['error' => 'Forbidden'], 403);
        }

        $kid = Kid::findOrFail($id);

        $validated = $request->validate([
            'wiseLevel' => 'required|in:1,2,3,4',
        ]);

        $kid->update($validated);
        return response()->json($kid, 200);
    }

    public function destroy(Request $request, $id)
    {
        $token = $request->user()->currentAccessToken();
        if (!$token->can('*')) {
            return response()->json(['error' => 'Forbidden'], 403);
        }

        $kid = Kid::findOrFail($id);
        $kid->delete();

        return response()->json(null, 204);
    }
}
7. Routes API (routes/api.php)
php
Copier le code
<?php

use App\Http\Controllers\KidsController;
use App\Http\Controllers\TokensController;
use Illuminate\Support\Facades\Route;

// Public
Route::post("kids", [KidsController::class, "store"]);

// Auth Required
Route::middleware("auth:sanctum")->group(function () {
    Route::get("kids", [KidsController::class, "index"]);
    Route::get("kids/{id}", [KidsController::class, "show"]);
    Route::patch("kids/{id}", [KidsController::class, "update"]);
    Route::delete("kids/{id}", [KidsController::class, "destroy"]);
});

// Tokens (admin)
Route::middleware(["auth:sanctum", "ability:*"])->group(function () {
    Route::apiResources([
        "tokens" => TokensController::class,
    ]);
});
8. Commandes utiles
bash
Copier le code
composer install
php artisan migrate
php artisan key:generate
php artisan serve
9. Version résumée en 30 secondes
POST kids → public

GET kids → kids:list / *

GET kids/{id} → * ou kids:read:unwise si wiseLevel=4

PATCH kids/{id} → kids:update / *

DELETE kids/{id} → *

Seul wiseLevel est modifiable

Père Fouettard → lit seulement wiseLevel=4

Codes : 201 / 200 / 204 / 403 / 404

10. Anti-sèche finale
pgsql
Copier le code
POST kids → public
GET kids → kids:list ou *
GET {id} → * ou unwise si wiseLevel=4
PATCH kids → update ou *
DELETE kids → *

store() : valider tout
update() : wiseLevel only
403 si permission manquante


# 🧠 CHEAT SHEET ULTIME – EPSIC LARAVEL  
Guide pour réussir TOUS les exercices (REST, CRUD, Login, Auth, Audit Trail, Tokens)

---

# 1️⃣ Mise en place d’un projet Laravel

Toujours commencer par :

```bash
composer install
copy .env.example .env
php artisan key:generate
php artisan migrate
php artisan serve
Si SQLite est utilisé :
Créer : database/database.sqlite

Dans .env :

ini
Copier le code
DB_CONNECTION=sqlite
DB_DATABASE=database/database.sqlite
2️⃣ Comment créer un modèle + migration
Si demandé de créer une ressource :

bash
Copier le code
php artisan make:model Kid -m
Ajoute les champs dans database/migrations/...create_kids_table.php :

php
Copier le code
$table->string('name');
$table->date('birthDate');
$table->string('address');
$table->string('zipCode');
$table->string('city');
$table->text('wishList')->nullable();
$table->tinyInteger('wiseLevel');
Relancer :

bash
Copier le code
php artisan migrate
Dans le modèle Kid :

php
Copier le code
protected $fillable = [
    'name', 'birthDate', 'address', 'zipCode', 'city', 'wishList', 'wiseLevel'
];
3️⃣ Comment créer un contrôleur REST API
Toujours :

bash
Copier le code
php artisan make:controller KidsController
Structure à respecter :

php
Copier le code
public function index()      // GET /kids
public function store()      // POST /kids
public function show($id)    // GET /kids/{id}
public function update($id)  // PATCH /kids/{id}
public function destroy($id) // DELETE /kids/{id}
4️⃣ Comment écrire une validation correcte
Toujours utiliser :

php
Copier le code
$request->validate([
    'name' => 'required|string|min:1|max:250',
    'birthDate' => 'required|date',
    'zipCode' => 'required|digits:4',
]);
Spécificité :
Si un seul champ modifiable → mettre uniquement ce champ dans validate()

5️⃣ Comment retourner les bons codes HTTP
Situation	Code
Ressource créée	201
Lecture / update OK	200
Suppression	204
Introuvable	404
Pas le droit	403
Mauvaise requête	400 ou auto par validator

Toujours renvoyer avec :

php
Copier le code
return response()->json([...], CODE);
6️⃣ Comment gérer les tokens (Sanctum)
Créer un token :

php
Copier le code
$user->createToken("admin", ["*"]);
Créer un token limité :

php
Copier le code
$user->createToken("reader", ["kids:list"]);
Récupérer token et permissions :

php
Copier le code
$token = $request->user()->currentAccessToken();
$token->can("kids:list"); // true/false
7️⃣ Comment protéger les routes
Dans routes/api.php :

php
Copier le code
Route::middleware("auth:sanctum")->group(function () {
    Route::get("/kids", [KidsController::class, "index"]);
});
Pour routes accessibles uniquement avec ability "*" :

php
Copier le code
Route::middleware(["auth:sanctum", "ability:*"])->group(function () {
    Route::delete("/kids/{id}", ...);
});
8️⃣ Comment vérifier une permission dans un contrôleur
Toujours :

php
Copier le code
$token = $request->user()->currentAccessToken();

if (!$token || !$token->can("kids:list")) {
    return response()->json(["error" => "Forbidden"], 403);
}
Cas accès total :

php
Copier le code
if ($token->can("*")) { ... }
Cas permission spéciale :

php
Copier le code
if ($token->can("kids:read:unwise") && $kid->wiseLevel == 4) {
    return $kid;
}
9️⃣ Comment gérer un login (Exercice Login)
Créer un user test dans seeder.

Login :

php
Copier le code
if (!Auth::attempt($credentials)) {
    return response()->json(["error" => "Invalid credentials"], 403);
}
Retourner un token :

php
Copier le code
return response()->json([
    "token" => $user->createToken("default", ["*"])->plainTextToken
]);
🔟 Comment structurer un CRUD propre
Voici un CRUD minimaliste à apprendre par cœur :

php
Copier le code
public function index() {
    return Model::all();
}

public function store(Request $r) {
    $data = $r->validate([...]);
    return response()->json(Model::create($data), 201);
}

public function show($id) {
    return Model::findOrFail($id);
}

public function update(Request $r, $id) {
    $item = Model::findOrFail($id);
    $data = $r->validate([...]);
    $item->update($data);
    return response()->json($item, 200);
}

public function destroy($id) {
    Model::findOrFail($id)->delete();
    return response()->json(null, 204);
}
1️⃣1️⃣ Comment réussir un exercice d’autorisation (Exercice 08)
Toujours appliquer :

Récupérer token

Vérifier permission via can(...)

Retourner 403 si pas autorisé

Retourner ressource si OK

1️⃣2️⃣ Comment réussir un audit trail (Exercice 09–10)
Un audit trail consiste à enregistrer TOUTES les actions.

Créer un modèle + migration :

bash
Copier le code
php artisan make:model Audit -m
Champs typiques :

php
Copier le code
$table->string('action');
$table->string('model');
$table->integer('model_id');
$table->json('before')->nullable();
$table->json('after')->nullable();
$table->timestamp('created_at');
Dans le contrôleur :

php
Copier le code
Audit::create([
    "action" => "update",
    "model" => "Kid",
    "model_id" => $kid->id,
    "before" => json_encode($kid->getOriginal()),
    "after" => json_encode($kid->getAttributes()),
]);
À déclencher dans store/update/delete.

1️⃣3️⃣ Comment identifier le type d’exercice pendant l’examen
Si on te demande…	C’est un exercice de…
Créer/Modifier/Supprimer	CRUD
login + token	Authentification
abilities	Autorisation
historique d’actions	Audit trail
écrire routes API	REST
validation de données	Validation

🧠 ASTUCE FINALE – Les 5 commandes à retenir absolument
bash
Copier le code
php artisan make:model Kid -m
php artisan make:controller KidsController
php artisan migrate
php artisan serve
php artisan tinker
🎯 Résumé ultra-court
Validation → validate()

Permissions → $token->can()

Codes HTTP → 201 / 200 / 204 / 403 / 404

CRUD propre → index/store/show/update/destroy

Audit trail → enregistrer action + before/after

Sanctum → abilities + auth:sanctum
