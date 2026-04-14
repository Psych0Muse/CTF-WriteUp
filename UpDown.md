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


![[Pasted image 20260413101923.png]]

J'ajoute le siteisup.htb dans mon /etc/hosts. 

Quand je mets le site google, j'obtiens que le serveur est down. Même en mode debug, ça ne fonctionne pas.

![[Pasted image 20260413102424.png]]




Quand on met 127.0.0.1
![[Pasted image 20260413102654.png]]


Si j'ouvre un listener et que je mets mon ip, je suis capable d'avoir une communication.
![[Pasted image 20260413102910.png]]


### command injection

J'ai essayé de rajouter
```
http://ip;ls
```

mais erreur de tentative de hacking.

## subdomain dev.

![[Pasted image 20260413104354.png]]


Après avoir rajouté dev dans /etc/hosts, je peux accéder au sous-domaine dev.

## port 80 /dev/.git

Il y a un git, mais mes scans initiaux n'ont pas fonctionné. En utilisant une liste qui contient les extensions.

En utilisant une liste qui detient les extensions.
![[Pasted image 20260413111958.png]]


Je telecharge 
![[Pasted image 20260413112114.png]]

Dans le fichier .htaccess,
![[Pasted image 20260413115149.png]]


On peut modifier les headers avec une extension.
![[Pasted image 20260413115601.png]]


On a maintenant accès au site.
![[Pasted image 20260413120107.png]]




## SSRF

Dans index.php, on retrouve la fonctionnalité d'include qui permet d'uploader du contenu php avant qu'il soit exécuté. Dans la partie du code ci-dessous, si on appelle la variable page et qu'elle n'inclut pas des répertoires, on peut avoir du code qui roule.
![[Pasted image 20260413142507.png]]

Essayer d'appeler un script php sur mon host avec page, mais ça n'a pas fonctionné.
![[Pasted image 20260413145005.png]]


### methode phar

Dans la documentation de php, phar permet d'exécuter du code php provenant d'une archive. En gros, on peut bypass les restrictions en utilisant phar qui est du php archive.
![[Pasted image 20260413154011.png]]


Ensuite je vais poster l'archive. Et on obtient du code qui roule quand on fait:
`http://dev.siteisup.htb/?page=phar://uploads/91556569835e1777eae98e498fd5f607/info.MK0/info`
![[Pasted image 20260413154659.png]]


Problème avec mon outil, il faut utiliser dfunc bypass mais ça ne fonctionnait pas. Trouvé le problème. L'outil ne prend pas en compte qu'on doit rajouter le header only4dev.
![[Pasted image 20260414131708.png]]



Après l'avoir rajouté, je trouve que proc_open peujt etre abuse
![[Pasted image 20260414115529.png]]



## proc_open 


Je trouve ce reverse shell qui abuse de proc_open : https://gist.github.com/noobpk/33e4318c7533f32d6a7ce096bc0457b7#file-reverse-shell-php-L62

Mais j'ai de la difficulté à le faire fonctionner.

Trouvé qu'on peut faire ça et ajouter notre reverse shell.
![[Pasted image 20260414122603.png]]

Après le re-upload et l'appeler avec phar, on a une connexion.
![[Pasted image 20260414122643.png]]



## dev


Dans le répertoire du développeur, on retrouve dev qui peut être lu par le groupe owner www-data (nous) et on y retrouve 2 scripts dont 1 qui a le setuid.
![[Pasted image 20260414124122.png]]



Dans le script siteisup_test.py

![[Pasted image 20260414124322.png]]



En python2, input() est techniquement eval(raw_input(prompt)) et eval permet donc de run du code. Donc dans le input, j'intègre la commande id.
![[Pasted image 20260414124601.png]]


J'ai vu qu'il y avait un répertoire .ssh. Je réussis à lire la clé.
![[Pasted image 20260414125838.png]]


## developer

Je me suis connecté avec ssh.
![[Pasted image 20260414130020.png]]

Et on retrouve le flag sur le /home de l'usager.


# PrivEsc


## Unix executable

![[Pasted image 20260414130311.png]]


Je retrouve la privilege escalation sur gtfo donc on fait ça. Les privilèges de sudo ne sont pas perdus.  On peut mettre une commande pour avoir un terminal dans un fichier setup.py. Celui-ci va être exécuté lorsqu'on roule easy_install.

![[Pasted image 20260414130513.png]]
