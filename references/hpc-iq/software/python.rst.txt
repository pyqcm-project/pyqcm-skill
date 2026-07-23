Python
======

L’informations ci-dessous est complémentaire à la documentation technique de
l’Alliance sur `Python <https://docs.alliancecan.ca/wiki/Python/fr>`_.

Modules
-------

Pour charger une version de Python compatible avec l’environnement logiciel
par défaut :


.. code-block:: console

    [alice@iv11 ~]$ module avail python
    ------------------------------------- Core Modules --------------------------------------
       ipython-kernel/3.10              python-build-bundle/2024a (D)
       ipython-kernel/3.11       (D)    python/3.10.13            (t,3.10)
       ipython-kernel/3.12              python/3.11.5             (t,D:3.11)
       python-build-bundle/2023b        python/3.12.4             (t)

    ...

    [alice@iv11 ~]$ module load python/3.11.5

La version recommandée (module par défaut) est indiquée par ``(D)``. Si vous
avez besoin d’une version qui n’est pas disponible dans l’environnement logiciel
par défaut, utilisez ``module spider python`` pour afficher toutes les versions
disponibles.

En plus de modules pour Python lui-même, les logiciels de l’Alliance contiennent
``scipy-stack``, des modules pour `Scientific Python` fournissant ``scipy`` mais
aussi ``numpy``, ``pandas``, ``matplotlib``, etc. Une fois votre module pour
Python lui-même chargé :

.. code-block:: console

    [alice@iv11 ~]$ module avail scipy-stack

    ------------------------------------- Core Modules --------------------------------------
       scipy-stack/2023b (math)    scipy-stack/2024a (math)    scipy-stack/2024b (math,D)

    ...

    [alice@iv11 ~]$ module load scipy-stack/2024b

Environnements virtuels
-----------------------

Tel qu’expliqué dans la section suivante, les paquets Python (autres que
`Scientific Python`) peuvent être installés avec la commande ``pip``, en les
téléchargeant de PyPI ou en les choisissant parmis les paquets précompilés par
l’équipe des logiciels de l’Alliance. Toutefois, nous recommandons de ne jamais
installer de paquets Python sans d’abord activer un environnement virtuel.

Les environnements virtuels Python sont des répertoires contenant un ensemble de
paquets installés pour une tâche donnée. Par exemple, vous pourriez avoir un
environnement virtuel pour simuler des systèmes quantiques avec QuTiP et un
autre pour l’apprentissage machine avec TensorFlow. Vous pourriez même avoir
plusieurs environnements virtuels pour différentes versions de QuTiP ou
TensorFlow au gré de vos projets.

En utilisant systématiquement un environnement virtuel pour vos tâches Python,
vous faciliterez l’installation de vos logiciels, le déboguage de vos calculs et
la reproducibilité de votre recherche. À l’inverse, si vous installez tous vos
paquets Python dans l’emplacement par défaut sans discernement, vous ferez face
à plus de problèmes de compatibilité et l’installation ou la mise à jour d’un
paquet pourrait boguer une tâche qui fonctionnait précédemment. De plus, vous ne
pourrez pas alterner entre différentes versions d’un paquet. Finalement, il sera
plus difficile de ré-exécuter une tâche sur un autre ordinateur ou de fournir
des instructions pour reproduire votre environnement logiciel et, donc, votre
recherche.

Pour suivre cette longue explication, voici une courte démonstration. D’abord,
chargeons les modules pour Python et `Scientific Python` :

.. code-block:: console

    [alice@iv11 ~]$ module load python/3.11.5
    [alice@iv11 ~]$ module load scipy-stack/2024b

Ensuite, créons un environnement virtuel :

.. code-block:: console

    [alice@iv11 ~]$ virtualenv --no-download $HOME/venv/qutip

Activons l’environnement :

.. code-block:: console

    [alice@iv11 ~]$ source $HOME/venv/qutip/bin/activate

Vous remarquerez que l’invite de commande change pour indiquer l’environnement
virtuel actif. Toutes les actions de la commande ``pip`` (installer,
désinstaller, mettre à jour des paquets) cibleront désormais le répertoire
``$HOME/venv/qutip``.

La première chose à faire est de mettre à jour ``pip`` :

.. code-block:: console

    (qutip) [alice@iv11 ~]$ pip install --no-index --upgrade pip

Ensuite, nous pouvons installer des paquets, par exemple QuTiP :

.. code-block:: console

    (qutip) [alice@iv11 ~]$ pip install --no-index qutip==5.0.1

Finalement, l’environnement peut être désactivé :

.. code-block:: console

    (qutip) [alice@iv11 ~]$ deactivate

Une fois l’environnement créé, il peut être réutilisé simplement en l’activant à
nouveau ; nul besoin de réinstaller les paquets. Par exemple, l’environnement
construit ci-dessus peut être utilisé dans un script de tâche avec :

.. code-block:: bash

   module purge
   module load python/3.11.5
   module load scipy-stack/2024b
   source $HOME/venv/qutip/bin/activate

Installer hors d’un environnement virtuel
'''''''''''''''''''''''''''''''''''''''''

Si vous essayez d’installer des paquets Python sans d’abord activer un
environnement virtuel, vous obtiendrez l’erreur suivante :

.. code-block:: console

    [alice@iv11 ~]$ pip install --no-index numpy
    ERROR: Could not find an activated virtualenv (required).

Si vous souhaitez néanmoins installer un paquet à l’extérieur d’un environnement
virtuel, vous pouvez le faire avec :

.. code-block:: console

    [alice@iv11 ~]$ PIP_REQUIRE_VIRTUALENV=false pip install --no-index numpy

.. note::

    Ce comportement est différent de celui des grappes de l’Alliance, où il est
    possible par défaut d’installer des paquets Python à l’extérieur d’un
    environnement virtuel.

Paquets Python précompilés
--------------------------

La commande ``avail_wheels`` liste les paquets logiciels Python précompilés par
l’équipe des logiciels de l’Alliance. Ces paquets sont optimisés pour le CHP.
Par exemple, pour chercher Qiskit:

.. code-block:: console

    [alice@iv11 ~]$ avail_wheels qiskit
    name    version    python    arch
    ------  ---------  --------  -------
    qiskit  1.2.4      cp38      generic

Pour installer cette version pré-compilée dans un environnement virtuel actif :


.. code-block:: console

    (qiskit) [alice@iv11 ~]$ pip install --no-index qiskit==1.2.4

Parallélisation avec Python
---------------------------

Le code Python n’est pas typiquement parallélisé. Par conséquent, demander
plusieurs cœurs CPU n’accélérera pas vos tâches automatiquement ! Vous devez
d’abord paralléliser votre code, soit explicitement, soit en utilisant des
fonctions parallélisées d’une bibliothèque, comme certaines fonctions de NumPy
ou SciPy.

À cause d’une limitation intrinsèque, le « global interpreter lock », le code
Python ne peut être parallélisé avec le modèle de mémoire partagée. Il existe
toutefois des alternatives. L’une d’elles est de coder une extension Python en
C/C++ en utilisant une bibliothèque de programmation parallèle telle qu’OpenMP.
Une autre est d’utiliser le modèle de mémoire distribué avec de multiple
processus Python. Pour ce faire, vous pouvez utiliser le module
``multiprocessing``, ou encore une bibliothèque telle que `mpi4py
<https://mpi4py.readthedocs.io/en/stable/>`_ (passage de messages) ou `Dask
<https://www.dask.org/>`_ (calcul distribué).

.. _python-fils-label:

Sur-souscription de fils
''''''''''''''''''''''''

Un problème commun avec le parallélisme dans Python est la sur-souscription de
fils d’éxécution (« thread oversubscription »), c’est-à-dire que le nombre de
fils d’exécution lancés dans une tâche est supérieur au nombre de cœurs CPU
alloués à la tâche. Le module ``multiprocessing``, en particulier, lance par
défaut autant de fils d’exécution qu’il y a de cœurs CPU, sans considérer si les
cœurs sont tous accessibles. Par exemple, ``multiprocessing`` lancera par défaut
64 fils d’exécution si vous l’utilisez dans une tâche sur un nœud CPU de la
Grappe IQ, même si n’avez demandé que 2, 4 ou 8 cœurs.

Ce problème est aggravé si l’on utilise aussi une fonction parallélisée qui
lance par défaut fils qu’il y a de cœurs (telle que
``scipy.sparse.linalg.eigsh``). Si l’on poursuit l’exemple ci-haut, dans une
tâche qui utilise à la fois ``multiprocessing`` et ``eigsh``, 4096 fils
d’exécution (64 × 64) seront lancés par défaut, même si la tâche n’a accès qu’à
2, 4 ou 8 cœurs. La performance sera drastiquement réduite.

Pour palier ce problème, vous devez spécifier à SciPy, ``multiprocessing``,
Dask, etc. le nombre de fils d’exécution à utiliser. En ajoutant les
instructions suivantes à votre script de tâche (avant votre calcul), vous
désactiverez la parallélisation implicite de la plupart des fonctions, incluant
celles de SciPy, qui utilise OpenMP ou Intel MKL en arrière-plan :

.. code-block:: bash

    export OMP_NUM_THREADS=1
    export MKL_NUM_THREADS=1

Pour contrôler le nombre de processus à lancer avec ``multiprocessing`` :

.. code-block:: python

    from multiprocessing import Pool
    from os import environ

    nprocesses = int(environ.get('SLURM_CPUS_PER_TASK', default=1))

    pool = Pool(nprocesses)

Avec Dask :

.. code-block:: python

    from os import environ
    from dask.distributed import LocalCluster

    nprocesses = int(environ.get('SLURM_CPUS_PER_TASK', default=1))

    cluster = LocalCluster(n_workers=nprocesses)

Si, au contraire, vous n’utilisez pas ``multiprocessing``, Dask, etc. mais que
vous souhaitez plutôt prendre avantage des fonctions parallèles de SciPy,
contrôlez le nombre de fils d’exécution avec :

.. code-block:: bash

    export OMP_NUM_THREADS=${SLURM_CPUS_PER_TASK:-1}
    export MKL_NUM_THREADS=${SLURM_CPUS_PER_TASK:-1}

.. seealso::

   - :ref:`Cette entrée <calcul-lent-label>` de notre FAQ discute les problèmes
     de performance et de fils d’exécution de manière plus générale.
