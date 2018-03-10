
Rapport de Spécification PSESI
================================================================================


Contexte
--------------------------------------------------------------------------------

### Situation du projet

Le but de ce projet est de concevoir un véhicule autonome miniature qui effectue
la cartographie du terrain dans lequel il se trouve.

Ce projet est destiné aux étudiants de Licence d'Informatique, le but est de
concevoir entièrement le prototype puis de retirer des parties du programme de
cartographie pour que les étudiants les reconstruisent.
Cet exercice a pour but de les sensibiliser aux enjeux de la programmation
embarquée, notamment l'élaboration d'un algorithme optimisé pour répondre à
des contraintes de temps-réel en puissance de calcul et espace mémoire limitées.

### Domaine de compétences requises

 * Programmation de microcontrolleur en C

Une grande partie du projet consistera en l’implémentation d’un algorithme de cartographie dans un système commandé par microcontrolleur, il sera donc nécéssaire de maitriser le langage C.

 * Programmation multitache

L’application logicielle contrôlant le véhicule serra constituée de plusieurs processus s’exécutant en parallèle, il est donc nécessaire d’être sensibilisé aux notions de programmation multitache.

 * Conception de circuits analogique élémentaires

Au delà de la partie logicielle, le projet inclut la réalisation d’un système muni de capteurs et d’actionneurs liés à un microcontrolleur central, il y a donc besoin d’interfacer le microcontrolleurs avec ces éléments via des circuits analogiques, d’où le besoin de compétences élementaires en électronique.


Objectif
--------------------------------------------------------------------------------

### Quelles sont les données initiales ?

Nous commençons ce projet en disposant des outils suivants :
Un châssis déjà construit
Deux roues motrices et deux MCC pour les actionner
Une roue avant directrice et un servomoteur pour l’orienter
Un capteur à ultrasons et un servomoteur pour l’orienter
Une carte Arduino basé sur le microcontrolleur ….

Au départ, le véhicule ne possède aucune information sur le terrain.

### Quels sont les résultats attendus ?

De manière générale, l’objectif final est d’obtenir un prototype ayant le comportement suivant :
Le véhicule explore le terrain et à la fin de son parcours, il obtient une représentation en mémoire de la carte du terrain qu’il a exploré, indiquant pour chaque zone élementaire s’il elle contient un obstacle ou si elle est libre.
La taille de la zone élémentaire est à définir plus précisément après une étude des limites du système en terme de précision de mesure, mais l’objectif serait d’atteindre une précision de l’ordre du cm.

Dans notre cas, l’objectif sera le suivant :
 1. Réalisation d’une étude de faisabilité déterminant s’il est possible d’aboutir à une cartographie du terrain avec pour seul objet de mesure un capteur à ultrasons.
 2. Dans le cas où le résultat de l’étude est positif, la réalisation d’un prototype fonctionnel.


Problématique
--------------------------------------------------------------------------------

Contrairement à l’autres systèmes recevant des informations par GPS, par un accéléromètre ou autres données absolues, ici notre système ne possède qu’un capteur à ultrasons et ne recevra donc qu’une distance relative à la position du véhicule.
Par conséquent, le véhicule aura besoin de se localiser pour construire la carte à partir des informations du capteur, sauf que le véhicule se localise par rapport à la carte donc il a besoin de connaître la carte pour se localiser. Ce problème “d’oeuf et de poule” est le problème de cartographie et localisation simulanées (SLAM Simultaneous Localization and Mapping) et ce sera la problématique principale de ce projet.





Identification des taches du projet
--------------------------------------------------------------------------------

### Tache : Commande des roues

#### Objectif : 

Pouvoir controler le déplacement du véhicule de manière logicielle.
Plus précisément, il s’agit d’obtenir une ou plusieurs fonctions logicielles pouvant être appelée par un programme utilisateur et permettant de controller la vitesse des roues pour faire avancer et tourner le véhicule.

#### Travail à réaliser :

- [ ] Concevoir l’interface électronique entre les MCC et la carte Arduino
  - [ ] Se documenter sur les différentes méthodes couramment utilisées pour interfacer une unité de commande faible-puissance (ici un microcontrolleur) et un moteur à courant continu.
  - [ ] Comparer ces différentes méthodes et sélectionner la plus appropriée
- [ ] Concevoir le driver de controle des MCC
  - [ ] Déterminer l’interface utilisateur (comment la fonction de commande des roues doit être appelée et avec quels arguments, par exemple est-ce que l’utilisateur fournit un vecteur vitesse que la fonction devra constamment suivre ou bien est-ce que l’utilisateur appelle la fonction pour aller à une vitesse v pendant n secondes)
  - [ ] Déterminer les signaux de commande (en sortie du microcontrolleur) à envoyer pour commander les roues à une vitesse v.
  - [ ] Implémenter la fonction de commande des roues, envoyant les signaux appropriés à l’interface électronique des MCC en fonction des paramètres utilisateurs fournis.

#### Validation :

- [ ] Commander les roues en translation avec une vitesse donnée et comparer la vitesse réelle du véhicule (mesurée à la règle et au chronomètre) avec la vitesse donnée en commande.
  - [ ] Test avec v = 0, le véhicule doit rester à l’arrêt
  - [ ] Test avec v = 0.5 m/s, chronométrer la durée nécéssaire au véhicule pour parcourir 1m, 2m, 5m, 10m.
  - [ ] Effectuer le même test que 1.2 avec v = 1m/s
  - [ ] Envoyer une suite de commande : 0.5 m/s pendant 1s puis 1m/s pendant 1s puis 0m/s, puis filmer le véhicule et mesurer sur la vidéo si sa vitesse au cours du temps correspond bien à la suite de commande effectuée.
- [ ] Commander les roues en rotations, avec une vitesse de rotation donnée et comparer la vitesse réelle avec la vitesse désirée.
  - [ ] Test avec v = 0 rad/s, le véhicule ne doit pas tourner.
  - [ ] Test avec v = pi/2 rad/s, le véhicule doit tourner à une allure constante.
  - [ ] Test avec plusieurs valeurs successives, v = pi/4, v = pi/2, v = -pi/2, v = - pi/4 rad/s


### Tache : Gestion du capteur à ultrasons

#### Objectif

Obtenir un ensemble de fonctions permettant d'orienter le capteur et de
récupérer l'information captée.

#### Travail à réaliser

- [ ] Concevoir l'interface électronique entre le capteur et le microcontrolleur
  - [ ] Se renseigner, notamment dans la documentation du capteur fourni, sur
l'interfacage électronique à mettre en place pour alimenter le capteur et récupérer les données captées.
  - [ ] Réaliser cette interface
- [ ] Ecrire la couche logicielle permettant d'accéder aux données captées
  - [ ] Déterminer la donnée lue du capteur en fonction de la distance à laquelle l'obstacle se situe (pour cela, placerons un obstacle à différentes distances du capteur et noterons les différentes valeurs lues)
  - [ ] Déterminer la portée du capteur, la distance minimale et maximale au delà desquelles le capteur ne renvoie plus de donnée pertinente.
  - [ ] Ecrire une fonction donnant la distance mesurée par le capteur.