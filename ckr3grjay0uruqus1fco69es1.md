## Laravel - Health Check

Ce n'est pas une surprise, il existe de nombreux packages Laravel permettant de contrôler la santé de votre application : accès à la base de données, écriture dans des dossiers de stockage, validité du certificat SSL, services Mail, Redis, ... et qui peuvent être étendus pour créer des contrôles spécifiques à votre application.

Mais pour des petits projets avec peu de fonctionnalités, ces packages nous semblent légèrement surdimensionnés (toujours un peu de scrupules à installer un package pour n'en utiliser qu'une infime partie).

## Ok, vous faîtes quoi alors pour les petits projets ?

Pour ceux-là, nous utilisons généralement un simple contrôleur Laravel qui va vérifier que l'accès à la base de données fonctionne et que l'écriture dans un dossier de stockage de l'application est possible.


**/app/Http/Controllers/HealthcheckController.php**

```
<?php

namespace App\Http\Controllers;

use Illuminate\Support\Facades\DB;
use Illuminate\Support\Facades\Storage;

class HealthcheckController extends Controller
{
    private bool $displayError = false;

    public function check(): \Illuminate\Http\Response
    {
        try {
            $healthcheck =
                $this->checkDatabase()
                && $this->checkStorage();

            return $this->makeResponse($healthcheck);
        } catch (\Exception $e) {
            return $this->makeResponse(false, $e->getMessage());
        }
    }

    private function makeResponse(bool $healthcheck = false, string $message = ''): \Illuminate\Http\Response
    {
        return response($message, ($healthcheck) ? 200 : 500);
    }

    private function checkDatabase(string $table = null): bool
    {
        if (null === $table) {
            $table = 'migrations';
        }

        try {
            DB::table($table)->take(1)->get();
        } catch (\Exception $e) {
            report($e);
            if ($this->displayError) {
                throw new \Exception($e);
            } else {
                return false;
            }
        }

        return true;
    }

    private function checkStorage(string $disk = 'local'): bool
    {
        try {
            $filename = 'healthcheck.txt';
            $date = now()->toDatetimeString();

            if (!Storage::disk($disk)->put($filename, $date)) {
                return false;
            }

            if (!Storage::disk($disk)->delete($filename)) {
                return false;
            }
        } catch (\Exception $e) {
            report($e);
            if ($this->displayError) {
                throw new \Exception($e);
            } else {
                return false;
            }
        }

        return true;
    }
}
``` 

> Les erreurs sont traitées par le logger par défaut de Laravel (par défaut, écriture dans le fichier `laravel...log`) et aucune information d'erreur n'est renvoyée par le contrôleur, simplement un code HTTP 500 au lieu de 200. Si vous souhaitez que le contrôleur affiche les erreurs, passez le booléen `$displayError` à `true`.

Pour rendre le contrôleur accessible (*https://monsite.com/healthcheck*), pensez à ajouter la route Web associée (l'appel à la fonction `name` est facultatif) :

**/routes/web.php**

```
use App\Http\Controllers\HealthcheckController;

Route::get('/healthcheck', [HealthcheckController::class, 'check'])->name('healthcheck');
``` 

## Et maintenant, vous en faîtes quoi de votre Health Check ?

Avoir une URL accessible sur son application c'est bien, mais à moins que vous n'ayez envie de passer votre temps devant votre navigateur à rafraîchir la page ou de développer un script d'appel de l'URL qui va ensuite vous notifier par le moyen de votre choix, il existe là aussi de nombreuses solutions qui peuvent s'en charger pour vous :

- Configuration d'une URL de Health Check qui va être appelée à intervalle régulier. D'autres types qu'un appel HTTP sont généralement possibles : API, DNS, ping, réponse sur un port donné, test sur le contenu de la réponse, …
- Notification en cas de problème : e-mail, SMS, notification Push, Slack, Twitter, Webhooks, …
- Informations détaillées sur la santé de l'application et de l'hébergement : temps de réponse, de rendu de la page, historique des incidents, …

Actuellement, nous utilisons la solution [UptimeRobot ](https://uptimerobot.com) qui propose une offre gratuite (jusqu'à 50 contrôleurs, notification par e-mail uniquement) suffisante pour de la surveillance de premier niveau. Vous pourrez en plus créer des pages publiques de statut, personnalisables avec la sélection des contrôleurs de votre choix à afficher, la protection par mot de passe, que vous pourrez ensuite afficher sur un écran en mode tableau de bord, communiquer à vos équipes ou vos clients.

**Exemple de page publique de statut UptimeRobot**

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1626265359150/a_NGhG5lc.png)
