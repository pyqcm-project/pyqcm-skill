.. faq

Foire aux questions (FAQ)
=========================

Problèmes fréquents
-------------------

« Illegal instruction » (``SIGILL``)
''''''''''''''''''''''''''''''''''''

Votre tâche cause une erreur de type « Illegal instruction » ou ``SIGILL``, par
example:

.. code-block:: console

   [alice@iv11 ~]$ cat slurm-3304643.out
   /var/spool/slurmd/job3304643/slurm_script: line 11: 25047 Illegal instruction   (core dumped) python my_script.py

Cette erreur se produit lorsqu’un programme optimisé pour une classe de
processeurs particuliers est exécuté sur un processeur qui n’est pas compatible.

Si vous compilez votre code sur le nœud de connexion avec GCC, utilisez l’option
d’optimisation ``-march=core-avx2``. Si vous utilisez les compilateurs Intel,
l’option correspondante est ``-xCORE-AVX2``. N’utilisez pas ``-march=native`` ou
``-xHost``. Ces dernières tentent d’optimiser pour les processeurs Intel du nœud
de connexion. Le programme résultant peut être incompatible avec les processeurs
AMD de certains nœuds de calcul.

.. _calcul-lent-label:

Mon calcul est beaucoup plus lent que sur mon portable
''''''''''''''''''''''''''''''''''''''''''''''''''''''

Vous avez testé un programme parallèle sur votre portable (ou un autre
ordinateur). Ce programme fonctionne aussi dans une tâche sur la Grappe IQ mais
il est beaucoup plus lent. Que se passe-t-il ?

D’abord, assurez-vous d’utiliser le même nombre de cœurs CPU sur votre portable
et dans votre tâche pour que la comparaison soit valide. Par exemple, si votre
portable a 8 cœurs CPU et que votre programme les utilise tous, vous devriez
demander 8 cœurs CPU dans votre tâche. Si vous demandez moins de cœurs, voire un
seul, il est normal que la performance soit moindre !

Si le problème persiste, vérifiez que votre tâche :ref:`utilise correctement les
ressources <tâches-actives-label>` qui lui sont allouées. Un problème commun est
un programme parallèle qui tente d’utiliser tous les cœurs CPU du nœud de calcul
même s’il ne sont pas tous alloués à la tâche. Par exemple, un programme
pourrait lancer 96 fils d’exécution pour tenter d’utiliser les 96 cœurs d’un
nœud CPU de la partition ``iq-main`` même si la tâche n’a accès qu’à un seul de
ces cœurs. Le système d’exploitation alterne alors entre les 96 fils pour que
chacun reçoive sa part de temps CPU. Cette alternance cause une dégradation
importante de la performance, sans compter les potentiels problèmes de
synchronisation entre les fils.

Dans la plupart des cas, le nombre de cœurs CPU alloués à une tâche devrait
correspondre exactement au nombre de processus ou de fils exécutés par la tâche.
Si une tâche exécute trop de fils ou de processus, ``htop`` montrera plus de
fils d’exécution qu’attendu, et ces fils n’utiliseront pas tous 100 % de temps
CPU comme ils le devraient.

Pour les programmes parallèles qui utilisent des fils d’exécution OpenMP, le
problème peut être corrigé en réglant le nombre de fils à exécuter pour qu’il
soit identique au nombre de cœurs alloués. Ajoutez cette instruction à votre
script de tâche avant l’exécution de votre programme :

.. code-block:: bash

   export OMP_NUM_THREADS=${SLURM_CPUS_PER_TASK:-1}

Certaines fonctions d’Intel MKL sont automatiquement parallélisées. En CHP, ce
parallélisme implicite est habituellement indésirable. Vous pouvez le désactiver
avec :

.. code-block:: bash

   export MKL_NUM_THREADS=1

Si vous souhaitez plutôt utiliser ces algorithmes parallèles, paramétrez le
nombre de fils d’exécution avec :

.. code-block:: bash

   export MKL_NUM_THREADS=${SLURM_CPUS_PER_TASK:-1}

.. seealso::

   - :ref:`Cette section <python-fils-label>` de notre guide Python traite du
     problème des fils d’exécution dans le contexte de ce language de
     programmation.
