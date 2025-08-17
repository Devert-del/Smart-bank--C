# Smart Bank (C)

A simple console banking demo in C: login with PIN, check balance, deposit, withdraw, transfer, view history, add interest, and rank users.

## Build & Run
```bash
gcc -Wall -Wextra -O2 -std=c11 -o smart_bank smart_bank.c
./smart_bankThis is educational demo code. It stores PINs in plain text and uses basic input handling. Do not use as-is for real banking.

Future improvements: input validation, bounds-safe string functions, secure PIN handling (hashing), persistent storage.
#include <stdio.h>
#include <string.h>
#include <stdlib.h>

#define MAX_USERS 2
#define MAX_NAME 20
#define MAX_PIN 10
#define MAX_TRANSACTIONS 100

typedef struct {
    char type[10];
    float balance;
    char pin[MAX_PIN];
    char transactions[MAX_TRANSACTIONS][50];
    int transaction_count;
    char name[MAX_NAME];
} User;

User users[MAX_USERS] = {
    {"Savings", 10000.0f, "1234", {}, 0, "Tony"},
    {"Current", 8000.0f, "5678", {}, 0, "Mary"}
};

int current_user = -1;
float interest_rate = 0.05f;

/* ---- Utility functions ---- */
void safe_input(char *buffer, size_t size) {
    if (fgets(buffer, (int)size, stdin)) {
        buffer[strcspn(buffer, "\n")] = '\0';  // remove newline
    }
}

void add_transaction(int user_index, const char *desc) {
    if (users[user_index].transaction_count < MAX_TRANSACTIONS) {
        strncpy(users[user_index].transactions[users[user_index].transaction_count++],
                desc, sizeof(users[user_index].transactions[0]) - 1);
    }
}

/* ---- Banking features ---- */
void show_menu() {
    printf("\n===== Smart Bank Menu =====\n");
    printf("1. Check Balance\n");
    printf("2. Deposit\n");
    printf("3. Withdraw\n");
    printf("4. Transfer\n");
    printf("5. Change PIN\n");
    printf("6. Transaction History\n");
    printf("7. Add Interest\n");
    printf("8. Rank Users\n");
    printf("9. Switch User\n");
    printf("10. Exit\n");
    printf("Choose an option: ");
}

void check_balance() {
    printf("💰 Balance: %.2f\n", users[current_user].balance);
}

void deposit() {
    float amount;
    printf("Enter amount to deposit: ");
    if (scanf("%f", &amount) != 1 || amount <= 0) {
        printf("❌ Invalid input.\n");
        while (getchar() != '\n'); // clear buffer
        return;
    }
    users[current_user].balance += amount;
    char desc[50];
    snprintf(desc, sizeof(desc), "Deposited %.2f", amount);
    add_transaction(current_user, desc);
    printf("✅ Deposit successful.\n");
}

void withdraw() {
    float amount;
    printf("Enter amount to withdraw: ");
    if (scanf("%f", &amount) != 1 || amount <= 0) {
        printf("❌ Invalid input.\n");
        while (getchar() != '\n'); // clear buffer
        return;
    }
    if (amount <= users[current_user].balance) {
        users[current_user].balance -= amount;
        char desc[50];
        snprintf(desc, sizeof(desc), "Withdrew %.2f", amount);
        add_transaction(current_user, desc);
        printf("✅ Withdrawal successful.\n");
    } else {
        printf("❌ Insufficient balance.\n");
    }
}

void transfer() {
    char name[MAX_NAME];
    float amount;
    printf("Enter recipient name: ");
    scanf("%s", name);
    printf("Enter amount to transfer: ");
    if (scanf("%f", &amount) != 1 || amount <= 0) {
        printf("❌ Invalid input.\n");
        while (getchar() != '\n');
        return;
    }

    int found = 0;
    for (int i = 0; i < MAX_USERS; i++) {
        if (strcmp(users[i].name, name) == 0 && i != current_user) {
            found = 1;
            if (amount <= users[current_user].balance) {
                users[current_user].balance -= amount;
                users[i].balance += amount;

                char send_desc[50], receive_desc[50];
                snprintf(send_desc, sizeof(send_desc), "Transferred %.2f to %s", amount, name);
                snprintf(receive_desc, sizeof(receive_desc), "Received %.2f from %s", amount, users[current_user].name);

                add_transaction(current_user, send_desc);
                add_transaction(i, receive_desc);

                printf("✅ Transfer successful.\n");
            } else {
                printf("❌ Insufficient funds.\n");
            }
            break;
        }
    }
    if (!found) printf("❌ User not found.\n");
}

void change_pin() {
    char old_pin[MAX_PIN], new_pin[MAX_PIN];
    printf("Enter old PIN: ");
    scanf("%s", old_pin);

    if (strcmp(users[current_user].pin, old_pin) == 0) {
        printf("Enter new PIN: ");
        scanf("%s", new_pin);
        strncpy(users[current_user].pin, new_pin, MAX_PIN - 1);
        users[current_user].pin[MAX_PIN - 1] = '\0'; // ensure null termination
        printf("✅ PIN changed successfully.\n");
    } else {
        printf("❌ Incorrect old PIN.\n");
    }
}

void show_transactions() {
    printf("===== Transaction History =====\n");
    for (int i = 0; i < users[current_user].transaction_count; i++) {
        printf("%d. %s\n", i + 1, users[current_user].transactions[i]);
    }
}

void add_interest() {
    float interest = users[current_user].balance * interest_rate;
    users[current_user].balance += interest;
    char desc[50];
    snprintf(desc, sizeof(desc), "Interest added: %.2f", interest);
    add_transaction(current_user, desc);
    printf("✅ Interest added.\n");
}

void rank_users() {
    printf("===== User Rankings (by Balance) =====\n");
    for (int i = 0; i < MAX_USERS - 1; i++) {
        for (int j = i + 1; j < MAX_USERS; j++) {
            if (users[i].balance < users[j].balance) {
                User temp = users[i];
                users[i] = users[j];
                users[j] = temp;
            }
        }
    }
    for (int i = 0; i < MAX_USERS; i++) {
        printf("%d. %s - %.2f\n", i + 1, users[i].name, users[i].balance);
    }
}

int login() {
    char name[MAX_NAME], pin[MAX_PIN];
    printf("Enter username: ");
    scanf("%s", name);
    printf("Enter PIN: ");
    scanf("%s", pin);

    for (int i = 0; i < MAX_USERS; i++) {
        if (strcmp(users[i].name, name) == 0 && strcmp(users[i].pin, pin) == 0) {
            return i;
        }
    }
    return -1;
}

/* ---- Main ---- */
int main() {
    printf("===== Welcome to Smart Bank =====\n");

    while (current_user == -1) {
        current_user = login();
        if (current_user == -1) {
            printf("❌ Invalid login. Try again.\n");
        }
    }

    int choice;
    do {
        show_menu();
        if (scanf("%d", &choice) != 1) {
            printf("❌ Invalid input.\n");
            while (getchar() != '\n');
            continue;
        }

        switch (choice) {
            case 1: check_balance(); break;
            case 2: deposit(); break;
            case 3: withdraw(); break;
            case 4: transfer(); break;
            case 5: change_pin(); break;
            case 6: show_transactions(); break;
            case 7: add_interest(); break;
            case 8: rank_users(); break;
            case 9: current_user = login(); break;
            case 10: printf("👋 Goodbye!\n"); break;
            default: printf("❌ Invalid option.\n");
        }
    } while (choice != 10);

    return 0;
}
