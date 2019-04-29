
Team
Animateur : Costes
Secrétaire : Gily
Scribe : Fantou
Gestionnaire : Nico

## Mots Clés
-	Tablespace :
-	SQL + :
-	PMON :
-	Control Files :
-	Base de données :
-	Fichiers de journalisation :
-	DBA :
-	~Fichiers de configuration~
-	~Instance~
-	~Dictionnaire de données~



## Contexte
Quoi ?
	Installer une base de données
Comment ?
	Utiliser SQL + (PL/SQL)
Pourquoi ?
	Respecter demande du client

## Contraintes

-	2 jours
-	Données confidentielles  Tablespace séparé
-	Fichiers de journalisation archivés
-	Processus PMON démarré automatiquement
-	Chaque BDD ne doit avoir qu'une seule instance
-	Conserver une copie du fichier de config
-	Pouvoir sélectionner l'emplacement des Control Files
-	Accès visuel à la BDD, pas de SQL + pour le client


## Problématique

~Comment mettre en place une BDD avec SQL + ?~
Comment mettre en place un environnement de BDD ? (Oracle 11gr2)


## Généralisation
Gestion de données

## Hypothèses
- 	Une API est une interface de programmation
-	Processus PMON = Journalisation automatique
-	Fichier de journalisation = Fichier de log
-	DBA = Database Administrator
-	SQL + = Ligne de commande en SQL
-	Tablespace est une partie encryptée
-	Tablespace peut être extensible ou non




## Plan d'action

### Etude :
o	Oracle GR2
	- Logique
	- Physique
o	Tablespace
o	SQL +
o	PMON
o	Control Files
o	Journalisation
o	Installation BDD
o	Dictionnaire de données
o	Vues
o	Requêtes
o	Etc (les bases des bdd)

### Réalisation
o	Installation BDD
o	Workshop (by Maaaaz ♫)

## I - Généralités

- Oracle database 11gR2 ⇒ septembre 2010
- 11.2 pour Windows depuis avril 2010
		- Contient APEX (outil de développement rapide) (Oracle Application Expresse)
		- Serveur HTTP intégré dans la BDD avec techno WebDAV ("Embedded PL/SQL Gateway")
		- Performances améliorées (stockage fichiers / fonctionnalités renforcées pour la sécurités...)
		- **Architecture physique** : 
		- Grid Computing : Matrices serveurs/systèmes pour économies de stockage. Le but est de créer des pools de ressources sur les deux. On a donc un partage de ressource sur plusieurs serveurs. Pour faire du grid computing, on retrouve : 
	- **ASM**
		- Automatic Storage Management 
		- Permet à la BDD de gérer directement les disques bruts, elle élimine le besoin pour un gestionnaire de fichiers de gérer à la fois des fichiers de données et de journaux.
		- Réparti automatiquement les data de BDD entre tous les disques, sans coût de gestion.
		- Actualise automatiquement la répartition des données
		- Les instances ASM sont donc responsable des métadas pour rendre les fichiers de données disponibles ailleurs.
	- **RAC** 
		- Regroupe les disques de fébricants différents dans des groupes disponibles. On ne gère ainsi que les groupes de disques.
	- Oracle Ressource Manager : Permet de contrôler l'allocation des ressources des noeuds de la grille 
	- Oracle Scheduler :  Contrôle la distribution des jobs aux noeuds de la grille 
	- Oracle streams : Transfère les données entre les noeud de la grille tout en assurant la synchronisation des copies.  

**Oracle c'est : **
Oracle offre l'accès à un choix d'outils et de processus de dévelppement : 
- Client Side Caching
- Binary XML
- Compilateur Java
- Intégration native avec VS Studio 2005 pour application .NET
- Oracle Application Expression pour outils de migration
- SQL Developer pour coder rapidement les routines SQL et PL/SQL

**Oracle propose plusieurs gammes de produits :**
- Entreprise Edition : Grosses applications critiques, intégrant des options supplémentaires tq partitionnement des tables
- Standard Edition : Serveurs possédant 4 proco et ne proposant que du RAC/ASM
- Standard Edition One : Serveur biproco, sans options
- Personnal Edition : Pour utilitsateur indépendant (dév, consultant...), et utilise un noyau Enteprise Edition

La 11g apporte dans l'Entreprise Edition :
	- Oracle Real Application Testing : Réduire délais / risques / coûts de test de ses modifications de façon contrôlée et économique.
	- Oracle Advanced Compression : Mécanisme de compression applicables à tous les types de données pour compresser 2x à 3x plus.
	- Oracle Total Recall : Conserver / retrouver les historiques des données modifiées
	- Oracle Active Data Guard : Protection des données jusqu'aux risques de défaillances des systèmes et de désastres.

NB : La structure Oracle doit respecter : 
- 30 cractères max
- Doit commencer par une lettre
- peut contenir lettre / chiffres / caractères spéciaux (_$#)
- Pas sensible à la casse
- Ne doit pas être un mot réservé Oracle

## II - Notions importantes 

**Notion de schéma :**

Schéma = Ensemble des objets qui appartiennent à un utilisateur
- Préfixés par le nom de l'utilisateur qui les a créés (ex : EMILIEN.AVION)
- On retrouve principalement comme types d'objets :	
	- Tables et index
	- Directory
	- Vue / séquences / synonymes
	- Programmes PL/SQL (procédures / fonctions / packages / triggers)
	
**Notion de dictionnaire de données :**

Ensemble de table et de vues qui donne des informations sur le contenu d'une BDD
- Strucutres de stockage
- Utilisateur et leurs droits
- Objets (table, vues, index, procédures, fonctions...)

**Notion de vues statiques :** 
Basées sur de vraies tables SYSTEM
- USER_* : Informations sur les objets qui appartiennent à l'utilisateur
- ALL_* : Information sur les objets auxquels l'utilisateur a accès
- DBA_* : Information sur tous les objets de la base

**Notion de vue dynamiques**
⇒ Les informations sont en mémoire ou extraites du fichier de contrôle
⇒ Remise à zéro si on arrête la base de données
Accéssible même si BDD pas complètement ouverte (MOUNT)
``PHP
V$INSTANCE 
V$DATABASE 
V$SGA 
V$DATABASE 
V$PARAMETER
``

**Notion de tablespace**
- Unité de stockage composée d'un ou de plusieurs fichiers physiques
- Fichier données découpés en bloc d'une taille définie à la création de la BDD
- La taille des blocks Oracle correspondent à un multiple du block géré par l'OS

## II - Architecture logique

- Instance : *Ensemble qui va permettre d'exploiter les données en ouvrant une BDD.* 
⇒ Une instance est unique, elle ne peut ouvrir qu'une BDD. 
⇒ MAIS avec RAC (Oracle Real Application Clusters), on peut utiliser Oracle sur des serveurs en cluster, et donc avoir plusieurs instances. Intéressante pour disponibilité mais complexe à mettre en œuvre.

**Une instance se compose de  :** 
	- Zone mémoire partagée (System Global Area - SGA) => Allouées au démarrage BDD
	- Des processus background rôle bien précis ==> Allouées au démarrage BDD
	- Des processus serveur chargés de traiter les requêtes utilisateurs
	- Dimensionnées par un ensemble de paramètre stockés dans un fichier de paramètres système "SPFILE<SID>.ora" généré à partir d'un fichier de paramètres caractères "PFILE<SID>.ora". Le SPFILE contient des variables qui configurent la SGA (Différentes zones mémoires des processus). Géré par le DBA (Database Administrator)

**Y accèder :**
⇒ On accède à l'instance via des processus utilisateurs, qui correspondent à des applications. 
⇒ Le client/serveur communiquent grâce à la couche "Oracle Net".

**Connexion à la BDD**
⇒ Lorsqu'on se connecte, on ouvre une session
⇒ Ps client pris en charge par processus serveur (traiter). On a un processus serveur par client (Dedicated Server Configuration) par défaut. On peut faire éventuellement en multithreaded server (MTS), pour avoir des processus serveur partagés par plusieurs processus utilisateur.
⇒ Lors de la connexion, on à une **PGA** qui est distribuée.

**La PGA** (Program Global Area)
 - Mémoire privée des différents processus distribuée au moment de la connexion d'un client
	 - Zone de tri (allouée dynamiquement lors d'un tri)
	 - Informations session
	 - Information des requêtes de la session
	 - Variables sessions
	 
**SGA -- System Global Area** : Zone de mémoire partagées par les différents processus allouée au démarrage de l'instance, configuré depuis le SPFILE.
La SGA est composé de :
	- SPA : Shared Pool Area : Zone de partage des requêtes et du dico Oracle
	- DBC : Database Buffer Cache : Zone de partage des données de la base (cache) ⇒ Utilise algo LRU (Least Recently Used) pour gérer le cache, en cas de de manque de place il delete les moins récents.
	- Redo Log Buffer : Stocke les infos sur les modifications apportées apportées à la base avant leur écriture dans un fichier Redo Log (activité sessions, journaux)
	- Large Pool : Espace réservé aux opérations parallèles
	- Java Pool : Zone réservées aux programmes Java
	- Streams Pool: Réplication de données entre BDD distante
	- Reserved Area : Zone réservée à l'enregistrement d'objets SQL 
	- Result_Cache : Composant pour initialiser le paramètre "MEMORY_TARGET" ==> Automatiser la gestion de l'instance, 
Chaque processus à également une zone de mémoire privée appellée "PGA" (Program Gbloal Area)

**Les vues permettent de suivre les valeurs des paramètres de l'instance en cours de fonctionnement
Ex : On peut suivre l'allocation dynamique et les valeurs d'allocations : `Select * from v$memory_target_advice ;`

**Les processus Arrière plan** :
- Réalisent des opérations sur l'instance et BDD (écriture/get/résolution d'erreurs/augmenter performance)
- Intéragis avec les nombreuses zones mémoires de la **SGA**
	- Database Writer (DBWRn) : écrit sur disque les données modifiées dans le Database Buffer Cache (DBC, Zone de partage des données de la base)
	- Log Writer (LGWR) : écrit sur le disque le contenu du Redo Log BUffer dans les fichiers Redo
	- Checkpoint (CKPT) : Crée des chkp via les entêtes, en enregistrant sur le disque, donnant la possibilité de lever un "Jalon" pour restaurer les données. 
	- Process Monitor (PMON) : Chargé du nettoyage des processus utilisateur qui plante. Libère les ressources qui se sont mal terminées.
	- System Monitor (SMON) : Restauration de l'instance après un arrêt normal. C'est le gardien de la cohésion des données. Lancée au démarrage.
	- CJQ : Utilisé par le schedular,  génère les ps pour exécuter les job planifiés dans file d'attente interne oracle
	- MMAN : distributeur de mémoire et coordonne la taille allouée aux différents composants
	- MMON : programme et déclanche le ADDM (Automatic Database Diagnostic Monitor) qui effectue des analyses pour déterminer des problèmes potentiels
	- [OPT] : Archiver : archive des fichiers de Redo Log quand celui ci est plein
	- [OPT] : Recover : Gère les BDD distribuées
	- [OPT] : Dispatcher : Présent en serveur partagé
	- [OPT] : Global Cache service 
	- [OPT] : Job queue : Rafraîchir les snapshots ou exécuter périodiquement des tâches programmée

- Qu'est ce qu'une BDD Oracle  :
Ensemble de fichiers qui permettent de gérer les données stockées dans la BDD.
	- Fichier de **contrôle** "*master*" qui contient toutes les infos sur les autres fichiers de la base (nom, emplacement, taille)
	- Fichier "**Redo Log**" 
		- Activité des sessions connectées à la base (Journaux de transaction organisé en groupe, possédant le même 	nombre de membres)
		- Éventuellement archives Redo Log
	- Un ou plusieurs fichiers de données qui contiennent les **données des tables de la base** , avec en plus au mini : 
		- Tablespace SYSTEM : dictionnaire de données
		- Tablespace SYSAUX : Auxiliaire de SYSTEM, contenant les fonctions Oracle ou des outils divers
		- Tablespace TEMP : segments temporaires utilisés par les requêtes SQL de la BDD
		- Tablespace UNDO : Récupère la version précédente des données en cours de modification par les transaction.
		- Tablespace USERS : Tablespace de travail
		- Le fichier de paramètre binaire SPFILE<SID>.ora
		- Un fichier password pour les privilèges du SYSDBA (Système DataBase Administrator)
	⇒ Les fichiers de données sont logiquement regoroupés en tablespaces



## IV - Gestion du stockage

⇒ Un tablespace est une untié logique de stockage composée d'un ou plusieurs fichier physiques. On travaille sur le tablespace et non sur le fichier de données. Sépare les différents objets de la BDD et assure de meilleures performances. Permet aussi une meilleure souplesse dans les tâches d'administration
⇒ Les objets dans des tablespace (Regroupement de tables) :
		⇒ Les **objets** (espace apppelé segment en Ko) appartient à un tablespace et est constitué d'**extends**
		⇒ Un extends est un semble de blocs voisins dans un fichier de donnée
		⇒ On les découpe pour éviter d'avoir des blocs de 5go, un autre de 500Mo... Au lieu de ça, ils font tous la même taille qu'on définit avec un paramètre "DB_BLOCK_SIZE", ce qui va allouer la taille d'un fichier, la capacité maximum que peut contenir d'objets. On attends ainsi qu'il soit plein avant d'en créer un autre. 
		⇒ On a ainsi des morceaux un peu partout "extends". On peut configurer la taille de chaque extends :
``PHP
Storage (INITIAL valeur
NEXT valeur
MINEXTENTS valeur
MAXEXTENTS valeur
PCTINCREASE valeur
)``
- Initial : Taille du premier extend crée à la création de l'objet
- NEXT : taille des extents suivants
- MINEXTENTS : Nombre min d'extentes crées à la création de l'objet
- MAXEXTENTS : Nombre max d'extends crées pendant la vie de l'objet
- PCTINCREASE : Pourcentage d'augmentation de la taille d'un extent par rapport au précédent.

- Il existe 4 types de segments :
	- Segment de table = espace occupé par les tables
	- Segment d''index = espace occupée par les index
	- Segment d'annulation = espace tempo pour stocker les infos d'annulation (ROLLBACK)
	- Segment temporaires = Tri lors de requêtes 
Chaque type de segment est stocké dans un tablespace qui lui est propre !!!

**nb** : à partir de la 9i, les tablespace sont gérés localement et plus par le dico de données, pour optimiser l'utilisation de l'espace, et éviter les phénomène de fragmentation

Dans les en-têtes de fichiers de données, on a une bit-map. C'est une enête.
Chaque bit correspond à un extent, correspondant à 1 si il est libre ou alloué.

**Gestion automatique des extents par Oracle**
- 3 bloc pour l'entête de fichier + taille d'un extent
-  Taille extent = 
	- Valeur explicite ou par défaut de la cluase SIZE pour un tablespace UNIFORM
	- 64 ko, 1M, 8M pour un tablespace AUTOALLOCATE
		- 64 ko SI segment < 1Mo
		- 1 Mo SI segment < 64 Mo
		- 8 Mo SI segment < 1024 Mo

## Tablespace SYSTEM & SYSAUX  ⇒ CF Dernier PDF


- La tablespace SYSTEM est la tablespace qui contient le dictionnaire des données.
- Le dictionnaire de données est instalé à la création de la BDD dans le tablespace SYSTEM par l'utilisateur SYS
- A partir de la 10g, les occupants sont situés dans SYSAUX.

SYSAUX :
- Obligatoire crée au moment de la BDD ou de son upgrade
- Permanante / Read Write / Extent Management Local / Segment Space Management Auto
- Fournit un emplacement centralisé pour toutes les méta-datas, auxiliares de lal BDD qui ne résident pas dans la tablespace SYSTEM. 
On peut crée nos propres tables à l'intérieur de SYSAUX mais c'est VIVEMENT DECONSEILLE.

Avanttages **SYSAUX** :
- Réduction du nombre de tablespace : table par défaut pour tous les produits oracles, centralise le tout
- Réduction de la charge sur le tablespace SYSTEM, ne pas impacter les performances
- Gestion simplifiée du RAC : On regroupe les tablespace des différents disques en une seule.

NB : 
- Modifier sysaux : ALTER TABLESPACE 
- Nécessité d'être SYSDBA pour la modifier

**Délocaliser les occupants du tablespace SYSAUX**

On peut aussi bouger certain composant de la tablespace SYSAUX si trop de place:
``Select occupant_name, schema_name, move_procedure
From v$sysaux occupants ;
Exec WKSYS.MOVE_WK(‘CWMLITE’);
Exec WKSYS.MOVE_WK(‘SYSAUX’);``


