# 1.Connessione al Database

La prima cosa che accade in un'applicazione è la connessione al database. Questa connessione viene creata una sola volta nel file principale, come ad esempio `index.php`, e viene poi passata al Data Access Object (DAO) per eseguire le varie operazioni sul database.

### Creazione della Connessione

Utilizziamo **PDO** (PHP Data Objects) per stabilire una connessione sicura e flessibile al database.

```php
<?php
// Creazione della connessione PDO
try {
    $db = new PDO('mysql:host=localhost;dbname=your_database', 'username', 'password');
    $db->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
} catch (PDOException $e) {
    echo "Errore nella connessione al database: " . $e->getMessage();
    die();
}
?>
```

# 2.DAO (Data Access Object)
Il DAO si occupa dell'interazione con il database. Il controller passa il modello al DAO, che esegue operazioni di CRUD (Create, Read, Update, Delete) utilizzando la connessione al database.
1. Il Controller istanzia il DAO:
    - Il controller crea un'istanza del DAO, passando la connessione al database (creata in precedenza) al costruttore del DAO. <br>
    ```php
    $dao = new YourModelDAO($db);
    ```

2. Il DAO usa il database
    - Il DAO esegue query SQL utilizzando la connessione PDO.
    - Ad esempio, quando il controller chiama getAll(), il DAO esegue la query SQL per recuperare tutti i record dal database.
```php
    public function getAll() {
    $query = $this->db->query("SELECT * FROM your_table");
    $rows = $query->fetchAll(PDO::FETCH_ASSOC);
    
    // Converte i dati in oggetti Model
    $models = [];
    foreach ($rows as $row) {
        $models[] = new YourModel($row['id'], $row['field1'], $row['field2']);
    }
    return $models;
}
```

# 3.Model
Il modello è una rappresentazione dei dati in memoria. Il DAO crea oggetti Model basati sui dati che recupera dal database. Ad esempio, ogni riga del database viene convertita in un oggetto YourModel.

1. DAO crea modelli dal database
    - Il DAO trasforma i dati recuperati dal database in istanze del modello (ad es. YourModel).
```php
$models[] = new YourModel($row['id'], $row['field1'], $row['field2']);
```

# 4.Controller
Il Controller è responsabile di orchestrare la logica. Riceve input dall'utente (via richiesta HTTP), utilizza il DAO per interagire con il database e infine passa i dati recuperati alle View.

1. Controller chiama il DAO
    - Ad esempio, quando l'utente visita l'URL index.php?action=index, il controller chiama il metodo index(), che a sua volta utilizza il DAO per ottenere tutti i dati:
```php
public function index() {
    $models = $this->dao->getAll();
    require 'views/yourmodel/index.php'; // Mostra la vista con i dati
}
```

# 5.View
La View si occupa della presentazione dei dati all'utente. Riceve i dati dal controller sotto forma di modelli e li visualizza.

1. Controller passa i dati alla View
    - Dopo aver ottenuto i modelli dal DAO, il controller include la view corrispondente e passa i dati (in questo caso, un array di modelli):
```php
require 'views/yourmodel/index.php'; // Mostra la vista con i modelli
```
2. La View mostra i dati all'utente
    - La View (ad esempio, index.php) accede ai dati e li mostra in formato HTML. In questo esempio, la view cicla tra i modelli e li stampa in una tabella:
```php
<?php foreach ($models as $model): ?>
    <tr>
        <td><?= $model->getId() ?></td>
        <td><?= $model->getField1() ?></td>
        <td><?= $model->getField2() ?></td>
        <td>
            <a href="index.php?action=show&id=<?= $model->getId() ?>">View</a>
            <a href="index.php?action=edit&id=<?= $model->getId() ?>">Edit</a>
            <a href="index.php?action=delete&id=<?= $model->getId() ?>">Delete</a>
        </td>
    </tr>
<?php endforeach; ?>
```

# 6. Catena di Interazione Completa
Utente interagisce con l'applicazione (ad esempio cliccando su un link).
Router (nel file index.php) decide quale azione chiamare in base all'URL (action=index, action=create, ecc.).
Controller riceve l'azione richiesta, chiama i metodi appropriati nel DAO.
DAO esegue query SQL, converte i risultati in oggetti Model.
Controller passa i modelli alla View.
View mostra i dati all'utente.
