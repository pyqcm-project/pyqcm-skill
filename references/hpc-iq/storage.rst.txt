Stockage
========

Répertoire personnel
--------------------

Votre répertoire personnel est accessible à ``$HOME``. Par exemple :

.. code-block:: console

    [alice@iv11 ~]$ echo $HOME
    /home/alice

C’est le bon emplacement pour vos fichiers de configuration, votre code et les
logiciels que vous installez. Dû à sa capacité et à sa performance limitées, ce
n’est pas le bon emplacement pour vos données de recherche et vous ne devriez
pas l’utiliser pour lire et écrire de telles données lorsque vous :doc:`exécutez
des tâches <jobs>`.

Données de recherche
--------------------

Les données de recherche sont stockées sur un serveur de 190T accessible à
``/net/nfs-iq/data``. Assurez-vous d’utiliser cet emplacement pour lire et
écrire des données lorsque vous :doc:`exécutez des tâches <jobs>` ; il offre
une meilleure performance entrée-sortie (« input/output, IO ») que votre
répertoire personnel.

Pour assurer leur sécurité en cas de défaillance d’un disque, les données de
recherche sont redondantes, c’est-à-dire qu’elles sont stockées en deux
exemplaires. (La capacité physique totale du serveur est de 384T, environ 2 ×
190T.) Les données sont également sauvegardées chaque jour et archivées sur
ruban chaque semaine.

Données individuelles
'''''''''''''''''''''

Chaque utilisateur dispose d’un espace individuel pour ses données de recherche,
accessible à ``/net/nfs-iq/data/$USER``. Par exemple :

.. code-block:: console

    [alice@iv11 ~]$ echo $USER
    alice
    [alice@iv11 ~]$ cd /net/nfs-iq/data/alice

Données des groupes
'''''''''''''''''''

Les groupes de recherche disposent d’un espace partagé accessible à
``/net/nfs-iq/data/def-$PROF``, où ``$PROF`` est le nom d’utilisateur du
directeur ou de la directrice du groupe de recherche. (Cette forme est identique
à celle du stockage projet des grappes de l’Alliance.) Par exemple :

.. code-block:: console

    [alice@iv11 ~]$ echo $USER
    alice
    [alice@iv11 ~]$ cd /net/nfs-iq/data/def-alice

Stockage local sur les nœuds de calcul
--------------------------------------

Quand l’ordonnanceur démarre une tâche, un répertoire temporaire est créé sur
chacun des nœuds alloués à cette tâche. La variable d’environnement
``SLURM_TMPDIR`` donne le chemin d’accès à ce répertoire. Lorsque la tâche se
termine, les fichiers du répertoire temporaire sont supprimés.

Puisque ``$SLURM_TMPDIR`` se trouve sur un disque local, les opérations
d’entrée-sortie sont plus rapides qu’avec le stockage réseau (tel que
``/net/nfs-iq/data``). Si une tâche fait beaucoup d’opérations de lecture et
d’écriture, elle s’exécutera probablement plus rapidement en utilisant
``$SLURM_TMPDIR`` plutôt que le stockage réseau. Vous devez toutefois vous
assurer de copier vos données vers le stockage réseau avant que votre tâche ne
se termine.

.. seealso::
   - `Stockage local sur les nœuds de calcul <https://docs.alliancecan.ca/wiki/Using_node-local_storage/fr>`_
     (documentation technique de l’Alliance)

Interconnexion avec la grappe MP2
---------------------------------

Le stockage pour les données de recherche, ``/net/nfs-iq/data``, est
accessible à partir des nœuds de connexion et de calcul de MP2. Les stockages
``/project`` et ``/scratch`` de MP2 ne sont pas accessibles à partir de la
Grappe IQ.
