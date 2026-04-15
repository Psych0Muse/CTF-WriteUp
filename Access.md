---
date: 2026-04-15
tags:
  - htb
  - medium
  - mdb
  - cached_creds
  - runas
  - access
---

# Recon

![[Pasted image 20260415112124.png]]


nothing on enum4linux


# Exploit

## FTP

On peut établir une connexion **anonymous** au serveur FTP et y trouver des documents.

![[Pasted image 20260415113244.png]]



### access control zip

On peut pas l'extraire a cause d'un mot de passe.

![[Pasted image 20260415113338.png]]


On peut utiliser **John** ou **fcrackzip** pour cracker le hash, mais aucun de ces outils ne m'a donné de résultats.


### backup.mdb

Je n'étais pas capable d'ouvrir le fichier initialement.
![[Pasted image 20260415120301.png]]


Dans la connexion FTP, il faut utiliser le **mode binary** pour télécharger le fichier octet par octet. Après cela, je suis capable de voir les tables du fichier `.mdb`.


![[Pasted image 20260415120354.png]]


La table `auth_user` semble intéressante car je peux peut-être y trouver des **credentials**. On y trouve effectivement des mots de passe que je vais ajouter à une liste.


![[Pasted image 20260415121613.png]]



### retour sur access control.zip


Je saisis le mot de passe trouvé et j'obtiens un fichier `.pst`, qui est un fichier de données **Outlook**.

![[Pasted image 20260415122109.png]]


En ligne, je découvre que ce type de fichier peut contenir des informations sur l'usager.
![[Pasted image 20260415123417.png]]



J'utilise **readpst** pour extraire l'information, ce qui génère un fichier `.mbox`.

![[Pasted image 20260415123707.png]]

![[Pasted image 20260415123712.png]]


On y retrouve l'usager **john** ainsi qu'un compte nommé **security** avec le mot de passe `4Cc3ssC0ntr0ller`



## port 23 telnet


Je suis capable de me loguer sur le compte **security**.

![[Pasted image 20260415124204.png]]


Et on retrouve le premier **flag**.
![[Pasted image 20260415124235.png]]









# PrivEsc



Je trouve un répertoire intéressant.

![[Pasted image 20260415124433.png]]




Au départ, je n'ai pas réussi à trouver un moyen de faire une escalade de privilèges. J'avais manqué un répertoire caché (**hidden**) dans l'utilisateur `Public`.

![[Pasted image 20260415134242.png]]


![[Pasted image 20260415134233.png]]



Je trouve un fichier .lnk

![[Pasted image 20260415134414.png]]


Je découvre que les **credentials** de l'administrateur sont sauvegardés avec la commande `/savecred`.

## methode Nishang

On peut se connecter à l'usager **Administrator** puisque les identifiants sont sauvegardés. La raison pour laquelle on ne peut pas exécuter `runas` directement est que cette application crée un nouveau processus, donc on ne peut pas le voir avec notre connexion actuelle.

Pour pallier a ça, j'utilise nishang.

![[Pasted image 20260415140526.png]]


Je copie le **reverse shell** pour y configurer mon adresse IP, afin qu'il sache se connecter à mon **listener**.

![[Pasted image 20260415140519.png]]

![[Pasted image 20260415140603.png]]


On peut lancer cette commande pour créer le nouveau processus avec `runas`, ce qui va entamer une connexion vers mon **listener**.

![[Pasted image 20260415141033.png]]


On voit que l'on reçoit la connexion.

![[Pasted image 20260415141040.png]]

Et je trouve le flag.
![[Pasted image 20260415141116.png]]



