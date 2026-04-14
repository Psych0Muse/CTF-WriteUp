---
date: 2026-04-13
tags:
  - htb
  - medium
  - SSRF
  - php
  - lfi
  - dfunc_bypass
  - sudo_binary
  - proc_open
---


# Recon

<img width="768" height="366" alt="image" src="https://github.com/user-attachments/assets/56ad189f-3846-4247-af33-37104ef74e64" />



# Exploit


## port 80


<img width="756" height="488" alt="image" src="https://github.com/user-attachments/assets/c95cad2e-052a-47e8-90d4-45ffe8b88520" />

J'ajoute le siteisup.htb dans mon /etc/hosts. 

Quand je mets le site google, j'obtiens que le serveur est down. Même en mode debug, ça ne fonctionne pas.

<img width="759" height="361" alt="image" src="https://github.com/user-attachments/assets/4aaae3ac-33f6-47f3-82cf-379dd230ad1e" />




Quand on met 127.0.0.1
<img width="738" height="236" alt="image" src="https://github.com/user-attachments/assets/0c41b5df-7a78-4463-8a60-a98905e5d7b6" />


Si j'ouvre un listener et que je mets mon ip, je suis capable d'avoir une communication.
<img width="695" height="168" alt="image" src="https://github.com/user-attachments/assets/b9ac59cc-fe2c-4389-a949-a8a698715486" />



### command injection

J'ai essayé de rajouter
```
http://ip;ls
```

mais erreur de tentative de hacking.

## subdomain dev.

<img width="752" height="534" alt="image" src="https://github.com/user-attachments/assets/43534bac-9b9d-4fb5-b83c-2786c76f85cd" />


Après avoir rajouté dev dans /etc/hosts, je peux accéder au sous-domaine dev.

## port 80 /dev/.git

Il y a un git, mais mes scans initiaux n'ont pas fonctionné. En utilisant une liste qui contient les extensions.

En utilisant une liste qui detient les extensions.
<img width="754" height="623" alt="image" src="https://github.com/user-attachments/assets/b381b2cc-5c3a-43d5-b72d-562b1971ad85" />


Je telecharge 
<img width="711" height="308" alt="image" src="https://github.com/user-attachments/assets/7d9c2cb3-4dd1-4225-b257-69e82d0fd529" />


Dans le fichier .htaccess,
<img width="679" height="196" alt="image" src="https://github.com/user-attachments/assets/d71639e2-7c63-418e-9edb-b201425ae004" />


On peut modifier les headers avec une extension.
<img width="759" height="141" alt="image" src="https://github.com/user-attachments/assets/580988e7-5120-4cb6-91b1-728cbe448713" />



On a maintenant accès au site.
<img width="753" height="452" alt="image" src="https://github.com/user-attachments/assets/f60598b7-736f-4eb4-83b1-c2c5f4a8d59d" />





## SSRF

Dans index.php, on retrouve la fonctionnalité d'include qui permet d'uploader du contenu php avant qu'il soit exécuté. Dans la partie du code ci-dessous, si on appelle la variable page et qu'elle n'inclut pas des répertoires, on peut avoir du code qui roule.
<img width="639" height="198" alt="image" src="https://github.com/user-attachments/assets/dbf9bc3a-68fb-4b8f-9ef1-4c92ba001372" />


Essayer d'appeler un script php sur mon host avec page, mais ça n'a pas fonctionné.
<img width="756" height="159" alt="image" src="https://github.com/user-attachments/assets/cea4862d-8544-4224-8e6f-21fc1e1263c4" />



### methode phar

Dans la documentation de php, phar permet d'exécuter du code php provenant d'une archive. En gros, on peut bypass les restrictions en utilisant phar qui est du php archive.
<img width="769" height="274" alt="image" src="https://github.com/user-attachments/assets/0323b66e-02b1-4ad4-b227-59aa8085891d" />



Ensuite je vais poster l'archive. Et on obtient du code qui roule quand on fait:
`http://dev.siteisup.htb/?page=phar://uploads/91556569835e1777eae98e498fd5f607/info.MK0/info`
<img width="754" height="304" alt="image" src="https://github.com/user-attachments/assets/66bb988f-3681-4271-a03b-416ccbcda2dc" />



Problème avec mon outil, il faut utiliser dfunc bypass mais ça ne fonctionnait pas. Trouvé le problème. L'outil ne prend pas en compte qu'on doit rajouter le header only4dev.
<img width="737" height="247" alt="image" src="https://github.com/user-attachments/assets/f86c81f6-4545-4be8-879e-5e6d8384123b" />




Après l'avoir rajouté, je trouve que proc_open peujt etre abuse
<img width="755" height="423" alt="image" src="https://github.com/user-attachments/assets/3b78ab07-9693-418a-baac-29621f60cc18" />




## proc_open 


Je trouve ce reverse shell qui abuse de proc_open : https://gist.github.com/noobpk/33e4318c7533f32d6a7ce096bc0457b7#file-reverse-shell-php-L62

Mais j'ai de la difficulté à le faire fonctionner.

Trouvé qu'on peut faire ça et ajouter notre reverse shell.
<img width="750" height="396" alt="image" src="https://github.com/user-attachments/assets/10b9bb2a-f720-420d-a5f6-6989e35a866c" />


Après le re-upload et l'appeler avec phar, on a une connexion.
<img width="767" height="253" alt="image" src="https://github.com/user-attachments/assets/7cba1c73-01f6-4346-90de-377db2235e4d" />



## dev


Dans le répertoire du développeur, on retrouve dev qui peut être lu par le groupe owner www-data (nous) et on y retrouve 2 scripts dont 1 qui a le setuid.
<img width="763" height="396" alt="image" src="https://github.com/user-attachments/assets/1d4c6731-4f98-4a0b-8bb7-7e9bebfa0b20" />


Dans le script siteisup_test.py

<img width="737" height="197" alt="image" src="https://github.com/user-attachments/assets/107a90ee-9350-42a5-add4-552a28cdbf08" />




En python2, input() est techniquement eval(raw_input(prompt)) et eval permet donc de run du code. Donc dans le input, j'intègre la commande id.
<img width="741" height="385" alt="image" src="https://github.com/user-attachments/assets/2eaa69c9-a3a5-4088-a0c9-eca974cab662" />


J'ai vu qu'il y avait un répertoire .ssh. Je réussis à lire la clé.
<img width="755" height="705" alt="image" src="https://github.com/user-attachments/assets/13f4ba82-28b3-4a33-8c9e-7953fc5f981a" />


## developer

Je me suis connecté avec ssh.
<img width="759" height="644" alt="image" src="https://github.com/user-attachments/assets/001fc77f-f6d2-4178-b2d9-5855f1fccdd0" />

Et on retrouve le flag sur le /home de l'usager.


# PrivEsc


## Unix executable

<img width="746" height="93" alt="image" src="https://github.com/user-attachments/assets/c56e71cd-56e2-4c56-aa8e-0f99af405210" />


Je retrouve la privilege escalation sur gtfo donc on fait ça. Les privilèges de sudo ne sont pas perdus.  On peut mettre une commande pour avoir un terminal dans un fichier setup.py. Celui-ci va être exécuté lorsqu'on roule easy_install.

<img width="760" height="218" alt="image" src="https://github.com/user-attachments/assets/7d2fbcab-b102-4843-8318-04e98264b089" />

