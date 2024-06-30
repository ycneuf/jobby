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

Par ailleurs, le *moniteur* remonte au *gestionnaire* les évènements relatifs à l'exécution des *runs* (démmarage, arrêt, plantage, etc ...).

Le *moniteur* s'enregistre auprès du *gestionnaire* en lui indiquant les *programmes* qu'il est en mesure d'exécuter.

Le *moniteur* émet un signal régulier (battement de coeur ou "hearbeat") permettant de savoir qu'il n'est pas bloqué et toujours disponible pour répondre aux requêtes.
Un *moniteur* qui n'émet plus ce signal régulier est considéré comme perdu est n'est plus sollicité pour exécuter des *programmes*.

### Collection des jobs

Comme son nom l'indique, la *collection des jobs* est une collection d'items. Elle est implémentée dans une base de données (pas nécessairement relationnelle).

### Dépot de médias

Le *dépôt de médias* assure la persistence des résultats produits par les *runs*.

### Gestionnaire

Le *gestionnaire* a en charge la cohérence entre le moniteur, la *collection des jobs* , le *depôt de médias* et le *collecteur*.

Le *gestionnaire* présente l'API REST qui est le point d'entrée de Jobby pour les applications.

### Collecteur
Le *collecteur* (ou *collecteur des flux*) est chargé 
1. de la récupération en temps réel des flux (stdout et stderr) produits par les *runs*
2. de la redirection des flux destinés à l'entrée standard des *runs"

Le *collecteur* expose une interface WebSocket permettant aux applications de consommer ces flux.

## Architecture

L'architecture de Jobby repose sur des serveurs communiquant via des connexions tcp pérennes (pour les flux de données) et un bus de messages (pour les évènements).











