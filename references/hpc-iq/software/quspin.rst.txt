QuSpin
======

QuSpin est un paquet Python `Open Source` pour la diagonalisation exacte et la
dynamique quantique de bosons, fermions et systèmes à plusieurs corps
arbitraires. Il supporte diverses symétries définies par l’utilisateur pour les
réseaux à dimension quelconque, ainsi que l’évolution temporelle suivant un
protocole spécifié par l’utilisateur.

Installation
------------

.. code-block:: bash

    module load StdEnv/2023
    module load python/3.11.5 scipy-stack/2024a
    virtualenv --no-download $HOME/venv/quspin
    source $HOME/venv/quspin/bin/activate
    pip install --no-index --upgrade pip
    pip install --no-index quspin==1.0.0
