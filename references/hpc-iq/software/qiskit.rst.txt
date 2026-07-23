Qiskit
======

Qiskit est une plateforme logicielle d’IBM pour le calcul quantique.

Installation
------------

.. code-block:: bash

    module load StdEnv/2023
    module load python/3.11.5 symengine/0.13.0
    virtualenv --no-download $HOME/venv/qiskit
    source $HOME/venv/qiskit/bin/activate
    pip install --no-index --upgrade pip
    pip install --no-index qiskit==2.3.1
    # Optionnel : simulateur Aer
    pip install --no-index qiskit-aer==0.17.2
    # Optionnel : support des GPU dans Aer
    pip install --no-index qiskit-aer-gpu==0.17.2
    # Optionnel : paquet pour les problèmes en sciences naturelles
    pip install --no-index qiskit-nature==0.7.2
