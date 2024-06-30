# Jobby

## Introduction
Les métiers ont développé, et continuent à développer, des petits programmes de calcul.
Historiquement, ces programmes étaient lancés en ligne de commande ; l'intégration dans des applications supposaient, soit l'intégration du code, soit l'appel à l'executable depuis une autre application.
Dans tous les cas, cela entraine des contraintes sur l'executable et sur l'application.

L'idée de Jobby est de proposer une petite plateforme simple permettant d'intégrer facilement des executables afin de les présenter comme des services asynchrones.

## Les concepts

### Exécutable
Un *exécutable*, pour Jobby, est un code, compilé ou interpété, pouvant être exécuté en ligne de commande et impliquant un certain nombre de contraintes locales (OS, librairies, variables d'environnement, autre exécutable, ...) mais aucune contrainte extra-locales (base de données, montage, application tiers, etc ...). Autrement dit, un *exécutable* peut être lancé avec succès sur un ordinateur adapté coupé du réseau.

Un *exécutable* travaille dans le répertoire courant. Il y trouve toutes les données dont il a besoin.

Un *exécutable* peut produire un résultat uniqument sur la sortie standard (stdout) et dans des fichiers du répertoire courant.

Un *exécutable* peut produire une trace (journal) sur la sortie standard d'erreur (stdout) et dans des fichiers du répertoire courant.

Un *exécutable* peut utiliser l'entrée standard (stdin) pour lire des données en entrées.

### Programme

Un *programme* est un *exécutable* associé à un ensemble de règles définissant ses conditions d'emploi :
- arguments acceptés en ligne de commande
- fichiers devant être présents dans le répertoire courant et ses sous-répertoires au lancement de l'exécution
- fichiers résultats pouvant être récupérés dans le répertoire courant à l'issue de l'exécution, en fonction de l'état final du *run* (réussite, échec, interruption, etc ...)
- ce qu'on fait de la sortie standard (flux exploitable en temps réel, redirigé vers un fichier du répertoire courant, ignorée)
- ce qu'on fait de la sortie d'erreur standard (flux exploitable en temps réél, redirigé vers un fichier du répertoire courant, ignorée)
- ce que l'*exécutable* fait de l'entrée standard (lecture du flux, ignorée)

### Run 
Un *run* est l'exécution contrôlée d'un *programme*.

### Job
Un *job* est la demande d'exécution d'un *programme*.
Un *job* est créé afin d'allouer les ressources nécessaires à la création d'un *run* (essentiellement le répertoire courant) et de permettre le téléversement des fichiers attendus dans le répertoire courant.
Un *job* existe même lorsque le *run* est terminé ; notamment, le *job* a en charge la récupération des résultats dans le répertoire courant du *run*

Un *job* est un ressource et est présentée aux applications à travers une interface REST.

### Moniteur

Le *moniteur* est un démon sur le serveur de calcul sur lequel s'exécute un *programme*. Le *moniteur* assure le respect des règles d'emploi définies par le *programme*, lance l'exécution et gère la redirection des flux standards (stdin, stdout et stderr). 

Le *moniteur* gère les *run* et les présente aux *jobs*.
Le *moniteur* permet à un *job* de :
- téleverser les données d'entrée dans le répertoire courant
- lancer l'exécution avec des arguments
- suspendre et reprendre l'exécution
- interrompre l'exécution (et de forcer l'interruption)
- télécharger les données produits dans le répertoire courant
- libérer les ressources allouées à un *run*

### Gestionnaire
Le *gestionnaire* a en charge les *jobs*.

Le *gestionnaire* 



### Programme
Pour Jobby, un exécutable est un *programme* qui consomme des entrées et produit un résultat. 

Les entrées sont : 
- l'entrée standard (stdin)
- des fichiers

Les sorties sont :
- la sortie standarad (stdout)
- la sortie d'erreur standard (stderr)
- des fichiers

### Run
Chaque fois qu'on lance un programme, un créé un *run* qui s'execute dans un répertoire de travail dédié. De fait, un programme ne doit pas avoir de dépendances avec des répertoires partagés ou des montages particuliers. Jobby se charge de fournir à chaque run tout ce dont il a besoin pour s'exécuter.

La possibilité, pour un Run, d'accéder dynamiquement à des données sans qu'elles soient téléchargées localement au préalable, reste un besoin à étuder. Différentes solutions sont envisageables, mais elles impliquent une contrainte sur le codage.

### Job
Un run correspond à l'exécution d'un programme. Toutefois, l'intention du run existe avant et après. En effet :
1. il faut créer l'environnement (répertoire dédié et fichiers) AVANT de lancer le programme.
2. les résultats d'un run doivent rester disponibles même après la fin d'exécution

Un *job* est une instance d'un programme. Le job "contient" le run.

## Les composants

Afin de faciliter les échanges entre composants et d'éléver la résilience du système, nous avons opté pour des échanges asynchrones par message plutôt que d'utiliser des connexions TCP persistentes entre composants.

### Bus de messages

Le bus de messages a pour objet de faire transiter les messages asynchrones.

Les échanges asynchrones :
- CREATED : un moniteur indique qu'il prend en charge l'exécution d'un job
- STOPED : un moniteur indique l'état terminal d'un run (COMPLETE, FAILED, SUSPENDED, RESUMED, INTERRUPTED, KILLED)
- HEARTBEAT : battement de coeur d'un moniteur
- CREATE : un gestionnaire indique un nouveau run à créer
- TERMINATE : un gestionnaire demande à ce qu'un run soit terminer proprement
- KILL : un gestionnaire demande à ce qu'un run soit tué
- SUSPEND : un gestionnaire demande à ce qu'un run soit suspendu (proprement)
- STOP : un gestionnaire demande à ce qu'un run soit suspendu immédiatement (méthode brutale)
- RESUME : un gestionnaire demande à ce qu'un run suspendu reprenne son exécution
- REMOVE : un gestionnaire demande la suppression d'un job

### Collecteur
Le collecteur assure la transmission des données en flux (stdin, stdout et stderr) en temps (quasi) réel.
Les données sont conservées par le collecteur jusqu'à ce que tous les clients connectés les aient lues ou que le run soit supprimé.
Un client ne peut pas demander une connection aux flux d'un run qui est dans un état terminal.

Le collecteur consomme les messages suivants sur le bus :
- HEARBEAT : le collecteur retransmet les battements de coeur
- REMOVE : suppression d'un run

### Moniteur
Le moniteur partage l'OS du run.

Le moniteur peut traiter les requêtes asynchrones suivantes en les récupérant sur le bus de messages:
- CREATE : si le moniteur dispose du programme demandé par le job, il peut le prendre en charge
- TERMINATE, KILL, SUSPEND, STOP, RESUME : si le moniteur gère le run indiqué, il est responsable de la prise en compte des demandes d'un gestionnaire concernant ce run
- REMOVE : si le moniteur gère le run, il est chargé de sa suppression

Le moniteur émet les messages suivants :
- CREATED : indique la prise en compte d'un run par le moniteur
- STOPED : indique l'état terminal d'un run (COMPLETE, FAILED, SUSPENDED, RESUMED, INTERRUPTED, KILLED)







