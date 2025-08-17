# Smart Bank (C)

A simple console banking demo in C: login with PIN, check balance, deposit, withdraw, transfer, view history, add interest, and rank users.

## Build & Run
```bash
gcc -Wall -Wextra -O2 -std=c11 -o smart_bank smart_bank.c
./smart_bankThis is educational demo code. It stores PINs in plain text and uses basic input handling. Do not use as-is for real banking.

Future improvements: input validation, bounds-safe string functions, secure PIN handling (hashing), persistent storage.
