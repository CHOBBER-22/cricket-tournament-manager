

#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#define MAX_NAME 40
#define DATA_FILE "cricket_data.dat"

typedef struct {
    int id;
    char name[MAX_NAME];
    int played;
    int won;
    int lost;
    int points;
    int runsFor;
    int runsAgainst;
} Team;

typedef struct {
    int teamA;
    int teamB;
    int scoreA;
    int scoreB;
    int winnerId; /* -1 means tie */
} Match;

Team *teams = NULL;
Match *matches = NULL;
int teamCount = 0, teamCapacity = 0;
int matchCount = 0, matchCapacity = 0;

void clearInput(void) {
    int c;
    while ((c = getchar()) != '\n' && c != EOF) { }
}

int getInt(const char *prompt) {
    int n;
    printf("%s", prompt);
    while (scanf("%d", &n) != 1) {
        printf("Enter a valid integer: ");
        clearInput();
    }
    clearInput();
    return n;
}

void getText(const char *prompt, char *text, int size) {
    printf("%s", prompt);
    if (fgets(text, size, stdin)) {
        size_t len = strlen(text);
        if (len && text[len - 1] == '\n') text[len - 1] = '\0';
        else clearInput();
    }
}

int growTeams(void) {
    if (teamCount < teamCapacity) return 1;
    int next = teamCapacity == 0 ? 4 : teamCapacity * 2;
    Team *temp = realloc(teams, (size_t)next * sizeof(Team));
    if (!temp) {
        printf("Memory allocation failed.\n");
        return 0;
    }
    teams = temp;
    teamCapacity = next;
    return 1;
}

int growMatches(void) {
    if (matchCount < matchCapacity) return 1;
    int next = matchCapacity == 0 ? 4 : matchCapacity * 2;
    Match *temp = realloc(matches, (size_t)next * sizeof(Match));
    if (!temp) {
        printf("Memory allocation failed.\n");
        return 0;
    }
    matches = temp;
    matchCapacity = next;
    return 1;
}

int teamIndexById(int id) {
    for (int i = 0; i < teamCount; i++) {
        if (teams[i].id == id) return i;
    }
    return -1;
}

/* Recursion example: prints a simple numbered list backwards. */
void printCountdown(int n) {
    if (n <= 0) return;
    printf("%d ", n);
    printCountdown(n - 1);
}

void addTeam(void) {
    if (!growTeams()) return;

    Team t = {0};
    t.id = getInt("Enter unique team ID: ");
    if (teamIndexById(t.id) != -1) {
        printf("That team ID already exists.\n");
        return;
    }

    getText("Enter team name: ", t.name, MAX_NAME);
    t.played = t.won = t.lost = t.points = 0;
    t.runsFor = t.runsAgainst = 0;

    teams[teamCount++] = t;
    printf("Team added successfully.\n");
}

void listTeams(void) {
    if (teamCount == 0) {
        printf("No teams added yet.\n");
        return;
    }

    printf("\n%-6s %-22s %-7s %-6s %-6s %-7s\n",
           "ID", "Team", "Played", "Won", "Lost", "Points");
    printf("-------------------------------------------------------------\n");
    for (int i = 0; i < teamCount; i++) {
        printf("%-6d %-22.22s %-7d %-6d %-6d %-7d\n",
               teams[i].id, teams[i].name, teams[i].played,
               teams[i].won, teams[i].lost, teams[i].points);
    }
}

void recordMatch(void) {
    if (teamCount < 2) {
        printf("Add at least two teams before recording a match.\n");
        return;
    }
    if (!growMatches()) return;

    int idA = getInt("Enter ID of Team A: ");
    int a = teamIndexById(idA);
    int idB = getInt("Enter ID of Team B: ");
    int b = teamIndexById(idB);

    if (a == -1 || b == -1 || a == b) {
        printf("Invalid team IDs. Make sure both teams exist and are different.\n");
        return;
    }

    int scoreA = getInt("Enter Team A score: ");
    int scoreB = getInt("Enter Team B score: ");
    if (scoreA < 0 || scoreB < 0) {
        printf("Scores cannot be negative.\n");
        return;
    }

    Match m;
    m.teamA = idA;
    m.teamB = idB;
    m.scoreA = scoreA;
    m.scoreB = scoreB;

    teams[a].played++;
    teams[b].played++;
    teams[a].runsFor += scoreA;
    teams[a].runsAgainst += scoreB;
    teams[b].runsFor += scoreB;
    teams[b].runsAgainst += scoreA;

    if (scoreA > scoreB) {
        m.winnerId = idA;
        teams[a].won++;
        teams[a].points += 2;
        teams[b].lost++;
        printf("Winner: %s by %d runs.\n", teams[a].name, scoreA - scoreB);
    } else if (scoreB > scoreA) {
        m.winnerId = idB;
        teams[b].won++;
        teams[b].points += 2;
        teams[a].lost++;
        printf("Winner: %s by %d runs.\n", teams[b].name, scoreB - scoreA);
    } else {
        m.winnerId = -1;
        teams[a].points++;
        teams[b].points++;
        printf("Match tied. Both teams receive 1 point.\n");
    }

    matches[matchCount++] = m;
    printf("Match recorded successfully.\n");
}

void showMatchHistory(void) {
    if (matchCount == 0) {
        printf("No match results recorded.\n");
        return;
    }

    printf("\n===== MATCH HISTORY =====\n");
    for (int i = 0; i < matchCount; i++) {
        int a = teamIndexById(matches[i].teamA);
        int b = teamIndexById(matches[i].teamB);
        const char *nameA = a >= 0 ? teams[a].name : "Unknown";
        const char *nameB = b >= 0 ? teams[b].name : "Unknown";
        printf("%d) %s %d - %d %s",
               i + 1, nameA, matches[i].scoreA, matches[i].scoreB, nameB);
        if (matches[i].winnerId == -1) {
            printf(" (Tie)\n");
        } else {
            int w = teamIndexById(matches[i].winnerId);
            printf(" (Winner: %s)\n", w >= 0 ? teams[w].name : "Unknown");
        }
    }
}

void searchTeam(void) {
    int id = getInt("Enter team ID to search: ");
    int i = teamIndexById(id);
    if (i == -1) {
        printf("Team not found.\n");
        return;
    }

    Team *t = &teams[i];
    printf("\nTeam: %s\nID: %d\nPlayed: %d\nWon: %d\nLost: %d\nPoints: %d\n",
           t->name, t->id, t->played, t->won, t->lost, t->points);
    printf("Runs scored: %d\nRuns conceded: %d\n",
           t->runsFor, t->runsAgainst);
}

void sortStandings(void) {
    if (teamCount < 2) {
        printf("Add at least two teams first.\n");
        return;
    }

    for (int i = 0; i < teamCount - 1; i++) {
        for (int j = 0; j < teamCount - i - 1; j++) {
            if (teams[j].points < teams[j + 1].points ||
                (teams[j].points == teams[j + 1].points &&
                 teams[j].runsFor - teams[j].runsAgainst <
                 teams[j + 1].runsFor - teams[j + 1].runsAgainst)) {
                Team temp = teams[j];
                teams[j] = teams[j + 1];
                teams[j + 1] = temp;
            }
        }
    }
    printf("Standings sorted by points, then run difference.\n");
    listTeams();
}

void showChampion(void) {
    if (teamCount == 0) {
        printf("Add teams first.\n");
        return;
    }

    int best = 0;
    for (int i = 1; i < teamCount; i++) {
        if (teams[i].points > teams[best].points) best = i;
        else if (teams[i].points == teams[best].points &&
                 (teams[i].runsFor - teams[i].runsAgainst) >
                 (teams[best].runsFor - teams[best].runsAgainst)) best = i;
    }

    printf("\n===== CURRENT LEADER =====\n");
    printf("Team: %s\nPoints: %d\nWins: %d\n",
           teams[best].name, teams[best].points, teams[best].won);
    printf("Countdown demo: ");
    printCountdown(3);
    printf("\nNote: This is the current points-table leader, not necessarily the final champion.\n");
}

void saveData(void) {
    FILE *fp = fopen(DATA_FILE, "wb");
    if (!fp) {
        printf("Unable to save data.\n");
        return;
    }

    fwrite(&teamCount, sizeof(int), 1, fp);
    fwrite(&matchCount, sizeof(int), 1, fp);
    if (teamCount > 0) fwrite(teams, sizeof(Team), (size_t)teamCount, fp);
    if (matchCount > 0) fwrite(matches, sizeof(Match), (size_t)matchCount, fp);
    fclose(fp);
    printf("Tournament data saved to %s.\n", DATA_FILE);
}

void loadData(void) {
    FILE *fp = fopen(DATA_FILE, "rb");
    if (!fp) {
        printf("No saved tournament found. Starting fresh.\n");
        return;
    }

    int tc = 0, mc = 0;
    if (fread(&tc, sizeof(int), 1, fp) != 1 ||
        fread(&mc, sizeof(int), 1, fp) != 1 ||
        tc < 0 || mc < 0 || tc > 10000 || mc > 100000) {
        printf("Saved file is invalid. Starting fresh.\n");
        fclose(fp);
        return;
    }

    Team *newTeams = NULL;
    Match *newMatches = NULL;

    if (tc > 0) {
        newTeams = malloc((size_t)tc * sizeof(Team));
        if (!newTeams || fread(newTeams, sizeof(Team), (size_t)tc, fp) != (size_t)tc) {
            free(newTeams);
            printf("Could not load team data.\n");
            fclose(fp);
            return;
        }
    }

    if (mc > 0) {
        newMatches = malloc((size_t)mc * sizeof(Match));
        if (!newMatches || fread(newMatches, sizeof(Match), (size_t)mc, fp) != (size_t)mc) {
            free(newTeams);
            free(newMatches);
            printf("Could not load match data.\n");
            fclose(fp);
            return;
        }
    }

    free(teams);
    free(matches);
    teams = newTeams;
    matches = newMatches;
    teamCount = teamCapacity = tc;
    matchCount = matchCapacity = mc;

    fclose(fp);
    printf("Loaded %d team(s) and %d match(es).\n", teamCount, matchCount);
}

void menu(void) {
    printf("\n========== CRICKET TOURNAMENT MANAGER ==========\n");
    printf("1. Add team\n");
    printf("2. Display teams\n");
    printf("3. Record match result\n");
    printf("4. Show match history\n");
    printf("5. Search team\n");
    printf("6. Sort points table\n");
    printf("7. Show current points-table leader\n");
    printf("8. Save tournament data\n");
    printf("9. Exit\n");
    printf("================================================\n");
}

int main(void) {
    int choice;
    loadData();

    do {
        menu();
        choice = getInt("Enter choice (1-9): ");

        switch (choice) {
            case 1: addTeam(); break;
            case 2: listTeams(); break;
            case 3: recordMatch(); break;
            case 4: showMatchHistory(); break;
            case 5: searchTeam(); break;
            case 6: sortStandings(); break;
            case 7: showChampion(); break;
            case 8: saveData(); break;
            case 9:
                saveData();
                printf("Thanks for using Cricket Tournament Manager!\n");
                break;
            default:
                printf("Invalid choice. Enter 1 to 9.\n");
        }
    } while (choice != 9);

    free(teams);
    free(matches);
    teams = NULL;
    matches = NULL;
    return 0;
}
<img width="1907" height="850" alt="Screenshot 2026-11-12 103402" src="https://github.com/user-attachments/assets/2014488b-f5ee-466f-9257-8cfb279bfcf8" />
<img width="1917" height="835" alt="Screenshot 2026-11-12 103434" src="https://github.com/user-attachments/assets/7e0470c4-823a-4461-a460-9ec5c1dc5e4e" />
<img width="1917" height="837" alt="Screenshot 2026-11-12 103459" src="https://github.com/user-attachments/assets/23f671fa-e0c0-4f83-bbb3-be4178507ce5" />
<img width="1917" height="837" alt="Screenshot 2026-11-12 103602" src="https://github.com/user-attachments/assets/64955e6c-c58f-42b9-8b3c-1e813172eaef" />



