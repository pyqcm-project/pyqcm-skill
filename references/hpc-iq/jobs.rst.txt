Lancer des tâches
=================

.. note::

   Cette page couvre les spécificités de la Grappe IQ. Si vous n’êtes pas
   familier avec les commandes pour lancer et gérer des tâches (telles que
   ``sbatch``, ``salloc``, ``squeue``), lisez d’abord `Exécuter des tâches
   <https://docs.alliancecan.ca/wiki/Running_jobs/fr>`_ dans la documentation de
   l’Alliance.

Nœuds de connexion
------------------

Utilisez le nœud de connexion (``iv11``) pour préparer vos tâches. Il est
toutefois interdit d’exécuter des tâches directement sur ce nœud ! Les nœuds de
connexion des grappes ne disposent pas de la puissance de calcul nécessaire pour
exécuter des tâches. De plus, exécuter une tâche sur un nœud de connexion peut
le ralentir considérablement, ce qui nuit à tous les chercheurs connectés.
Toutes les tâches doivent être soumises à l’ordonnanceur en utilisant les
commandes appropriées : ``sbatch``, ``salloc``, ``srun``.

Lecture-écriture
----------------

Utilisez le stockage dans ``/net/nfs-iq/data`` pour lire et écrire des données
de recherche dans vos tâches. Cet emplacement offre une bien meilleure
performance que votre répertoire personnel.

Nœuds publics
-------------

Les tâches doivent être soumises à la partition ``iq-main``, ce qui est l’option
par défaut. La partition peut néanmoins être indiquée explicitement si désiré
avec l’option ``--partition`` ou sa forme courte ``-p``. Par exemple, dans un
script de tâche :

.. code-block:: bash

    #!/bin/bash
    #SBATCH --job-name=my-job
    #SBATCH --partition=iq-main

    ...

La Grappe IQ offre trois nœuds de calcul avec 96 cœurs CPU et 500G de mémoire
chacun. La durée maximale des tâches est de sept jours. Référez-vous au tableau
des :ref:`nœuds de calcul publics <public-label>` pour plus de détails.

Tâches GPU
''''''''''

La Grappe IQ offre aussi un nœud de calcul avec deux GPU Nvidia A40, qui peuvent
être demandés à l’aide des options ``--gpus-per-node``, ``--gpus-per-task`` et
``--gres``. Ce même nœud a 48 cœurs CPU et 500G de mémoire.

Par exemple, pour utiliser un GPU dans une tâche interactive :

.. code-block:: console

    [alice@iv11 ~]$ salloc --gres=gpu

Pour utiliser les deux GPU dans un script de tâche :

.. code-block:: bash

    #!/bin/bash
    #SBATCH --job-name=my-job
    #SBATCH --partition=iq-main
    #SBATCH --gpus-per-node=nvidia_a40:2

    ...

Nœuds contribués
----------------

Pour lancer une tâche sur un ou plusieurs nœuds contribués auxquels vous avez
accès, demandez la partition correspondante avec ``-p`` ``--partition``.
Référez-vous au tableau des :ref:`nœuds de calcul contribués <contrib-label>`.
La durée maximale des tâches varie selon la partition.

Pour utiliser à la fois les nœuds publics et un ou plusieurs nœuds contribués,
listez les partitions correspondantes, séparées par des virgules. Par exemple :
``--partition=iq-main,iq-alice``. Les nœuds publics seront utilisés
préférentiellement. L’ordre des partitions n’a aucun effet.

Accès partagé
'''''''''''''

Certains nœuds contribués sont partagés via la partition ``iq-preempt``. Si
votre groupe de recherche partage un de ses nœuds de calcul, vous avez accès à
cette partition. Toutefois, les tâches en cours sont annulées automatiquement si
le propriétaire d’un nœud où elles s’exécutent en a besoin pour ses propres
tâches. Le propriétaire dispose ainsi d’un « droit de préemption ».

Les tâches annulées reçoivent un signal ``SIGTERM`` et disposent d’une minute de
grâce pour sauvegarder leurs résultats partiels. Une tâche ainsi annulée peut
demander à être remise dans la file d’attente avec l’option ``--requeue``.

La partition ``iq-preempt`` est utile pour les lots de courtes tâches et pour
les tâches qui utilisent des logiciels capables d’écrire des points de contrôle
afin de reprendre un calcul interrompu.

Pour utiliser à la fois les nœuds publics et les nœuds partagés, utilisez
``--partition=iq-main,iq-preempt``. Les nœuds publics seront utilisés
préférentiellement. Pour utiliser à la fois les nœuds publics, les nœuds
partagés et un ou plusieurs nœuds contribués, utilisez e.g.
``--partition=iq-main,iq-alice,iq-preempt``. Pour exclure un nœud particulier,
tel que le vôtre pour le réserver à d’autres tâches, ajoutez e.g.
``--exclude=cp3799``.

Ressources disponibles
----------------------

La commande ``susage`` affiche les ressources allouées ou disponibles dans une
ou plusieurs partitions. Utilisée sans option ni argument, elle affiche les
ressources allouées dans les partitions auxquelles vous avez accès :

.. code-block:: console

    [alice@iv11 ~]$ susage
    iq-main:  nodes: (2+1)/4 cpus: 104/336 mem: 1.0T/2.0T gpus: 0/2
              ██████▓▓▓░░░░░ ████░░░░░░░░░ ███████░░░░░░░ ░░░░░░░░░
    iq-alice:                cpus: 192/192 mem: 612G/2.2T
                             █████████████ ████░░░░░░░░░░

À titre d’exemple, pour afficher les ressources disponibles, avec une légende,
dans la partition ``iq-main`` seulement :

.. code-block:: console

    [alice@iv11 ~]$ susage --available --legend iq-main
    iq-main: nodes: (1+1)/4 cpus: 232/336 mem: 1.0T/2.0T gpus: 2/2
             ███▓▓▓░░░░░░░░ █████████░░░░ ███████░░░░░░░ █████████

    Resources are reported as available/total. For nodes, (idle+mixed) are
    reported. Mixed nodes are those with both allocated and idle CPUs. █ indicates
    available resources, ░ allocated resources, and ▓ nodes in the mixed state.

Gestion des tâches
------------------

La commande ``squeue`` liste toutes les tâches dans l’ordonnanceur, incluant les
tâches de tous les utilisateurs. Utilisez ``sq`` pour lister uniquement vos
tâches. (Cette dernière commande est aussi disponible sur les grappes de
l’Alliance.)

.. _tâches-actives-label:

Suivre les tâches actives
'''''''''''''''''''''''''

Lorsqu’une de vos tâches démarre, il est important de vérifier qu’elle utilise
adéquatement les ressources qui lui ont été assignées. Par exemple, si une tâche
a accès à 4 cœurs CPU et 80G de mémoire, utilise-t-elle vraiment ces 4 cœurs à
100% et sa consommation de mémoire est-elle dans cet ordre de grandeur ?

Pour le vérifier, connectez-vous avec ``ssh`` à un nœud de calcul assigné à
votre tâche et exécutez la commande ``htop``, qui donne un aperçu de la
consommation de CPU et de mémoire. Dans l’exemple suivant,
``alice`` utilise la sortie de ``sq`` pour identifier le nœud ``cp1433`` avant
de s’y connecter. ``htop`` montre 4 processus à 100% CPU appartenant à Alice, ce
qui correspond aux 4 CPU assignés à sa tâche.

.. code-block:: console

   [alice@iv11 ~]$ sq
             JOBID     USER      ACCOUNT           NAME  ST  TIME_LEFT NODES CPUS       GRES MIN_MEM NODELIST (REASON)
           5623630 alice    def-alice         md-job.sh   R      14:56     1    4     (null)      1G cp1433 (None)
   [alice@iv11 ~]$ ssh cp1433
   Last login: Wed Aug 21 11:16:34 2024 from iv11.m
   [alice@cp1433-mp2 ~]$ htop

       0[||||||||100.0%]    8[          0.0%]    16[          0.0%]   24[          0.0%]
       1[||||||||100.0%]    9[          0.0%]    17[|         0.7%]   25[          0.0%]
       2[||||||||100.0%]   10[          0.0%]    18[          0.0%]   26[          0.0%]
       3[||||||||100.0%]   11[          0.0%]    19[          0.0%]   27[          0.0%]
       4[          0.0%]   12[          0.0%]    20[          0.0%]   28[          0.0%]
       5[          0.0%]   13[          0.0%]    21[          0.0%]   29[          0.0%]
       6[          0.0%]   14[          0.0%]    22[          0.0%]   30[          0.0%]
       7[          0.0%]   15[          0.0%]    23[|         0.7%]   31[          0.0%]
     Mem[|||                      6.82G/252G]   Tasks: 63, 174 thr; 5 running
     Swp[                              0K/0K]   Load average: 2.40 0.71 1.22
                                             Uptime: 1 day, 20:53:58

      PID USER      PRI  NI  VIRT   RES   SHR S CPU%▽MEM%   TIME+  Command
    35160 alice      20   0  457M 97680 19588 R  99.  0.0  0:51.67 /cvmfs/soft.computecanada.
    35161 alice      20   0  454M 96376 19248 R  99.  0.0  0:51.93 /cvmfs/soft.computecanada.
    35162 alice      20   0  454M 95832 19248 R  99.  0.0  0:51.83 /cvmfs/soft.computecanada.
    35163 alice      20   0  446M 93644 19252 R 99.3  0.0  0:51.82 /cvmfs/soft.computecanada.
    35449 alice      20   0 58960  4812  3044 R  0.7  0.0  0:00.08 htop
        1 root       20   0  122M  4116  2636 S  0.0  0.0  0:47.60 /usr/lib/systemd/systemd -
     1041 root       20   0 39060  8500  8172 S  0.0  0.0  0:01.65 /usr/lib/systemd/systemd-j
     1074 root       20   0 45472  1840  1352 S  0.0  0.0  0:11.67 /usr/lib/systemd/systemd-u
     1318 root       20   0 48920  1328  1012 S  0.0  0.0  0:00.00 /usr/sbin/rdma-ndd --syste
     1393 root       16  -4 55532   860   456 S  0.0  0.0  0:00.37 /sbin/auditd
     1394 root       16  -4 55532   860   456 S  0.0  0.0  0:00.00 /sbin/auditd
     1395 root       12  -8 84556   888   740 S  0.0  0.0  0:00.39 /sbin/audispd
   F1Help  F2Setup F3SearchF4FilterF5Tree  F6SortByF7Nice -F8Nice +F9Kill  F10Quit

Tâches GPU
""""""""""

Pour les tâches GPU, il importe également de vérifier qu’elles utilisent
adéquatement le ou les GPU qui lui ont été alloués. Pour ce faire,
connectez-vous au nœud de calcul et utilisez la commande ``nvidia-smi``, qui
liste les GPU et les programmes qui les utilisent. Par exemple :

.. code-block:: console

   [alice@iv11 ~]$ ssh cp3705
   Last login: Wed Aug 21 13:47:44 2024 from iv11.m
   [alice@cp3705-mp2 ~]$ nvidia-smi
   Wed Aug 21 13:52:41 2024
   +-----------------------------------------------------------------------------------------+
   | NVIDIA-SMI 550.54.15              Driver Version: 550.54.15      CUDA Version: 12.4     |
   |-----------------------------------------+------------------------+----------------------+
   | GPU  Name                 Persistence-M | Bus-Id          Disp.A | Volatile Uncorr. ECC |
   | Fan  Temp   Perf          Pwr:Usage/Cap |           Memory-Usage | GPU-Util  Compute M. |
   |                                         |                        |               MIG M. |
   |=========================================+========================+======================|
   |   0  NVIDIA A40                     Off |   00000000:65:00.0 Off |                    0 |
   |  0%   30C    P0             81W /  300W |     370MiB /  46068MiB |      0%      Default |
   |                                         |                        |                  N/A |
   +-----------------------------------------+------------------------+----------------------+
   |   1  NVIDIA A40                     Off |   00000000:CA:00.0 Off |                    0 |
   |  0%   29C    P0             70W /  300W |     276MiB /  46068MiB |      0%      Default |
   |                                         |                        |                  N/A |
   +-----------------------------------------+------------------------+----------------------+

   +-----------------------------------------------------------------------------------------+
   | Processes:                                                                              |
   |  GPU   GI   CI        PID   Type   Process name                              GPU Memory |
   |        ID   ID                                                               Usage      |
   |=========================================================================================|
   |    0   N/A  N/A     14734      C   gmx_mpi                                       362MiB |
   |    1   N/A  N/A     14734      C   gmx_mpi                                       268MiB |
   +-----------------------------------------------------------------------------------------+

On remarque que le processus ``gmx_mpi`` (id 14734) utilise les deux GPU.

Statistiques des tâches terminées
'''''''''''''''''''''''''''''''''

La commande ``seff`` affiche des statistiques pour les tâches terminées,
incluant leur efficacité en CPU et en mémoire. Par exemple :

.. code-block:: console

   [alice@ip15-mp2 ~]$ seff 5623631
   Job ID: 5623631
   Cluster: mp2
   User/Group: alice/alice
   State: COMPLETED (exit code 0)
   Nodes: 1
   Cores per node: 4
   CPU Utilized: 01:00:09
   CPU Efficiency: 99.59% of 01:00:24 core-walltime
   Job Wall-clock time: 00:15:06
   Memory Utilized: 353.91 MB (estimated maximum)
   Memory Efficiency: 8.64% of 4.00 GB (1.00 GB/core)

Typiquement, l’efficacité en CPU devrait être proche de 100%. Une efficacité
plus basse indique que du temps CPU est perdu, possiblement parce que la tâche
n’utilise pas toutes les ressources allouées. Si l’efficacité d’une de vos
tâches est sous 70%, vous ne devriez pas soumettre d’autres tâches similaires
avant de régler ce problème.

L’efficacité en mémoire, pour sa part, devrait être d’au moins 50%. Si une de
vos tâches est sous ce seuil, réduisez la quantité de mémoire demandée pour les
tâches similaires. (Si vous demandez la quantité de mémoire par défaut, 1G par
cœur, ignorez l’efficacité mémoire puisque votre consommation absolue est de
toute façon très basse.)

En surveillant l’efficacité de vos tâches, vous ne vous assurez pas seulement
qu’elles soient plus rapides : vous permettez aussi à un plus grand
nombre de tâches d’être exécutées simultanément, ce qui réduit le temps
d’attente pour tous les chercheurs.
