#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#define MAX_CANDIDATES 10

struct Candidate {
    char name[50];
    char aadhar[13]; 
    char address[100];  
    char signature[50]; 
    char photoFile[100];   
   // Increased size for full path
    unsigned char *photoData; 
    size_t photoSize;       
};

// Function to read photo file into memory
int readPhoto(struct Candidate *c)
 {
    FILE *file = fopen(c->photoFile, "rb"); 
// Open in binary mode
    if(file == NULL) 
{
        printf("Error: Cannot open photo file '%s'.\n", c->photoFile);
        printf("Make sure the file exists and path is correct.\n");
        return 0;
    }

    fseek(file, 0, SEEK_END);
    c->photoSize = ftell(file);
    fseek(file, 0, SEEK_SET);
    c->photoData = (unsigned char *)malloc(c->photoSize);
    if(c->photoData == NULL)
 {
        printf("Memory allocation failed for photo.\n");
        fclose(file);
        return 0;
    }

    size_t readBytes = fread(c->photoData, 1, c->photoSize, file);
    fclose(file);
    if(readBytes != c->photoSize) {
        printf("Error: Could not read complete photo file.\n");
        free(c->photoData);
        c->photoData = NULL;
        return 0;
    }

    return 1;
}

// Function to input candidate details
void inputCandidate(struct Candidate *c)
 {
    printf("\nEnter Name: ");
    scanf(" %[^\n]s", c->name);
    printf("Enter Aadhar Number: ");
    scanf("%s", c->aadhar);
    printf("Enter Address: ");
    scanf(" %[^\n]s", c->address);
    printf("Enter Signature: ");
    scanf(" %[^\n]s", c->signature);
    printf("Enter Photo Filename (with full path if not in program folder): ");
    scanf(" %[^\n]s", c->photoFile);
    if(readPhoto(c)) 
{
        printf("Photo loaded successfully (%zu bytes).\n", c->photoSize);
} 
else 
{
        printf("Photo could not be loaded. Please check file path.\n");
 }
}
// Function to verify signature
int verifySignature(struct Candidate *c, char inputSig[50])
 {
    return (strcmp(c->signature, inputSig) == 0);
}
// Function to find candidate by name
struct Candidate* findCandidate(struct Candidate *list, int count, char name[50]) 
{
    for(int i = 0; i < count; i++) 
    {
        if(strcmp(list[i].name, name) == 0)
        {
            return &list[i];
        }
    }
    return NULL;
}

// Function to print first N bytes of photo
void printPhotoBytes(struct Candidate *c, int n) {
    if(c->photoData == NULL) return;
    printf("First %d bytes of photo '%s':\n", n, c->photoFile);
    for(int i = 0; i < n && i < c->photoSize; i++) {
        printf("%02X ", c->photoData[i]);  
// Hexadecimal
        if((i+1) % 16 == 0) printf("\n");
    }
    printf("\n");
}

int main() {
    struct Candidate candidates[MAX_CANDIDATES];
    int candidateCount = 0;
    printf("=== Biometric System: Candidate Registration ===\n");
    int n;
    printf("How many candidates do you want to register? ");
    scanf("%d", &n);
    if(n > MAX_CANDIDATES) n = MAX_CANDIDATES;
    for(int i = 0; i < n; i++) {
        printf("\n--- Candidate %d ---\n", i+1);
        inputCandidate(&candidates[i]);
        candidateCount++;
    }
    char searchName[50];
    char inputSig[50];
    printf("\n=== Signature Verification ===\n");
    printf("Enter candidate name to verify: ");
    scanf(" %[^\n]s", searchName);
    struct Candidate *cptr = findCandidate(candidates, candidateCount, searchName);
    if(cptr == NULL) 
    {
        printf("Candidate not found.\n");
    } 
    else 
    {
        printf("Enter signature to verify: ");
        scanf(" %[^\n]s", inputSig);
        if(verifySignature(cptr, inputSig))
        {
            printf("\nSignature Matched! Verification Successful.\n");
            printf("Candidate Name: %s\n", cptr->name);
            printf("Aadhar Number: %s\n", cptr->aadhar);
            printf("Address: %s\n", cptr->address);
            printf("Photo Filename: %s\n", cptr->photoFile);
            printf("Photo Size: %zu bytes\n", cptr->photoSize);
            // Print first 100 bytes of photo
            printPhotoBytes(cptr, 100);
        }
        else
        {
            printf("\nSignature Not Matched! Verification Failed.\n");
        }
    }
    // Free allocated photo memory
    for(int i = 0; i < candidateCount; i++)
     {
        if(candidates[i].photoData != NULL)
            free(candidates[i].photoData);
    }

    return 0;
}