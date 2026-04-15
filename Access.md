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

<img width="944" height="697" alt="image" src="https://github.com/user-attachments/assets/7089d948-4ee1-4b69-b9fa-7b5dd388ee7d" />



nothing on enum4linux


# Exploit

## FTP

On peut établir une connexion **anonymous** au serveur FTP et y trouver des documents.

<img width="755" height="338" alt="image" src="https://github.com/user-attachments/assets/43479589-78e5-4ae3-a7d4-5d83ed13e0b4" />



### access control zip

On peut pas l'extraire a cause d'un mot de passe.

<img width="537" height="417" alt="image" src="https://github.com/user-attachments/assets/af8349e0-a51c-4c37-ac11-33735bebf3ae" />


On peut utiliser **John** ou **fcrackzip** pour cracker le hash, mais aucun de ces outils ne m'a donné de résultats.


### backup.mdb

Je n'étais pas capable d'ouvrir le fichier initialement.

<img width="707" height="118" alt="image" src="https://github.com/user-attachments/assets/88b1810d-af58-4520-9850-f5fca09d96b3" />



Dans la connexion FTP, il faut utiliser le **mode binary** pour télécharger le fichier octet par octet. Après cela, je suis capable de voir les tables du fichier `.mdb`.


<img width="943" height="434" alt="image" src="https://github.com/user-attachments/assets/5582864f-23e5-42e9-81a8-c026697f0df2" />


La table `auth_user` semble intéressante car je peux peut-être y trouver des **credentials**. On y trouve effectivement des mots de passe que je vais ajouter à une liste.


<img width="624" height="182" alt="image" src="https://github.com/user-attachments/assets/7271c221-91f6-4dae-8459-2cf5d5d67691" />




### retour sur access control.zip


Je saisis le mot de passe trouvé et j'obtiens un fichier `.pst`, qui est un fichier de données **Outlook**.

<img width="220" height="122" alt="image" src="https://github.com/user-attachments/assets/5b954d79-2dd6-4f18-9949-67c065efe80e" />


En ligne, je découvre que ce type de fichier peut contenir des informations sur l'usager.

<img width="1025" height="293" alt="image" src="https://github.com/user-attachments/assets/5e1965ca-e3d5-4ff3-a0f6-ec7f8210acd7" />




J'utilise **readpst** pour extraire l'information, ce qui génère un fichier `.mbox`.

<img width="753" height="231" alt="image" src="https://github.com/user-attachments/assets/ed78b5d2-fe8b-45b5-9d02-c1e42b0af6ed" />

<img width="992" height="549" alt="image" src="https://github.com/user-attachments/assets/544a5cad-2576-44e4-bf90-493df57a8c81" />



On y retrouve l'usager **john** ainsi qu'un compte nommé **security** avec le mot de passe `4Cc3ssC0ntr0ller`



## port 23 telnet


Je suis capable de me loguer sur le compte **security**.

<img width="772" height="838" alt="image" src="https://github.com/user-attachments/assets/879c448e-8a8a-4c66-be2e-367584905346" />



Et on retrouve le premier **flag**.

<img width="580" height="278" alt="image" src="https://github.com/user-attachments/assets/5ef29160-2ca7-4e06-8d0a-5122e181c0f3" />










# PrivEsc



Je trouve un répertoire intéressant.

<img width="587" height="369" alt="image" src="https://github.com/user-attachments/assets/c3d0717e-d1ee-45af-9d27-6d7ecbd03aeb" />





Au départ, je n'ai pas réussi à trouver un moyen de faire une escalade de privilèges. J'avais manqué un répertoire caché (**hidden**) dans l'utilisateur `Public`.

<img width="594" height="326" alt="image" src="https://github.com/user-attachments/assets/735c4de1-1323-4b08-975e-192c829036c8" />



<img width="554" height="451" alt="image" src="https://github.com/user-attachments/assets/d3a93754-7c89-470e-86d3-387f211fc626" />



Je trouve un fichier .lnk

<img width="961" height="436" alt="image" src="https://github.com/user-attachments/assets/94e56606-a5ed-4844-896a-2d7e4d1eada6" />



Je découvre que les **credentials** de l'administrateur sont sauvegardés avec la commande `/savecred`.

## methode Nishang

On peut se connecter à l'usager **Administrator** puisque les identifiants sont sauvegardés. La raison pour laquelle on ne peut pas exécuter `runas` directement est que cette application crée un nouveau processus, donc on ne peut pas le voir dans notre fenêtre actuelle.
Pour pallier a ça, j'utilise nishang.

<img width="777" height="206" alt="image" src="https://github.com/user-attachments/assets/d54b8f70-fa44-4497-bc18-4dd191af8404" />



Je copie le **reverse shell** pour y configurer mon adresse IP, afin qu'il sache se connecter à mon **listener**.

<img width="546" height="67" alt="image" src="https://github.com/user-attachments/assets/96a0abc7-1ea3-4a74-919f-6415ef84fb37" />

<img width="872" height="656" alt="image" src="https://github.com/user-attachments/assets/1c570925-283d-493c-8df4-3e8934ece8b7" />



On peut lancer cette commande pour créer le nouveau processus avec `runas`, ce qui va entamer une connexion vers mon **listener**.

<img width="961" height="173" alt="image" src="https://github.com/user-attachments/assets/374a88ee-3e21-49be-9be3-4de93bc6fd60" />


On voit que l'on reçoit la connexion.

<img width="712" height="252" alt="image" src="https://github.com/user-attachments/assets/38ed18db-4abc-4dbe-ab46-402f5760977f" />


Et je trouve le flag.

<img width="662" height="227" alt="image" src="https://github.com/user-attachments/assets/4cd8354d-7095-4d58-9748-760012a6a0e5" />




