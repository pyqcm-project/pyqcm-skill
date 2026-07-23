Mathematica
===========

Mathematica est un logiciel commercial pour le calcul formel et numérique
développé par Wolfram Research.

License
-------

Les licences Mathematica sont distribuées par un serveur du département de
physique. Aucune configuration n’est requise mais chaque utilisateur est
responsable de se renseigner quant aux politiques d’accès à ces licences.

Utilisation
-----------

Pour voir les versions de Mathematica disponibles :

.. code-block:: console

    [alice@iv11 ~]$ ls /net/nfs-iq/data/software/Mathematica/
    12.1  14.0

Pour utiliser Mathematica dans un script de tâche :

.. code-block:: bash

    #!/bin/bash
    #SBATCH --time=02:00:00
    #SBATCH --cpus-per-task=2
    #SBATCH --mem=8G

    PATH="$PATH:/net/nfs-iq/data/software/Mathematica/12.1/Executables"

    wolframscript -file script.wls

Pour utiliser Mathematica dans une tâche interactive :

.. code-block:: console

    [alice@iv11 ~]$ salloc -t 2:00:00 -c 2 --mem=8G
    salloc: Granted job allocation 5843885
    salloc: Waiting for resource configuration
    salloc: Nodes cp3702 are ready for job
    [alice@cp3702 ~]$ PATH="$PATH:/net/nfs-iq/data/software/Mathematica/12.1/Executables"
    [alice@cp3702 ~]$ wolfram
    Mathematica 12.1.0 Kernel for Linux x86 (64-bit)
    Copyright 1988-2020 Wolfram Research, Inc.

    In[1]:=

Graphiques
----------

Mathematica ne peut être utilisé en mode graphique sur la Grappe IQ. Les tâches
doivent être réalisées en mode texte, soit avec des scripts (``wolframscript``)
ou interactivement (``wolfram``).
