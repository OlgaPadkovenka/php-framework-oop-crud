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

//CREATE
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

 //Update
 16. Dans le formulaire pour faire un update, j'ai un input avec name="edit". Le formulaire est en get par défaut.
   <!-- start update-->
                                    <form>
                                        <input type="hidden" name="edit" value="<?= $game->getId() ?>" />
                                        <button type="submit" class="btn btn-primary btn-sm">
                                            <i class="fas fa-edit"></i>
                                        </button>
                                    </form>
                                    <!-- end update-->

Cela me permet d'avoir un query quand je click sur le bouton d'update.
videogames/?edit=
J'ajoute un id dans value de game value="<?= $game->getId() ?>" 
Le résultat: Par exemple, videogames/?edit=24

17. Dans mon html, je dis que pour chaque jeu vidéo tu me fabrique une ligne de tableau: 
<?php foreach ($games as $game) : ?>
Ensuite, je dis s'il existe une valeur edit dans les paramètres GET et que ce paramètre $_GET["edit"] contient la même valeur que l'id du jeu, à ce moment-là, tu me fais un formulaire:

18. Si je fais un dd à cet endoit
 <?php foreach ($games as $game) : ?>
                        <?php dd(['GET parameter' => $_GET['edit'], 'Game id' => $game->getId()]); ?>
                        <?php if (isset($_GET['edit']) && $_GET['edit'] === $game->getId()) : ?>

Je peux voir que GET parameter est une chaine de caractère, alors que un id est un nombre.
^ array:2 [▼
  "GET parameter" => "6"
  "Game id" => 1
]

19. Dans la condition j'ajoute la fonction intval()
 <?php if (isset($_GET['edit']) && intval($_GET['edit']) === $game->getId()) : ?>

^ array:2 [▼
  "GET parameter" => 6
  "Game id" => 1
]

20. Je voudrais dire que s'il y a un id, je modifier un jeu, s'il n'y a pas d'id, je veux crée un jeu.
  $id = null;
    if (isset($_POST['id'])) {
        $id = intval($_POST['id']);
    }

Si  $_POST['id'] existe, $id est égale à $_POST['id'] transformé en nombre. 

21. 
Dans ce cas la, je peux changer mon objet new Game dans edot.php
de   $game = new Game(
        null,
        $_POST['title'],
        $_POST['release_date'],
        $_POST['link'],
        $_POST['developer'],
        $_POST['platform']
    );

à

  $game = new Game(
        $id,
        $_POST['title'],
        $_POST['release_date'],
        $_POST['link'],
        $_POST['developer'],
        $_POST['platform']
    );

Je peux le tester en ajoutant un dd($game);
 $game = new Game(
        $id,
        $_POST['title'],
        $_POST['release_date'],
        $_POST['link'],
        $_POST['developer'],
        $_POST['platform']
    );
    dd($game);

  Si je crée un jeu, l'id est null, si je fais un update, l'id existe déjà.

22. Je dis
   // Si aucun ID n'a été envoyé dans les données de formulaire, c'est donc qu'on souhaite créer un nouveau jeu
     if (is_null($id)) {
         // Crée un nouvel enregistrement en base de données à partir des informations contenues dans l'objet
         $game->create();
     // Sinon, c'est qu'on souhaite modifier un jeu déjà existant
     } else {
         // Met à jour un enregistrement existant en base de données à partir des propriétés de cet objet
         $game->update();
     }

23. Je crée une méthode update() dans Game.php

  /**
      * Met à jour un enregistrement existant en base de données à partir des propriétés de cet objet
      *
      * @return void
      */
     public function update()
     {
         // Configure une connexion au serveur de base de données
         $databaseHandler = new \PDO('mysql:host=localhost;dbname=videogames', 'root', 'root');
         // Crée un modèle de requête "à trous" dans lequel on pourra injecter des variables
         $statement = $databaseHandler->prepare('UPDATE `game`
             SET
                 `title` = :title,
                 `link` = :link,
                 `release_date` = :release_date,
                 `developer_id` = :developer_id,
                 `platform_id` = :platform_id
             WHERE `id` = :id
         ');
         // Exécute la requête en remplaçant chaque champ variable par la valeur associée dans le tableau
         $statement->execute([
             ':id' => $this->id,
             ':title' => $this->title,
             ':link' => $this->link,
             ':release_date' => $this->releaseDate->format('Y-m-d H:i:s'),
             ':developer_id' => $this->developerId,
             ':platform_id' => $this->platformId,
         ]);
     }

24. Je crée un fichier delete.php
<?php

 require_once __DIR__ . '/vendor/autoload.php';

 use App\Model\Game;

 try {
     // Si la méthode HTTP utilisée dans cette requête n'est pas POST, c'est donc que l'utilisateur a tenté d'accéder à ce script manuellement
     if ($_SERVER['REQUEST_METHOD'] !== 'POST') {
         throw new Exception('This script must be accessed via a POST HTTP request.', 0);
     }

     // S'il manque un seul des champ présents dans le formulaire, c'est donc que l'utilisateur a contourné le formulaire
     if (!isset($_POST['id'])) {
         throw new Exception('Form field missing in request.', 1);
     }

     // Récupère une copie de l'enregistrement à supprimer sous forme d'objet
     $game = Game::findById($_POST['id']);
     // Supprime un enregistrement existant en base de données correspondant à cet objet
     $game->delete();

     // Redirige sur la liste des jeux
     header('Location: /');
 }
 catch (Exception $exception) {
     // Redirige sur la liste des jeux
     header('Location: /?error=' . $exception->getCode());
 }

 25. Je crée un formulaire avec name="delete". Je crée une méthode delete() dans Game.php et je l'appele dans delete.php.

     /**
      * Supprime un enregistrement existant en base de données correspondant à cet objet
      *
      * @return void
      */
     public function delete()
     {
         // Configure une connexion au serveur de base de données
         $databaseHandler = new \PDO('mysql:host=localhost;dbname=videogames', 'root', 'root');
         // Crée un modèle de requête "à trous" dans lequel on pourra injecter des variables
         $statement = $databaseHandler->prepare('DELETE FROM `game` WHERE `id` = :id');
         // Exécute la requête préparée en remplaçant chaque champ variable par le contenu reçu du champ correspondant dans le formulaire
         $statement->execute([
             ':id' => $this->id
         ]);
     }
