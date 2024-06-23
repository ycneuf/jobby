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







