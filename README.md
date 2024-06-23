# Jobby

## Introduction
Les métiers ont développé, et continuent à développer, des petits programmes de calcul.
Historiquement, ces programmes étaient lancés en ligne de commande ; l'intégration dans des applications supposaient, soit l'intégration du code, soit l'appel à l'executable depuis une autre application.
Dans tous les cas, cela entraine des contraintes sur l'executable et sur l'application.

L'idée de Jobby est de proposer une petite plateforme simple permettant d'intégrer facilmement des executables afin de les présenter comme des services asynchrones.

## Principes
Pour Jobby, un executable est un *programme* qui consomme des entrées et produit un résultat. 

Les entrées sont : 
- l'entrée standard (stdin)
- des fichiers

Les sorties sont :
- la sortie standarad (stdout)
- la sortie d'erreur standard (stderr)
- des fichiers

Chaque fois qu'on lance un programme, un créé un *run* qui s'execute dans un répertoire de travail dédié. De fait, un programme ne doit pas avoir de dépendances avec des répertoires partagés ou des montages particuliers. Jobby se charge de fournir à chaque run tout ce dont il a besoin pour s'executer.




