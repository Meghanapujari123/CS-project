#include <stdio.h>
#include <conio.h>
#include <stdlib.h>
#include <string.h>
#include <ctype.h>
#include <windows.h>
#include <time.h>

// Define structures for items in the inventory
struct Item {
    char id[20] ;
    char name[50] ;
    double price ;
    int quantity ;
} ;

// Define structures for customers
struct Customer {
    int id ;
    char name[50] ;
    char phoneNumber[20] ; 
    int pin ;
} ;

// Define structure for discounts
struct Discount {
    int customerId ;
    double discountPercentage ;
} ;


// Define structure for admin
struct Admin {
    char username[50];
    char password[50];
};

void cursor(int x, int y) ;

// Inventory management Functions
void displayInventory(void);
void addItem(struct Item newItem) ;
void removeItem(char *itemId) ;
void modifyItem(char *itemId) ;
void manageInventory(void) ;
void loadInventoryFromFile(const char *fileName) ;
void updateInventoryFile(const char *fileName) ;
void checkTransactions(void) ;

// Customer management Functions
void addCustomer(struct Customer newCustomer) ;
void removeCustomer(int customerId) ;
void displayCustomers(void) ;
void manageCustomers(void);
void loadCustomersWithDiscountsFromFile(void);
void writeCustomersWithDiscountsToFile(void) ;

// Discounts management Functions
void addDiscount(int customerId, double discountPercentage) ;
void removeDiscount(int customerId) ;
double getDiscount(int customerId) ;
void displayDiscounts(void) ;
void manageDiscounts(void) ;

// Admin Functions
void adminMenu(void) ;
void changeAdminKeys(void) ;
int authenticateAdmin(char *username, char *password) ;

// Shopping Functions
void storeCheckoutRecords(struct Item *cart, int cartItemCount, char *name, int customerId, double *checkoutPrices) ;
void shopping(struct Customer customer) ;

// Functions for managing inventory items
#define MAX_ITEMS 100 

struct Item inventory[MAX_ITEMS]; // Array to hold all inventory items
int itemCount = 0; // To track the current number of items in inventory

// Was given in OLD Turbo C but declared for new 64-bit compilers as ConsoleCursor function is part of windows.h
void cursor(int x, int y) {
    HANDLE hConsoleOutput;
    COORD dwCursorPosition;

    hConsoleOutput = GetStdHandle(STD_OUTPUT_HANDLE);
    dwCursorPosition.X = x;
    dwCursorPosition.Y = y;

    SetConsoleCursorPosition(hConsoleOutput, dwCursorPosition);
}

// Function to print formatted headings
void printMenu( char name[ ] ) {
    printf(" ") ;
    for(int i = 0; i < 16 ; ++i ){
  
        printf("*");
    }

    for(int i = 0; i < strlen(name) ; ++i ){
        
        printf(" %c", name[i] );
    }

    for(int i = 0; i < 16; i++){
        
        printf("*");
    }
    printf(" ") ; // To reset to default color.
}

void loadInventoryFromFile(const char *filename) {
    FILE *file = fopen(filename, "r");
    if (file == NULL) {
        cursor(0 , 0) ;
        printf("Error opening file %s.", filename);
        return;
    }

// !feof(file) checks if the end-of-file indicator is not set at the current position in the file,
    while (!feof(file) && itemCount < MAX_ITEMS) {
        fscanf(file, "%s %s %lf %d", inventory[itemCount].id, inventory[itemCount].name,
               &inventory[itemCount].price, &inventory[itemCount].quantity);
        itemCount++;
    }

    fclose(file);
}

void displayInventory(void) {
    if (itemCount == 0) {
        cursor(20, 20);
        printf("\nInventory is empty.\n");
        return;
    }

    int row = 5 ;
    cursor( 29, 1 ) ;
    printMenu("INVENTORY-DASHBOARD ") ;
    // printf("Inventory-Dashboard:\n");
    // cursor( 55, 2 ) ;
    // printf("********************\n") ;
    cursor( 45 , row ) ;
    printf("ID     |     Name     |  Price  | Quantity\n");
    for (int i = 0; i < itemCount; i++) {
        cursor( 45, ++row ) ; // As previously used so I pre-incremented for not to be on same line
        printf("%-6s | %-12s | %-7.2lf | %-3d\n", inventory[i].id, inventory[i].name, inventory[i].price, inventory[i].quantity);
    }
}

void updateInventoryFile(const char *filename) {
    FILE *file = fopen(filename, "w");
    if (file == NULL) {
        cursor( 20, 22) ;
        printf("Error opening file %s for writing.\n", filename);
        return;
    }

    // Write the updated inventory to the file
    for (int i = 0; i < itemCount; ++i) {
        if( i == itemCount - 1 )
            fprintf(file, "%s %s %.2lf %d", inventory[i].id, inventory[i].name, inventory[i].price, inventory[i].quantity);
        else
            fprintf(file, "%s %s %.2lf %d\n", inventory[i].id, inventory[i].name, inventory[i].price, inventory[i].quantity);
    }

    fclose(file);
}

void addItem(struct Item newItem) {
    int found = 0 ;
    if (itemCount >= MAX_ITEMS) {
        cursor( 20 , 22 ) ;
        printf("Inventory is full. Cannot add more items\n");
        
        return;
    }

    // Check if the item ID already exists
    for (int i = 0; i < itemCount; ++i) {
        if ( strcmp (inventory[i].id , newItem.id ) == 0 ) {
            cursor( 20, 22 );
            printf("An item with the same ID already exists. Item not added.");
            
            return;
        }
        else if( newItem.id[0] == 'D' || newItem.id[0] == 'G'|| newItem.id[0] == 'C'|| newItem.id[0] == 'S'|| newItem.id[0] == 'B' || newItem.id[0] == 'O') {
            found = 1 ; // Don't break as need to check similiar IDs
        }
    }

    if( ! found ) {
        cursor( 20, 24 ) ;
        printf("Item unable to add cause of invalid ID-format\nTry Again!\n") ;
        
        return ;
    }
        inventory[itemCount] = newItem;
        itemCount++;
        cursor( 20 , 22 );
        printf("Item added to inventory!\n") ;
        
        updateInventoryFile("inventory_data.txt") ;
}

void removeItem(char *itemId) {
    int found = 0;
    for (int i = 0; i < itemCount; ++i) {
        if ( strcmp (inventory[i].id , itemId ) == 0 ) {
            // Shift elements to fill the gap caused by removing the item
            for (int j = i; j < itemCount - 1; ++j) {
                inventory[j] = inventory[j + 1];
            }
            itemCount--;
            found = 1;
            cursor( 20 , 20);
            printf("Item removed from inventory!\n");
            
            updateInventoryFile("inventory_data.txt") ;
            break;
        }
    }

    if (!found) {
        cursor( 20 , 20 );
        printf("Item with ID %s not found in inventory. Removal failed.\n", itemId);
        
    }
}

void modifyItem(char *itemId) {
    int itemIndex = -1;

    for (int i = 0; i < itemCount; ++i) {
        if (strcmp(inventory[i].id, itemId) == 0) {
            itemIndex = i;
            break;
        }
    }

    if (itemIndex == -1) {
        cursor( 20 , 20 ) ;
        printf("Item with ID %s not found in inventory. Update failed\n", itemId);
        
        return;
    }

    char userChoice[20];
    cursor( 20 , 20 ) ;
    printf("Enter which thing to update (Quantity or Price or Both): ");
    fgets(userChoice, sizeof(userChoice), stdin);
    userChoice[strcspn(userChoice, "\n")] = '\0';

    if (strcasecmp(userChoice, "quantity") == 0) {
        int newQuantity;
        cursor( 20 , 22 ) ;
        printf("Enter new quantity: ");
        if (scanf("%d", &newQuantity) != 1) {
            fflush(stdin) ;
            cursor( 20 , 24 ) ;
            printf("Invalid input for quantity. Update failed\n");
            
            return;
        }
        inventory[itemIndex].quantity = newQuantity;
    } 
    else if (strcasecmp(userChoice, "price") == 0) {
        double newPrice;
        cursor( 20 , 22 ) ;
        printf("Enter new price: ");
        if (scanf("%lf", &newPrice) != 1) {
            fflush(stdin) ;
            cursor( 20 , 24 ) ;
            printf("Invalid input for price. Update failed\n");
            
            return;
        }
        inventory[itemIndex].price = newPrice;
    } 
    else if (strcasecmp(userChoice, "both") == 0) {
        int newQuantity;
        cursor( 20 , 22 ) ;
        printf("Enter new quantity: ");
        if (scanf("%d", &newQuantity) != 1) {
            fflush(stdin) ;
            cursor( 20 , 24 ) ;
            printf("Invalid input for quantity. Update failed\n");
            
            return;
        }
        inventory[itemIndex].quantity = newQuantity;

        double newPrice;
        cursor( 20 , 24 ) ;
        printf("Enter new price: ");
        if (scanf("%lf", &newPrice) != 1) {
            fflush(stdin) ;
            cursor( 20 , 26 ) ;
            printf("Invalid input for price. Update failed\n");
            
            return;
        }
        inventory[itemIndex].price = newPrice;
    } 
    else {
        cursor( 20 , 22 ) ;
        printf("Wrong Choice Entered!\n");
        return;
    }

    cursor( 18 , 26 ) ;
    printf("Item modified to %d quantity and %.2lf price!\n", inventory[itemIndex].quantity, inventory[itemIndex].price);
    
    updateInventoryFile("inventory_data.txt");
}

// Functions for managing customers
#define MAX_CUSTOMERS 150 

struct Customer customers[MAX_CUSTOMERS]; // Array to hold all customer details
int customerCount = 0; // Variable to track the current number of customers

// Functions for managing discounted customers
#define MAX_DISCOUNTS 100 
struct Discount discounts[MAX_DISCOUNTS]; // Array to hold all discounts
int discountCount = 0; // Variable to track the current number of discounts

void loadCustomersWithDiscountsFromFile(void) {
    FILE *file = fopen("customers_data.bin", "rb");
    if (file == NULL) {
        printf("Error opening file %s for reading.\n", "customers_data.bin");
        return;
    }

    fread(&customerCount, sizeof(int), 1, file); // Read the total number of customers first

    for (int i = 0; i < customerCount; ++i) {
        fread(&customers[i], sizeof(struct Customer), 1, file); // Read each customer structure
    }

    fread(&discountCount, sizeof(int), 1, file) ;

    for (int i = 0; i < customerCount; ++i) {
        fread(&discounts[i], sizeof(struct Discount), 1, file); // Read each discount structure then in sequence
    }

    fclose(file);
}

void writeCustomersWithDiscountsToFile(void) {
    FILE *file = fopen("customers_data.bin", "wb");
    if (file == NULL) {
        printf("Error opening file %s for writing.\n", "customers_data.bin");
        return;
    }

    fwrite(&customerCount, sizeof(int), 1, file); // Write the total number of customers first

    for (int i = 0; i < customerCount; ++i) {
        fwrite(&customers[i], sizeof(struct Customer), 1, file); // Write each customer structure
    }

    fwrite(&discountCount, sizeof(int), 1, file) ;

    for (int i = 0; i < customerCount; ++i) {
        fwrite(&discounts[i], sizeof(struct Discount), 1, file); // Write each discount structure then in sequence
    }

    fclose(file);
}

void addCustomer(struct Customer newCustomer) {
    if (customerCount >= MAX_CUSTOMERS) {
        cursor(20,23);
        printf("Maximum customers reached. Cannot add more customers\n");
        
        return;
    }

    customers[customerCount] = newCustomer;
    customerCount++;
    cursor(20,23);
    printf("%s Customer is allotted with ID-Number %d.\n", newCustomer.name, newCustomer.id);
    
    // After adding the customer, save the updated data to the file. Otherwise it was not properly reading .
    writeCustomersWithDiscountsToFile( ); // This function should be called separately not integrated with in it otherwise not properly updating
}

void removeCustomer(int customerId) {
    int found = 0;
    for (int i = 0; i < customerCount; ++i) {
        if (customers[i].id == customerId) {
            // Shift elements to fill the gap caused by removing the customer
            for (int j = i; j < customerCount - 1; ++j) {
                customers[j] = customers[j + 1];
                customers[j].id -- ; // -1 from IDs also
            }
            customerCount--;
            found = 1;
            cursor( 20 , 20);
            printf("Customer with %d-ID removed!\n", customerId );
            
            writeCustomersWithDiscountsToFile( ) ;
            break;
        }
    }

    if (!found) {
        cursor(20,20);
        printf("Customer with ID %d not found. Removal failed.\n", customerId);
        
    }
}

void displayCustomers(void) {
    if (customerCount == 0) {
        cursor(20,16);
        printf("No customers added in the list.\n");
        
        return;
    }
    int row = 5;
    cursor(15,2);
    printMenu("CUSTOMERS-DASHBOARD ") ;
    // printf("==========Customers Dasboard==========\n");
    // cursor(15,3);
    // printf("\t\t ******************\n") ;
    cursor(20,4);
    printf("ID |     Name     | Phone_Num\n");
    for (int i = 0; i < customerCount; ++i) {
        cursor( 20 , ++row );
        printf("%d  | %-12s | %s\n", customers[i].id, customers[i].name, customers[i].phoneNumber);
    }
}

// Functions for managing discounts
void addDiscount(int customerId, double discountPercentage) {
    if (discountCount >= MAX_DISCOUNTS) {
        cursor(20,18);
        printf("Maximum number of discounts reached. Cannot add more discounts\n");
        
        return; 
    }
    else if ( customerId > customerCount ) {
        cursor(20,18);
        printf("Customer with %d-ID not exists\n", customerId ) ;
        
        return ;
    }

    // Check if the discount for the customer already exists
    for (int i = 0; i < discountCount; ++i) {
        if (discounts[i].customerId == customerId) {
            cursor(20,18);
            printf("A discount for the same customer already exists. Discount not added\n");
            
            return ;
        }
    }

    discounts[discountCount].customerId = customerId;
    discounts[discountCount].discountPercentage = discountPercentage;
    discountCount++;
    cursor(20,18);
    printf("Discount added for the %d-ID Customer!\n", customerId ) ;
    
    writeCustomersWithDiscountsToFile( ) ;
}

void removeDiscount(int customerId) {
    int found = 0;
    for (int i = 0; i < discountCount; ++i) {
        if (discounts[i].customerId == customerId) {
            // Shift elements to fill the gap caused by removing the discount
            for (int j = i; j < discountCount - 1; ++j) {
                discounts[j] = discounts[j + 1];
            }
            discountCount--;
            found = 1;
            cursor( 20 , 16 );
            printf("Discount removed for the %d-ID customer!\n", customerId );
            
            writeCustomersWithDiscountsToFile( ) ;
            break;
        }
    }

    if (!found) {
        cursor(20,16);
        printf("No discount found for customer ID %d. Removal failed\n", customerId);
        
    }
}


double getDiscount(int customerId) {
    double discount = 0.0;
    for (int i = 0; i < discountCount; ++i) {
        if (discounts[i].customerId == customerId) {
            discount = discounts[i].discountPercentage;
            break;
        }
    }
    return discount;
}

void displayDiscounts(void) {
    int row = 10 ;
    cursor( 32, 5 ) ;
    printMenu("ALL DISCOUNTEES ") ;
    // printf("--- All Discounts ---\n");
    // cursor( 50, 6 ) ;
    // printf("    ************\n") ;

    cursor( 50, 8 ) ;
    printf("|Customer ID|Discount %%|\n");
    for (int i = 0; i < discountCount; ++i) {
        cursor( 50, row++ ) ; // Now here since not previously used so post-incremenr is better
        printf("| %-9d | %-7.2lf%% |\n", discounts[i].customerId, discounts[i].discountPercentage);
    }
}

void checkTransactions(void) {
    FILE *file = fopen("Records.txt", "r") ;

    if(file == NULL) {
        cursor( 20, 18 ) ;
        printf("Error opening file %s for writing.\n", "Records.bin");
        return;
    }

    char info ;
    while( ( info = fgetc(file) ) != EOF ) {
        printf("%c", info ) ;
    }
}

// Functions for admin interface
struct Admin adminUser = {"admin", "pass123"}; // Sample password


int authenticateAdmin(char *username, char *password) {
    if (strcmp(username, adminUser.username) == 0 && strcmp(password, adminUser.password) == 0) 
        return 1; 
    else
        return 0; 
}

// Functions for managing inventory in the admin menu
void manageInventory(void) {
    int choice;
    do {
        cursor(15,3);
        printf("========Inventory Management========\n");
        cursor(20,4);
        printf("1. Add Item\n");
        cursor(20,6);
        printf("2. Remove Item\n");
        cursor(20,8); 
        printf("3. Modify Item\n");
        cursor(20,10); 
        printf("4. Display Inventory\n");
        cursor(20,12); 
        printf("5. Display All Transactions\n");
        cursor(20,14); 
        printf("0. Back to Admin Menu\n");
        cursor(20,16); 
        printf("Enter your choice: ");
        scanf("%d", &choice);
        fflush( stdin ) ;
        
        switch (choice) {
            case 1: {
                system("cls");
                struct Item newItem;
                cursor(12,3);
                printf("Instructions:-");
                cursor(15,4); 
                printf("ID must follow these prefixes for specific sections in the format: 1stFixedChar(NUM or CHAR)_NUM ") ;
                cursor(15,5); 
                printf("For Dairy (D) section: ID should be like DD_188\n");
                cursor(15,6);
                printf("For Cooking Products (G or C) section: ID should be like G/C_NUM\n");
                cursor(15,7);
                printf("For Snacks (S) section: ID should be like S_NUM\n");
                cursor(15,8);
                printf("For Beverages (B) section: ID should be like B_NUM\n");
                cursor(15,9);
                printf("For Other Products (O) section: ID should be like O_NUM\n") ;
                cursor(20,14);
                printf("Enter item ID: ");
                fgets(newItem.id, sizeof(newItem.id), stdin ) ;
                newItem.id[ strcspn( newItem.id, "\n" ) ] = '\0' ;
                cursor(20,16);
                printf("Enter item name: ");
                fgets(newItem.name, sizeof(newItem.name), stdin ) ;
                newItem.name[ strcspn( newItem.name, "\n" ) ] = '\0' ;
                cursor(20,18);
                printf("Enter item price: ");
                scanf("%lf", &newItem.price);
                fflush(stdin) ;
                cursor(20,20);
                printf("Enter item quantity: ");
                scanf("%d", &newItem.quantity);
                fflush(stdin) ;
                addItem(newItem) ;
                system("cls") ;
                break;
            }
            case 2: {
                char itemId[20] ;
                cursor(20,18); 
                printf("Enter item ID to remove: ");
                fgets(itemId, sizeof(itemId), stdin) ;
                itemId[ strcspn( itemId, "\n" ) ] = '\0' ;
                removeItem(itemId);
                system("cls") ;
                break;
            }
            case 3: {
                char itemId[20] ;
                int newQuantity ;
                double newPrice ;
                cursor(20,18);
                printf("Enter item ID to update: ");
                fgets(itemId, sizeof(itemId), stdin) ;
                itemId[ strcspn( itemId, "\n" ) ] = '\0' ;
                modifyItem(itemId);
                system("cls") ;
                break;
            }
            case 4:
                system("cls") ;
                displayInventory();
                printf("\nPress any key to exit viewing.\n") ;
                getch( ) ;
                system("cls") ;
                break;
            case 5: 
                system("cls") ;
                checkTransactions() ;
                printf("\nPress any key to go back.\n") ;
                getch( ) ;
                fflush(stdin) ;
                system("cls") ;
            case 0:
                cursor(20,19);
                printf("Returning to Admin Menu...\n");
                
                system("cls");
                break;
            default:
                fflush(stdin) ;
                cursor(20,19);
                printf("Invalid choice! Please enter a valid option..\n");
                break ;
        }
    } while (choice != 0);
}

// Functions for managing customers in the admin menu
void manageCustomers(void) {
    int choice;
    do {
        cursor(15,3);
        printf("========Customer Management========");
        cursor(20,6);
        printf("1. Add Customer\n");
        cursor(20,8);
        printf("2. Remove Customer\n");
        cursor(20,10);
        printf("3. Display Customers\n");
        cursor(20,12);
        printf("0. Back to Admin Menu\n");
        cursor(20,14);
        printf("Enter your choice: ") ;
        scanf("%d", &choice);
        fflush(stdin) ;

        switch (choice) {
            case 1: {
                struct Customer newCustomer;
                newCustomer.id = customerCount + 1 ;
                cursor(20,17);
                printf("Enter customer name: ");
                cursor(20,19);
                fgets(newCustomer.name, sizeof(newCustomer.name), stdin ) ;
                newCustomer.name[ strcspn( newCustomer.name,"\n" ) ] = '\0' ;
                cursor( 20 , 21 ) ;
                printf("Enter customer phone_number: ") ;
                fgets(newCustomer.phoneNumber, sizeof(newCustomer.phoneNumber), stdin ) ;
                newCustomer.phoneNumber[ strcspn( newCustomer.phoneNumber, "\n" ) ] = '\0' ;

                for(int i = 0; i < customerCount; ++i) {
                    if( strcmp ( newCustomer.phoneNumber , customers[i].phoneNumber ) == 0 ) {
                        cursor(25, 24) ;
                        printf("%s Phone_Number is already registered.\n\t\t\t\tPlease Try Again!\n", newCustomer.phoneNumber ) ;
                        
                        system("cls") ;
                        return ;
                    }
                }
                addCustomer(newCustomer);
                system("cls") ;
                break;
            }
            case 2: {
                int customerId;
                cursor(20,17);
                printf("Enter customer ID to remove: ");
                scanf("%d", &customerId);
                fflush(stdin) ;
                removeCustomer(customerId);
                system("cls") ;
                break;
            }
            case 3:
                system("cls") ;
                displayCustomers();
                printf("\nPress Any Key to Go back to Management menu:\n");
                getch( ) ;
                system("cls") ;
                break;
            case 0:
                cursor(20,17);
                printf("Returning to Admin Menu.\n");
                
                system("cls") ;
                break;
            default:
                fflush(stdin) ;
                cursor(20,17);
                system("cls") ;
                printf("Invalid choice! Please enter a valid option.\n");
        }
    } while (choice != 0);
}

// Functions for managing discounts in the admin menu
void manageDiscounts(void) {
    int choice;
    do {
        cursor(12,2);
        printf("========Discount Management========");
        cursor(20,4);
        printf("1. Add Discount\n");
        cursor(20,6);
        printf("2. Remove Discount\n");
        cursor(20,8);
        printf("3. Display Customer Discount List\n");
        cursor(20, 10);
        printf("0. Back to Admin Menu\n");
        cursor(20,12);
        printf("Enter your choice: ");
        scanf("%d", &choice);
        fflush( stdin ) ;

        switch (choice) {
            case 1: {
                int customerId;
                double discountPercentage;
                cursor(20,14);
                printf("Enter customer ID for discount: ");
                scanf("%d", &customerId);
                fflush( stdin ) ;
                cursor(20,16);
                printf("Enter discount percentage: ");
                scanf("%lf", &discountPercentage);
                fflush( stdin ) ;
                addDiscount(customerId, discountPercentage);
                system("cls") ;
                break;
            }
            case 2: {
                int customerId;
                cursor(20,14);
                printf("Enter customer ID to remove discount: ");
                scanf("%d", &customerId);
                fflush( stdin ) ;
                removeDiscount(customerId);
                system("cls") ;
                break;
            }
            case 3:
                system("cls") ;
                displayDiscounts();
                printf("\n Press any key to go back.\n") ;
                getch( ) ;
                system("cls") ;
                break;
            case 0:
                cursor(20,14);
                printf("Returning to Admin Menu.\n") ;
                
                system("cls") ;
                break;
            default:
                fflush( stdin ) ;
                cursor(20,14);
                printf("Invalid choice! Please enter a valid option.\n");
                
                system("cls") ;
        }
    } while (choice != 0);
}

void changeAdminKeys( void ) {
    char existingPass[50] ;
    char newUsername[50] ;
    char newPassword[50] ;
    cursor( 20, 4 );
    printf("Enter the previous password for admin : ") ;
    
    int i = 0;
        while (i < 49) { 
            if (_kbhit()) {
                char ch = _getch() ;
                if (ch == 13) { 
                    existingPass[i] = '\0'; 
                    break;
                } else if (ch == 8) { 
                    if (i > 0) {
                        printf("\b \b"); 
                        i--;
                    }
                } else {
                    existingPass[i] = ch;
                    printf("*"); 
                    i++;
                }
            }
        }

    cursor( 0 , 7 ) ;
    printf("PLEASE WAIT!!\n\n\t\tVALIDATING CREDENTIALS...\n") ;
    
    if ( strcmp (existingPass, adminUser.password) == 0 ) {
        cursor(15, 9) ;
        printf("Note:-Change Password At your own risk.\n") ;
        cursor(15, 11) ;
        printf("Enter new admin username: ");
        fgets(newUsername, sizeof(newUsername), stdin ) ;
        newUsername[ strcspn(newUsername, "\n") ] = '\0' ;
        cursor(15, 13) ;
        printf("Enter new admin password: ");
        fgets(newPassword, sizeof(newPassword), stdin ) ;
        newPassword[ strcspn(newPassword, "\n") ] = '\0' ;

        // Update admin credentials
        strcpy(adminUser.username, newUsername);
        strcpy(adminUser.password, newPassword);

        cursor(20, 18) ;
        printf("Admin credentials updated successfully!\n");
    } 
    else {
        cursor(20, 18) ;
        printf("\nSecurity-Alert Sent to Server!") ;
        // Playing siren like sound via Beep
        int initialFrequency = 1000 ;
        for (int i = 0; i < 20; ++i) {
            Beep(initialFrequency, 200);
            initialFrequency += 50; 
                        
        }
        exit(1) ;
    }
}

void adminMenu(void) {
    char username[50];
    char password[50];
    int choice;

    system("cls");
    cursor(20,5);
    printMenu( "Authentication For Admin ") ;
    // printf("========Authentication For Admin========");
    // cursor(18,6);
    // printf("*********************************************\n");
    cursor(24,8);
    printf("Enter Admin Username: ");
    fgets(username, sizeof(username), stdin) ;
    username[ strcspn( username, "\n" ) ] = '\0'; 
    fflush(stdin);
    cursor(24,10);
    printf("Enter Admin Password: ");
    
    int i = 0;
    while (i < 49) { 
        if (_kbhit()) {
            char ch = _getch() ;
            if (ch == 13) { // If character is not ENTER KEY (\r For Conio) then
                password[i] = '\0';
                break;
            } else if (ch == 8) { // Backspace
                if (i > 0) {
                    printf("\b \b"); // Move cursor back, erase character, move cursor back again
                    i--;
                }
            } else {
                password[i] = ch;
                printf("*"); // Masking the input
                i++;
            }
        }
    }

    cursor( 0, 14 ) ;
    printf("PLEASE WAIT....\n\nYOUR DATA IS BEING PROCESSED....");
    

    if (authenticateAdmin(username, password)) {
        system("cls");
        cursor(10,3);
        printf("========Authentication successful! Welcome, Admin========\n");
        do {
           cursor(20,5);
            printf("========Admin Menu========");
            cursor(20,7);
            printf("1. Manage Inventory");
            cursor(20,9);
            printf("2. Manage Customers\n");
            cursor(20,11);
            printf("3. Manage Discounts\n");
            cursor(20,13);
            printf("4. Change Admin Credentials\n");
            cursor(20,15);
            printf("5. Exit Admin Menu\n");
            cursor(20,17); 
            printf("Enter your choice: ");
            scanf("%d", &choice);
            fflush( stdin ) ;

            switch (choice) {
                case 1:
                    system("cls") ;
                    manageInventory() ;
                    break;
                case 2:
                    system("cls") ;
                    manageCustomers() ;
                    break;
                case 3:
                    system("cls") ;
                    manageDiscounts() ;
                    break;
                case 4:
                    system("cls") ;
                    changeAdminKeys() ;
                    
                    system("cls") ;
                    break ;
                case 5:
                    cursor(20,20);
                    printf("Exiting Admin Menu.\n");
                    
                    system("cls") ;
                    break;
                default:
                    fflush(stdin) ;
                    cursor(20,20);
                    printf("Invalid choice! Please enter a valid option\n");
                    break ;
            }
        } while (choice != 5);
    } 
    else {
        cursor(15,18);
        printf("!!!!!Authentication failed! Access denied!!!!!.\n") ;
        
        system("cls");
    }
}

// Function to store checkout records in a file with date and time
void storeCheckoutRecords(struct Item *cart, int cartItemCount, char *name, int customerId , double *checkoutPrices ) {
    FILE *file = fopen("Records.txt", "a");
    if (file == NULL) {
        printf("Error opening file for storing records.\n");
        return;
    }

    // Get current date and time
    time_t currentTime;
    struct tm *localTime;
    time(&currentTime);
    localTime = localtime(&currentTime);

    double totalPrice = 0.0 ;
    if ( getDiscount( customerId ) > 0 ) {
        fprintf(file, "\n  (Old SignedUp Customer) Name : %s", name ) ;
        // Write timestamp to file
        fprintf(file, "\t\tCheckout Date and Time: %s", asctime(localTime));

        // Write cart details to file
        fprintf(file, "ID     |     Name     |  Price  | Quantity\n");
        for (int i = 0; i < cartItemCount; ++i) {
            fprintf(file, "%-6s | %-12s | %-7.2lf | %-3d\n", cart[i].id, cart[i].name, checkoutPrices[i] , cart[i].quantity);
            totalPrice += checkoutPrices[i] ;
        }
            fprintf(file, "\t\t\tTotal Purchase Price = %lf (With %lf%% Discount)\n", totalPrice, getDiscount(customerId) ) ;
    } 
    else {
        fprintf(file, "\n  (New Customer ) Name : %s", name ) ;
        fprintf(file, "\t\tCheckout Date and Time: %s", asctime(localTime));

        fprintf(file, "ID     |     Name     |  Price  | Quantity\n");
        for (int i = 0; i < cartItemCount; ++i) {
            fprintf(file, "%-6s | %-12s | %-7.2lf | %-3d\n", cart[i].id, cart[i].name, cart[i].price, cart[i].quantity);
            totalPrice += checkoutPrices[i] ;
        }
            fprintf(file, "\t\t\tTotal Purchase Price = %lf\n", totalPrice ) ;
    }
    fprintf(file, "------------------------------------\n");

    fclose(file);
}

// Function for shopping
void shopping(struct Customer customer) {
    int choice;
    int customerId = customer.id;
    double discount = getDiscount(customerId); // Get customer's discount
    struct Item cart[MAX_ITEMS]; // Array to hold items in the cart
    int cartItemCount = 0; // Variable to track the number of items in the cart
    char *NewCustomer = customer.name ;

    do {
        cursor(30, 3);
        printMenu("SHOPPING MENU ") ;
        // printf("Shopping-Menu:\n");
        // cursor(25, 4);
	    // printf("_____________") ;
        cursor( 20, 7 ) ;
        printf("1. Display Whole-Inventory\n");
        cursor( 20, 9 ) ;
        printf("2. Display Section Wise Items\n") ;
        cursor( 20, 11 ) ;
        printf("3. Add Item to Cart\n");
        cursor( 20, 13 ) ;
        printf("4. Remove Item from Cart\n");
        cursor( 20, 15 ) ;
        printf("5. Display Cart\n");
        cursor( 20, 17 ) ;
        printf("6. Checkout\n");
        cursor( 20, 19 ) ;
        printf("0. Exit Shopping\n");
        cursor( 20, 21 ) ;
        printf("\nEnter your choice: ");
        scanf("%d", &choice);
        fflush(stdin);

        switch (choice) {
            case 1:
                system("cls") ;
                displayInventory();
                // 33 for yellow color and 5 for blinking effect
                printf("\nPress any key to exit viewing.\n") ;
                getch( ) ;
                system("cls") ;
                break;
            case 2: {
                system("cls") ;
                int category_choice ;
                cursor( 10, 1 ) ;
                printMenu("SECTION-WISE INVENTORY ") ;
                // printf("Section-Wise Inventory\n") ;
                // cursor( 10, 2 ) ;
	            // printf("______________________\n\n") ;
                printf("\n\n1. Dairy\n\n") ;
                printf("2. Grains_and_cooking_products\n\n") ;
                printf("3. Snacks\n\n") ;
                printf("4. Beverages\n\n") ;
                printf("5. Others Section\n\n") ;
                printf("Enter the option_number: ") ;
                scanf("%d", &category_choice ) ;
                fflush( stdin ) ;
                if( category_choice == 1 ) {
                    printf("\n\tDairy Inventory:\n") ;
                    printf("ID     |     Name     |  Price  | Quantity\n") ;
                    for (int i = 0; i < itemCount; i++) {
                        if ( inventory[i].id[0] == 'D')
                        printf("%-6s | %-12s | %-7.2lf | %-3d\n", inventory[i].id, inventory[i].name, inventory[i].price, inventory[i].quantity);
                    }
                }
                else if( category_choice == 2 ) {
                    printf("\n\tGrains Inventory:\n") ;
                    printf("ID     |     Name     |  Price  | Quantity\n") ;
                    for (int i = 0; i < itemCount; i++) {
                        if ( inventory[i].id[0] == 'G' || inventory[i].id[0] == 'C' )
                            printf("%-6s | %-12s | %-7.2lf | %-3d\n", inventory[i].id, inventory[i].name, inventory[i].price, inventory[i].quantity);
                    }
                }
                else if( category_choice == 3 ) {
                    printf("\n\tSnacks Inventory:\n") ;
                    printf("ID     |     Name     |  Price  | Quantity\n") ;
                    for (int i = 0; i < itemCount; i++) {
                        if ( inventory[i].id[0] == 'S')
                            printf("%-6s | %-12s | %-7.2lf | %-3d\n", inventory[i].id, inventory[i].name, inventory[i].price, inventory[i].quantity);
                    }
                }
                else if( category_choice == 4 ) {
                    printf("\n\tBeverages Inventory:\n") ;
                    printf("ID     |     Name     |  Price  | Quantity\n") ;
                    for (int i = 0; i < itemCount; i++) {
                        if ( inventory[i].id[0] == 'B')
                            printf("%-6s | %-12s | %-7.2lf | %-3d\n", inventory[i].id, inventory[i].name, inventory[i].price, inventory[i].quantity);
                    }
                }
                else if( category_choice == 5 ) {
                    printf("\n\tOthers Inventory:\n") ;
                    printf("ID     |     Name     |  Price  | Quantity\n") ;
                    for (int i = 0; i < itemCount; i++) {
                        if ( inventory[i].id[0] == 'O')
                            printf("%-6s | %-12s | %-7.2lf | %-3d\n", inventory[i].id, inventory[i].name, inventory[i].price, inventory[i].quantity);
                    }
                }
                else {
                    printf("Invalid choice!\n ") ;
                }
                printf("\nEnter any key to go back to shopping...\n") ;
                getch( ) ;
                system("cls") ;
                break ;
            }
            case 3: { // Can declare varaibles in case-switch with delimitters
                char itemId[25] ; 
                int quantity;
                printf("\nEnter item ID to add to cart: ");
                fgets( itemId, sizeof(itemId), stdin ) ;
                itemId[ strcspn( itemId, "\n" ) ] = '\0' ;

                printf("\nEnter quantity: ");
                scanf("%d", &quantity);
                fflush( stdin ) ;
                // Find the item in inventory
                int itemIndex = -1;
                for (int i = 0; i < itemCount; ++i) {
                    if ( strcmp ( inventory[i].id , itemId ) == 0 ) {
                        itemIndex = i;
                        break;
                    }
                }

                if (itemIndex != -1 && inventory[itemIndex].quantity >= quantity) {
                    double totalPrice;
                    if ( discount > 0  ) {
                        // Calculate discounted price for existing customer
                        double discountedPrice = inventory[itemIndex].price * (1.0 - (discount / 100.0));
                        totalPrice = discountedPrice * quantity;
                    } 
                    else {
                        // Regular price for new customers (no discount)
                        totalPrice = inventory[itemIndex].price * quantity;
                    }

                    // Add item to the cart
                    cart[cartItemCount] = inventory[itemIndex];
                    cart[cartItemCount].quantity = quantity;
                    cartItemCount++;

                    // Display item details and total price
                    printf("\nAdded to Cart:\n");
                    printf("Item: %s | Quantity: %d | Total Price: %.2lf\n", inventory[itemIndex].name, quantity, totalPrice);
                    getch( ) ;
                    system("cls") ;

                    // Previously Used to Update inventory quantity after individual purchases
                    // inventory[itemIndex].quantity -= quantity;

                } 
                else {
                    system("cls") ;
                    printf("Item not found or insufficient quantity in inventory.(Max-%d Quantity In Stock).\n", inventory[itemIndex].quantity);
                }
                break;
            }

            case 4: {
                if (cartItemCount == 0) {
                    system("cls") ;
                    printf("\nCart is empty. Unable to remove.\n");
                } 
                else {
                    char itemIdToRemove[25];
                    printf("\nEnter item ID to remove from cart: ");
                    fgets(itemIdToRemove, sizeof(itemIdToRemove), stdin);
                    itemIdToRemove[strcspn(itemIdToRemove, "\n")] = '\0';

                    int removeIndex = -1;
                    for (int i = 0; i < cartItemCount; ++i) {
                        if (strcmp(cart[i].id, itemIdToRemove) == 0) {
                            removeIndex = i;
                            break;
                        }
                    }

                    system("cls") ;
                    if (removeIndex != -1) {
                        // Shift elements to remove the specified item from the cart
                        for (int i = removeIndex; i < cartItemCount - 1; ++i) {
                            cart[i] = cart[i + 1];
                        }
                        cartItemCount--;

                        printf("\n%s-Item removed from cart successfully.\n", itemIdToRemove );
                    } 
                    else {
                        printf("\n%s Item not found in the cart.\n", itemIdToRemove );
                    }
                }
                break;
            }

            case 5: {
                if (cartItemCount == 0) {
                    printf("\n\nCart is empty.\n");
                } 
                else {
                    int row = 29 ;
                    cursor( 55, 25 ) ;
                    printf("Cart Contents:\n");
                    cursor( 55, 26 ) ;
                    printf("______________\n") ;
                    cursor( 45, row ) ;
                    printf("ID     |     Name     |  Price  | Quantity\n") ;
                    double totalCartPrice = 0.0;
                    for (int i = 0; i < cartItemCount; ++i) {
                        cursor( 45, ++row ) ;
                        double itemPrice ;
                        if ( discount > 0 ) {
                            double discountedPrice = cart[i].price * (1.0 - (discount / 100.0));
                            itemPrice = discountedPrice * cart[i].quantity ;
                            printf("%-6s | %-12s | %-7.2lf | %-3d\n", cart[i].id, cart[i].name, itemPrice, cart[i].quantity);
                            totalCartPrice += itemPrice ;
                        }
                        else {
                            itemPrice = cart[i].price * cart[i].quantity;
                            printf("%-6s | %-12s | %-7.2lf | %-3d\n", cart[i].id, cart[i].name, cart[i].price, cart[i].quantity);
                            totalCartPrice += cart[i].price ;
                        }
                    }
                    printf("Total Cart Price: %.2lf\nPress any key to move back.\n", totalCartPrice) ;
                    getch( ) ;
                    system("cls") ;
                }
                break;
            }
            case 6: {
                system("cls") ;
                if (cartItemCount == 0) {
                    printf("Cart is empty. Unable to checkout.\n");
                } 
                else {
                    int row = 5 ;
                    printf("\nCheckout Process:\n");
                    printf("\nItems Purchased are:\n");
                    cursor( 15, row ) ;
                    printf("ID     |     Name     |  Price  | Quantity\n") ;
                    double totalCheckoutPrice = 0.0;
                    double checkoutPrices[cartItemCount] ;
                    for (int i = 0; i < cartItemCount; ++i) {
                        cursor( 15 , ++row ) ;
                        if ( discount > 0 ) {
                            double discountedPrice = cart[i].price * (1.0 - (discount / 100.0));
                            checkoutPrices[i] = discountedPrice * cart[i].quantity ;
                            printf("%-6s | %-12s | %-7.2lf | %-3d\n", cart[i].id, cart[i].name, checkoutPrices[i], cart[i].quantity);
                        }
                        else {
                            checkoutPrices[i] = cart[i].price * cart[i].quantity;
                            printf("%-6s | %-12s | %-7.2lf | %-3d\n", cart[i].id, cart[i].name, cart[i].price, cart[i].quantity);
                        }
                        totalCheckoutPrice += checkoutPrices[i] ;
                    }
                    cursor(0 , ++row ) ;
                    if( discount > 0 ) 
                        printf("\nTotal Checkout Price(With %.2lf%%-Discount): %.2lf\n",discount, totalCheckoutPrice) ;
                    else 
                        printf("\nTotal Checkout Price: %.2lf\n", totalCheckoutPrice) ;

                    // Perform checkout - Reset the cart
                    cursor(0 , ++row ) ;
                    printf("\n Checkout completed. Thanks you for shopping!\nWe hope to see you again soon!(Regards Group 3)\n") ;
                    getch( ) ;
                    system("cls") ;
                    for (int i = 0; i < cartItemCount; ++i) {
                        // Update quantity in inventory after purchase
                        for (int j = 0; j < itemCount; ++j) {
                            if ( strcmp( cart[i].id , inventory[j].id ) == 0 ) {
                                inventory[j].quantity -= cart[i].quantity;
                                break;
                            }
                        }
                    }
                    // Update the inventory file with the modified quantities
                    updateInventoryFile("inventory_data.txt");
                    // Store checkout records with date and time into Records.txt
                    storeCheckoutRecords(cart, cartItemCount, NewCustomer, customerId , checkoutPrices );

                    cartItemCount = 0; // Reset cart
                }
                break;
            }
            case 0:
                printf("Exiting Shopping...\n");
                
                system("cls") ;
                break;
            default:
                fflush(stdin) ;
                system("cls") ;
                printf("\nInvalid choice! Please enter a valid option.\n");
        }
    } while (choice != 0);
}

int main() {
    struct Customer newCustomer;
    char customerPhoneNum[20] ;
    int pin;
    int choice;
    loadCustomersWithDiscountsFromFile( ) ;
    loadInventoryFromFile("inventory_data.txt");

    do {
        // cursor(20, 3);
        // printf("========= GENERAL STORE MANAGEMENT SYSTEM =========\n");
        // cursor(20, 4);
	    // printf("\t      ********************************\n");
        cursor(13, 2) ;
        printMenu("GENERAL STORE MANAGEMENT SYSTEM ") ;
        cursor(35, 7) ;
	    printf("DEVELOPED BY GROUP 3");
        cursor(23, 10);
        printf("1. New Customer Shopping\n");
        cursor(23, 12);
        printf("2. Existing Customer Shopping\n");
        cursor(23, 14);
        printf("3. Admin Menu\n");
        cursor(23, 16);
        printf("0. Exit\n");
        cursor(23, 18);
        printf("\nEnter your choice: ");
        scanf("%d", &choice);
        fflush(stdin) ; // Used flush buffer where necessary to remove white spaces or new lines wrongly given in integer to avoid code breakage

        switch (choice) {
            case 1:
                system("cls") ;
                printf("\n\n!!CREATING NEW CUSTOMER ACCOUNT!!") ;
                newCustomer.id = customerCount + 1 ;
                printf("\n\nEnter customer name: ");
                fgets(newCustomer.name, sizeof(newCustomer.name), stdin ) ;
                newCustomer.name[ strcspn( newCustomer.name, "\n" ) ] = '\0' ;
                int check = 1 ;
                printf("\n\nEnter customer phone number: ");
                fgets(newCustomer.phoneNumber, sizeof(newCustomer.phoneNumber), stdin ) ;
                newCustomer.phoneNumber[ strcspn( newCustomer.phoneNumber, "\n" ) ] = '\0' ;
                fflush( stdin ) ;
                for(int i = 0; i < strlen( newCustomer.phoneNumber); ++i) {
                    if( ! ( newCustomer.phoneNumber[i] >= '0' && newCustomer.phoneNumber[i] <= '9' || ( newCustomer.phoneNumber[0] == '+' || newCustomer.phoneNumber[i] == '-' ) ) ) { 
                        check = 0  ;
                        break ;
                    }
                }

                if( check == 1 ) {
                    char c ;
                    printf("\n\nSet PIN for customer authentication(Only Numbers): ");
                    while( ( scanf("%d", &newCustomer.pin) ) != 1 ) {
                        printf("Invalid input.Please enter a valid number.\n") ;
                        while(( c = getchar() )!= '\n' && c != EOF ) ;
                        fflush( stdin );
                    }

                    for(int i = 0; i < customerCount; ++i) {
                        if( strcmp ( newCustomer.phoneNumber , customers[i].phoneNumber ) == 0 ) {
                            cursor(15, 18) ;
                            // ANSI escape code for red text
                            printf("%s Phone_Number is already registered.\n\t\t\t\tPlease Try Again!\n", newCustomer.phoneNumber ) ;
                            return 1 ;
                        }
                    }

                    system("cls");
                    printf("PLEASE WAIT....\n\nYOUR DATA IS BEING PROCESSED....");
                    

                    cursor(30, 10);

                    printf("ACCOUNT CREATED SUCCESSFULLY....");
                    cursor(0, 20);
                    addCustomer(newCustomer);
                    printf("Press enter to login");

                    getch();
                    // Perform shopping for the new customer
                    system("cls") ;
                    shopping(newCustomer) ;
                }
                else {
                    printf("Phone Number Format is wrong !\nCan start with (+ code) and contain hypens(-) but now other alphabets or characters.\n") ;
                    
                    system("cls") ;
                }
                break;
            case 2:
                system("cls") ;
                printf("\n\n!!LOGGING IN CURRENT CUSTOMER!!") ;
                printf("\n\nEnter customer phone number: ");
                fgets(customerPhoneNum, sizeof(customerPhoneNum), stdin) ;
                customerPhoneNum[ strcspn( customerPhoneNum, "\n" ) ] = '\0' ;
                fflush( stdin ) ;
                printf("\n\nEnter PIN for authentication: ");
                scanf("%d", &pin);
                fflush( stdin );

                // Authenticate existing customer
                int customerIndex = -1;
                for (int i = 0; i < customerCount; ++i) {
                    if ( ( strcmp ( customers[i].phoneNumber , customerPhoneNum ) == 0 ) && customers[i].pin == pin) {
                        customerIndex = i;
                        break;
                    }
                }

                system("cls");
                printf("PLEASE WAIT....\n\nFETCHING ACCOUNT DETAILS....");
                
                if (customerIndex != -1) {
                    cursor(30, 10);
                    printf("Authentication successful! Welcome back, %s.\n", customers[customerIndex].name);
                    cursor(0 , 20) ;
                    printf("Enter any key to login ...\n") ;
                    getch() ;
                    system("cls") ;
                    // Perform shopping for the existing customer
                    shopping(customers[customerIndex]);
                } 
                else {
                    system("cls") ;
                    cursor(0, 21) ;
                    printf("Authentication failed! Access denied.\n");
                }
                break;
            case 3:
                adminMenu();
                break;
            case 0:
                printf("\nThanks For Using Our App.\nRegards Group 3\nExiting...\n");
                break;
            default:
                system("cls") ;
                fflush(stdin) ;
                cursor(0, 21) ;
                printf("Invalid choice! Please enter a valid option.\n");
        }
    } while (choice != 0);

    return 0;
}