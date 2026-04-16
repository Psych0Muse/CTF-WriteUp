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

<img width="711" height="261" alt="image" src="https://github.com/user-attachments/assets/53a6ee91-ddae-4fd6-aba3-b6f27071ab14" />




# Exploit


## port 135 rpc

Je tente une connexion avec un null session (usager vide), mais je ne peux rien interroger.

<img width="426" height="166" alt="image" src="https://github.com/user-attachments/assets/e9f14009-eae4-446a-bbf4-f110b89dfec5" />




## port 389 ldap

Il en est de même pour ldap.

<img width="880" height="359" alt="image" src="https://github.com/user-attachments/assets/2c5fd783-e51d-435a-a448-a26bacf15721" />



## port 88 kerberos

Il n'y a pas grand-chose à faire avec les derniers ports ouverts. Je tente donc, comme dernier recours, de brute-forcer une liste d'utilisateurs. Je parviens à récupérer quelques noms.

<img width="942" height="367" alt="image" src="https://github.com/user-attachments/assets/f92cbd96-eb79-4684-b491-061479902879" />



Jai trouvé comme usager : support, guest et administrator



### AS-REP Roasting

Puisque j'ai une liste d'utilisateurs, je peux vérifier si le KDC est mal configuré. Cela permet d'obtenir un AS-REP sans mot de passe et d'en extraire le hash pour tenter de le cracker. J'utilise la commande:

`impacket-GetNPUsers BLACKFIELD.local/ -dc-ip 10.129.229.17 -no-pass -usersfile users.txt`

<img width="953" height="349" alt="image" src="https://github.com/user-attachments/assets/933b6e98-9942-486e-abfe-774e15aed8cf" />


On est capable de le crack avec hashcat et le mdp est `#00^BlackKnight`

## retour sur rpc

De retour sur le port RPC, on peut maintenant interroger le domaine avec les **credentials** que l'on vient d'obtenir.

<img width="568" height="310" alt="image" src="https://github.com/user-attachments/assets/6c85b140-1876-4373-ab2f-12a1741dce84" />



J'obtiens quelques autres usagers

<img width="495" height="184" alt="image" src="https://github.com/user-attachments/assets/492c1ca2-ec60-4e2e-86a7-6583e03a29f8" />




## port 445 smb

Je regarde les shares accessibles via SMB avec ces identifiants, mais on n'y trouve rien d'exploitable.

<img width="912" height="474" alt="image" src="https://github.com/user-attachments/assets/98e59739-485f-401f-ab94-75365d2c886f" />




<img width="798" height="679" alt="image" src="https://github.com/user-attachments/assets/3f2e51f8-a637-4518-89f8-6bcf83b1e89d" />






## bloodhound

J'utilise BloodHound pour déterminer s'il existe des chemins d'attaque (attack paths) possibles.

<img width="1012" height="275" alt="image" src="https://github.com/user-attachments/assets/38fe73e6-e9ab-48c9-a28e-70e79dc639b3" />






### forcechangepassword


On voit que l'usager que je contrôle détient le privilège ForceChangePassword sur un autre compte.

<img width="1025" height="280" alt="image" src="https://github.com/user-attachments/assets/0a88158a-2d88-4f9a-84ba-dbb1ab159c34" />



J'essaie la méthode montrée dans BloodHound, mais elle ne fonctionne pas directement.

<img width="904" height="180" alt="image" src="https://github.com/user-attachments/assets/59312979-2e16-4e11-9bba-82462c19a386" />



Par contre, on peut le faire avec une connection rpc. 

<img width="636" height="128" alt="image" src="https://github.com/user-attachments/assets/4bc5c4ef-5813-451f-913d-beb409f0b56b" />


Même si la sortie n'indique pas explicitement que cela a réussi, le nouveau mot de passe fonctionne bel et bien.

<img width="890" height="463" alt="image" src="https://github.com/user-attachments/assets/d018e19f-6b72-423b-aae8-5e9c7bbc9534" />


## smb avec audit 2020

nsuite, je regarde si l'usager **audit2020** a des accès spéciaux. Il possède les droits de lecture sur le partage forensic.
`smbmap -H 10.129.229.17  -u audit2020 -p 'newP@ssword2022'`

<img width="897" height="441" alt="image" src="https://github.com/user-attachments/assets/3c687773-1454-4d5c-b393-4bef624bccde" />


On y retrouve des choses interessantes que je telecharge.

<img width="794" height="247" alt="image" src="https://github.com/user-attachments/assets/1cf0421a-afb6-47dd-8148-16926500ef70" />



Je récupère un dump de LSASS (Local Security Authority Subsystem Service). Ce processus contient les hashs et parfois les mots de passe des usagers connectés. J'extrais le zip pour obtenir le .DMP. 

<img width="788" height="158" alt="image" src="https://github.com/user-attachments/assets/72ca4366-9955-4740-b768-ae3e61394111" />



### pypykatz

Pour lire le fichier .DMP, je dois installer pypykatz.

<img width="799" height="243" alt="image" src="https://github.com/user-attachments/assets/1de93027-4735-4097-8b5d-87ba1a65d6e5" />



J'exécute l'application et j'y retrouve les hashs pour svc_backup et administrator.

<img width="866" height="552" alt="image" src="https://github.com/user-attachments/assets/e3787e60-34cb-49ac-bbc6-b7ff5b07e92f" />





```
svc_backup / 9658d1d1dcd9250115e2205d9f48400d
administrator / 7f1e4ff8c6a8e6b6fcae2d9c0572cd62
```


### hash spray

Je tente un pass-the-hash (ou password spray avec les hashs) sur le domaine. Seul celui de svc_backup marche.

<img width="1015" height="337" alt="image" src="https://github.com/user-attachments/assets/8c1d0256-ceb6-4ecb-8370-27cb73ca93e9" />



<img width="1018" height="104" alt="image" src="https://github.com/user-attachments/assets/c3e208a6-906e-41bb-a102-82348cee6dac" />



<img width="946" height="260" alt="image" src="https://github.com/user-attachments/assets/1e99b0fa-29da-45ad-8945-fa95d2a15064" />



Je me connecte avec cette session et je récupère le flag.

<img width="545" height="274" alt="image" src="https://github.com/user-attachments/assets/d12d05cd-cd58-42ef-b6a3-4791efbd6090" />





# PrivEsc

On voit que l'usager détient le privilège SeBackupPrivilege. Cela permet de lire n'importe quel fichier, même s'il est verrouillé par le système, en créant une Volume Shadow Copy.

<img width="740" height="313" alt="image" src="https://github.com/user-attachments/assets/3a01fe16-d52f-47de-8f40-bf1f2bc76d76" />



## seBackupPrivilege avec shadow copy


Je me base sur ca : https://www.hackingarticles.in/windows-privilege-escalation-sebackupprivilege/

Je crée un fichier de script `shadow.txt` pour l'outil `diskshadow`.

<img width="420" height="245" alt="image" src="https://github.com/user-attachments/assets/66c97f69-13e5-40e2-99f6-722271794681" />



Il faut le convertir a un format DOS.

<img width="566" height="99" alt="image" src="https://github.com/user-attachments/assets/d3102391-dd46-46ed-b8a4-1924240ac94e" />



Je transfère le fichier sur la cible et j'exécute diskshadow.

<img width="979" height="358" alt="image" src="https://github.com/user-attachments/assets/93df39a7-3ac7-4827-bd76-9e64f6833068" />


<img width="665" height="166" alt="image" src="https://github.com/user-attachments/assets/584ac073-a741-47c5-9379-f3edc6349ca2" />


Cela crée une copie du fichier ntds.dit, qui contient tous les hashs des comptes du domaine.

<img width="1079" height="617" alt="image" src="https://github.com/user-attachments/assets/d38d41f2-8c98-4a1b-a18a-3356f5bfb5ae" />


J'ai également besoin du fichier SYSTEM (ruche du registre) qui contient la clé de démarrage permettant de décrypter le contenu de `ntds.dit`.

<img width="1096" height="114" alt="image" src="https://github.com/user-attachments/assets/9f8397ba-596a-4dd0-a4e0-7dc918426b04" />




Je telecharge les 2 fichiers.

<img width="782" height="230" alt="image" src="https://github.com/user-attachments/assets/a0f33d60-5ed5-478e-8cd4-f764bfc37227" />


Ensuite, j'utilise secretsdump pour extraire les hashs localement

<img width="1007" height="312" alt="image" src="https://github.com/user-attachments/assets/ef6b241c-2ccc-4fee-9c6f-d14925affc1e" />


J'obtiens ainsi le hash de l'administrateur, ce qui me permet de récupérer le dernier flag.

<img width="1043" height="547" alt="image" src="https://github.com/user-attachments/assets/aba18c54-2519-48e5-b4bb-f1df766f1362" />




