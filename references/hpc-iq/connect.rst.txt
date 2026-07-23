Connexion
=========

L’accès à la Grappe IQ se fait par SSH (Secure Shell). L’adresse de la grappe
est ``hpc.iq.ccs.usherbrooke.ca``.

Les systèmes d’exploitation récents (Linux, MacOS, Windows 11) incluent
habituellement l’ensemble d’outils OpenSSH. Pour vous connecter à la grappe en
mode ligne de commande, ouvrez d’abord une fenêtre de terminal (nommé PowerShell
dans Windows). Dans cette fenêtre, utilisez la commande ``ssh`` pour établir une
connexion :

.. code-block:: console

   $ ssh alice@hpc.iq.ccs.usherbrooke.ca

Remplacez ``alice`` par votre nom d’utilisateur.

Lorsque vous vous connectez pour la première fois, votre programme affichera
l’empreinte de la clé de chiffrement du serveur et vous demandera de confirmer
que vous souhaitez vous connecter. Une fois cette confirmation donnée, votre
programme enregistrera la clé du nœud de connexion dans la liste des hôtes
connus. Ce dialogue prendra la forme suivante :

.. code-block:: console

    The authenticity of host 'hpc.iq.ccs.usherbrooke.ca (204.19.23.210)' can't be established.
    ED25519 key fingerprint is SHA256:hVAo6KoqKOEbtOaBh6H6GYHAvsStPsDEcg4LXBQUP50.
    Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
    Warning: Permanently added 'hpc.iq.ccs.usherbrooke.ca' (ED25519) to the list of known hosts.

L’empreinte de la clé du serveur devrait correspondre à celle donnée dans
l’exemple ci-haut.

Entrez ensuite votre mot de passe CCDB. Notez qu’aucun caractère ne s’affichera
à l’écran pendant la saisie de votre mot de passe.

.. code-block:: console

   (alice@hpc.iq.ccs.usherbrooke.ca) Password:

Utilisez un second facteur d’authentification pour compléter le processus de
connexion. Cette invite différera selon le ou les appareils que vous avez
enregistrés dans CCDB. Dans l’exemple suivant, Alice a enregistré un téléphone
Android :

.. code-block:: console

    (alice@hpc.iq.ccs.usherbrooke.ca) Duo two-factor login for alice

    Enter a passcode or select one of the following options:

     1. Duo Push to Samsung S23 Alice (Android)

    Passcode or option (1-1):

En tapant ``1``, Alice obtiendrait une notification sur son téléphone.

Finalement, vous obtiendrez un message indiquant que la connexion a été
établie :

.. code-block:: console

    Success. Logging you in...
    Last login: Wed Apr  9 10:45:16 2025 from 24.203.57.88
      _____ ____
     |_   _/ __ \   Grappe IQ / IQ Cluster
       | || |  | |  hpc.iq.ccs.usherbrooke.ca
       | || |  | |
      _| || |__| |  Documentation fr: https://ccs-udes.github.io/hpc-iq/fr
     |_____\___\_\                en: https://ccs-udes.github.io/hpc-iq/en
                       Support fr/en: olivier.fisette@usherbrooke.ca
                                      michel.l.barrette@usherbrooke.ca (backup)

    [alice@iv11 ~]$

.. seealso::
   - `SSH <https://docs.alliancecan.ca/wiki/SSH/fr>`_ (documentation technique
     de l’Alliance)

Clients SSH alternatifs
-----------------------

Bien qu’un client SSH soit maintenant inclus avec Windows (depuis la version 10
1803), certains chercheurs préfèrent utiliser un client tierce partie pour
bénéficier de fonctionnalités supplémentaires ou simplement par habitude. Des
choix populaires sont :

* `MobaXterm <https://mobaxterm.mobatek.net/>`_
* `PuTTY <https://www.chiark.greenend.org.uk/~sgtatham/putty/>`_
* `Windows Subsystem for Linux (WSL) <https://docs.microsoft.com/en-us/windows/wsl/install>`_

Nous recommandons MobaXterm aux nouveaux utilisateurs plutôt que PuTTY puisque
le premier offre davantage de fonctionnalités. Pour sa part, WSL n’est pas un
client SSH proprement dit mais plutôt un système Linux virtuel installé à
l’intérieur de Windows. WSL donne accès aux commandes Linux de base, incluant
les commandes pour SSH.

.. seealso::
   - Documentation technique de l’Alliance :
       - `Connexion à un serveur avec MobaXterm <https://docs.alliancecan.ca/wiki/Connecting_with_MobaXTerm/fr>`_
       - `Connexion à un serveur avec PuTTY <https://docs.alliancecan.ca/wiki/Connecting_with_PuTTY/fr>`_

Clés SSH
--------

Il est préférable d’utiliser une paire de clés SSH pour vous connecter plutôt
que votre mot de passe. Cela améliore la sécurité tout en étant plus pratique.
Si vous n’avez pas déjà une paire de clés, vous pouvez en générer une dans votre
terminal sur votre ordinateur avec :

.. code-block:: console

    $ ssh-keygen
    Generating public/private ed25519 key pair.
    Enter file in which to save the key (/home/alice/.ssh/id_ed25519):
    Enter passphrase (empty for no passphrase):
    Enter same passphrase again:
    Your identification has been saved in /home/alice/.ssh/id_ed25519.
    Your public key has been saved in /home/alice/.ssh/id_ed25519.pub.
    The key fingerprint is:
    SHA256:GLyOJ2mS7f7d4MPEGu00sa34oSmr7/cTJremUD/G2hI alice@laptop
    The key's randomart image is:
    +--[ED25519 256]--+
    |                 |
    |     .           |
    |      o          |
    |       +.        |
    |      +oS+       |
    |   o =Eo@ .      |
    |  o B o&B=       |
    |   +.+=OO=       |
    |  .=*=***o.      |
    +----[SHA256]-----+

À l’invite, entrez une phrase de passe sécuritaire : au moins 12 caractères,
incluant des lettres minuscules et majuscules, des chiffres et des caractères
spéciaux. Cette phrase sera utilisée pour chiffrer votre clé privée.

Une fois votre paire de clés SSH créée, configurez votre clé publique sur la
Grappe IQ :

.. code-block:: console

    $ ssh-copy-id hpc.iq.ccs.usherbrooke.ca
    /usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: ssh-add -L
    /usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
    /usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
    (alice@hpc.iq.ccs.usherbrooke.ca) Password:
    (alice@hpc.iq.ccs.usherbrooke.ca) Duo two-factor login for alice

    Enter a passcode or select one of the following options:

     1. Duo Push to Samsung S23 Alice (Android)

    Passcode or option (1-1):

    Number of key(s) added: 1

Vous devrez vous authentifier avec votre mot de passe CCDB et un deuxième
facteur Duo pour que votre clé publique soit copiée vers le serveur par
``ssh-copy-id``.

Vous n’aurez plus à utiliser votre mot de passe pour les connexions
subséquentes : votre clé privée sera utilisée à la place. Vous devrez toutefois
entrer votre phrase de passe afin de déchiffrer cette clé. Si vous utilisez un
gestionnaire de mots de passe (e.g. Windows Credential Manager, MacOS Passwords,
SSH Agent), vous n’aurez à déchiffer votre clé qu’une fois par session.

.. note::
    - Le terme « phrase de passe »  est utilisé en cryptographie par analogie
      avec les mots de passe. Pour votre paire de clés SSH, il n’est toutefois
      pas nécessaire que cette « phrase » soit plus longue que les mots de passe
      sécuritaires habituels.
    - La Grappe IQ n’utilise pas les clés SSH publiques configurées dans votre
      compte CCDB. Vous devez configurer votre clé avec la commande
      ``ssh-copy-id`` tel qu’expliqué ci-haut.
    - Les clés SSH améliorent la sécurité en évitant d’exposer votre mot de
      passe. Ainsi, même si la Grappe IQ était compromise, votre compte CCDB
      demeurerait sécurisé.

.. seealso::
    - `Clés SSH <https://docs.alliancecan.ca/wiki/SSH_Keys/fr>`_ (documentation
      technique de l’Alliance)

Configuration d’OpenSSH
-----------------------

Les outils OpenSSH peuvent être configurés sur votre ordinateur à l’aide du
fichier de configuration ``$HOME/.ssh/config``, que vous pouvez éditer à l’aide
d’un éditeur de texte (e.g. Nano, Notepad). Si vous configurez OpenSSH pour la
première fois, vous devrez créer ce fichier.

Simplifier la commande de connexion
'''''''''''''''''''''''''''''''''''

Nous recommandons de créer un raccourci afin d’éviter d’avoir à taper l’adresse
``hpc.iq.ccs.usherbrooke.ca`` au long à chaque connexion :

.. code-block::

    Host iq
    Hostname hpc.iq.ccs.usherbrooke.ca

Si votre nom d’utilisateur CCDB n’est pas le même que votre nom d’utilisateur
sur votre ordinateur, ajoutez l’option ``User`` :

.. code-block::
    :emphasize-lines: 3

    Host iq
    Hostname hpc.iq.ccs.usherbrooke.ca
    User alice

Avec cette configuration en place, vous pourrez vous connecter à la Grappe IQ
avec simplement :

.. code-block:: console

    $ ssh iq

Éviter les invites MFA répétitives
''''''''''''''''''''''''''''''''''

L’authentification multifacteur avec Duo demande l’utilisation d’un second
facteur à chaque connexion. Pour éviter cette torture, nous recommandons
d’activer le multiplexeur OpenSSH. Ainsi, une connexion authentifiée sera
réutilisée lorsque vous établissez une nouvelle session. Vous n’aurez donc pas à
utiliser un second facteur Duo ou même à déchiffrer votre clé SSH privée plus
d’une fois.

Ajoutez les options ``Control...`` suivantes :

.. code-block::
    :emphasize-lines: 3-5

    Host iq
    Hostname hpc.iq.ccs.usherbrooke.ca
    ControlPath ~/.ssh/cm-%r@%h:%p
    ControlMaster auto
    ControlPersist 60m

Éviter les déconnexions intempestives
'''''''''''''''''''''''''''''''''''''

Certains serveurs et réseaux interrompent les connexions SSH inactives après
quelques minutes. Pour éviter ce désagrément, vous pouvez configurer OpenSSH
afin d’envoyer périodiquement un signal au serveur indiquant que la connexion
est toujours active.

Ajoutez l’option ``ServerAliveInterval`` au début de votre fichier de
configuration, avant la première entrée ``Host`` :

.. code-block::
    :emphasize-lines: 1

    ServerAliveInterval 60

    Host iq
    Hostname hpc.iq.ccs.usherbrooke.ca
