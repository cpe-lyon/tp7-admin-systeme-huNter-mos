## Adrien GALAN

# TP 7 - Boot, services et processus / Tâches d’administration (2)

## Exercice 1. Personnalisation de GRUB

**1. Commencez par changer l’extension du fichier /etc/default/grub.d/50-curtin-settings.cfg s’il
est présent dans votre environnement (vous pouvez aussi commenter son contenu).**


**2. Modifiez le fichier /etc/default/grub pour que le menu de GRUB s’affiche pendant 10 secondes ;
passé ce délai, le premier OS du menu doit être lancé automatiquement.**


**3. Lancez la commande update-grub
 Cette commande fait appel au script grub-mkconfig qui construit le fichier de configuration
”final” de GRUB (/boot/grub/grub.cfg) à partir du fichier de paramètres et des scripts.**


**4. Redémarrez votre VM pour valider que les changements ont bien été pris en compte
 Pensez à lancer la commande update-grub après chaque modification de la configuration de
GRUB !**


**5. On va augmenter la résolution de GRUB et de notre VM. Cherchez sur Internet le ou les paramètres
à rajouter au fichier grub.**


**6. On va à présent ajouter un fond d’écran. Il existe un paquet en proposant quelques uns : grub2-splash-images
(après installation, celles-ci sont disponibles dans /usr/share/images/grub).**


**7. Il est également possible de configurer des thèmes. On en trouve quelques uns dans les dépôts (grub2-themes-*).
Installez-en un.**


**8. Ajoutez une entrée permettant d’arrêter la machine, et une autre permettant de la redémarrer.


**9. Configurer GRUB pour que le clavier soit en français


## Exercice 2. Noyau
**Dans cet exercice, on va créer et installer un module pour le noyau.**


**1. Commencez par installer le paquet build-essential, qui contient tous les outils nécessaires (compilateurs, bibliothèques) à la compilation de programmes en C (entre autres).**


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


**5. Chargez le module ; vérifiez dans le journal du noyau que le message ”La fonction init_module() est
appelée” a bien été inscrit, synonyme que le module a été chargé ; confirmez avec la commande lsmod.**


**6. Utilisez la commande modinfo pour obtenir des informations sur le module hello.ko ; vous devriez**
**notamment voir les informations figurant dans le fichier C.**


**7. Déchargez le module ; vérifiez dans le journal du noyau que le message ”La fonction cleanup_module()
**est appelée” a bien été inscrit, synonyme que le module a été déchargé ; confirmez avec la commande**
**lsmod.**


**8. Pour que le module soit chargé automatiquement au démarrage du système, il faut l’inscrire dans le**
**fichier /etc/modules. Essayez, et vérifiez avec la commande lsmod après redémarrage de la machine.**


## Exercice 3. Interception de signaux
**La commande interne trap permet de redéfinir des gestionnaires pour les signaux reçus par un processus.**
**Un cas d’utilisation typique est la suppression des fichiers temporaires créés par un script lorsque celui-ci est**
**interrompu.**

**1. Commencez par écrire un script qui recopie dans un fichier tmp.txt chaque ligne saisie au clavier par**
**l’utilisateur**


**2. Lancez votre script et appuyez sur CTRL+Z. Que se passe-t-il ? Comment faire pour que le script poursuive son exécution ?**


**3. Toujours pendant l’exécution du script, appuyez sur CTRL+C. Que se passe-t-il ?**


**4. Modifiez votre script pour redéfinir les actions à effectuer quand le script reçoit les signaux SIGTSTP**
**(= CTRL+Z) et SIGINT (= CTRL+C) : dans le premier cas, il doit afficher ”Impossible de me placer en**
**arrière-plan”, et dans le second cas, il doit afficher ”OK, je fais un peu de ménage avant” avant de**
**supprimer le fichier temporaire et terminer le script.**


**5. Testez le nouveau comportement de votre script en utilisant d’une part les raccourcis clavier, d’autre**
**part la commande kill**


**6. Relancez votre script et faites immédiatement un CTRL+C : vous obtenez un message d’erreur vous**
**indiquant que le fichier tmp.txt n’existe pas. A l’aide de la commande interne test, corrigez votre**
**script pour que ce message n’apparaisse plus.**


## Exercice 4. Surveillance de l’activité du système

**1. Dans tty1, lancez la commande htop, puis tapez la commande w dans tty2. Qu’affiche cette commande ?**


**2. Comment afficher l’historique des dernières connexions à la machine ?**


**3. Quelle commande permet d’obtenir la version du noyau ?**


**4. Comment récupérer toutes les informations sur le processeur, au format JSON ?**


**5. Comment obtenir la liste des derniers démarrages de la machine avec la commande journalctl ?**
**Comment afficher tout ce qu’il s’est passé sur la machine lors de l’avant-dernier boot ?**


**6. Comment obtenir la liste des derniers démarrages de la machine avec la commande journalctl ?**


**7. Faites en sortes que lors d’une connexion à la machine, les utilisateurs soient prévenus par un message**
**à l’écran d’une maintenance le 26 mars à minuit.**


**8. Ecrivez un script bash qui permet de calculer le k-ième nombre de Fibonacci : Fk = Fk−1 + Fk−2,**
**avec F0 = F1 = 1. Lancez le calcul de F100 puis lancez la commande tload depuis un autre terminal**
**virtuel. Que constatez-vous ? Interrompez ensuite le calcul avec CTRL+C et observez la conséquence sur**
**l’affichage de tload.**
