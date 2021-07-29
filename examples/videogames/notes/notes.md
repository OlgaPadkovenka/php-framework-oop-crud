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

4. J'adapte html.