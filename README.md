# Travaux pratiques

## Cloner ce repository

Utiliser votre client git préféré pour cloner ce repository.
Vous écrirez votre code source dans ce dossier.

Chaque partie dans la suite s'accompagnera d'un commit (et pourquoi pas de commits intermédiaires)

## Ecrire avec Postman sur Slack

Slack est un outil de communication où on peut créer des workspaces.
Un workspace comporte un certain nombre de membres.
Chaque membre est invité via une adresse mail.

Ce qui a fait le succès de Slack c'est que :

- chaque workspace a une isolation totale avec les autres workspaces : on ne partage pas son Slack comme on partage son Skype. On a généralement une seule adresse Skype (comme un numéro de téléphone) là où on appartient à X workspaces Slack (celui que vous avez créé avec 2 amis autour d'un projet perso, celui avec votre boulot, celui avec un projet auquel vous participez pour un de vos clients de votre activité freelance...)
- des bots peuvent écrire dans Slack. On peut ajouter programmatiquement des messages dans Slack. Et donc un certain nombre d'intégrations avec des applications tierces ont été rendues possibles. Ainsi, on peut configurer Github pour qu'à chaque push sur un repository, il écrire sur le channel #github de son Slack qu'un nouveau commit est arrivé, son contenu, la personne qui l'a écrit (...)

Slack propose un système de Webhook.
Ce qu'on appelle un Webhook est simplement une url appartenant à Slack de la forme : `slack.com/xxxxxxxxxxx/yyyyyy` sur laquelle on peut envoyer des requêtes HTTP POST.

Ces requêtes HTTP POST doivent contenir un raw body.
Ce raw body doit être une chaîne de caractères représentant un json valide.
Ce json doit respecter un format que Slack impose.

1. Créer un channel Slack public correspondant à la concaténation de son prenom et de son nom
2. Se renseigner pour faire en sorte de créer ce fameux Webhook
3. Utiliser Postman pour écrire dans Slack le message de votre choix

## Créer un script Python3 permettant d'écrire dans Slack

Postman permet d'exporter des requêtes HTTP créées en son sein dans différents langages et avec différentes librairies.
L'export proposé permet surtout d'avoir une idée de l'allure du code source à appeler mais on peut se séparer de bon nombre de headers pas nécessairement utiles.

Python3 est un langage de programmation à la syntaxe assez lisible.

pip est un outil permettant d'installer des packages Python développés par des personnes généreuses et mis à disposition de la communauté pour faciliter la vie de cette dernière sur des problématiques courantes du type `Envoyer une requête HTTP de façon plus confortable que ce que permet nativement le langage de façon un peu trop verbeuse`

On peut lancer un container et partager le volume de l'hôte avec ce dernier. Ainsi, pour lancer un container Python avec montage du volume courant sur le volume /data du container, on peut lancer cette commande :

```
docker run -it -v $(pwd):/data python bash
```

Bien vérifier que le dossier dans lequel vous travaillez est bien vu par docker comme pouvant être montable (à configurer dans docker).
Sur windows, la syntaxe `$(pwd)` qui ne fait rien d'autre qu'avoir le path courant peut ne pas fonctionner en fonction de ce que vous utilisez (Powershell, cmd, git bash...). L'adapter en fonction.

Ce container ainsi lancé nous permet de :

- modifier un script sur notre machine hôte
- exécuter ce script dans le contexte du container en faisant par exemple `python myScript.py`
- voir directement l'output rendu

Avec la commande qui suit, on peut demander au container docker de faire en sorte que le dossier sur lequel on arrive au lancement du container soit /data. Ce qui évite d'avoir à naviguer dans le container avec des `cd`. Ca correspond à donner un dossier à l'option `-w`.

```
docker run -it -v $(pwd):/data -w /data python bash
```

1. Chercher une image docker contenant Python3. On utilisera Python3 dans un container lancé à partir de cette image plutôt que d'installer Python3 sur son ordinateur via Anaconda ou tout autre méthode... Mais si vous n'avez pas pu installer docker, il faudra trouver une solution.
2. Exporter la requête Postman de la partie précédente en Python3. On pourra utiliser la librairie requests
3. Créer un script Python3 slackSender.py qui a pour seule mission d'envoyer un message écrit en dur à un webhook slack écrit en dur dans le code. Ce script s'arrête ensuite

4) faire en sorte que ce script puisse être appelé ainsi :

```bash
slackSender.py "slack.com/xxxxxxxxxxx/yyyyyy" "Here is my message"
```

et donc que le webhook et le message soient donnés en arguments.

## Créer un serveur web en Python3 permettant de traiter des requêtes HTTP

Flask est une librairie open-source permettant de créer en Python un serveur web. Un serveur web sert à être capable de réceptionner des requêtes HTTP et d'y repondre. Un serveur web doit nécessaire écouter, ce n'est pas un script qu'on lance et qui s'arrête. Il écoute en permance sur le port demandé. Il est en attente de requêtes.

1. Voir pour installer flask avec pip
2. Lancer le plus simple des serveurs web qui quand on visite une route en GET, par exemple /hello affiche juste le célèbre `Hello World`. Le script qui lancera ledit serveur s'appellera `httpSenderService.py`. Ce script ne sera pas lancé avec un `python httpSenderService.py` maus avec un `flask run` auquel on pourra passer des options et arguments si nécessaire. La documentation de Flask vous indiquera comme il faut à ce sujet.
3. Pour tester votre serveur web, on pourra utiliser un navigateur pour ce cas précis (GET), mais on préfèrera bien évidemment toujours... Postman. Si vous voulez pouvoir consulter le port d'un container depuis votre machine hôte, penser bien à faire le nécessaire lors du lancement de votre container. Il y a peut-être un mappage de ports à faire (...). Attention, une application écoutant sur localhost (127.0.0.1) ne peut pas être jointe depuis autre part que localhost. Elle est "bindée" sur localhost. On peut généralement faire en sorte que notre application puisse être consultée depuis autre chose que localhost avec une simple config (on parle de bind 0.0.0.0). Une requête provenant de l'hôte est considérée comme extérieure au container.
4. Faire en sorte que vous puissiez accepter des requêtes POST et soyez capables de transmettre des paramètres dans le body

On peut envoyer de plusieurs façons des paramètres textuels en POST :

- x-www-form-urlencoded
- form-data
- raw

On peut aussi envoyer des binaires en POST (fichiers...), mais ce n'est pas le sujet du jour.

Ces trois façons de transmettre des paramètres sont simplement plusieurs façons de séparer les paramètres entre eux. On doit préciser dans notre requête HTTP, dans les headers, comment on a envoyé nos paramètres, sinon le serveur ne pourrait pas (tout le temps) le deviner.

Le plus couramment, on utilise maintenant `raw` c'est à dire véritablement une chaîne de caractères brute. On confie à l'application le soin de `parser` le body. Généralement dans le `raw` on envoie une chaîne de caractères représentant un json (cf Slack et son webhook plus haut qui faisaient exactement ceci).

5. Faire en sorte d'être capable d'envoyer un raw body à notre serveur web en HTTP POST. Ce raw body représentera un objet JSON valide et il faudra qu'on soit capable, côté Python, d'obtenir cet objet sous la forme d'un dictionnaire qu'on pourra manipuler
6. En reprenant des morceaux du script `slackSender.py` réalisé plus haut et reprendre notre mécanique capable d'écrire sur Slack. On fera en sorte d'avoir une route HTTP POST /slackSend à laquelle on enverra l'url du webhook + le contenu du message. Bien évidemment, le client qui envoie la requête HTTP voudrait que son message parte. Le service fera en sorte que ce soit le cas et indiquera en retour si tout s'est bien passé ou non.
7. Si pas déjà fait, faire en sorte que notre service soit disponible sur le port 80 du container. Sur notre hôte, ça importe peu mais c'est tant mieux si c'est aussi sur le port 80 qu'on peut consulter le service.

## Création d'une image docker avec notre service

Notre service ne fait rien d'autre qu'être un passe-plat pour le moment mais on est quand même fiers de nous parce qu'on sait :

- utiliser ("consommer") un webservice proposé par Slack
- créer un webservice nous-mêmes

On voudrait avoir une image docker s'appelant `httpsenderservice`. Elle sera publiée sur vos comptes docker respectifs avec le même nom (mais pas le même owner, donc pas de problèmes).

Cette image devra :

- embarquer flask
- embarquer requests
- embarquer le script python nécessaire
- faire en sorte que lorsqu'un container est lancée à partir d'elle, la tâche soit de lancer le serveur web comme il faut, sur le port 80

La façon dont on voudra se servir de l'image :

```
docker run -it -p80:80 {{yourDockerUsername}}/httpsenderservice
```

On veut que sur le port 80 de notre hôte, on puisse communiquer avec le service.
