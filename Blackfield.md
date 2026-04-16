---
date: 2026-04-16
tags:
  - htb
  - hard
  - AD
  - asrep-roast
  - shadow_credential
  - sebackupprivilege
---


# Recon

tcp scan
<img width="943" height="714" alt="image" src="https://github.com/user-attachments/assets/0d0e9f25-336a-4726-9708-dce5475dd811" />



udp scan
<img width="943" height="714" alt="image" src="https://github.com/user-attachments/assets/a4ef07c4-2b50-443d-a76f-fc76eae8695c" />



# Exploit


## port 135 rpc

Je tente une connexion avec un null session (usager vide), mais je ne peux rien interroger.

<img width="943" height="714" alt="image" src="https://github.com/user-attachments/assets/06fee88f-7a8d-4a60-af96-4cd73a3c7770" />



## port 389 ldap

Il en est de même pour ldap.

<img width="943" height="714" alt="image" src="https://github.com/user-attachments/assets/41341314-eec6-41c2-8d2b-4419cbec1ac9" />



## port 88 kerberos

Il n'y a pas grand-chose à faire avec les derniers ports ouverts. Je tente donc, comme dernier recours, de brute-forcer une liste d'utilisateurs. Je parviens à récupérer quelques noms.

<img width="943" height="714" alt="image" src="https://github.com/user-attachments/assets/2bc0ccf3-8aaa-458f-a313-27143eb1b542" />


Jai trouvé comme usager : support, guest et administrator



### AS-REP Roasting

Puisque j'ai une liste d'utilisateurs, je peux vérifier si le KDC est mal configuré. Cela permet d'obtenir un AS-REP sans mot de passe et d'en extraire le hash pour tenter de le cracker. J'utilise la commande:

`impacket-GetNPUsers BLACKFIELD.local/ -dc-ip 10.129.229.17 -no-pass -usersfile users.txt`

<img width="943" height="714" alt="image" src="https://github.com/user-attachments/assets/dea02465-7b57-43f8-b97e-d3e38879246d" />


On est capable de le crack avec hashcat et le mdp est `#00^BlackKnight`

## retour sur rpc

De retour sur le port RPC, on peut maintenant interroger le domaine avec les **credentials** que l'on vient d'obtenir.

<img width="943" height="714" alt="image" src="https://github.com/user-attachments/assets/a0add4ae-afe1-4fb4-a0ea-d38483e777eb" />


J'obtiens quelques autres usagers

<img width="943" height="714" alt="image" src="https://github.com/user-attachments/assets/f8f6856b-547a-4cac-9f6c-5bfa99f69f78" />



## port 445 smb

Je regarde les shares accessibles via SMB avec ces identifiants, mais on n'y trouve rien d'exploitable.

<img width="943" height="714" alt="image" src="https://github.com/user-attachments/assets/06bd1fbc-b146-4f60-8cf5-2d67d2ed6340" />




<img width="943" height="714" alt="image" src="https://github.com/user-attachments/assets/f4b2bf5f-a69d-443a-9b1d-fbef01f2176a" />





## bloodhound

J'utilise BloodHound pour déterminer s'il existe des chemins d'attaque (attack paths) possibles.

<img width="943" height="714" alt="image" src="https://github.com/user-attachments/assets/b9a77ffe-d54a-48ea-9c8a-15b2e29d9267" />





### forcechangepassword


On voit que l'usager que je contrôle détient le privilège ForceChangePassword sur un autre compte.

<img width="943" height="714" alt="image" src="https://github.com/user-attachments/assets/86ad52e1-d16d-4106-8fcd-e87d0e97b14d" />


J'essaie la méthode montrée dans BloodHound, mais elle ne fonctionne pas directement.

<img width="943" height="714" alt="image" src="https://github.com/user-attachments/assets/113518a9-d50f-4fd8-a2b9-80aee6bc2d91" />



Par contre, on peut le faire avec une connection rpc. 

<img width="943" height="714" alt="image" src="https://github.com/user-attachments/assets/4f628b63-a158-4414-b39e-62ea236a2c34" />


Même si la sortie n'indique pas explicitement que cela a réussi, le nouveau mot de passe fonctionne bel et bien.

<img width="943" height="714" alt="image" src="https://github.com/user-attachments/assets/99f0fc0f-31d1-44f4-bb46-8ca85015bb59" />



## smb avec audit 2020

nsuite, je regarde si l'usager **audit2020** a des accès spéciaux. Il possède les droits de lecture sur le partage forensic.
`smbmap -H 10.129.229.17  -u audit2020 -p 'newP@ssword2022'`

<img width="943" height="714" alt="image" src="https://github.com/user-attachments/assets/7779edff-3733-4293-89b1-05139f4386da" />

On y retrouve des choses interessantes que je telecharge.

<img width="943" height="714" alt="image" src="https://github.com/user-attachments/assets/c7d4f107-fe5a-4929-9acb-80ba0917f6ec" />



Je récupère un dump de LSASS (Local Security Authority Subsystem Service). Ce processus contient les hashs et parfois les mots de passe des usagers connectés. J'extrais le zip pour obtenir le .DMP. 

<img width="943" height="714" alt="image" src="https://github.com/user-attachments/assets/a3285426-9f7d-4d10-b9fd-aa1cd2d939ab" />


### pypykatz

Pour lire le fichier .DMP, je dois installer pypykatz.

<img width="943" height="714" alt="image" src="https://github.com/user-attachments/assets/54d8e428-1e4c-4943-8023-352b1ffd9ba0" />


'exécute l'application et j'y retrouve les hashs pour svc_backup et administrator.

<img width="943" height="714" alt="image" src="https://github.com/user-attachments/assets/ef0a4a53-b9a0-4e5b-b7eb-427f09191dea" />




```
svc_backup / 9658d1d1dcd9250115e2205d9f48400d
administrator / 7f1e4ff8c6a8e6b6fcae2d9c0572cd62
```


### hash spray

Je tente un pass-the-hash (ou password spray avec les hashs) sur le domaine. Seul celui de svc_backup marche.

<img width="943" height="714" alt="image" src="https://github.com/user-attachments/assets/5f159c92-7b5c-4cfe-88f9-c088fecc0317" />


<img width="943" height="714" alt="image" src="https://github.com/user-attachments/assets/34f7728d-4f4e-4ba5-ad63-1a8966e1824a" />


<img width="943" height="714" alt="image" src="https://github.com/user-attachments/assets/cfce9842-e29a-4887-9066-e3f9aaa2f0f5" />


Je me connecte avec cette session et je récupère le flag.

<img width="943" height="714" alt="image" src="https://github.com/user-attachments/assets/32e82df2-2c83-439c-8fc1-d3ea9c02ca51" />




# PrivEsc

On voit que l'usager détient le privilège SeBackupPrivilege. Cela permet de lire n'importe quel fichier, même s'il est verrouillé par le système, en créant une Volume Shadow Copy.

<img width="943" height="714" alt="image" src="https://github.com/user-attachments/assets/57452fbc-a843-4890-9749-d918b3c4bcbf" />



## seBackupPrivilege avec shadow copy


Je me base sur ca : https://www.hackingarticles.in/windows-privilege-escalation-sebackupprivilege/

Je crée un fichier de script `shadow.txt` pour l'outil `diskshadow`.

<img width="943" height="714" alt="image" src="https://github.com/user-attachments/assets/33de0973-e3c4-4f72-805d-0661d46bd405" />


Il faut le convertir a un format DOS.

<img width="943" height="714" alt="image" src="https://github.com/user-attachments/assets/d9e88b99-bb61-44d2-a41c-bedcad26ceb2" />


Je transfère le fichier sur la cible et j'exécute diskshadow.

<img width="943" height="714" alt="image" src="https://github.com/user-attachments/assets/f49e3ff5-a439-478d-b3b6-c3f652cfc065" />


<img width="943" height="714" alt="image" src="https://github.com/user-attachments/assets/647641b6-c8c3-42a4-a33e-06828ee24134" />

Cela crée une copie du fichier ntds.dit, qui contient tous les hashs des comptes du domaine.

<img width="943" height="714" alt="image" src="https://github.com/user-attachments/assets/08f578dc-d39b-4176-b344-8f95c9a05689" />


J'ai également besoin du fichier SYSTEM (ruche du registre) qui contient la clé de démarrage permettant de décrypter le contenu de `ntds.dit`.

<img width="943" height="714" alt="image" src="https://github.com/user-attachments/assets/756e7cad-b74f-49da-ae6c-f31770cb25bf" />



Je telecharge les 2 fichiers.

<img width="782" height="230" alt="image" src="https://github.com/user-attachments/assets/e07af5f7-b9da-4c60-ad08-3c571c51a5e8" />


Ensuite, j'utilise secretsdump pour extraire les hashs localement

<img width="782" height="230" alt="image" src="https://github.com/user-attachments/assets/1e46981e-1d44-4823-9f48-2a8b40bb37fe" />

J'obtiens ainsi le hash de l'administrateur, ce qui me permet de récupérer le dernier flag.

<img width="782" height="230" alt="image" src="https://github.com/user-attachments/assets/8d777824-914c-4d25-9385-d4574f4c7cc4" />



