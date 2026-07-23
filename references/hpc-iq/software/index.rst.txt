Logiciels
=========

Lorsque vous vous connectez à la Grappe IQ, votre environnement ne contient
initialement qu’un petit nombre de logiciels. Vous pouvez en ajouter à partir du
large éventail de `logiciels disponibles
<https://docs.alliancecan.ca/wiki/Available_software/fr>`_. Pré-installés et
optimisés pour le CHP, ces logiciels sont les mêmes qui sont disponibles sur les
grappes nationales, ce qui vous assure que vos tâches peuvent être exécutées de
manière reproductible sur toutes les grappes.

L’ajout de logiciels à votre environnement se fait principalement à l’aide de la
commande ``module``. Si vous n’êtes pas familier avec son utilisation, lisez
d’abord comment `utiliser des modules
<https://docs.alliancecan.ca/wiki/Utiliser_des_modules>`_. À titre d’exemple,
vous pouvez lister les modules chargés avec ``module list``, chercher des
modules avec ``module spider`` et les ajouter à votre environnement avec
``module load``. Par exemple :

.. code-block:: console

    [alice@iv11 ~]$ module list

    Currently Loaded Modules:
      1) CCconfig            6) ucx/1.14.1            11) flexiblas/3.3.1
      2) gentoo/2023   (S)   7) libfabric/1.18.0      12) imkl/2023.2.0            (math)
      3) gcccore/.12.3 (H)   8) pmix/4.2.4            13) StdEnv/2023              (S)
      4) gcc/12.3      (t)   9) ucc/1.2.0             14) mii/1.1.2
      5) hwloc/2.9.1        10) openmpi/4.1.5    (m)  15) slurm-completion/23.02.7

      Où :
       S:     Module is Sticky, requires --force to unload or purge
       m:     MPI implementations / Implémentations MPI
       math:  Mathematical libraries / Bibliothèques mathématiques
       t:     Tools for development / Outils de développement
       H:                Hidden Module

    [alice@iv11 ~]$ module spider gromacs

    --------------------------------------------------------------------------------------
      gromacs:
    --------------------------------------------------------------------------------------
        Description:
          GROMACS is a versatile package to perform molecular dynamics, i.e. simulate the
          Newtonian equations of motion for systems with hundreds to millions of
          particles.

         Versions:
            gromacs/2016.6
            gromacs/2020.4
            gromacs/2020.6
            gromacs/2021.2
            gromacs/2021.4
            gromacs/2021.6
            gromacs/2022.2
            gromacs/2022.3
            gromacs/2023
            gromacs/2023.2
            gromacs/2023.3
            gromacs/2023.5
            gromacs/2024.1
            gromacs/2024.4
         Autres candidats possibles :
            gromacs-colvars  gromacs-cp2k  gromacs-ls  gromacs-plumed  gromacs-ramd
            gromacs-swaxs

    --------------------------------------------------------------------------------------
      Pour trouver d'autres correspondances à votre recherche, exécutez :

          $ module -r spider '.*gromacs.*'

    --------------------------------------------------------------------------------------
      Pour de l'information détaillée à propos d'un module "gromacs" spécifique (incluant
      comment charger ce module), utilisez le nom complet. Par exemple :

         $ module spider gromacs/2024.4
    --------------------------------------------------------------------------------------

    [alice@iv11 ~]$ module spider gromacs/2024.4

    --------------------------------------------------------------------------------------
      gromacs: gromacs/2024.4
    --------------------------------------------------------------------------------------
        Description:
          GROMACS is a versatile package to perform molecular dynamics, i.e. simulate the
          Newtonian equations of motion for systems with hundreds to millions of
          particles.

        Propriétés:
          Chemistry libraries/apps / Logiciels de chimie

        Vous devrez charger tous les modules de l'un des lignes suivantes avant de pouvoir
        charger le module "gromacs/2024.4".

          StdEnv/2023  gcc/12.3  openmpi/4.1.5
          StdEnv/2023  gcc/12.3  openmpi/4.1.5  cuda/12.2

        Help:
          Description
          ===========
          GROMACS is a versatile package to perform molecular dynamics, i.e. simulate the
          Newtonian equations of motion for systems with hundreds to millions of
          particles.

          More information
          ================
           - Homepage: http://www.gromacs.org

    [alice@iv11 ~]$ module load StdEnv/2023 gcc/12.3 openmpi/4.1.5
    [alice@iv11 ~]$ module load gromacs/2024.4
    [alice@iv11 ~]$ module list

    Currently Loaded Modules:
      1) CCconfig                         10) hwloc/2.9.1
      2) gentoo/2023              (S)     11) ucx/1.14.1
      3) imkl/2023.2.0            (math)  12) libfabric/1.18.0
      4) StdEnv/2023              (S)     13) pmix/4.2.4
      5) mii/1.1.2                        14) ucc/1.2.0
      6) slurm-completion/23.02.7         15) openmpi/4.1.5    (m)
      7) gcc/12.3                 (t)     16) fftw/3.3.10      (math)
      8) flexiblas/3.3.1                  17) gromacs/2024.4   (chem)
      9) gcccore/.12.3            (H)

      Où :
       S:     Module is Sticky, requires --force to unload or purge
       m:     MPI implementations / Implémentations MPI
       math:  Mathematical libraries / Bibliothèques mathématiques
       t:     Tools for development / Outils de développement
       chem:  Chemistry libraries/apps / Logiciels de chimie
       H:                Hidden Module

Version de l’environnement logiciel
-----------------------------------

L’équipe de l’Alliance en charge des logiciels crée périodiquement de nouvelles
versions de l’environnement, par exemple ``StdEnv/2023`` ou ``StdEnv/2020``. Sur
la Grappe IQ, ``StdEnv/2023`` est l’environnement initialement chargé lorsque
vous vous connectez. Les autres environnements sont disponibles et peuvent être
chargés. Par exemple :

.. code-block:: console

    [alice@iv11 ~]$ module load StdEnv/2020

    Modules inactifs:
      1) slurm-completion

    Dû à un changement dans la variable MODULEPATH, les modules suivants ont été rechargés :
      1) mii/1.1.2

    Les modules suivants ont été rechargés avec un changement de version :
      1) StdEnv/2023 => StdEnv/2020           5) libfabric/1.18.0 => libfabric/1.10.1
      2) gcccore/.12.3 => gcccore/.9.3.0      6) openmpi/4.1.5 => openmpi/4.0.3
      3) gentoo/2023 => gentoo/2020           7) ucx/1.14.1 => ucx/1.8.0
      4) imkl/2023.2.0 => imkl/2020.1.217

Cible d’optimisation
--------------------

Les logiciels de l’Alliance disponibles sur la Grappe IQ sont optimisés pour
l’utilisation de processeurs x86 qui supportent le jeu d’instructions AVX2. Ces
logiciels n’ont pas été compilés pour utiliser de jeux d’instructions plus
récents tels qu’AVX512. C’est le meilleur choix pour supporter à la fois les
processeurs Intel et AMD des nœuds de calcul tout en assurant une bonne
performance. Si vous compilez votre code avec GCC, l’option d’optimisation
correspondante est ``-march=core-avx2``. Avec les compilateurs Intel, utilisez
``-xCORE-AVX2``.

Nous ne recommendons pas d’utiliser une architecture différente (e.g. ``module
load arch/avx512``) ou de compiler votre code avec une option différente (e.g.
``-march=native`` ou ``-xHost``) car cela peut mener à des problèmes de
compatibilité.

Bibliothèques BLAS/LAPACK
-------------------------

Les logiciels de l’Alliance offrent plusieurs implémentations des bibliothèques
`BLAS et LAPACK <https://docs.alliancecan.ca/wiki/BLAS_and_LAPACK/fr>`_ pour
l’algèbre linéaire. Intel MKL est utilisé par défaut sur la Grappe IQ et la
plupart des grappes de l’Alliance.

Guides logiciels
----------------

Les pages suivantes portent sur l’utilisation de logiciels spécifiques. Certains
sont également disponibles sur les grappes nationales, auquel cas l’accent est
mis sur les particularités de leur utilisation avec la Grappe IQ.

.. toctree::
   :maxdepth: 1

   ansys
   mathematica
   pyqcm
   python
   qiskit
   quimb
   quspin
