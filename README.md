# Jobby

## Introduction
Les métiers ont développé, et continuent à développer, des petits programmes de calcul.
Historiquement, ces programmes étaient lancés en ligne de commande ; l'intégration dans des applications supposaient, soit l'intégration du code, soit l'appel à l'executable depuis une autre application.
Dans tous les cas, cela entraine des contraintes sur l'executable et sur l'application.

L'idée de Jobby est de proposer une petite plateforme simple permettant d'intégrer facilement des executables afin de les présenter comme des services asynchrones.

## Principes

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

### Job
Un run correspond à l'exécution d'un programme. Toutefois, l'intention du run existe avant et après. En effet :
1. il faut créer l'environnement (répertoire dédié et fichiers) AVANT de lancer le programme.
2. les résultats d'un run doivent rester disponibles même après la fin d'exécution

Un *job* est une instance d'un programme. Le job "contient" le run.

## Les composants

Afin de faciliter les échanges entre composants et d'éléver la résilience du système, nous avons opté pour des échanges asynchrones par message plutôt que d'utiliser des connexions TCP persistentes entre composants.

# Bus de messages
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
- REMOVE : un gestionnaire demande la suppression d'un run
- REMOVED : un moniteur indique qu'il a supprimer un run
# Collecteur
Le collecteur assure la transmission des données en flux (stdin, stdout et stderr).
Les données sont conservées par le collecteur jusqu'à ce que tous les clients connectés les aient lues ou que le run soit supprimé.
Un client ne peut pas demander une connection aux flux d'un run qui est dans un état terminal.

# Moniteur
Le moniteur partage l'OS du run.

Le moniteur peut traiter les requêtes asynchrones suivantes :
- CONFIG : transmettre des éléments de configuration (liste des dispatchers, ...)
- INIT : initialiser l'environnement d'un run (répertoire dédié)
- UPLOAD : téléverser des données dans un fichier
- DOWNLOAD : télécharger des données depuis un fichier
- RUN : démarrer un run (associe le pid à l'identifiant du job)
- TERMINATE : terminer (proprement) un run
- KILL : terminer (brutalement) un run
- TSTP : suspendre (proprement) un run
- STOP : suspendre (brutalement) un run
- CONT : reprendre l'exécution d'un run interrompu
- REMOVE : supprimer l'environnement d'un run (uniquement si le run est terminé)





