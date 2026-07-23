Pyqcm
=====

Pyqcm est une bibliothèque Python développée par David Sénéchal, qui implémente
les méthodes des grappes pour les systèmes quantiques hautement corrélés.

Pyqcm est disponible comme paquet Python précompilé :

.. code-block:: bash

    module load StdEnv/2023
    module load python/3.11.5 scipy-stack/2025a nlopt/2.10.0
    virtualenv --no-download $HOME/venv/qcm
    source $HOME/venv/qcm-dev/bin/activate
    pip install --no-index --upgrade pip
    pip install --no-index pyqcm==2.12.1

Pour une installation personnalisée, par exemple une version de développement ou
pour expérimenter avec des options :

.. code-block:: bash

    git clone https://bitbucket.org/dsenechQCM/qcm_wed
    cd qcm_wed
    module load StdEnv/2023 gcc/12.3
    module load python/3.11.5 scipy-stack/2025a
    module load flexiblas/3.3.1 eigen/3.4.0 cuba/4.2.2 primme/3.2 nlopt/2.10.0
    virtualenv --no-download $HOME/venv/qcm-dev
    source $HOME/venv/qcm-dev/bin/activate
    pip install --no-index --upgrade pip
    export CMAKE_ARGS="-DEIGEN_HAMILTONIAN=1 -DWITH_PRIMME=1 -DBLA_VENDOR=FlexiBLAS -DPRIMME_DIR=$EBROOTPRIMME -DCUBA_DIR=$EBROOTCUBA -DWITH_GF_OPT_KERNEL=0"
    pip install . --no-index

.. warning::

    Pyqcm offre plusieurs méthodes pour les calculs parallèles, dont les fils
    d’exécution OpenMP. Si Pyqcm est compilé avec GCC mais en utilisant MKL
    comme bibliothèque BLAS avec l’option multi-fils OpenMP d’Intel, deux
    bibliothèques OpenMP seront utilisées en même temps : celle de GCC et celle
    d’Intel. Cela résulte en un nombre incorrect de fils d’exécution : si
    :math:`n` fils OpenMP sont demandés, :math:`n \times n` sont créés. Cela
    cause un ralentissement important des calculs.

    Pour éviter ce problème, compilez Pyqcm avec l’interface FlexiBLAS.

.. warning::

    Si vous utilisez des optimisations spécifiques au CPU local (e.g.
    ``-march=native`` ou ``-xHost``) ou si vous activez
    ``-DWITH_GF_OPT_KERNEL=1``, assurez-vous de faire la compilation dans une
    tâche interactive sur le même nœud de calcul où vous utiliserez Pyqcm. Une
    compilation faite sur le nœud de connexion pourrait être incompatible avec
    certains nœuds de calcul, générant des erreurs de type « illegal
    instruction ».

.. note::

    Les auteurs de Pyqcm `rapportent une faible performance avec BLIS
    <https://qcm-wed.readthedocs.io/en/stable/parallel.html#numerical-integration>`_.
    Sur la Grappe IQ, Intel MKL est utilisé par défaut à travers
    l’interface FlexiBLAS.

.. seealso::

    - `Documentation de Pyqcm <https://dsenech.github.io/qcm_wed_doc/>`_
