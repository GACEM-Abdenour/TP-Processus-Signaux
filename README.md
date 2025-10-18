# TP Processus et Signaux — USTHB

**Module:** Système d'exploitation  
**Sujet:** Gestion des processus et signaux sous Linux  
**Année:** 2025-2026  
**Étudiant:** Abdenour GACEM  
**Filière:** M1 RSD  

---

## ⚙️ Environnement

- **OS:** Linux (Parrot)
- **Compilateur:** `gcc`
- **Commandes utiles:** `ps`, `pstree`, `kill`, `man`
- **Notions étudiées:** `fork()`, `wait()`, `waitpid()`, `exit()`, `getpid()`, `getppid()`

---

## 🧩 Exercice 1 — Création de processus avec `fork()`

## Énoncé

### Question 1
A partir d'une console, lancer un éditeur de texte et saisir le code du programme suivant.

```c
// fils.c
#include <stdio.h>
#include <unistd.h> // fork
#include <stdlib.h> // exit
void main()
{
	int pid= fork();
	if (pid == - 1)
	{ /* code si échec : */
		perror ( "fork " ) ;
		exit(1) ; //sortir sur un code d'erreur
	}
	if (pid==0)
	{
		// Code du fils
		printf("Début fils \n ");
		printf("Processus fils de pid=%d, ppid=%d \n ", getpid(), getppid());
		sleep(6);
		exit(0);
		// Fin du processus fils
	}
	// Suite code du père, si pid > 0
	sleep(2);
	printf("Processus père de pid=%d, ppid=%d \n ", getpid(), getppid());
}
```

### Question 2
Compiler et exécuter le programme généré. Que font les fonctions `getpid()`, `getppid()` et `exit()` ? Modifier pour que le père affiche aussi le pid du fils créé. Utiliser `man`.

### Question 3
Exécuter la commande shell `ps aux` à partir d'une autre fenêtre du terminal pour visualiser les processus créés. Trouver qui est le processus père de votre programme (le père ici).

### Question 4
Modifier le programme précédent pour que le père crée n fils. Chaque fils devra afficher un message du type "fils de pid x : bonjour, le pid de mon père est y" au départ de son exécution, puis dormir 6 secondes et afficher un message du type "fils de pid x : au revoir, pid de mon père est y" à la fin de son exécution.

### Question 5
Que remarquez-vous ? NB. vérifier les valeurs du pid du père au bonjour et au revoir de chaque fils.

### Question 6
Processus orphelins : Que se passe-t-il quand un processus devient orphelin ? Que faut-il faire pour éviter cette situation ?

---

## Réponses

### 1. Code initial

```c
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>

void main()
{
	int pid = fork();
	
	if (pid == -1) {
		perror("fork");
		exit(1);
	}
	
	if (pid == 0) {
		// Code du fils
		printf("Début fils \n ");
		printf("Processus fils de pid=%d, ppid=%d \n ", getpid(), getppid());
		sleep(6);
		exit(0);
	}
	
	// Suite code du père
	sleep(2);
	printf("Processus père de pid=%d, ppid=%d \n ", getpid(), getppid());
}
```

**Compilation et exécution :**
```bash
gcc fils.c -o fils
./fils
```

```
Début fils 
Processus fils de pid=19814, ppid=19813
Processus père de pid=19813, ppid=15674
```

### 2. Fonctions et modification

**Rôle des fonctions :**

- `getpid()` : Retourne le PID (Process ID) du processus courant
- `getppid()` : Retourne le PPID (Parent Process ID), c'est-à-dire le PID du processus père
- `exit(code)` : Termine le processus et retourne un code de sortie au processus père

**Code modifié avec affichage du PID du fils par le père :**

```c
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>

void main()
{
	int pid = fork();
	
	if (pid == -1) {
		perror("fork");
		exit(1);
	}
	
	if (pid == 0) {
		// Code du fils
		printf("Début fils \n ");
		printf("Processus fils de pid=%d, ppid=%d \n ", getpid(), getppid());
		sleep(6);
		exit(0);
	}
	
	// Suite code du père
	sleep(2);
	printf("Processus père de pid=%d, ppid=%d, pid_fils=%d \n ", 
		   getpid(), getppid(), pid);
}
```
On lance le programme: 
```
./fils
```

```
Début fils
Processus fils de pid=20060, ppid=20059
Processus père de pid=20059, ppid=15674, pid_fils=20060
```

### 3. Visualisation des processus

**Commande à exécuter dans un autre terminal :**
```bash
ps aux | grep fils
```

**Le père :**
```
abdenour    8422  0.0  0.0   2472  1024 pts/1    S    18:36   0:00 ./fils

```

### 4. Création de n fils

```c
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>

int main(int argc, char *argv[])
{
	int n = 1; 
	
	if (argc == 2) {
		n = atoi(argv[1]);
	}
	
	printf("Création de %d fils... \n ", n);
	
	for(int i = 0; i < n; i++) {
		int pid = fork();
		if (pid == -1) {
			perror("fork");
			exit(1);
		}
		if (pid == 0) {
			// Code du fils
			int ppid_avant = getppid();
			printf("fils de pid %d : bonjour, le pid de mon père est %d \n ", 
				   getpid(), ppid_avant);
			sleep(6);
			int ppid_apres = getppid();
			printf("fils de pid %d : au revoir, pid de mon père est %d \n ", 
				   getpid(), ppid_apres);
			exit(0);
		}
	}
	
	// Le père NE attend PAS ses fils - il se termine immédiatement
	printf("Père terminé, pid=%d \n ", getpid());
	return 0;
}
```

### 5. Observation

**Ce qu'on remarque :**

Lorsqu'on compare les valeurs du PPID (pid du père) au moment du "bonjour" et du "au revoir" de chaque fils, on peut observer que :

- Au moment du "bonjour", le PPID correspond bien au PID du processus père
- Au moment du "au revoir" (après 6 secondes), si le père s'est terminé avant, le PPID change et devient généralement 1 (ou le PID du processus init/systemd)

**Exemple de sortie :**
On lance le programme: 
```
./fils 3
```

```
Création de 3 fils...
Père terminé, pid=17939
fils de pid 17940 : bonjour, le pid de mon père est 17939
fils de pid 17942 : bonjour, le pid de mon père est 17939
fils de pid 17941 : bonjour, le pid de mon père est 17939
fils de pid 17940 : au revoir, pid de mon père est 1575
fils de pid 17942 : au revoir, pid de mon père est 1575
fils de pid 17941 : au revoir, pid de mon père est 1575
```

Les fils deviennent **orphelins** car le père se termine avant eux.

### 6. Processus orphelins

**Qu'est-ce qu'un processus orphelin ?**

Un processus devient orphelin lorsque son processus père se termine avant lui. Dans ce cas, le système adopte automatiquement le processus orphelin en changeant son PPID pour qu'il devienne un enfant du processus init (PID = 1) ou le PId du systemd ( dans notre cas PID = 1575 ).

**Conséquences :**
- Le processus continue de s'exécuter normalement
- Son PPID devient 1 (init/systemd)
- Le processus init se charge de récupérer son code de sortie

**Comment éviter cette situation ?**

Pour éviter que les fils deviennent orphelins, le père doit **attendre** la terminaison de tous ses fils avant de se terminer lui-même. On utilise les fonctions :

- `wait(NULL)` : Attend la terminaison d'un fils quelconque
- `waitpid(pid, &status, 0)` : Attend la terminaison d'un fils spécifique

**Code corrigé avec wait() :**

```c
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>
#include <sys/wait.h>

int main(int argc, char *argv[])
{
	int n = 1;
	
	if (argc == 2) {
		n = atoi(argv[1]);
		if (n <= 0) {
			printf("Erreur: le nombre de fils doit être positif \n");
			exit(1);
		}
	} else if (argc > 2) {
		printf("Usage: %s [nombre_de_fils] \n ", argv[0]);
		printf("Exemple: %s 3    (crée 3 fils) \n ", argv[0]);
		printf("Exemple: %s      (crée 1 fils par défaut) \n ", argv[0]);
		exit(1);
	}
	
	printf("Création de %d fils... \n", n);
	
	for(int i = 0; i < n; i++) {
		int pid = fork();
		if (pid == -1) {
			perror("fork");
			exit(1);
		}
		if (pid == 0) {
			printf("fils de pid %d : bonjour, le pid de mon père est %d \n ", 
				   getpid(), getppid());
			sleep(6);
			printf("fils de pid %d : au revoir, pid de mon père est %d \n ", 
				   getpid(), getppid());
			exit(0);
		}
	}
	
	for(int i = 0; i < n; i++) {
		wait(NULL);
	}
	
	printf("Tous les fils sont terminés. Processus père (%d) se termine.\n ", getpid());
	return 0;
}
```

**Résultat avec wait() :**
On lance le programme: 
```
./fils 4
```

```
Création de 4 fils...
fils de pid 14036 : bonjour, le pid de mon père est 14035
fils de pid 14037 : bonjour, le pid de mon père est 14035
fils de pid 14038 : bonjour, le pid de mon père est 14035
fils de pid 14039 : bonjour, le pid de mon père est 14035
fils de pid 14036 : au revoir, pid de mon père est 14035
fils de pid 14037 : au revoir, pid de mon père est 14035
fils de pid 14038 : au revoir, pid de mon père est 14035
fils de pid 14039 : au revoir, pid de mon père est 14035
Tous les fils sont terminés. Processus père (14035) se termine.
```

Maintenant, le PPID reste constant et les fils ne deviennent plus orphelins.
'''
---
## 🧩 Exercice 2 — Synchronisation avec `wait()` et `waitpid()`

## Énoncé
Les fonctions `wait(..)` et `waitpid(..)`.
	
Écrire un programme dont le père, après avoir créé trois fils (f1, f2, f3), attend le retour de ces trois fils pour réaliser le calcul 3 × 10 + 5.
	
Les données :
	- le fils f1 retourne la valeur 5
	- le fils f2 retourne la valeur 10
	- le fils f3 retourne la valeur 3
	
---
	
## Réponse
	
	
```c
	#include <stdio.h>
	#include <unistd.h>
	#include <stdlib.h>
	#include <sys/wait.h>
	
	int main()
	{
		int f1, f2, f3;
		int status;
		int valeur_f1 = 0, valeur_f2 = 0, valeur_f3 = 0;
		
		// Création du fils f1
		f1 = fork();
		if (f1 == -1) {
			perror("fork f1");
			exit(1);
		}
		if (f1 == 0) {
			// Code du fils f1
			printf("Fils f1 (pid=%d) : retourne 5 \n", getpid());
			exit(5);
		}
		
		// Création du fils f2
		f2 = fork();
		if (f2 == -1) {
			perror("fork f2");
			exit(1);
		}
		if (f2 == 0) {
			// Code du fils f2
			printf("Fils f2 (pid=%d) : retourne 10 \n", getpid());
			exit(10);
		}
		
		// Création du fils f3
		f3 = fork();
		if (f3 == -1) {
			perror("fork f3");
			exit(1);
		}
		if (f3 == 0) {
			// Code du fils f3
			printf("Fils f3 (pid=%d) : retourne 3 \n", getpid());
			exit(3);
		}
		
		// Code du père : attente des trois fils
		printf("\n Père (pid=%d) : attente des fils... \n \n", getpid());
		
		// Attente et récupération de la valeur de f1
		waitpid(f1, &status, 0);
		if (WIFEXITED(status)) {
			
			valeur_f1 = WEXITSTATUS(status);
			printf("Père : f1 terminé, valeur récupérée = %d \n", valeur_f1);
		}
		
		// Attente et récupération de la valeur de f2
		waitpid(f2, &status, 0);
		if (WIFEXITED(status)) {
			valeur_f2 = WEXITSTATUS(status);
			printf("Père : f2 terminé, valeur récupérée = %d \n", valeur_f2);
		}
		
		// Attente et récupération de la valeur de f3
		waitpid(f3, &status, 0);
		if (WIFEXITED(status)) {
			valeur_f3 = WEXITSTATUS(status);
			printf("Père : f3 terminé, valeur récupérée = %d \n", valeur_f3);
		}
		
		// Calcul : 3 × 10 + 5
		int resultat = valeur_f3 * valeur_f2 + valeur_f1;
		
		printf("\n ================================= \n");
		printf("Calcul : %d × %d + %d = %d \n", valeur_f3, valeur_f2, valeur_f1, resultat);
		printf("================================= \n");
		
		return 0;
	}
```
	
### Compilation et exécution
	
```bash
	gcc exo2.c -o exo2
	./exo2
```
	
### Sortie attendue
	
```
	Fils f1 (pid=9421) : retourne 5
	Fils f2 (pid=9422) : retourne 10
	Fils f3 (pid=9423) : retourne 3
	
	Père (pid=9420) : attente des fils...
	
	Père : f1 terminé, valeur récupérée = 5
	Père : f2 terminé, valeur récupérée = 10
	Père : f3 terminé, valeur récupérée = 3
	
	=================================
	Calcul : 3 × 10 + 5 = 35
	=================================
```
	
	
## 🧩 Exercice 4 — Signaux
	
## Énoncé
	
Signaux
	
NB. La commande `kill -l` permet d'obtenir la liste complète des signaux sous Linux.
	
### Question 1
Écrire un programme « infini.c » qui réalise une boucle infinie. Lancer le programme puis taper les touches "Ctrl+C". Que se passe-t-il ? Quel est le signal associé à l'action Ctrl+C ?
	
### Question 2
Relancer le programme infini. Sur un autre terminal, déterminer le PID du processus correspondant au programme infini en utilisant "ps -A". Ensuite taper "kill -9 pid". Que se passe-t-il ? Quel est le signal envoyé par kill -9 ?
	
### Question 3
Créer un programme « assassin.c ». Ce programme demande la saisie d'un PID, puis fait appel à la fonction `kill(pid, SIGINT)`. Lancer le programme infini. Sur un autre terminal, déterminer le PID du processus correspondant à infini. Lancer assassin et saisir le PID de infini. Que se passe-t-il ?
	
### Question 4
Quel est le comportement par défaut d'un processus à la réception d'un signal (selon le signal reçu) ?
	
### Question 5
Modifier le programme infini de telle sorte qu'il affiche un message « Hello ! I am here » au lieu de se terminer lorsqu'il reçoit un signal SIGINT (par Ctrl+C ou par kill).
	
---
	
## Réponses
	
### 1. Programme infini.c et signal Ctrl+C
	
**Code infini.c :**
	
```c
	#include <stdio.h>
	#include <unistd.h>
	
	int main()
	{
		printf("Programme infini lancé (PID=%d) \n", getpid());
		printf("Appuyez sur Ctrl+C pour terminer... \n \n");
		
		while(1) {
			printf("En cours d'exécution... \n");
			sleep(1);
		}
		
		return 0;
	}
```
	
**Compilation et exécution :**
	
```bash
	gcc infini.c -o infini
	./infini
```
	
**Sortie :**
	
```
	Programme infini lancé (PID=10234)
	Appuyez sur Ctrl+C pour terminer...
	
	En cours d'exécution...
	En cours d'exécution...
	En cours d'exécution...
	^C
```
	
**Que se passe-t-il ?**
	
Lorsqu'on appuie sur `Ctrl+C`, le programme se termine immédiatement.
	
**Signal associé :**
	
Le signal envoyé par `Ctrl+C` est **SIGINT** (signal numéro 2). C'est un signal d'interruption demandant au processus de se terminer proprement.
	
**Vérification :**
	
```bash
	kill -l | grep INT
	# Affiche : 2) SIGINT
```
	
### 2. Commande kill -9
	
**Étapes :**
	
1. Lancer le programme infini dans un terminal :
```bash
	./infini
```
	
2. Dans un autre terminal, trouver le PID :
```bashObservations
	ps -A | grep infini
	# Résultat : 24436 pts/1    00:00:00 infini
```
	
3. Envoyer le signal avec kill -9 :
	```bash
	kill -9 24436
	```
	
	**Que se passe-t-il ?**
	
	Le programme se termine **immédiatement et brutalement** sans possibilité de résistance, et on voit ce message :
	```
	[1]    24436 killed     ./infini
	```
	
	**Signal envoyé :**
	
	`kill -9` envoie le signal **SIGKILL** (signal numéro 9). C'est un signal de terminaison forcée qui **ne peut pas être intercepté, ignoré ou géré** par le processus.
	
	**Différence entre SIGINT et SIGKILL :**
	
	| Signal | Numéro | Peut être intercepté ? | Usage |
	|--------|--------|------------------------|-------|
	| SIGINT | 2 | ✅ Oui | Demande de terminaison "propre" |
	| SIGKILL | 9 | ❌ Non | Terminaison forcée immédiate |
	
	### 3. Programme assassin.c
	
	**Code assassin.c :**
	
	```c
	#include <stdio.h>
	#include <signal.h>
	#include <stdlib.h>
	
	int main()
	{
		int pid;
		
		printf("=== Programme Assassin ===\\n");
		printf("Entrez le PID du processus à terminer : ");
		scanf("%d", &pid);
		
		printf("\\nEnvoi du signal SIGINT au processus %d...\\n", pid);
		
		if (kill(pid, SIGINT) == 0) {
			printf("✓ Signal SIGINT envoyé avec succès au PID %d\\n", pid);
		} else {
			perror("✗ Erreur lors de l'envoi du signal");
			exit(1);
		}
		
		return 0;
	}
	```
	
	**Compilation :**
	
	```bash
	gcc assassin.c -o assassin
	```
	
	**Test :**
	
	Terminal 1 :
	```bash
	./infini
	Programme infini lancé (PID=10456)
	Appuyez sur Ctrl+C pour terminer...
	
	En cours d'exécution...
	En cours d'exécution...
	```
	
	Terminal 2 :
	```bash
	ps -A | grep infini
	# 10456 pts/1    00:00:00 infini
	
	./assassin
	=== Programme Assassin ===
	Entrez le PID du processus à terminer : 10456
	
	Envoi du signal SIGINT au processus 10456...
	✓ Signal SIGINT envoyé avec succès au PID 10456
	```
	
	**Que se passe-t-il ?**
	
	Dans le Terminal 1, le programme `infini` se termine exactement comme si on avait appuyé sur `Ctrl+C`. Le programme `assassin` envoie le même signal SIGINT mais de manière programmatique via la fonction `kill()`.
	
	### 4. Comportement par défaut des signaux
	
	Le comportement par défaut d'un processus à la réception d'un signal dépend du type de signal :
	
	| Action par défaut | Signaux concernés | Description |
	|-------------------|-------------------|-------------|
	| **Term** (Terminaison) | SIGINT, SIGTERM, SIGQUIT | Le processus se termine |
	| **Kill** (Terminaison forcée) | SIGKILL, SIGSTOP | Terminaison immédiate, non interceptable |
	| **Core** (Core dump) | SIGSEGV, SIGABRT, SIGFPE | Terminaison + génération d'un fichier core |
	| **Ign** (Ignoré) | SIGCHLD, SIGURG | Le signal est ignoré |
	| **Stop** (Suspension) | SIGTSTP (Ctrl+Z) | Le processus est suspendu |
	| **Cont** (Continuation) | SIGCONT | Reprend un processus suspendu |
	
	**Principaux signaux :**
	
	```bash
	kill -l
	```
	
	Sortie (extrait) :
	```
	1) SIGHUP       2) SIGINT       3) SIGQUIT      4) SIGILL
	5) SIGTRAP      6) SIGABRT      7) SIGBUS       8) SIGFPE
	9) SIGKILL     10) SIGUSR1     11) SIGSEGV     12) SIGUSR2
	13) SIGPIPE     14) SIGALRM     15) SIGTERM     16) SIGSTKFLT
	17) SIGCHLD     18) SIGCONT     19) SIGSTOP     20) SIGTSTP
	```
	
	**Signaux non interceptables :**
	- **SIGKILL (9)** : Terminaison immédiate
	- **SIGSTOP (19)** : Suspension immédiate
	
	Tous les autres signaux peuvent être interceptés et gérés par le processus via la fonction `signal()` ou `sigaction()`.
	
	### 5. Interception de SIGINT
	
	**Code infini_modifie.c :**
	
	```c
	#include <stdio.h>
	#include <signal.h>
	#include <unistd.h>
	
	// Gestionnaire de signal pour SIGINT
	void gestionnaire_sigint(int sig)
	{
		printf("\n \n✋ Hello ! I am here \n \n");
		// Le programme continue après avoir affiché le message
	}
	
	int main()
	{
		// Installation du gestionnaire de signal
		signal(SIGINT, gestionnaire_sigint);
		
		printf("Programme infini modifié lancé (PID=%d) \n", getpid());
		printf("Appuyez sur Ctrl+C (le programme ne se terminera pas) \n");
		printf("Pour terminer : utilisez Ctrl+\\\\ ou kill -9 %d \n \n", getpid());
		
		while(1) {
			printf("En cours d'exécution... \n");
			sleep(4);
		}
		
		return 0;
	}
	```
	
	**Compilation et exécution :**
	
	```bash
	gcc infini_modifie.c -o infini_modifie
	./infini_modifie
	```
	
	**Sortie avec Ctrl+C :**
	
	```
	Programme infini modifié lancé (PID=10678)
	Appuyez sur Ctrl+C (le programme ne se terminera pas)
	Pour terminer : utilisez Ctrl+\ ou kill -9 10678
	
	En cours d'exécution...
	En cours d'exécution...
	En cours d'exécution...
	^C
	
	✋ Hello ! I am here
	
	En cours d'exécution...
	En cours d'exécution...
	^C
	
	✋ Hello ! I am here
	
	En cours d'exécution...
	```
	
	
