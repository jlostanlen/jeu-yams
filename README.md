# Jeu de Yams
Création d'un jeu de Yams en langage C s'exécutant dans un terminal. Le Yams est un jeu de dé pour deux joueurs. Le but est d'obtenir le plus grand nombre de points en réalisant des combinaisons.
J'ai réalisé ce projet à l'automne 2021, au tout début de ma première année d'informatique. Je n'avais jamais vraiment fait de code avant cette première année d'étude... 

## Code source : 
```/**
 * @file jeu_yams.c
 * @author Lostanlen Jérémie (jeremie.lostanlen@etudiant.univ-rennes1.fr)
 * @brief 
 * @version 0.1
 * @date 2021-11-25
 * 
 * @copyright Copyright (c) 2021
 * 
 */

#include <stdlib.h>
#include <stdio.h>
#include <stdbool.h>
#include <time.h>
#include <string.h>

#define D 5        //nombre d'éléments des tableaux qui affichent les dés
#define MAX_COL 2  //nombre de colonnes du tableau marque
#define MAX_LIG 17 //nombre de lignes du tableau marque

typedef int typeTab[MAX_LIG][MAX_COL];
typedef int autreTab[D];
const int BORNE = 6;
const int FINCONSERVE = -1;
const int INITIALISER = 0;

//declaration des procédures et fonctions:
void demandeNom(char *nom1, char *nom2);
void combinaisonsEtValeurs();
void feuilleMarque(char *nom1, char *nom2, typeTab marque);
int alea();
void lancerDe(autreTab t, int nbrDeConserve);
void afficheDe(autreTab t, int nbrDe);
void proposeValideLancer(autreTab t, autreTab u, int numDe, int nbrDe, int *nbrLancers, int k);
void chercheDe(autreTab t, autreTab u, int numDe, int k);
int calculeSommeValeur(autreTab t, int *a);
void triSelection(autreTab t);
int sommeBrelanCarreFullHouseYams(autreTab t, int *x, int doubl, int tripl, int *nbrVariables);
int sommePetiteEtGrandeSuite(autreTab t, int *x);
int sommeChance(autreTab t);
void enregistreActualise(typeTab t, int *somme, int *numTour, int *numCombinaison, int *totalPartieJoueur);
void afficheScores(typeTab t, char *nom1, char *nom2);
void initialiseTableau2dimensions(typeTab t);
void initialiserTableauUneDimension(autreTab t);
int affectationJoueur2(int stockageDe, autreTab t, int *numCombinaison);
int affectationJoueur1(int stockageDe, autreTab t, int *numCombinaison);

int main()
{
    srand(time(NULL));
    typeTab marque;     //Tableau des scores des deux joueurs
    autreTab des;       //Tableau qui contient les dés après un lancer
    autreTab desFinaux; //Tableau qui stocke les valeurs des dés que l'utilisateur désire conserver
    int compt;

    int stockageDe; //variable de stockage dans laquelle on met les dés que l'utilisateur désire conserver
    int nbrDe;      //Variable qui compte le nombre de dés que l'utilisateur veut conserver
    //int stockageDe;          //Variable qui contient le score du joueur
    int somme;               //récupère la somme retournée par les fonctions qui calculent la somme des dés pour chaque combinaison, pour le joueur 1
    int numDe;               //Variable qui contient le numéro du dé que le joueur veut conserver
    int nbrLancers;          //Variable qui compte le nombre de lancers du joueur
    int numTour;             //vaut 1 quand c'est au tour du joueur 1 et 2 quand c'est au tour du joueur 2
    int numCombinaison;      //contient le numéro de ligne du tableau "marque" qui correspond à la combinaison à laquelle le joueur souhaite affecter son score
    int totalPartieJoueur;   //comptabilise le nombre total de points de chaque joueur
    int k;                   //compte à quelle case du tableau desFinaux on est arrivé, pour la procedure chercheDe
    char nom1[11], nom2[11]; //Nom du joueur 1 et du joueur 2
    char rep1;               //Le Joueur répond s'il désire lancer les dés ou non

    k = 0;
    compt = 0;
    numDe = 0;
    nbrDe = 5;
    stockageDe = 0;
    nbrLancers = 0;
    somme = 0;
    totalPartieJoueur = 0;

    initialiserTableauUneDimension(desFinaux);
    printf("\t\t\t--JEU DE YAMS--\n\n");
    demandeNom(nom1, nom2);
    printf("\n");
    combinaisonsEtValeurs();
    printf("\n\n");
    initialiseTableau2dimensions(marque);
    //Tours de jeu
    for (compt = 0; compt < 13; compt++)
    {
        //Pour joueur 1
        printf("\t\tTour du premier joueur:\n\n");
        initialiserTableauUneDimension(desFinaux);
        numTour = 1;
        feuilleMarque(nom1, nom2, marque); //affichage de la feuille de score des deux joueurs
        printf("\n\n");
        printf("souhaitez-vous lancer les dés (o/N) ?\n");
        scanf(" %c", &rep1);
        while ((rep1 != 'o') && (rep1 != 'N')) //On vérifie si l'utilisateur entre bien la réponse voulue
        {
            printf("Entrez 'o' ou 'N' ");
            scanf(" %c", &rep1);
        }
        if (rep1 == 'o')
        {
            lancerDe(des, nbrDe);
            nbrLancers = nbrLancers + 1;
            proposeValideLancer(des, desFinaux, numDe, nbrDe, &nbrLancers, k);
            somme = affectationJoueur1(stockageDe, desFinaux, &numCombinaison);
            enregistreActualise(marque, &somme, &numTour, &numCombinaison, &totalPartieJoueur);
            printf("\n\n");
        }
        else //Si l'utilisateur entre 'N'
        {
            printf("Fin du tour");
        }

        //Pour joueur 2
        printf("\t\tTour du deuxième joueur:");
        nbrLancers = 0;
        initialiserTableauUneDimension(desFinaux);
        numTour = 2;
        feuilleMarque(nom1, nom2, marque); //affichage de la feuille de marque
        printf("souhaitez-vous lancer les dés (o/N) ? \t");
        scanf(" %c", &rep1);
        while ((rep1 != 'o') && (rep1 != 'N')) //On vérifie si l'utilisateur entre bien la réponse voulue
        {
            printf("Entrez 'o' ou 'N' ");
            scanf(" %c", &rep1);
        }
        if (rep1 == 'o')
        {
            lancerDe(des, nbrDe);
            nbrLancers = nbrLancers + 1;
            proposeValideLancer(des, desFinaux, numDe, nbrDe, &nbrLancers, k);
            somme = affectationJoueur2(stockageDe, desFinaux, &numCombinaison);
            enregistreActualise(marque, &somme, &numTour, &numCombinaison, &totalPartieJoueur);
        }
        else //Si l'utilisateur entre 'N'
        {
            printf("Fin du tour");
        }
    }
    feuilleMarque(nom1, nom2, marque);
    afficheScores(marque, nom1, nom2);
    return EXIT_SUCCESS;
}

//définition des procédures et fonctions

//fonction pour demander les noms des joueurs
/**
 * @brief 
 * 
 * @param nom1 
 * @param nom2 
 */
void demandeNom(char *nom1, char *nom2)
{
    printf("Nom du premier joueur (si votre nom dépasse 10 carractères, veuillez saisir à la place les deux premières lettres de votre nom):");
    scanf("%s", nom1);
    printf("nom du deuxième joueur (si votre nom dépasse 10 carractères, veuillez saisir à la place les deux premières lettres de votre nom):");
    scanf("%s", nom2);
    if (strcmp(nom1, nom2) == 0)
    {
        printf("%s saisissez de nouveau votre nom en y ajoutant un 2 à la fin, sans espace", nom2);
        scanf("%s", nom2);
    }
}

//fonction pour afficher les combinaisons
/**
 * @brief 
 * 
 */
void combinaisonsEtValeurs()
{
    printf("Combinaisons:\n\n");
    printf("\t-Brelan: 3 dés identiques\n");
    printf("\t-Carré: 4 dés identiques \n");
    printf("\t-Full House: Paire + Brelan\n");
    printf("\t-Petite Suite: 4 dés qui se suivent\n");
    printf("\t-Grande Suite: 5 dés qui se suivent\n");
    printf("\t-Yams: 5 dés de même valeur\n");
    printf("\t-Chance: Somme des valeurs des 5 dés\n");
}

//procedure feuille de marque
/**
 * @brief 
 * 
 * @param nom1 
 * @param nom2 
 * @param marque1 
 * @param marque2 
 */
void feuilleMarque(char *nom1, char *nom2, typeTab marque)
{
    //valeurs:
    printf("Somme des dés:\n\n");
    printf("-----------------------------------------\n");
    printf("|                       |%10s|%10s|\n", nom1, nom2);
    printf("-----------------------------------------\n");
    printf("|0 - Valeur de 1        |%10d|%10d|\n", marque[0][0], marque[0][1]);
    printf("-----------------------------------------\n");
    printf("|1 - Valeur de 2        |%10d|%10d|\n", marque[1][0], marque[1][1]);
    printf("-----------------------------------------\n");
    printf("|2 - Valeur de 3        |%10d|%10d|\n", marque[2][0], marque[2][1]);
    printf("-----------------------------------------\n");
    printf("|3 - Valeur de 4        |%10d|%10d|\n", marque[3][0], marque[3][1]);
    printf("-----------------------------------------\n");
    printf("|4 - Valeur de 5        |%10d|%10d|\n", marque[4][0], marque[4][1]);
    printf("-----------------------------------------\n");
    printf("|5 - Valeur de 6        |%10d|%10d|\n", marque[5][0], marque[5][1]);
    printf("-----------------------------------------\n");
    printf("|6 - Bonus si>62[35]    |%10d|%10d|\n", marque[6][0], marque[6][1]);
    printf("-----------------------------------------\n");
    printf("|7 - Total supérieur    |%10d|%10d|\n", marque[7][0], marque[7][1]);
    printf("\n\n");
    //combinaisons:
    printf("Combinaisons:\n\n");
    printf("-----------------------------------------\n");
    printf("|                       |%10s|%10s|\n", nom1, nom2);
    printf("-----------------------------------------\n");
    printf("|8 - Brelan             |%10d|%10d|\n", marque[8][0], marque[8][1]);
    printf("-----------------------------------------\n");
    printf("|9 - Carré              |%10d|%10d|\n", marque[9][0], marque[9][1]);
    printf("-----------------------------------------\n");
    printf("|10 - Full House        |%10d|%10d|\n", marque[10][0], marque[10][1]);
    printf("-----------------------------------------\n");
    printf("|11 - Petite Suite      |%10d|%10d|\n", marque[11][0], marque[11][1]);
    printf("-----------------------------------------\n");
    printf("|12 - Grande Suite      |%10d|%10d|\n", marque[12][0], marque[12][1]);
    printf("-----------------------------------------\n");
    printf("|13 - Yams              |%10d|%10d|\n", marque[13][0], marque[13][1]);
    printf("-----------------------------------------\n");
    printf("|14 - Chance            |%10d|%10d|\n", marque[14][0], marque[14][1]);
    printf("-----------------------------------------\n");
    printf("|15 - Total inférieur   |%10d|%10d|\n", marque[15][0], marque[15][1]);
    printf("\n\n");
    //scores finaux:
    printf("-----------------------------------------\n");
    printf("|                       |%10s|%10s|\n", nom1, nom2);
    printf("-----------------------------------------\n");
    printf("|16 - Total             |%10d|%10d|\n", marque[16][0], marque[16][1]);
    printf("-----------------------------------------\n");
}

//Fonction qui produit une valeur aléatoire
/**
 * @brief 
 * 
 * @return int 
 */
int alea()
{
    int n;
    do
    {
        n = rand() % BORNE;
    } while (n == 0);
    return n;
}
//Fonction qui lance les dés
/**
 * @brief 
 * 
 * @param t 
 * @param nbrDe 
 */
void lancerDe(autreTab t, int nbrDe)
{
    int i;
    for (i = 0; i < D; i++) //D-nbrDe permet de relancer le nombre de dés voulu par le joueur. nbrDe est déterminé dans la procédure qui demande à l'utilisateur s'il désire relancer les dés ou non. Par exemple, si l'utilisateur veut conserver 3 dés, nbrDe vaut 3.
    {
        t[i] = alea();
    }
    afficheDe(t, nbrDe);
}

//Procédure qui affiche un de
/**
 * @brief 
 * 
 * @param t 
 */
void afficheDe(autreTab t, int nbrDe)
{
    int i;
    printf("---------------\n");
    for (i = 0; i < (nbrDe); i++)
    {
        printf("|%1d|", t[i]);
    }
    printf("\n");
    printf("---------------");
    printf("\n");
}

//procedure qui propose au joueur de valider son lancer ou de relancer
/**
 * @brief 
 * 
 * @param rep2 
 * @param t 
 * @param u 
 * @param numDe 
 * @param nbrDeConserve 
 * @param nbrLancers 
 */
void proposeValideLancer(autreTab t, autreTab u, int numDe, int nbrDe, int *nbrLancers, int k)
{
    char rep2; //le joueur indique s'il veut valider son lancer
    int i;
    int j;
    int nbrDeConserve;
    j = 0;
    nbrDeConserve = 0;
    do
    {
        printf("Souhaitez-vous valider votre lancer (o/N) ? \n");
        scanf(" %c", &rep2);
        if ((rep2 == 'N') && (*nbrLancers < 3))
        {
            do
            {
                printf("Indiquez le numéro du dé que vous souhaitez conserver (le numéro du premier dé est 0) ou bien -1 si vous ne voulez plus conserver de dés:\t");
                scanf("%d", &numDe);
                nbrDeConserve = nbrDeConserve + 1; //compte le nombre de dés conservés
                nbrDe = (6 - nbrDeConserve);
                    //variable qui correspond au nombre de dés que le joueur veut relancer.
                if (numDe != FINCONSERVE)
                {
                    chercheDe(t, u, numDe, k);
                    k++;
                }
            } while ((numDe != FINCONSERVE) && (nbrDeConserve < 5)); //On boucle tant que l'utilisateur désire conserver un dé et que le nombre de dés qu'il veut conserver est inférieur ou égal à 5
        }
        if ((numDe != FINCONSERVE) || (nbrDeConserve < 5))
        {
            lancerDe(t, nbrDe);
        }
        *nbrLancers = *nbrLancers + 1;
    }while ((rep2 != 'o') && (*nbrLancers < 3)); //on renntre dans la boucle tant que le joueur ne souhaite pas valider son lancer et que le nombre de lancers est inférieur à 3)
    
    //la partie qui suit permet de recopier, dans le tableau qui doit contenir les 5 dés finaux du joueur, les dés qui restent après que l'utilisateur ait validé son lancer ou qu'il ait épuisé ses trois lancers
    for (i = 0; i < D; i++)
    {
        if (u[i] == 0)
        {
            u[i] = t[j];
            j++;
        }
    }
    nbrDe = 5;
    if (*nbrLancers == 3)
    {
        afficheDe(u, nbrDe);
        printf("\n");
        printf("Vous avez épuisé tous vos lancers.");
    }
    else
    {
        afficheDe(u, nbrDe);
        printf("\n");
    }
    nbrDeConserve = 0;
}

//fonction qui cherche le dé que le joueur veut conserver
/**
 * @brief 
 * 
 * @param t 
 * @param u
 * @param  
 * @param numDe 
 */
void chercheDe(autreTab t, autreTab u, int numDe, int k)
{
    u[k] = t[numDe];
}

/**
 * @brief 
 * 
 * @param stockageDe 
 * @param t 
 * @return somme retourne la somme des dés correspondant à une combinaison
 */
//procédure pour affecter le score réalisé à une combinaison
int affectationJoueur1(int stockageDe, autreTab t, int *numCombinaison)
{
    int somme; //récupère la somme retournée par les fonctions qui calculent la somme des dés pour chaque combinaison
    int nbrVariables;
    //Compte le nombre de variables qui sont utilisées pour une combinaison (par exemple, "Chance" peut en utiliser 5)

    int a;    //correspond à la valeur à rechercher dans la série de dé du joueur quand il demande d'affecter son score à une valeur de dé. Par exemple, quand il choisit 2, a vaut 2
    int x; //Correspond au nombre de dés pour une combinaison combinaison. Par exemple, pour le Brelan, x=3.

    bool un, deux, trois, quatre, cinq, six, brelan, carre, fullHouse, petiteSuite, grandeSuite, yams, chance; /*Variables booléennes correspondants aux différentes combinaisons. 
                                                                                                                Elles sont initialisées à false et prennent la valeur true quand l'utilisateur y affecte sont score. */
    int doubl, tripl; //correspond à la valeur du double et du triple que le joueur veut affecter au Full house
    //On initialise tous ces booléens à false car aucun score n'y a été affecté:
    un = false;
    deux = false;
    trois = false;
    quatre = false;
    cinq = false;
    six = false;
    brelan = false;
    carre = false;
    fullHouse = false;
    petiteSuite = false;
    grandeSuite = false;
    yams = false;
    chance = false;
    printf("Indiquez le numéro correspondant à la combinaison ou à la valeur de dé à laquelle vous souhaitez affecter votre score. \nSi votre score ne vous convient pas, entrez -1");
    scanf("%d", numCombinaison);
    triSelection(t);
    if (*numCombinaison == 0)
    {
        if (un == true)
        {
            printf("Erreur: vous avez déjà sélectionné cette valeur. Choisissez-en une autre.\n");
        }
        else
        {
            a = 1;
            un = true;
            somme = calculeSommeValeur(t, &a);
        }
    }
    else if (*numCombinaison == 1)
    {
        if (deux == true)
        {
            printf("Erreur: vous avez déjà sélectionné cette valeur. Choisissez-en une autre.\n");
        }
        else
        {
            a = 2;
            deux = true;
            somme = calculeSommeValeur(t, &a);
        }
    }
    else if (*numCombinaison == 2)
    {
        if (trois == true)
        {
            printf("Erreur: vous avez déjà sélectionné cette valeur. Choisissez-en une autre.\n");
        }
        else
        {
            a = 3;
            trois = true;
            somme = calculeSommeValeur(t, &a);
        }
    }
    else if (*numCombinaison == 3)
    {
        if (quatre == true)
        {
            printf("Erreur: vous avez déjà sélectionné cette valeur. Choisissez-en une autre.\n");
        }
        else
        {
            a = 4;
            quatre = true;
            somme = calculeSommeValeur(t, &a);
        }
    }
    else if (*numCombinaison == 4)
    {
        if (cinq == true)
        {
            printf("Erreur: vous avez déjà sélectionné cette valeur. Choisissez-en une autre.\n");
        }
        else
        {
            a = 5;
            cinq = true;
            somme = calculeSommeValeur(t, &a);
        }
    }
    else if (*numCombinaison == 5)
    {
        if (six == true)
        {
            printf("Erreur: vous avez déjà sélectionné cette valeur. Choisissez-en une autre.\n");
        }
        else
        {
            a = 6;
            six = true;
            somme = calculeSommeValeur(t, &a);
        }
    }
    else if (*numCombinaison == 8)
    {
        if (brelan == true)
        {
            printf("Erreur: vous avez déjà sélectionné cette combinaison. Choisissez-en une autre.\n");
        }
        else
        {
            brelan = true;
            nbrVariables = 1; //Il n'y a besoin que d'une variable pour le Brelan
            x = 3;            //Car, dans un Brelan, il y a trois valeurs à rechercher
            somme = sommeBrelanCarreFullHouseYams(t, &x, doubl, tripl, &nbrVariables);
        }
    }
    else if (*numCombinaison == 9)
    {
        if (carre == true)
        {
            printf("Erreur: vous avez déjà sélectionné cette combinaison. Choisissez-en une autre.\n");
        }
        else
        {
            carre = true;
            nbrVariables = 1;
            x = 4; //car, dans un carré, il y a 4 valleurs à rechercher
            somme = sommeBrelanCarreFullHouseYams(t, &x, doubl, tripl, &nbrVariables);
        }
    }
    else if (*numCombinaison == 10)
    {
        if (fullHouse == true)
        {
            printf("Erreur: vous avez déjà sélectionné cette combinaison. Choisissez-en une autre.\n");
        }
        else
        {
            printf("Quelle est la valeur de votre double ?");
            scanf("%d" ,&doubl);
            printf("Quelle est la valeur de voter triple ?");
            scanf("%d" ,&tripl);
            fullHouse = true;
            nbrVariables = 2;
            somme = sommeBrelanCarreFullHouseYams(t, &x, doubl, tripl, &nbrVariables);
        }
    }
    else if (*numCombinaison == 11)
    {
        if (petiteSuite == true)
        {
            printf("Erreur: vous avez déjà sélectionné cette combinaison. Choisissez-en une autre.\n");
        }
        else
        {
            petiteSuite = true;
            x = 4;
            somme = sommePetiteEtGrandeSuite(t, &x);
        }
    }
    else if (*numCombinaison == 12)
    {
        if (grandeSuite == true)
        {
            printf("Erreur: vous avez déjà sélectionné cette combinaison. Choisissez-en une autre.\n");
        }
        else
        {
            grandeSuite = true;
            x = 5;
            somme = sommePetiteEtGrandeSuite(t, &x);
        }
    }
    else if (*numCombinaison == 13)
    {
        if (yams == true)
        {
            printf("Erreur: vous avez déjà sélectionné cette combinaison. Choisissez-en une autre.\n");
        }
        else
        {
            nbrVariables = 2;
            yams = true;
            x = 5;
            somme = sommeBrelanCarreFullHouseYams(t, &x, doubl, tripl, &nbrVariables);
        }
    }
    else if (*numCombinaison == 14)
    {
        if (chance == true)
        {
            printf("Erreur: vous avez déjà sélectionné cette combinaison. Choisissez-en une autre.\n");
        }
        else
        {
            chance = true;
            somme = sommeChance(t);
        }
    }
    else if (*numCombinaison == -1)
    {
        printf("Entrez un numéro de ligne correspondant à une combinaison qui n'a pas encore étée choisie. \nVous ne pourrez plus rien y affecter. \nLa case correspondant à cette combinaison sera remplie par la valeur -1");
        scanf("%d", &somme);
    }
    return somme;
}

int affectationJoueur2(int stockageDe, autreTab t, int *numCombinaison)
{
    int somme; //récupère la somme retournée par les fonctions qui calculent la somme des dés pour chaque combinaison
    int nbrVariables;
    //Compte le nombre de variables qui sont utilisées pour une combinaison (par exemple, "Chance" peut en utiliser 5)

    int a;    //correspond à la valeur à rechercher dans la série de dé du joueur quand il demande d'affecter son score à une valeur de dé. Par exemple, quand il choisit 2, a vaut 2
    int x; //Correspond au nombre de dés pour chaque "partie" de la combinaison. Par exemple, pour le Brelan, x=3.
    int doubl, tripl; //correspond à la valeur du double et du triple que le joueur veut affecter au Full house
    bool un, deux, trois, quatre, cinq, six, brelan, carre, fullHouse, petiteSuite, grandeSuite, yams, chance; /*Variables booléennes correspondants aux différentes combinaisons. 
                                                                                                                Elles sont initialisées à false et prennent la valeur true quand l'utilisateur y affecte sont score. */

    //On initialise tous ces booléens à false car aucun score n'y a été affecté:
    un = false;
    deux = false;
    trois = false;
    quatre = false;
    cinq = false;
    six = false;
    brelan = false;
    carre = false;
    fullHouse = false;
    petiteSuite = false;
    grandeSuite = false;
    yams = false;
    chance = false;
    printf("Indiquez le numéro correspondant à la combinaison ou à la valeur de dé à laquelle vous souhaitez affecter votre score. \nSi votre score ne vous convient pas, entrez -1");
    scanf("%d", numCombinaison);
    triSelection(t);
    if (*numCombinaison == 0)
    {
        if (un == true)
        {
            printf("Erreur: vous avez déjà sélectionné cette valeur. Choisissez-en une autre.\n");
        }
        else
        {

            a = 1;
            un = true;
            somme = calculeSommeValeur(t, &a);
        }
    }
    else if (*numCombinaison == 1)
    {
        if (deux == true)
        {
            printf("Erreur: vous avez déjà sélectionné cette valeur. Choisissez-en une autre.\n");
        }
        else
        {

            a = 2;
            deux = true;
            somme = calculeSommeValeur(t, &a);
        }
    }
    else if (*numCombinaison == 2)
    {
        if (trois == true)
        {
            printf("Erreur: vous avez déjà sélectionné cette valeur. Choisissez-en une autre.\n");
        }
        else
        {

            a = 3;
            trois = true;
            somme = calculeSommeValeur(t, &a);
        }
    }
    else if (*numCombinaison == 3)
    {
        if (quatre == true)
        {
            printf("Erreur: vous avez déjà sélectionné cette valeur. Choisissez-en une autre.\n");
        }
        else
        {

            a = 4;
            quatre = true;
            somme = calculeSommeValeur(t, &a);
        }
    }
    else if (*numCombinaison == 4)
    {
        if (cinq == true)
        {
            printf("Erreur: vous avez déjà sélectionné cette valeur. Choisissez-en une autre.\n");
        }
        else
        {

            a = 5;
            cinq = true;
            somme = calculeSommeValeur(t, &a);
        }
    }
    else if (*numCombinaison == 5)
    {
        if (six == true)
        {
            printf("Erreur: vous avez déjà sélectionné cette valeur. Choisissez-en une autre.\n");
        }
        else
        {

            a = 6;
            six = true;
            somme = calculeSommeValeur(t, &a);
        }
    }
    else if (*numCombinaison == 6)
    {
        if (brelan == true)
        {
            printf("Erreur: vous avez déjà sélectionné cette combinaison. Choisissez-en une autre.\n");
        }
        else
        {
            brelan = true;
            nbrVariables = 1; //Il n'y a besoin que d'une variable pour le Brelan
            x = 3;            //Car, dans un Brelan, il y a trois valeurs à rechercher
            somme = sommeBrelanCarreFullHouseYams(t, &x, doubl, tripl, &nbrVariables);
        }
    }
    else if (*numCombinaison == 7)
    {
        if (carre == true)
        {
            printf("Erreur: vous avez déjà sélectionné cette combinaison. Choisissez-en une autre.\n");
        }
        else
        {

            carre = true;
            nbrVariables = 1;
            x = 4; //car, dans un carré, il y a 4 valleurs à rechercher
            somme = sommeBrelanCarreFullHouseYams(t, &x, doubl, tripl, &nbrVariables);
        }
    }
    else if (*numCombinaison == 8)
    {
        if (fullHouse == true)
        {
            printf("Erreur: vous avez déjà sélectionné cette combinaison. Choisissez-en une autre.\n");
        }
        else
        {
            printf("Quelle est la valeur de votre double ?");
            scanf("%d" ,&doubl);
            printf("Quelle est la valeur de voter triple ?");
            scanf("%d" ,&tripl);
            fullHouse = true;
            nbrVariables = 2;
            somme = sommeBrelanCarreFullHouseYams(t, &x, doubl, tripl, &nbrVariables);
        }
    }
    else if (*numCombinaison == 9)
    {
        if (petiteSuite == true)
        {
            printf("Erreur: vous avez déjà sélectionné cette combinaison. Choisissez-en une autre.\n");
        }
        else
        {

            petiteSuite = true;
            x = 4;
            somme = sommePetiteEtGrandeSuite(t, &x);
        }
    }
    else if (*numCombinaison == 10)
    {
        if (grandeSuite == true)
        {
            printf("Erreur: vous avez déjà sélectionné cette combinaison. Choisissez-en une autre.\n");
        }
        else
        {

            grandeSuite = true;
            x = 5;
            somme = sommePetiteEtGrandeSuite(t, &x);
        }
    }
    else if (*numCombinaison == 11)
    {
        if (yams == true)
        {
            printf("Erreur: vous avez déjà sélectionné cette combinaison. Choisissez-en une autre.\n");
        }
        else
        {
            nbrVariables = 2;
            yams = true;
            x = 5;
            somme = sommeBrelanCarreFullHouseYams(t, &x, doubl, tripl, &nbrVariables);
        }
    }
    else if (*numCombinaison == 12)
    {
        if (chance == true)
        {
            printf("Erreur: vous avez déjà sélectionné cette combinaison. Choisissez-en une autre.\n");
        }
        else
        {

            chance = true;
            somme = sommeChance(t);
        }
    }
    else if (*numCombinaison == -1)
    {
        printf("Entrez un numéro de ligne correspondant à une combinaison qui n'a pas encore étée choisie. \nVous ne pourrez plus rien y affecter. \nLa case correspondant à cette combinaison sera remplie par la valeur -1");
        scanf("%d", &somme);
    }
    return somme;
}

//procedure qui initialise un tableau de 5 cases avec des 0. Cela va servir ensuite pour vérifier si une case contient une valeur de dé ou pas
/**
 * @brief 
 * 
 * @param t 
 */
void initialiserTableauUneDimension(autreTab t)
{
    int i;
    for (i = 0; i < D; i++)
    {
        t[i] = INITIALISER;
    }
}

//fonction qui, en parcourant le tableau "desFinaux", fait la somme des dés correspondants à la valeur de dé choisie par le joueur
/**
 * @brief 
 * 
 * @param t 
 * @param a 
 * @return int 
 */
int calculeSommeValeur(autreTab t, int *a)
{
    int somme = 0;
    int i;
    for (i = 0; i < D; i++)
    {
        if (t[i] == *a)
        {
            somme = somme + t[i]; //on ajoute à somme la valeur de dé qui correspond à celle choisie par le joueur
        }
    }
    if (somme == 0)
    {
        printf("Erreur: votre série de dé ne comporte pas la valeur demandée");
    }
    return somme;
}

/**
 * @brief 
 * 
 * @param t 
 */
//procedure qui trie un tableau
void triSelection(autreTab t)
{
    int i, j, min, indMin, temp;
    for (i = 0; i < D - 1; i++)
    {
        min = t[i];
        indMin = i;
        for (j = i + 1; j < D; j++)
        {
            if (t[j] < min)
            {
                min = t[j];
                indMin = j;
            }
        }
        temp = t[i];
        t[i] = t[indMin];
        t[indMin] = temp;
    }
}

/**
 * @brief 
 * 
 * @param t 
 * @param x 
 * @param y 
 * @param nbrVariables 
 * @return int 
 */
//fonction pour calculer la somme des dés correspondant au Brelan, Carre, Full House et Yams
int sommeBrelanCarreFullHouseYams(autreTab t, int *x, int doubl, int tripl, int *nbrVariables)
{
    int verif = 1;  //compte le nombre de valeurs identiques que l'on a trouvées
    int verif2 = 1; //même chose que pour "verif" mais, celle-ci est utlisée dans la deuxième "partie" du Full House
    int trouve;     //permet de stocker la valeur qui correspond à la combinaison
    int i = 1;
    int somme = 0;
    //pour le Brelan, le carré et le Yams
    if (*nbrVariables == 1)
    {
        while ((verif < *x) && (i < D))
        {
            if (t[i] == t[i - 1])
            {
                trouve = t[i];
                verif = verif + 1;
            }
            i++;
        }
        somme = somme + (trouve * *x);
        verif = 1;
        if (verif != *x)
        {
            printf("Erreur, votre série de dé ne comporte pas cette combinaison\t");
        }
    }
    //pour le Full House
    else if (*nbrVariables == 2)
    {
        while ((verif < doubl) && (i < D)) //première "partie" de la combinaison
        {
            if (t[i] == t[i - 1])
            {
                trouve = t[i];
                verif = verif + 1;
            }
            i++;
        }
        somme = somme + (trouve * 2);
        i = 1;
        while ((verif2 < tripl) && (i < D)) //deuxième "partie" de la combinaison
        {
            if (t[i] == t[i - 1])
            {
                trouve = t[i];
                verif2 = verif2 + 1;
            }
            i++;
        }
        somme = somme + (trouve * tripl);
        if ((verif != doubl) && (verif2 != tripl))
        {
            printf("Erreur, votre série de dé ne comporte pas cette combinaison\t");
        }
    }
    return somme;
}

/**
 * @brief fonction pour calculer somme des dés correspondant aux Petite Suite et Grande Suite
 * 
 * @param t 
 * @param x 
 * @return int 
 */
int sommePetiteEtGrandeSuite(autreTab t, int *x)
{
    int verif;
    int i = 1;
    int somme = 0;
    bool conditionFinBoucle; //booléen qui marque la condition de fin de boucle
    conditionFinBoucle = (t[i - 1] == (t[i] - 1));
    while (conditionFinBoucle)
    {
        if (conditionFinBoucle)
        {
            somme = somme + t[i - 1];
            verif++;
        }
        i++;
    }
    verif = verif + 1;
    if (verif != *x)
    {
        printf("Erreur, votre série de dé ne comporte pas cette combinaison\t");
    }
    somme = somme + t[i - 1];
    return somme;
}

/**
 * @brief 
 * 
 * @param t 
 * @return int 
 */
//fonction pour calculer somme des dés correspondant à la combinaison Chance
int sommeChance(autreTab t)
{
    int somme = 0;
    int i;
    for (i = 0; i < D; i++)
    {
        somme = t[i];
    }
    return somme;
}

/**
 * @brief 
 * 
 * @param t 
 * @param somme 
 * @param numTour 
 * @param numCombinaison 
 * @param totalPartieJoueur 
 */
//procedure qui enregistre et qui actualise la feuille de marque d'un joueur
void enregistreActualise(typeTab t, int *somme, int *numTour, int *numCombinaison, int *totalPartieJoueur)
{

    if (*numTour == 1) //quand c'est le tour du joueur 1
    {

        t[*numCombinaison][0] = *somme;
        t[16][0] = t[16][0] + *somme;
        if ((*numCombinaison >= 0) && (*numCombinaison <= 5)) //si le joueur a choisi d'affecter son score à une valeur de dé
        {
            t[7][0] = t[7][0] + *somme;
        }
        else if ((*numCombinaison >= 8) && (*numCombinaison <= 14)) //si le joueur a choisi d'affecter son score à une combinaison
        {
            t[15][0] = t[15][0] + *somme;
        }
        if (t[7][0] > 62)
        {
            t[6][0] = 35;
            t[7][0] = t[7][0] + 35;
            t[16][0] = *totalPartieJoueur + 35;
        }
    }
    else if (*numTour == 2) //quand c'est le tour du joueur 2
    {
        t[*numCombinaison][1] = *somme;
        t[16][1] = t[16][1] + *somme;
        if ((*numCombinaison >= 0) && (*numCombinaison <= 5)) //si le joueur a choisi d'affecter son score à une valeur de dé
        {
            t[7][1] = t[7][1] + *somme;
        }
        else if ((*numCombinaison >= 8) && (*numCombinaison <= 14)) //si le joueur a choisi d'affecter son score à une combinaison
        {
            t[15][1] = t[15][1] + *somme;
        }
        if (t[7][1] > 62)
        {
            t[6][1] = 35;
            t[7][1] = t[7][1] + 35;
            t[16][1] = *totalPartieJoueur + 35;
        }
    }
}

/**
 * @brief 
 * 
 * @param t 
 * @param nom1 
 * @param nom2 
 */
//procedure qui affiche les scores finaux et indique le vainqueur
void afficheScores(typeTab t, char *nom1, char *nom2)
{
    printf("-----------------------------------------\n");
    printf("|                       |%10s|%10s|\n", nom1, nom2);
    printf("-----------------------------------------\n");
    printf("|20 - Total             |%10d|%10d|\n", t[MAX_LIG][0], t[MAX_LIG][1]);
    printf("-----------------------------------------\n\n");
    if (t[MAX_LIG][0] > t[MAX_LIG][1])
    {
        printf("Felicitation %s, vous avez gagné !\n", nom1);
    }
    else if (t[MAX_LIG][0] < t[MAX_LIG][1])
    {
        printf("Félicitaion %s, vous avez gagné !", nom2);
    }
}

/**
 * @brief 
 * 
 * @param t 
 */
//procedure pour initialiser un tableau à deux dimensions avec des 0
void initialiseTableau2dimensions(typeTab t)
{
    int i;
    int j;
    for (i = 0; i < MAX_LIG; i++)
    {
        for (j = 0; j < MAX_COL; j++)
        {
            t[i][j] = 0;
        }
    }
}
```
