## Adrien GALAN

# TP 7 - Boot, services et processus / Tâches d’administration (2)

## Exercice 1. Personnalisation de GRUB

**1. Commencez par changer l’extension du fichier /etc/default/grub.d/50-curtin-settings.cfg s’il
est présent dans votre environnement (vous pouvez aussi commenter son contenu).**


**2. Modifiez le fichier /etc/default/grub pour que le menu de GRUB s’affiche pendant 10 secondes ;
passé ce délai, le premier OS du menu doit être lancé automatiquement.**

On ajoute au fichier /etc/default/grub les lignes de commande suivantes :
```
GRUB_TIMEOUT=10
GRUB_TIMEOUT_STYLE=menu
```

**3. Lancez la commande update-grub
 Cette commande fait appel au script grub-mkconfig qui construit le fichier de configuration
”final” de GRUB (/boot/grub/grub.cfg) à partir du fichier de paramètres et des scripts.**

On exécute la commande..

**4. Redémarrez votre VM pour valider que les changements ont bien été pris en compte
 Pensez à lancer la commande update-grub après chaque modification de la configuration de
GRUB !**

Les modifications ont bien eu l'effet escompté.

**5. On va augmenter la résolution de GRUB et de notre VM. Cherchez sur Internet le ou les paramètres
à rajouter au fichier grub.**

On ajoute ces lignes au fichier /etc/default/grub
```
GRUB_GFXMODE=1280x1024x32 
GRUB_GFXPAYLOAD_LINUX=keep
```

**6. On va à présent ajouter un fond d’écran. Il existe un paquet en proposant quelques uns : grub2-splash-images
(après installation, celles-ci sont disponibles dans /usr/share/images/grub).**

```console
adrien@server:~$ sudo apt-get install grub2-splash-images
Lecture des listes de paquets... Fait
Construction de l'arbre des dépendances
Lecture des informations d'état... Fait
E: Impossible de trouver le paquet grub2-splash-images
```
Je n'ai pas réussi à installer le paquet..
Si j'avais réussi, la commande pour ajouter le fond d'écran à grub aurait été :

```GRUB_BACKGROUND="chemin"```

**7. Il est également possible de configurer des thèmes. On en trouve quelques uns dans les dépôts (grub2-themes-*).
Installez-en un.**

Pour installer un thème, il suffit de télécharger le thème sur internet puis de mettre le paramètre :

```
GRUB_THEME="/boot/grub/themes/ubuntu-mate/theme.txt"
```


**8. Ajoutez une entrée permettant d’arrêter la machine, et une autre permettant de la redémarrer.

On modifie le fichier /etc/grub.d/40_custom et on rajoute :
```
menuentry ’Arrêt du système’ {
halt
}
menuentry ’Redémarrage du système’ {
reboot
}
```

**9. Configurer GRUB pour que le clavier soit en français

On commence par créer un dossier pour accueillir la configuration en français

sudo mkdir /boot/grub/layouts

On y génère la configuration en français

sudo grub-kbdcomp -o /boot/grub/layouts/fr.gkb fr

Dans /etc/default/grub : rajouter GRUB_TERMINAL_INPUT=at_keyboard

Puis on ajoute la configuration au grub

On ajoute les lignes suivantes au fuchier /etc/grub.d/40_custom :
# Clavier fr
insmod keylayouts
keymap fr


## Exercice 2. Noyau
**Dans cet exercice, on va créer et installer un module pour le noyau.**


**1. Commencez par installer le paquet build-essential, qui contient tous les outils nécessaires (compilateurs, bibliothèques) à la compilation de programmes en C (entre autres).**

```
sudo apt-get install build-essential
```

**2. Créez un fichier hello.c contenant le code suivant :**

```bash
1 #include <linux/module.h>
2 #include <linux/kernel.h>
3
4 MODULE_LICENSE("GPL");
5 MODULE_AUTHOR("John Doe");
6 MODULE_DESCRIPTION("Module hello world");
7 MODULE_VERSION("Version 1.00");
8
9 int init_module(void)
10 {
11 printk(KERN_INFO "[Hello world] - La fonction init_module() est appelée.\n");
12 return 0;
13 }
14
15 void cleanup_module(void)
16 {
17 printk(KERN_INFO "[Hello world] - La fonction cleanup_module() est appelée.\n");
18 }
```

3. Créez également un fichier Makefile :

```
1 obj-m += hello.o
2
3 all:
4 make -C /lib/modules/$(shell uname -r)/build M=$(PWD) modules
5
6 clean:
7 make -C /lib/modules/$(shell uname -r)/build M=$(PWD) clean
8
9 install:
10 cp ./hello.ko /lib/modules/$(shell uname -r)/kernel/drivers/misc
```
 Les lignes 4, 7 et 10 doivent commencer par une tabulation.


**4. Compilez le module à l’aide de la commande make, puis installez-le à l’aide de la commande make**
**install.**
** Le module est installé dans le dossier spécifié à la ligne 10.**

```console
adrien@server:~$ adrien@server:~$ make
make -C /lib/modules/5.0.0-31-generic/build M=/home/adrien modules
make[1]: Entering directory '/usr/src/linux-headers-5.0.0-31-generic'
  CC [M]  /home/adrien/hello.o
  Building modules, stage 2.
  MODPOST 1 modules
  CC      /home/adrien/hello.mod.o
  LD [M]  /home/adrien/hello.ko
make[1]: Leaving directory '/usr/src/linux-headers-5.0.0-31-generic'
adrien@server:~$ sudo make install
cp ./hello.ko /lib/modules/5.0.0-31-generic/kernel/drivers/misc
```

**5. Chargez le module ; vérifiez dans le journal du noyau que le message ”La fonction init_module() est
appelée” a bien été inscrit, synonyme que le module a été chargé ; confirmez avec la commande lsmod.**

```sudo insmod /lib/modules/5.0.0-31-generic/kernel/drivers/misc/hello.ko```

```console
adrien@server:~$ tail -n 1 /var/log/kern.log
Oct 21 08:34:17 server kernel: [ 4439.178000] [Hello world] - La fonction init_module() est appelée.
```

```console
adrien@server:~$ lsmod | grep "hello"
hello                  16384  0
```

**6. Utilisez la commande modinfo pour obtenir des informations sur le module hello.ko ; vous devriez**
**notamment voir les informations figurant dans le fichier C.**

```console
adrien@server:~$ modinfo hello.ko
filename:       /home/adrien/hello.ko
version:        Version 1.00
description:    Module hello world
author:         John Doe
license:        GPL
srcversion:     4398A2271F215E3A6F58078
depends:
retpoline:      Y
name:           hello
vermagic:       5.0.0-31-generic SMP mod_unload
```


**7. Déchargez le module ; vérifiez dans le journal du noyau que le message ”La fonction cleanup_module()**
**est appelée” a bien été inscrit, synonyme que le module a été déchargé ; confirmez avec la commande**
**lsmod.**

```
adrien@server:~$ sudo rmmod hello.ko
adrien@server:~$ lsmod | grep hello
adrien@server:~$ tail -n 1 /var/log/kern.log
Oct 21 08:43:55 server kernel: [ 5017.585900] [Hello world] - La fonction cleanup_module() est appelée.
```

**8. Pour que le module soit chargé automatiquement au démarrage du système, il faut l’inscrire dans le**
**fichier /etc/modules. Essayez, et vérifiez avec la commande lsmod après redémarrage de la machine.**

J'ai essayé de lui ajouter le nom du module seul et aussi le chemin complet et aucun des deux ne fonctionne.
Je ne sais pas vraiment quoi tester de plus..

## Exercice 3. Interception de signaux
**La commande interne trap permet de redéfinir des gestionnaires pour les signaux reçus par un processus.**
**Un cas d’utilisation typique est la suppression des fichiers temporaires créés par un script lorsque celui-ci est**
**interrompu.**

**1. Commencez par écrire un script qui recopie dans un fichier tmp.txt chaque ligne saisie au clavier par**
**l’utilisateur**

```
#!/bin/bash
while :; do
read texte
echo $texte >> tmp.txt
done
```


**2. Lancez votre script et appuyez sur CTRL+Z. Que se passe-t-il ? Comment faire pour que le script poursuive son exécution ?**

```
adrien@server:~$ ./monscript.sh
^Z
[2]+  Stopped                 ./monscript.sh
```
Le script est mis en pause.
Pour reprendre l'exécution, on utilise la commande fg X (x étant ici 2).
On peut obtenir le numéro du processus avec la commande ``` jobs ```.


**3. Toujours pendant l’exécution du script, appuyez sur CTRL+C. Que se passe-t-il ?**

Cette fois-ci le script est complètement arrêté.
ctrl+c est le raccourci pour le signal signint (à savoir le signal d'interruption).


**4. Modifiez votre script pour redéfinir les actions à effectuer quand le script reçoit les signaux SIGTSTP**
**(= CTRL+Z) et SIGINT (= CTRL+C) : dans le premier cas, il doit afficher ”Impossible de me placer en**
**arrière-plan”, et dans le second cas, il doit afficher ”OK, je fais un peu de ménage avant” avant de**
**supprimer le fichier temporaire et terminer le script.**

On ajoute les lignes suivantes au début du script :
```
trap 'echo impossible de me placer en arrière-plan !' TSTP
trap 'echo OK, je fais juste un peu de ménage avant ; exit' INT
trap 'rm tmp.txt' EXIT
```


**5. Testez le nouveau comportement de votre script en utilisant d’une part les raccourcis clavier, d’autre**
**part la commande kill**

```console
adrien@server:~$ ./monscript.sh
^COK, je fais juste un peu de ménage avant
adrien@server:~$ ./monscript.sh
^Zimpossible de me placer en arrière-plan !
^X^Zimpossible de me placer en arrière-plan !
g
g
g
g^COK, je fais juste un peu de ménage avant
```

**6. Relancez votre script et faites immédiatement un CTRL+C : vous obtenez un message d’erreur vous**
**indiquant que le fichier tmp.txt n’existe pas. A l’aide de la commande interne test, corrigez votre**
**script pour que ce message n’apparaisse plus.**

```console
adrien@server:~$ ./monscript.sh
^COK, je fais juste un peu de ménage avant
rm: cannot remove 'tmp.txt': No such file or directory
```

On modifie le trap du signal exit :
``` trap 'if [ -e tmp.txt ]; then rm tmp.txt; fi' EXIT ```
De cette manière, on vérifie que le fichier existe avant de le supprimer.

Résultat :

```console
adrien@server:~$ ./monscript.sh
^COK, je fais juste un peu de ménage avant
```




## Exercice 4. Surveillance de l’activité du système

**1. Dans tty1, lancez la commande htop, puis tapez la commande w dans tty2. Qu’affiche cette commande ?**

Cette commande permet d'afficher les utilisateurs connectés et ce qu'ils sont en train de faire.

**2. Comment afficher l’historique des dernières connexions à la machine ?**

Pour afficher l'historique des dernières connexions à la machine, on utilise la commande ```last```


**3. Quelle commande permet d’obtenir la version du noyau ?**

```uname -r``` permet d'obtenir la version du noyau.
```console
adrien@server:~$ uname -r
5.0.0-31-generic
```


**4. Comment récupérer toutes les informations sur le processeur, au format JSON ?**

```console
adrien@server:~$ lshw -class Processor -json >> infoProc.json
WARNING: you should run this program as super-user.
WARNING: output may be incomplete or inaccurate, you should run this program as super-user.
adrien@server:~$ cat infoProc.json
                {
    "id" : "cpu",
    "class" : "processor",
    "claimed" : true,
    "product" : "Intel(R) Core(TM) i5-8265U CPU @ 1.60GHz",
    "vendor" : "Intel Corp.",
    "physid" : "1",
    "businfo" : "cpu@0",
    "width" : 64,
    "capabilities" : {
      "fpu" : "mathematical co-processor",
      "fpu_exception" : "FPU exceptions reporting",
      "wp" : true,
      "vme" : "virtual mode extensions",
      "de" : "debugging extensions",
      "pse" : "page size extensions",
      "tsc" : "time stamp counter",
      "msr" : "model-specific registers",
      "pae" : "4GB+ memory addressing (Physical Address Extension)",
      "mce" : "machine check exceptions",
      "cx8" : "compare and exchange 8-byte",
      "apic" : "on-chip advanced programmable interrupt controller (APIC)",
      "sep" : "fast system calls",
      "mtrr" : "memory type range registers",
      "pge" : "page global enable",
      "mca" : "machine check architecture",
      "cmov" : "conditional move instruction",
      "pat" : "page attribute table",
      "pse36" : "36-bit page size extensions",
      "clflush" : true,
      "mmx" : "multimedia extensions (MMX)",
      "fxsr" : "fast floating point save/restore",
      "sse" : "streaming SIMD extensions (SSE)",
      "sse2" : "streaming SIMD extensions (SSE2)",
      "ht" : "HyperThreading",
      "syscall" : "fast system calls",
      "nx" : "no-execute bit (NX)",
      "rdtscp" : true,
      "x86-64" : "64bits extensions (x86-64)",
      "constant_tsc" : true,
      "rep_good" : true,
      "nopl" : true,
      "xtopology" : true,
      "nonstop_tsc" : true,
      "cpuid" : true,
      "tsc_known_freq" : true,
      "pni" : true,
      "pclmulqdq" : true,
      "monitor" : true,
      "ssse3" : true,
      "cx16" : true,
      "pcid" : true,
      "sse4_1" : true,
      "sse4_2" : true,
      "x2apic" : true,
      "movbe" : true,
      "popcnt" : true,
      "aes" : true,
      "xsave" : true,
      "avx" : true,
      "rdrand" : true,
      "hypervisor" : true,
      "lahf_lm" : true,
      "abm" : true,
      "3dnowprefetch" : true,
      "invpcid_single" : true,
      "fsgsbase" : true,
      "avx2" : true,
      "invpcid" : true,
      "rdseed" : true,
      "clflushopt" : true,
      "md_clear" : true,
      "flush_l1d" : true,
      "arch_capabilities" : true
    }
  },

```


**5. Comment obtenir la liste des derniers démarrages de la machine avec la commande journalctl ?**
**Comment afficher tout ce qu’il s’est passé sur la machine lors de l’avant-dernier boot ?**

```console
adrien@server:~$ adrien@server:~$ journalctl --list-boots | tail -n 3
 -2 831d86bd35014f63950d72b5b64a0d96 Mon 2019-10-21 07:20:23 UTC—Mon 2019-10-21 08:51:24 UTC
 -1 7f1874e64e1f4a29bf43b78761fdb5b9 Mon 2019-10-21 08:52:54 UTC—Mon 2019-10-21 08:55:14 UTC
  0 5cc101f5b5704597924c233a3fe405c9 Mon 2019-10-21 08:56:43 UTC—Mon 2019-10-21 09:41:21 UTC
```
(tail inutile mais c'est pour que ce soit plus clair à la correction/relecture)

```journal ctl -b -l``` conformément au manuel.
-b pour boot et -l pour toutes les informations

**6. Comment obtenir la liste des derniers démarrages de la machine avec la commande journalctl ?**

```journalctl --list-boots | tail -n 3```

**7. Faites en sortes que lors d’une connexion à la machine, les utilisateurs soient prévenus par un message**
**à l’écran d’une maintenance le 26 mars à minuit.**

On écrit le message que l'on souhaite afficher aux utilisateurs dans le fichier /etc/motd, que l'on crée.


**8. Ecrivez un script bash qui permet de calculer le k-ième nombre de Fibonacci : Fk = Fk−1 + Fk−2,**
**avec F0 = F1 = 1. Lancez le calcul de F100 puis lancez la commande tload depuis un autre terminal**
**virtuel. Que constatez-vous ? Interrompez ensuite le calcul avec CTRL+C et observez la conséquence sur**
**l’affichage de tload.**

```
function fib(){
  if [ $1 -le 1 ]; then
    echo 1
  else
    echo $[`fib $[$1-2]` + `fib $[$1 - 1]` ]
  fi
}
fib $1
```

Ce script demande beaucoup de calculs et donc de ressources et on peut donc voir que la consommation du processus augmente énormément. 
La charge du processeur baisse directement si on ferme le processus.
