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
