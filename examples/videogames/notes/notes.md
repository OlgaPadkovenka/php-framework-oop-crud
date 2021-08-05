1. Je supprime la requette sql dans index.php
$sql = 'SELECT
`game`.`id`,
`game`.`title`,
`game`.`release_date`,
`game`.`link`,
`game`.`developer_id`,
`game`.`platform_id`,
`developer`.`name` as `developer_name`,
`developer`.`link` as `developer_link`,
`platform`.`name` as `platform_name`,
`platform`.`link` as `platform_link`
FROM `game`
JOIN `developer` ON `game`.`developer_id` = `developer`.`id`
JOIN `platform` ON `game`.`platform_id` = `platform`.`id`
ORDER BY `' . $orderBy . '`';

$databaseHandler = new PDO('mysql:host=localhost;dbname=videogames', 'root', 'root');
$statement = $databaseHandler->query($sql);
$games = $statement->fetchAll();

$statement = $databaseHandler->query('SELECT * FROM `developer`');
$developers = $statement->fetchAll();

$statement = $databaseHandler->query('SELECT * FROM `platform`');
$platforms = $statement->fetchAll();

2. J'importe dans index.php
require_once __DIR__ . '/vendor/autoload.php';

use App\Model\Game;
use App\Model\Platform;
use App\Model\Developer;

3. Je cherche toute les tables dans index.php.

$games = Game::findAll();
$developers = Developer::findAll();
$platforms = Platform::findAll();

P.S. la méthode findAll n'est pas dans Game.php. Parce que Game hérite les méthodes de la super-class AbstractModel (class Game extends AbstractModel) qui a la méthode findAll. Cette méthode va utiliser SqlDatabaseHandler pour lui dire d'aller chercher tous le contenu d'une certaine table. Cette table on trouve à travers un attribut qu'on a donner à la classe (getTableName()). La classe Game a l'attribut game (#[Table('game')]) qui permet à AbstractModel de savoir que qunad on fait fildAll en partant de Game, il faut chercher sur la table Game. Donc, ca active le SqlDatabaseHandler qui va chercher tout le contenu de la table game. 

4. J'adapte html.
5. Jadapte Game.php
6. Je change $releaseDate.  Ce n'est pas un string, mais un objetDateTime

$game->getReleaseDate()->format('Y-m-d') à index.php

  /**
     * Date de sortie du jeu
     * @var \DateTime
     */
    protected \DateTime $releaseDate;

7. Je crée un fichier edit.php
8. Je charge autoloader.
<?php
require_once __DIR__ . '/vendor/autoload.php';

9. Je fais toutes les verifications.

try {
    // Si la méthode HTTP utilisée dans cette requête n'est pas POST, c'est donc que l'utilisateur a tenté d'accéder à ce script manuellement
    if ($_SERVER['REQUEST_METHOD'] !== 'POST') {
        throw new Exception('This script must be accessed via a POST HTTP request.', 0);
    }

    // S'il manque un seul des champ présents dans le formulaire, c'est donc que l'utilisateur a contourné le formulaire
    if (!isset($_POST['title']) ||
        !isset($_POST['link']) ||
        !isset($_POST['release_date']) ||
        !isset($_POST['developer']) ||
        !isset($_POST['platform'])
    ) {
        throw new Exception('Form field missing in request.', 1);
    }

    // Teste si l'un des champs est vide
    if (empty($_POST['title']) ||
        empty($_POST['link']) ||
        empty($_POST['release_date']) ||
        empty($_POST['developer']) ||
        empty($_POST['platform'])
    ) {
        throw new Exception('Form should not have empty fields.', 2);
    }

  
}
catch (Exception $exception) {
    // Redirige sur la liste des jeux
    header('Location: /?error=' . $exception->getCode());
}

10. Je crée un objet Game dans le fichier edit.php pour pouvoir modifier un game.
 $game = new Game(
        null,
        $_POST['title'],
        $_POST['release_date'],
        $_POST['link'],
        $_POST['developer'],
        $_POST['platform']

    );
    Je peux faire dd:
 dd($game);

Résultat:
^ App\Model\Game {#3 ▼
  #id: null
  #title: "title"
  #releaseDate: DateTime @1628121600 {#2 ▶}
  #link: "link"
  #developerId: 2
  #platformId: 2
}

11. Dans Game.php, je crée une méthode create().
 public function create()
    {
    }

12. La méthode que j'ai eu avant.
   public function create()
    {
        // Configure une connexion au serveur de base de données
        $databaseHandler = new \PDO('mysql:host=localhost;dbname=videogames', 'root', 'root');
        // Crée un modèle de requête "à trous" dans lequel on pourra injecter des variables
        $statement = $databaseHandler->prepare('INSERT INTO `game`
    (`title`, `link`, `release_date`, `developer_id`, `platform_id`)
VALUES (:title, :link, :release_date, :developer_id, :platform_id)');
        // Exécute la requête préparée en remplaçant chaque champ variable par le contenu reçu du champ correspondant dans le formulaire
        $statement->execute([
            ':title' => $_POST['title'],
            ':link' => $_POST['link'],
            ':release_date' => $_POST['release_date'],
            ':developer_id' => $_POST['developer'],
            ':platform_id' => $_POST['platform'],
        ]);
    }

13. Je change execute.
 $statement->execute([
            ':title' => $this->title,
            ':link' => $this->link,
            ':release_date' => $this->release_date,
            ':developer_id' => $this->developerId,
            ':platform_id' => $this->platformId,
        ]);

J'ai vérifié, si le formulaire est valide, a partir de quoi je crée un nouvel objet game avec les propriétés et je dit à l'objet create (à edit.php).
$game->create();

14. Ca marche pas, parce qu'il faut transformer release_date de l'objet en chaine de caractère.
On passe un objet Datetime à release_date, mais je vais envoyer la requette sql dans mon serveur de la base de donées, ce sera enoyé sous forme de chaine de caractère. 

 ':release_date' => $this->release_date->format('Y-m-d H:i:s'),

15. Si je remplis le formulaire, ca me redirige sur une page vide. Je fais la redirection dans edit.php
 header('Location: ./');



