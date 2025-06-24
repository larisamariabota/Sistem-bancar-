#define _CRT_NO_SECURE_WARNINGS
#include <stdio.h>
#include <string.h>
#include <stdlib.h>

typedef enum { ADMIN, USER } TipUtilizator;
typedef enum { depozit, retragere, transfer } TipOperatiune;
typedef enum { initiated, processing, canceled, blocked, successful } StareOperatiune;

typedef struct Extras {
    TipOperatiune tip;
    StareOperatiune stare;
    char motiv[100];
    struct Extras* urm;
} Extras;

typedef struct Cont {
    char iban[30];
    char valuta[4];
    float sold;
    Extras* extrase;
    struct Cont* urm;
} Cont;

typedef struct User {
    char username[30];
    char parola[30];
    char nume[30];
    char prenume[30];
    char cnp[20];
    TipUtilizator tip;
    Cont* conturi;
    struct User* urm;
} User;

void adauga_extras(Cont* cont, TipOperatiune tip, StareOperatiune stare, const char* motiv) {
    Extras* e = (Extras*)malloc(sizeof(Extras));
    e->tip = tip;
    e->stare = stare;
    strncpy(e->motiv, motiv, sizeof(e->motiv));
    e->urm = cont->extrase;
    cont->extrase = e;
}

User* lista_ordonata(User* lista, User* nou) {
    if (lista == NULL || strcmp(nou->username, lista->username) < 0) {
        nou->urm = lista;
        return nou;
    }
    User* p = lista;
    while (p->urm != NULL && strcmp(p->urm->username, nou->username) < 0) {
        p = p->urm;
    }
    nou->urm = p->urm;
    p->urm = nou;
    return lista;
}

User* citeste_User() {
    User* p = (User*)malloc(sizeof(User));
    printf(" Bine ai venit:\n");
    printf("username: "); scanf("%s", p->username);
    printf("parola: "); scanf("%s", p->parola);
    printf("nume si prenume: "); scanf("%s %s", p->nume, p->prenume);
    printf("cnp: "); scanf("%s", p->cnp);
    char tipStr[10];
    printf("tip (ADMIN/USER): "); scanf("%s", tipStr);
    p->tip = (strcmp(tipStr, "ADMIN") == 0) ? ADMIN : USER;
    p->conturi = NULL;
    p->urm = NULL;
    return p;
}

void afiseaza_utilizatori(User* lista) {
    printf("=== Lista utilizatorilor ===\n");
    while (lista != NULL) {
        printf("Username: %s | Nume: %s %s | CNP: %s | Tip: %s\n",
            lista->username, lista->nume, lista->prenume, lista->cnp,
            lista->tip == ADMIN ? "ADMIN" : "USER");
        lista = lista->urm;
    }
}

Cont* creaza_cont() {
    Cont* c = (Cont*)malloc(sizeof(Cont));
    printf("IBAN: "); scanf("%s", c->iban);
    printf("Valuta: "); scanf("%s", c->valuta);
    printf("Sold: "); scanf("%f", &c->sold);
    c->extrase = NULL;
    c->urm = NULL;
    return c;
}

void adaug_IBAN(User* lista, const char* cnp) {
    while (lista != NULL) {
        if (strcmp(lista->cnp, cnp) == 0) {
            Cont* nou = creaza_cont();
            if (lista->conturi == NULL)
                lista->conturi = nou;
            else {
                Cont* p = lista->conturi;
                while (p->urm != NULL) p = p->urm;
                p->urm = nou;
            }
            printf("Cont adaugat cu succes.\n");
            return;
        }
        lista = lista->urm;
    }
    printf("Utilizatorul nu a fost găsit.\n");
}

void sterg_IBAN(User* lista, const char* iban) {
    while (lista) {
        Cont* prev = NULL;
        Cont* c = lista->conturi;
        while (c) {
            if (strcmp(c->iban, iban) == 0) {
                if (prev == NULL) lista->conturi = c->urm;
                else prev->urm = c->urm;
                free(c);
                printf("Cont șters cu succes.\n");
                return;
            }
            prev = c;
            c = c->urm;
        }
        lista = lista->urm;
    }
    printf("Contul nu a fost găsit.\n");
}

void editeaza_user(User* lista, const char* cnp) {
    while (lista != NULL) {
        if (strcmp(lista->cnp, cnp) == 0) {
            printf("Introdu noul nume și prenume: ");
            scanf("%s %s", lista->nume, lista->prenume);
            return;
        }
        lista = lista->urm;
    }
    printf("Utilizatorul nu a fost găsit.\n");
}

void schimbare_parola(User* lista, const char* cnp) {
    char parola_veche[30], parola_noua[30];
    while (lista != NULL) {
        if (strcmp(lista->cnp, cnp) == 0) {
            printf("Parola veche: "); scanf("%s", parola_veche);
            if (strcmp(lista->parola, parola_veche) != 0) {
                printf("Parolă greșită.\n"); return;
            }
            printf("Noua parolă: "); scanf("%s", parola_noua);
            strcpy(lista->parola, parola_noua);
            printf("Parolă schimbată.\n");
            return;
        }
        lista = lista->urm;
    }
    printf("Utilizatorul nu a fost găsit.\n");
}

void meniu_admin(User** lista) {
    int opt;
    char cnp[90], iban[90];
    do {
        printf("\n=== Meniu Admin ===\n");
        printf("1. Adaugare utilizator\n2. Afisare utilizatori\n3. Adauga cont\n4. Sterge cont\n5. Editare date\n6. Schimbare parola\n7. Delogare\n0. Iesire\n");
        scanf("%d", &opt);
        switch (opt) {
        case 1: *lista = lista_ordonata(*lista, citeste_User()); break;
        case 2: afiseaza_utilizatori(*lista); break;
        case 3: printf("CNP: "); scanf("%s", cnp); adaug_IBAN(*lista, cnp); break;
        case 4: printf("IBAN: "); scanf("%s", iban); sterg_IBAN(*lista, iban); break;
        case 5: printf("CNP: "); scanf("%s", cnp); editeaza_user(*lista, cnp); break;
        case 6: printf("CNP: "); scanf("%s", cnp); schimbare_parola(*lista, cnp); break;
        case 7: return;
        case 0: exit(0);
        default: printf("Optiune invalida.\n");
        }
    } while (1);
}

void meniu_user(User* user) {
    int opt;
    Cont* cont = NULL;
    char iban[30];
    do {
        printf("\n=== Meniu Utilizator ===\n");
        printf("1. Selectare cont\n2. Depozit\n3. Retragere\n4. Afișare extrase\n5. Schimbare parolă\n6. Delogare\n");
        scanf("%d", &opt);
        switch (opt) {
        case 1:
            printf("IBAN: "); scanf("%s", iban);
            cont = user->conturi;
            while (cont && strcmp(cont->iban, iban) != 0) cont = cont->urm;
            if (cont) printf("Cont selectat.\n");
            else printf("Cont inexistent.\n");
            break;
        case 2:
            if (cont) {
                float s; printf("Suma: "); scanf("%f", &s);
                cont->sold += s;
                adauga_extras(cont, depozit, successful, "Depunere");
            } else printf("Selecteaza un cont.\n");
            break;
        case 3:
            if (cont) {
                float s; printf("Suma: "); scanf("%f", &s);
                if (s <= cont->sold) {
                    cont->sold -= s;
                    adauga_extras(cont, retragere, successful, "Retragere");
                } else {
                    printf("Fonduri insuficiente.\n");
                    adauga_extras(cont, retragere, canceled, "Fonduri insuficiente");
                }
            } else printf("Selecteaza un cont.\n");
            break;
        case 4:
            if (cont) {
                Extras* e = cont->extrase;
                while (e) {
                    printf("- %s (%s): %s\n",
                        e->tip == depozit ? "Depozit" : e->tip == retragere ? "Retragere" : "Transfer",
                        e->stare == successful ? "Succes" : "Eșuat",
                        e->motiv);
                    e = e->urm;
                }
            } else printf("Selecteaza un cont.\n");
            break;
        case 5:
            schimbare_parola(user, user->cnp); break;
        case 6:
            return;
        default:
            printf("Optiune invalida.\n");
        }
    } while (1);
}

int main() {
    User* lista = NULL;
    User* utilizator = NULL;
    int opt;
    char username[30], parola[30];
    char tip_utilizator[10];

    
    do {
    printf("\n=== Alegere tip utilizator ===\n");
    printf("Alege tipul de utilizator (ADMIN/USER): ");
    scanf("%s", tip_utilizator);

   
    if (strcmp(tip_utilizator, "ADMIN") == 0 || strcmp(tip_utilizator, "USER") == 0) {
        break;
    } else {
        printf("Tip utilizator invalid. Te rugăm să alegi din nou.\n");
    }
} while (1);

do {
    printf("\n=== Autentificare ===\n");
    printf("Username: "); scanf("%s", username);
    printf("Parola: "); scanf("%s", parola);
    
    utilizator = lista;
    while (utilizator != NULL) {
        if (strcmp(utilizator->username, username) == 0 && strcmp(utilizator->parola, parola) == 0) {
            // Verifică dacă tipul de utilizator selectat corespunde
            if ((strcmp(tip_utilizator, "ADMIN") == 0 && utilizator->tip == ADMIN) || 
                (strcmp(tip_utilizator, "USER") == 0 && utilizator->tip == USER)) {
                printf("Autentificare reusita!\n");
                break;
            } else {
                printf("Tipul de utilizator nu corespunde.\n");
                utilizator = NULL;  
                break;
            }
        }
        utilizator = utilizator->urm;
    }

    if (utilizator == NULL) {
        printf("Username sau parola incorecta. Incercati din nou.\n");
        continue;  
    }

   
    if (utilizator->tip == ADMIN) {
        meniu_admin(&lista); 
    } else {
        meniu_user(utilizator); 
    }

    printf("Vrei sa te delogezi? (1 - Da, 0 - Nu): ");
    scanf("%d", &opt);
} while (opt != 0);

    return 0;
}

