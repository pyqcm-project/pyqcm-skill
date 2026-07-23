QUIMB
=====

Quim est une bibliothèque Python pour l’information quantique et le problème à
plusieurs corps.

Installation
------------

.. code-block:: bash

    module load StdEnv/2023
    module load python/3.11.5
    virtualenv --no-download $HOME/venv/quimb
    source $HOME/venv/quimb/bin/activate
    pip install --no-index quimb==1.12.1

Pour générer des figures :

.. code-block:: bash

    pip install --no-index matplotlib==3.10.8 networkx==3.6.1

Pour utiliser Jax comme moteur d’optimisation :

.. code-block:: bash

    pip install --no-index jax==0.8.3

Pour utiliser Autograd comme moteur d’optimisation :

.. code-block:: bash

    pip install --no-index autograd==1.8.0
