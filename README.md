# Exercice 11 – API Laravel Kids  
README – Solution Complète

---

## 1. Mise en place du projet

### Étapes d’installation
# 1️⃣ Étapes de mise en place


1. Installer les dépendances :
```bash
composer install
Copier l’environnement :

bash
Copier le code
copy .env.example .env
Générer la clé :

bash
Copier le code
php artisan key:generate
Créer la base SQLite :

pgsql
Copier le code
database/database.sqlite
Modifier .env :

ini
Copier le code
DB_CONNECTION=sqlite
DB_DATABASE=database/database.sqlite
Lancer les migrations :

bash
Copier le code
php artisan migrate
Exécuter les seeders (pour créer l’utilisateur Père Noël et les Kids) :

bash
Copier le code
php artisan db:seed
Lancer le serveur :

bash
Copier le code
php artisan serve

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


# 📘 Synthèse complète – Fichiers modifiés, étapes et code ajouté  
Examen Kids API – Laravel (CRUD + Login + Tokens + Abilities)

---

# 1️⃣ Étapes de mise en place

1. Télécharger l’archive `api-base.zip`
2. L’ouvrir dans VSCode
3. Installer les dépendances :

```bash

composer install
Copier l’environnement :

bash

Copier le code

copy .env.example .env
Générer la clé :

bash

Copier le code

php artisan key:generate
Créer la base SQLite :

pgsql

Copier le code

database/database.sqlite
Modifier .env :

ini

Copier le code

DB_CONNECTION=sqlite
DB_DATABASE=database/database.sqlite
Lancer les migrations :

bash

Copier le code

php artisan migrate
Exécuter les seeders (pour créer l’utilisateur Père Noël et les Kids) :

bash

Copier le code

php artisan db:seed
Lancer le serveur :

bash

Copier le code

php artisan serve
2️⃣ Fichiers exactement modifiés
Voici la liste complète :

Fichier	Rôle	Modifications
routes/api.php	Déclaration des routes Kids + Login + Tokens	Ajout des routes REST + middleware Sanctum
app/Http/Controllers/KidsController.php	CRUD des Kids	Tout le CRUD complet + validations
app/Http/Controllers/TokensController.php	Gestion login + tokens + abilities	Code pour créer token admin/élèves
app/Models/Kid.php	Modèle + attributs mass-assignables	Ajout $fillable
database/seeders/UserSeeder.php	Création Père Noël	Ajout user + mot de passe hashé
database/seeders/KidSeeder.php	Données des enfants	Ajout d’un tableau de Kids
app/Http/Middleware/Authenticate.php	Réponse JSON en cas d’absence d’auth	Remplacement du redirect() par JSON
app/Http/Middleware/ValidateSignature.php	(automatique)	Aucun changement majeur
app/Http/Middleware/VerifyCsrfToken.php	Pas utilisé en API	Aucun changement

3️⃣ Code EXACT ajouté dans chaque fichier
✅ A. routes/api.php

php

Copier le code

use App\Http\Controllers\KidsController;
use App\Http\Controllers\TokensController;
use Illuminate\Support\Facades\Route;

// Login
Route::post("/login", [TokensController::class, "login"]);

// Routes protégées par Sanctum
Route::middleware("auth:sanctum")->group(function () {

    // Kids CRUD
    Route::get("/kids", [KidsController::class, "index"]);
    Route::post("/kids", [KidsController::class, "store"]);
    Route::get("/kids/{id}", [KidsController::class, "show"]);
    Route::patch("/kids/{id}", [KidsController::class, "update"]);
    Route::delete("/kids/{id}", [KidsController::class, "destroy"]);

    // Gestion tokens
    Route::post("/tokens/create", [TokensController::class, "create"]);
    Route::get("/tokens", [TokensController::class, "list"]);
});
✅ B. app/Http/Controllers/KidsController.php
php

Copier le code

<?php

namespace App\Http\Controllers;

use App\Models\Kid;
use Illuminate\Http\Request;

class KidsController extends Controller
{
    public function index()
    {
        return Kid::all();
    }

    public function store(Request $request)
    {
        $data = $request->validate([
            "name" => "required|string|max:255",
            "birthDate" => "required|date",
            "address" => "required|string|max:255",
            "zipCode" => "required|string",
            "city" => "required|string",
            "wishList" => "nullable|string",
            "wiseLevel" => "required|integer|min:1|max:5"
        ]);

        return response()->json(Kid::create($data), 201);
    }

    public function show($id)
    {
        return Kid::findOrFail($id);
    }

    public function update(Request $request, $id)
    {
        $kid = Kid::findOrFail($id);

        $data = $request->validate([
            "wiseLevel" => "required|integer|min:1|max:5"
        ]);

        $kid->update($data);

        return response()->json($kid, 200);
    }

    public function destroy($id)
    {
        Kid::findOrFail($id)->delete();
        return response()->json(null, 204);
    }
}
✅ C. app/Http/Controllers/TokensController.php

php

Copier le code

<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\Support\Facades\Auth;

class TokensController extends Controller
{
    public function login(Request $request)
    {
        $credentials = $request->validate([
            "email" => "required|email",
            "password" => "required"
        ]);

        if (!Auth::attempt($credentials)) {
            return response()->json(["error" => "Invalid credentials"], 403);
        }

        $user = Auth::user();

        $token = $user->createToken("default", ["*"])->plainTextToken;

        return ["token" => $token];
    }

    public function create(Request $request)
    {
        $request->validate([
            "name" => "required",
            "abilities" => "required|array"
        ]);

        $token = $request->user()->createToken(
            $request->name,
            $request->abilities
        );

        return ["token" => $token->plainTextToken];
    }

    public function list(Request $request)
    {
        return $request->user()->tokens;
    }
}
✅ D. app/Models/Kid.php

php

Copier le code

<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Kid extends Model
{
    protected $fillable = [
        "name", "birthDate", "address", "zipCode",
        "city", "wishList", "wiseLevel"
    ];
}
✅ E. database/seeders/UserSeeder.php

php

Copier le code
<?php

namespace Database\Seeders;

use Illuminate\Database\Seeder;
use Illuminate\Support\Facades\DB;
use Illuminate\Support\Facades\Hash;

class UserSeeder extends Seeder
{
    public function run(): void
    {
        DB::table("users")->insert([
            "name" => "Père noël",
            "email" => "pere@noel.com",
            "password" => Hash::make("salut"),
            "created_at" => now(),
            "updated_at" => now()
        ]);
    }
}
✅ F. database/seeders/KidSeeder.php
php
Copier le code
<?php

namespace Database\Seeders;

use Illuminate\Database\Seeder;
use Illuminate\Support\Facades\DB;

class KidSeeder extends Seeder
{
    public function run(): void
    {
        $kids = [
            ["name" => "Alice", "birthDate" => "2016-01-01", "address" => "Rue A", "zipCode" => "1000", "city" => "Lausanne", "wishList" => "Lego", "wiseLevel" => 5],
            ["name" => "Bob", "birthDate" => "2015-02-12", "address" => "Rue B", "zipCode" => "1004", "city" => "Lausanne", "wishList" => "PS5", "wiseLevel" => 3]
        ];

        foreach ($kids as $kid) {
            $kid["created_at"] = now();
            $kid["updated_at"] = now();
            DB::table("kids")->insert($kid);
        }
    }
}
✅ G. app/Http/Middleware/Authenticate.php

php

Copier le code

protected function unauthenticated($request, array $guards)
{
    return response()->json(["error" => "Unauthenticated"], 401);
}
4️⃣ Résultat final : ce que ton API sait faire
Fonction	Route	Protection
Login	POST /login	Public
Lister tokens	GET /tokens	Auth sanctum
Créer token	POST /tokens/create	Auth sanctum
Lister kids	GET /kids	Auth sanctum
Ajouter kid	POST /kids	Auth sanctum
Voir kid	GET /kids/{id}	Auth sanctum
Modifier kid (wiseLevel)	PATCH /kids/{id}	Auth sanctum
Supprimer kid	DELETE /kids/{id}	Auth sanctum

1. Lance Postman
2. Clique sur **"New" → Request"**
3. Donne un nom à ta requête (ex : “Login”)
4. Choisis ou crée une collection (ex : “Kids API Test”)

---

# ✅ 2. Tester le Login (obtenir un token)

⭐ Cette étape est **obligatoire**, car toutes les routes de ton API sont protégées par *Sanctum*.

### 📌 Requête
- **Méthode :** `POST`
- **URL :** `http://127.0.0.1:8000/api/login`

### 📌 Body → JSON
Sélectionne : `Body → Raw → JSON`

Colle :

```json
{
  "email": "pere@noel.com",
  "password": "salut"
}
📌 Réponse attendue
json
Copier le code
{
  "token": "xxxxxxxxxxxxxxxxxxxxxxxxxxxx"
}
➡️ Copie ce token, tu en auras besoin pour toutes les autres requêtes.

✅ 3. Ajouter le token à Postman
Pour éviter de recoller le token à chaque requête, fais ceci :

Clique sur Authorization dans ta requête

Type : Bearer Token

Colle le token reçu

📌 Poste d’authentification :

yaml
Copier le code
Authorization → Type: Bearer Token
Token : <ton_token_ici>
Super important :
➡️ Toutes les requêtes protégées doivent avoir ce Bearer Token !

✅ 4. Tester les routes Kids API
🟦 4.1. Lister tous les kids
GET

http://127.0.0.1:8000/api/kids

Auth : oui

Résultat attendu : un tableau d’enfants

🟩 4.2. Ajouter un enfant
POST

http://127.0.0.1:8000/api/kids

Auth : oui

Body → Raw JSON :
json
Copier le code
{
  "name": "Marie",
  "birthDate": "2015-11-05",
  "address": "Rue du Nord 5",
  "zipCode": "1000",
  "city": "Lausanne",
  "wishList": "Poupée",
  "wiseLevel": 4
}
Réponse :
➡️ Code 201 CREATED
➡️ Le kid nouvellement créé

🟨 4.3. Voir un enfant
GET

http://127.0.0.1:8000/api/kids/1

Auth : oui

🟧 4.4. Modifier le niveau de sagesse (wiseLevel)
PATCH

http://127.0.0.1:8000/api/kids/1

Auth : oui

Body :

json
Copier le code
{
  "wiseLevel": 2
}
Réponse :
➡️ Code 200 OK

🟥 4.5. Supprimer un kid
DELETE

http://127.0.0.1:8000/api/kids/1

Auth : oui

Réponse :
➡️ Code 204 NO CONTENT

✅ 5. Tester la gestion des tokens (optionnel)
🔐 5.1. Voir tous mes tokens
GET

http://127.0.0.1:8000/api/tokens

Auth : Oui

🔑 5.2. Créer un token avec permissions spécifiques
POST

http://127.0.0.1:8000/api/tokens/create

Auth : Oui

Body :

json
Copier le code
{
  "name": "admin",
  "abilities": ["*"]
}
Réponse :

json
Copier le code
{
  "token": "nouveau_token_ici"
}
📌 Résumé visuel (workflow Postman)
markdown
Copier le code
1. LOGIN → Récupérer token
2. Ajouter token dans Authorization (Bearer)
3. Tester les routes protégées :
   - GET kids
   - POST kids
   - GET kid/{id}
   - PATCH kid/{id}
   - DELETE kid/{id}
4. Gestion des tokens (optionnel)
🎯 Conseils pour réussir l’examen
✔ Si tu reçois 401 Unauthenticated → tu as oublié le Bearer Token
✔ Si tu reçois 422 Unprocessable Entity → ton JSON n’est pas correct
✔ Si tu reçois 404 Not Found → mauvais ID ou mauvaise URL
✔ Toujours vérifier :

Méthode HTTP correcte (GET / POST / PATCH / DELETE)

Body en JSON + Raw

Token dans Authorization

Dans Postman → Barre du haut →
👉 New → HTTP Request

C’EST cette option-là qu’il te faut pour tester ton API.

✅ 2. Une fois que tu as créé une requête HTTP

Tu dois remplir :

🔹 A — La méthode

GET
POST
PATCH
DELETE
(selon la route que tu veux tester)

🔹 B — L’URL

Par exemple :

http://127.0.0.1:8000/api/login
http://127.0.0.1:8000/api/kids

🔹 C — L’onglet Body (pour POST / PATCH)

Clique sur :

Body → Raw → JSON


Puis mets ton JSON.

🔹 D — Onglet Authorization

Sélectionne :

Type → Bearer Token
Token → <ton_token>

🚨 IMPORTANT

WiseLVL => rulein XS S M L XL etc
Créer un fichier UpdateKidRequest => Faire en sorte qu'une contrainte soit mise afin que Wise etc... ne puisse pas être update

Tu ne dois PAS utiliser les collections qui sont affichées sur ton écran ("Contract Testing", "Integration Testing", etc.).

👉 Ce sont des exemples fournis par Postman — ils ne servent à rien pour ton projet Laravel.

🔥 Voici EXACTEMENT l’endroit où être dans Postman

Sur la colonne de gauche :

✔ Clique sur Collections
✔ Puis sur New Collection (optionnel pour organiser)

Mais ce qui compte vraiment :

👉 Clique sur New → HTTP Request

Puis travaille UNIQUEMENT dans cette requête.

🎯 Résumé visuel (super simple)
1️⃣ New → HTTP Request
2️⃣ Méthode GET/POST/PATCH/DELETE
3️⃣ URL = http://127.0.0.1:8000/...
4️⃣ Authorization → Bearer Token
5️⃣ Body → Raw + JSON (si nécessaire)
6️⃣ SEND 🚀

<?php

namespace App\Http\Requests\Kids;

use App\Models\Kid;
use Illuminate\Foundation\Http\FormRequest;
use Illuminate\Validation\Rule;

class UpdateKidRequest extends FormRequest
{
    /
     
Determine if the user is authorized to make this request.*/
  public function authorize(): bool{
      return true;}

    /
     
Get the validation rules that apply to the request.*
@return array<string, \Illuminate\Contracts\Validation\ValidationRule|array<mixed>|string>*/
public function rules(): array{
    return ["wiseLevel" => ["nullable", "string",Rule::in([Kid::WISE_LEVEL_1,Kid::WISE_LEVEL_2,Kid::WISE_LEVEL_3,Kid::WISE_LEVEL_4,])]];}
}
Créer un fichier UpdateKidRequest.

Ajouter des règles de validation.

Empêcher wiseLevel (ou "WiseLVL" selon ton code) d’être modifié → même si l’utilisateur l’envoie, Laravel doit l’ignorer.

🧠 1. Créer la Request

Dans le terminal :

php artisan make:request UpdateKidRequest


Laravel crée un fichier :

app/Http/Requests/UpdateKidRequest.php

🧠 2. Autoriser la requête

Dans la méthode authorize() :

public function authorize(): bool
{
    return true; // On autorise l'accès, pas le sujet ici
}

🧠 3. A

Dans rules() :

public function rules(): array
{
    return [
        'name' => 'sometimes|string|max:255',
        'birthDate' => 'sometimes|date',
        'address' => 'sometimes|string',
        'zipCode' => 'sometimes|string',
        'city' => 'sometimes|string',
        'wishList' => 'sometimes|string',

        // WiseLVL a des valeurs OBLIGATOIRES mais NE DOIT PAS être modifiable
        'wiseLevel' => 'prohibited', 
    ];
}

👉 Pourquoi prohibited ?

Parce que ça empêche totalement l’utilisateur d’envoyer ce champ dans une requête update.

Si quelqu’un tente :

{
  "wiseLevel": "*"
}


Alors Laravel renvoie :

422 Unprocessable Entity → Ce champ ne peut PAS être modifié.

🧠 OPTION : Si tu veux juste valider les valeurs sans empêcher la modif

Tu utiliserais :

'wiseLevel' => 'in:*,*,*',


Mais PAS dans ce cas.
Dans l’examen → on doit empêcher la mise à jour ⇒ donc prohibited est parfait.

🧠 4. Modifier ton controller pour utiliser la Request

Dans ton KidsController → méthode update :

Avant (probable) :

public function update(Request $request, Kid $kid)
{
    $kid->update($request->all());
    return $kid;
}


Après (correct) :

public function update(UpdateKidRequest $request, Kid $kid)
{
    $data = $request->validated();   // wiseLevel sera automatiquement supprimé si envoyé

    $kid->update($data);
    return response()->json($kid, 200);
}
