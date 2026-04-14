---
date: 2026-04-14
tags:
  - htb
  - medium
  - mssql
  - adcs
  - esc7
---

# Recon

![[Pasted image 20260414171927.png]]
![[Pasted image 20260414171942.png]]

port udp
![[Pasted image 20260414171954.png]]



# Exploit

## port 135

pas de connexion possible
![[Pasted image 20260414172012.png]]

## port 139

A l aide de smbmap
![[Pasted image 20260414172015.png]]
rien de trouver dans les shares accessible


### liste d'usager

J'obtiens la liste d'usager : `nxc smb manager.htb -u 'anonymous' -p '' --rid-brute`

![[Pasted image 20260414172027.png]]
Je prend en note les usagers qui me semble pertinent.


## AS-REP Roasting 

J'entame du AS-REP roasting avec :
`impacket-GetNPUsers  manager.htb/ -dc-ip 10.129.20.162 -no-pass -usersfile user.txt`

![[Pasted image 20260414172035.png]]
aucun des usagers dans ma liste ont le UF_DONT_REQUIRE_PREAUTH.



## port 389 ldap

pas d'information de trouver

![[Pasted image 20260414172042.png]]


## port 80

Sur le port 80, on retrouve un site web

![[Pasted image 20260414172052.png]]

rien de trouver


## password spray

Il fallait password spray les nom avec lui meme et s assurer qu on met les nom d usager en caratere petit

`nxc smb manager.htb -u user.txt -p user.txt --continue-on-success`

ca nous permet de trouve l'usager operator

![[Pasted image 20260414172102.png]]

## bloodhound 

Je roule l'application bloodhound
`bloodhound-python -u 'Operator' -p 'operator' -d manager.htb -c All -ns 10.129.20.162`


![[Pasted image 20260414172109.png]]

rien de trouver



## mssql

Je peux me connecter a mssql

![[Pasted image 20260414172118.png]]

je peux aller voir le C: et je vais dans le root du webserver (inetpub\wwwroot)

![[Pasted image 20260414172132.png]]

je trouve web.config et un zip. Web.config ne me donne rien mais je peux telecharger le zip.

dans le zip, je retrouve un vieux fichier qui comporte les credentials de raven.
![[Pasted image 20260414172139.png]]

raven / R4v3nBe5tD3veloP3r!123


on voit que ces credentials marche

![[Pasted image 20260414172147.png]]


raven est un membre de remote management, donc on peut s'y connecter

![[Pasted image 20260414172152.png]]


on se connecte avec evil-winrm et on trouve le flag

![[Pasted image 20260414172157.png]]

# PrivEsc


## ADCS

Je regarde les certificats avec
`certipy-ad find -ns 10.129.20.162  -u 'raven@manager.htb' -p 'R4v3nBe5tD3veloP3r!123' -stdout -vulnerable`


on trouve le ESC7 est exploitable. ESC7 est une vulnerabilite de faible controle d acces provenant du ADCS.  On peut se donner des droit du ManageCA et eleve notre privilege

![[Pasted image 20260414172810.png]]

On ajoute notre usager au managerCA.
![[Pasted image 20260414172817.png]]


On request le SubCa
![[Pasted image 20260414172822.png]]


On request le id qu'on a recu
![[Pasted image 20260414172834.png]]

on request le certificat
![[Pasted image 20260414172857.png]]


On update le temps pour etre en concordance avec le domaine
![[Pasted image 20260414172902.png]]

On se connecte
![[Pasted image 20260414172906.png]]


on se connecte a l usager administrator et on obtient le flag
![[Pasted image 20260414172911.png]]
