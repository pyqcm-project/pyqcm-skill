Ansys
=====

Ansys est une suite logicielle commerciale pour la conception 3D et la
simulation.

.. seealso::


   - `Ansys <https://docs.alliancecan.ca/wiki/Ansys/fr>`_ (documentation
     technique de l’Alliance)

License
-------

La Grappe IQ a un serveur de licence CMC Microsystems dédié pour Ansys. Chaque
utilisateur doit configurer sa license :

#. Créez le fichier ``~/.licences/ansys.lic`` avec ce contenu :

.. code-block::

    setenv("ANSYSLMD_LICENSE_FILE", "6624@ip39.ccs.usherbrooke.ca")
    setenv("ANSYSLI_SERVERS", "2325@ip39.ccs.usherbrooke.ca")

#. Envoyez un courriel à CMC Microsystems (`mcsupport@cmc.ca`) avec votre nom
   complet, le nom de la personne qui vous fournit la license, votre nom
   d’utilisateur sur la Grappe IQ et l’adresse du serveur de licence
   (``ip39.ccs.usherbrooke.ca``).

#. CMC Microsystem activera votre licence en quelques heures, au plus quelques
   jours. Une fois cela fait, vous pourrez utiliser Ansys tel que décrit
   ci-dessous.

Modules
-------

Les modules Ansys sont les mêmes que sur les grappes de l’Alliance. Par
exemple :

.. code-block:: console

    [alice@iv11 ~]$ module load ansys/2023R2
