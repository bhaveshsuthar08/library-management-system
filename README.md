#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <ctype.h>

#ifdef _WIN32
#include <conio.h>
#include <windows.h>
#else
#include <termios.h>
#include <unistd.h>
#endif

#define MAX_BOOKS 1000
#define MAX_MEMBERS 500
#define MAX_TRANSACTIONS 2000
#define DATA_FILE "library.dat"
#define TRANSACTION_FILE "transactions.dat"

int i;
// Structure definitions
typedef struct {
    int id;
    char title[100];
    char author[50];
    char publisher[50];
    int year;
    int is_issued;
    int member_id;
    char issue_date[20];
    char due_date[20];
} Book;

typedef struct {
    int id;
    char name[50];
    char email[50];
    char phone[15];
    int books_issued;
} Member;

typedef struct {
    int transaction_id;
    int book_id;
    int member_id;
    char issue_date[20];
    char due_date[20];
    char return_date[20];
} Transaction;

typedef struct {
    Book books[MAX_BOOKS];
    Member members[MAX_MEMBERS];
    Transaction transactions[MAX_TRANSACTIONS];
    int book_count;
    int member_count;
    int transaction_count;
} Library;

// Function prototypes
void initLibrary(Library *lib);
void saveLibrary(Library *lib);
void loadLibrary(Library *lib);
void saveTransactions(Library *lib);
void loadTransactions(Library *lib);
void displayMenu();
void addBook(Library *lib);
void viewBooks(Library *lib);
void searchBook(Library *lib);
void issueBook(Library *lib);
void returnBook(Library *lib);
void addMember(Library *lib);
void viewMembers(Library *lib);
void deleteMember(Library *lib);
void drawHeader(const char *title);
void drawLine(int length);
void clearScreen();
void waitForEnter();
int getIntInput(const char *prompt);
void getStringInput(const char *prompt, char *buffer, int length);
char *strcasestr_custom(const char *haystack, const char *needle);
Member* findMemberById(Library *lib, int member_id);

// Case-insensitive substring search
char *strcasestr_custom(const char *haystack, const char *needle) {
    if (!*needle) return (char *)haystack;
    for (; *haystack; haystack++) {
        const char *h = haystack;
        const char *n = needle;
        while (*h && *n && tolower((unsigned char)*h) == tolower((unsigned char)*n)) {
            h++;
            n++;
        }
        if (!*n) return (char *)haystack;
    }
    return NULL;
}

// Cross-platform getch implementation
int getch(void) {
#ifdef _WIN32
    return _getch();
#else
    struct termios oldattr, newattr;
    int ch;
    tcgetattr(STDIN_FILENO, &oldattr);
    newattr = oldattr;
    newattr.c_lflag &= ~(ICANON | ECHO);
    tcsetattr(STDIN_FILENO, TCSANOW, &newattr);
    ch = getchar();
    tcsetattr(STDIN_FILENO, TCSANOW, &oldattr);
    return ch;
#endif
}

// Cross-platform clear screen
void clearScreen() {
#ifdef _WIN32
    system("cls");
#else
    system("clear");
#endif
}

int main() {
    Library lib;
    int choice;
    
    initLibrary(&lib);
    loadLibrary(&lib);
    loadTransactions(&lib);
    
    do {
        clearScreen();
        drawHeader("LIBRARY MANAGEMENT SYSTEM");
        displayMenu();
        printf("\nEnter your choice: ");
        if (scanf("%d", &choice) != 1) {
            printf("\nInvalid input! Please enter a number.\n");
            while (getchar() != '\n');    // clear input buffer
            waitForEnter();
            continue;
        }
        while (getchar() != '\n');       // clear buffer after number input
        
        switch(choice) {
            case 1: addBook(&lib); break;
            case 2: viewBooks(&lib); break;
            case 3: searchBook(&lib); break;
            case 4: issueBook(&lib); break;
            case 5: returnBook(&lib); break;
            case 6: addMember(&lib); break;
            case 7: viewMembers(&lib); break;
            case 8: deleteMember(&lib); break;
            case 9:
                printf("\n.....Saving data...\n");
                saveLibrary(&lib);
                saveTransactions(&lib);
                printf("Thank you for using Library Management System!\n");
                break;
            default:
                printf("\nInvalid choice! Please try again.\n");
                waitForEnter();
        }
    } while (choice != 9);
    
    return 0;
}

void initLibrary(Library *lib) {
    lib->book_count = 0;
    lib->member_count = 0;
    lib->transaction_count = 0;
}

void saveLibrary(Library *lib) {
    FILE *file = fopen(DATA_FILE, "wb");
    if (file == NULL) {
        printf("Error saving data!\n");
        return;
    }
    fwrite(lib, sizeof(Library), 1, file);
    fclose(file);
}

void loadLibrary(Library *lib) {
    FILE *file = fopen(DATA_FILE, "rb");
    if (file == NULL) {
        printf("No existing data found. Starting with empty library.\n");
        return;
    }
    fread(lib, sizeof(Library), 1, file);
    fclose(file);
    printf("Data loaded successfully!\n");
}

void saveTransactions(Library *lib) {
    FILE *file = fopen(TRANSACTION_FILE, "wb");
    if (file == NULL) {
        printf("Error saving transaction data!\n");
        return;
    }
    fwrite(&lib->transaction_count, sizeof(int), 1, file);
    fwrite(lib->transactions, sizeof(Transaction), lib->transaction_count, file);
    fclose(file);
}

void loadTransactions(Library *lib) {
    FILE *file = fopen(TRANSACTION_FILE, "rb");
    if (file == NULL) {
        printf("No existing transaction data found.\n");
        return;
    }
    fread(&lib->transaction_count, sizeof(int), 1, file);
    fread(lib->transactions, sizeof(Transaction), lib->transaction_count, file);
    fclose(file);
    printf("Transaction data loaded successfully!\n");
}

void drawHeader(const char *title) {
    printf("\n");
    drawLine(60);
    printf("\t\t%s\n", title);
    drawLine(60);
    printf("\n");
}

void drawLine(int length) {
    int i;
    for (i = 0; i < length; i++) {
        printf("=");
    }
    printf("\n");
}

void displayMenu() {
    printf("1. Add Book\n");
    printf("2. View Books\n");
    printf("3. Search Book\n");
    printf("4. Issue Book\n");
    printf("5. Return Book\n");
    printf("6. Add Member\n");
    printf("7. View Members\n");
    printf("8. Delete Member\n");
    printf("9. Exit\n");
}

void waitForEnter() {
    printf("\nPress Enter to continue...");
    int ch;
    do {
        ch = getch();
    } while (ch != '\n' && ch != '\r');
}

int getIntInput(const char *prompt) {
    int value;
    printf("%s", prompt);
    while (scanf("%d", &value) != 1) {
        printf("Invalid input! Enter a number: ");
        while (getchar() != '\n');
    }
    while (getchar() != '\n');
    return value;
}

void getStringInput(const char *prompt, char *buffer, int length) {
    printf("%s", prompt);
    if (fgets(buffer, length, stdin)) {
        buffer[strcspn(buffer, "\n")] = 0;
    }
}

Member* findMemberById(Library *lib, int member_id) {
	int i;
    for (i = 0; i < lib->member_count; i++) {
        if (lib->members[i].id == member_id) {
            return &lib->members[i];
        }
    }
    return NULL;
}

void addBook(Library *lib) {
    clearScreen();
    drawHeader("ADD NEW BOOK");
    
    if (lib->book_count >= MAX_BOOKS) {
        printf("Maximum book capacity reached!\n");
        waitForEnter();
        return;
    }
    
    Book *newBook = &lib->books[lib->book_count];
    newBook->id = lib->book_count + 1;
    
    getStringInput("Enter Book Title: ", newBook->title, sizeof(newBook->title));
    getStringInput("Enter Author Name: ", newBook->author, sizeof(newBook->author));
    getStringInput("Enter Publisher: ", newBook->publisher, sizeof(newBook->publisher));
    newBook->year = getIntInput("Enter Publication Year: ");
    
    newBook->is_issued = 0;
    newBook->member_id = -1;
    strcpy(newBook->issue_date, "");
    strcpy(newBook->due_date, "");
    
    lib->book_count++;
    printf("\nBook added successfully! (ID: %d)\n", newBook->id);
    waitForEnter();
}

void viewBooks(Library *lib) {
    clearScreen();
    drawHeader("BOOK LIST");
    
    if (lib->book_count == 0) {
        printf("No books in the library!\n");
        waitForEnter();
        return;
    }

    printf("%-5s %-30s %-20s %-15s %-6s %-10s\n", 
           "ID", "Title", "Author", "Publisher", "Year", "Status");
    drawLine(90);

    for ( i = 0; i < lib->book_count; i++) {
        Book *book = &lib->books[i];
        printf("%-5d %-30s %-20s %-15s %-6d %-10s\n", 
               book->id, book->title, book->author, book->publisher, book->year,
               book->is_issued ? "Issued" : "Available");
    }
    
    printf("\nTotal Books: %d\n", lib->book_count);
    waitForEnter();
}

void searchBook(Library *lib) {
    clearScreen();
    drawHeader("SEARCH BOOK");
    
    if (lib->book_count == 0) {
        printf("No books in the library!\n");
        waitForEnter();
        return;
    }
    
    char title[100];
    getStringInput("Enter book title to search: ", title, sizeof(title));
    
    int found = 0;
    printf("\n%-5s %-30s %-20s %-15s %-6s %-10s\n", 
           "ID", "Title", "Author", "Publisher", "Year", "Status");
    drawLine(90);

    for ( i = 0; i < lib->book_count; i++) {
        Book *book = &lib->books[i];
        if (strcasestr_custom(book->title, title)) {
            printf("%-5d %-30s %-20s %-15s %-6d %-10s\n", 
                   book->id, book->title, book->author, book->publisher, book->year,
                   book->is_issued ? "Issued" : "Available");
            
            // Show borrower details if book is issued
            if (book->is_issued) {
                Member *borrower = findMemberById(lib, book->member_id);
                if (borrower) {
                    printf("     Issued to: %s (ID: %d, Phone: %s, Email: %s)\n", 
                           borrower->name, borrower->id, borrower->phone, borrower->email);
                    printf("     Issue Date: %s, Due Date: %s\n", 
                           book->issue_date, book->due_date);
                }
                printf("\n");
            }
            found = 1;
        }
    }
    
    if (!found) {
        printf("\nNo books found with that title.\n");
    }
    waitForEnter();
}

void issueBook(Library *lib) {
    clearScreen();
    drawHeader("ISSUE BOOK");
    
    if (lib->book_count == 0) {
        printf("No books in the library!\n");
        waitForEnter();
        return;
    }
    
    if (lib->member_count == 0) {
        printf("No members registered!\n");
        waitForEnter();
        return;
    }
    
    int book_id = getIntInput("Enter Book ID: ");
    int member_id = getIntInput("Enter Member ID: ");
    
    if (book_id < 1 || book_id > lib->book_count) {
        printf("Invalid Book ID!\n");
        waitForEnter();
        return;
    }
    
    if (member_id < 1 || member_id > lib->member_count) {
        printf("Invalid Member ID!\n");
        waitForEnter();
        return;
    }
    
    Book *book = &lib->books[book_id - 1];
    Member *member = &lib->members[member_id - 1];
    
    if (book->is_issued) {
        printf("Book is already issued!\n");
        waitForEnter();
        return;
    }
    
    if (member->books_issued >= 3) {
        printf("Member has reached the maximum issue limit (3 books)!\n");
        waitForEnter();
        return;
    }
    
    book->is_issued = 1;
    book->member_id = member_id;
    
    printf("Enter issue date (DD-MM-YYYY): ");
    scanf("%19s", book->issue_date);
    printf("Enter due date (DD-MM-YYYY): ");
    scanf("%19s", book->due_date);
    while (getchar() != '\n');
    
    member->books_issued++;
    
    // Add to transaction record
    if (lib->transaction_count < MAX_TRANSACTIONS) {
        Transaction *trans = &lib->transactions[lib->transaction_count++];
        trans->transaction_id = lib->transaction_count;
        trans->book_id = book_id;
        trans->member_id = member_id;
        strcpy(trans->issue_date, book->issue_date);
        strcpy(trans->due_date, book->due_date);
        strcpy(trans->return_date, "");
    }
    
    printf("Book '%s' issued to %s successfully!\n", book->title, member->name);
    waitForEnter();
}

void returnBook(Library *lib) {
    clearScreen();
    drawHeader("RETURN BOOK");
    
    int book_id = getIntInput("Enter Book ID: ");
    
    if (book_id < 1 || book_id > lib->book_count) {
        printf("Invalid Book ID!\n");
        waitForEnter();
        return;
    }
    
    Book *book = &lib->books[book_id - 1];
    if (!book->is_issued) {
        printf("Book was not issued!\n");
        waitForEnter();
        return;
    }
    
    Member *member = &lib->members[book->member_id - 1];
    member->books_issued--;
    
    // Update transaction record
    for ( i = 0; i < lib->transaction_count; i++) {
        Transaction *trans = &lib->transactions[i];
        if (trans->book_id == book_id && strcmp(trans->return_date, "") == 0) {
            printf("Enter return date (DD-MM-YYYY): ");
            char return_date[20];
            scanf("%19s", return_date);
            while (getchar() != '\n');
            strcpy(trans->return_date, return_date);
            break;
        }
    }
    
    book->is_issued = 0;
    book->member_id = -1;
    strcpy(book->issue_date, "");
    strcpy(book->due_date, "");
    
    printf("Book '%s' returned successfully!\n", book->title);
    waitForEnter();
}

void addMember(Library *lib) {
    clearScreen();
    drawHeader("ADD NEW MEMBER");
    
    if (lib->member_count >= MAX_MEMBERS) {
        printf("Maximum member capacity reached!\n");
        waitForEnter();
        return;
    }
    
    Member *newMember = &lib->members[lib->member_count];
    newMember->id = lib->member_count + 1;
    
    getStringInput("Enter Member Name: ", newMember->name, sizeof(newMember->name));
    getStringInput("Enter Email: ", newMember->email, sizeof(newMember->email));
    getStringInput("Enter Phone: ", newMember->phone, sizeof(newMember->phone));
    
    newMember->books_issued = 0;
    lib->member_count++;
    
    printf("\nMember added successfully! (ID: %d)\n", newMember->id);
    waitForEnter();
}

void viewMembers(Library *lib) {
    clearScreen();
    drawHeader("MEMBER LIST");
    
    if (lib->member_count == 0) {
        printf("No members registered!\n");
        waitForEnter();
        return;
    }
    
    printf("%-5s %-20s %-30s %-15s %-10s\n", 
           "ID", "Name", "Email", "Phone", "Books Issued");
    drawLine(85);

    for ( i = 0; i < lib->member_count; i++) {
        Member *member = &lib->members[i];
        printf("%-5d %-20s %-30s %-15s %-10d\n", 
               member->id, member->name, member->email, member->phone, member->books_issued);
    }
    
    printf("\nTotal Members: %d\n", lib->member_count);
    waitForEnter();
}

void deleteMember(Library *lib) {
    clearScreen();
    drawHeader("DELETE MEMBER");
    
    if (lib->member_count == 0) {
        printf("No members registered!\n");
        waitForEnter();
        return;
    }
    
    int member_id = getIntInput("Enter Member ID to delete: ");
    
    if (member_id < 1 || member_id > lib->member_count) {
        printf("Invalid Member ID!\n");
        waitForEnter();
        return;
    }
    
    Member *member = &lib->members[member_id - 1];
    
    // Check if member has issued books
    if (member->books_issued > 0) {
        printf("Cannot delete member! Member has %d book(s) issued.\n", member->books_issued);
        waitForEnter();
        return;
    }
    
    printf("Are you sure you want to delete member !!: %s (ID: %d)? (y/n): ", 
           member->name, member->id);
    char confirm;
    scanf(" %c", &confirm);
    while (getchar() != '\n');
    
    if (confirm == 'y' || confirm == 'Y') {
        // Shift all members after this one to fill the gap
        for ( i = member_id - 1; i < lib->member_count - 1; i++) {
            lib->members[i] = lib->members[i + 1];
            lib->members[i].id = i + 1; // Update ID
        }
        lib->member_count--;
        printf("Member deleted successfully!\n");
    } else {
        printf("Deletion cancelled.\n");
    }
    waitForEnter();
}

