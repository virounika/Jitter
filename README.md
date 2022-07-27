# Jitter

#include <iostream>
#include <cstring>
#include <cassert>
using namespace std;

// nStandards is the number of match standards in the collection

const int MAX_WORD_LENGTH = 20;
int editStandards(int distance[],
                  char word1[][MAX_WORD_LENGTH + 1],
                  char word2[][MAX_WORD_LENGTH + 1],
                  int nStandards) {
    // countStandards returned by function:
    int countStandards = nStandards;
    int index = 0;
    
    // Treat negative nStandards as no match standards:
    if (nStandards < 0) {
        return 0;
    }
    // Check for empty strings and negative distances:
    for (int i = 0; i < nStandards; i++) {
        if ((strlen(word1[index]) == 0) | (strlen(word2[index]) == 0) | (distance[index] < 0)) {
            // Branch taken if any string is empty or distance negative:
            // Visit countStandards - 1 to move all following strings up:
            for (int j = index; j < countStandards - 1; j++) {
                // Copy string/distance after index to index:
                strcpy(word1[j], word1[j + 1]);
                strcpy(word2[j], word2[j + 1]);
                distance[j] = distance[j + 1];
            }
            // Invalid string requires decrement of countStandards:
            countStandards--;
        } else {
            // if valid, continue visiting each string:
            index++;
        }
    }
    // Check for non-alphabetic characters:
    nStandards = countStandards;
    index = 0;
    for (int i = 0; i < nStandards; i++) {
        bool foundInvalid = false;
        for (int j = 0; j < strlen(word1[index]); j++) {
            if (isalpha(word1[index][j])) {
                word1[index][j] = tolower(word1[index][j]);
            }
            else {
                // If non-alphabetic character:
                for (int k = index; k < countStandards - 1; k++) {
                    strcpy(word1[k], word1[k + 1]);
                    strcpy(word2[k], word2[k + 1]);
                    distance[k] = distance[k + 1];
                }
                countStandards--;
                foundInvalid = true;
                // decrement countStandards max once if not letter at same index, exit k loop:
                break;
            }
        }
        // If non-alphabetic character detected, don't go to next loop:
        if (foundInvalid) {
            continue;
        }
        // Loop only entered if character in word1 is alphabetic character:
        for (int j = 0; j < strlen(word2[index]); j++) {
            if (isalpha(word2[index][j])) {
                word2[index][j] = tolower(word2[index][j]);
            }
            else {
                for (int k = index; k < countStandards - 1; k++) {
                    strcpy(word1[k], word1[k + 1]);
                    strcpy(word2[k], word2[k + 1]);
                    distance[k] = distance[k + 1];
                }
                countStandards--;
                foundInvalid = true;
                break;
            }
        }
        // If invalid, continue to next iteration of i loop:
        if (foundInvalid) {
            continue;
        } else {
            // Check next row if word1 and word2 at index are valid:
            index++;
        }
    }
    // Check for duplicates:
    nStandards = countStandards;
    // Loop through each word:
    for (int i = 0; i < nStandards - 1; i++) {
        if (i >= countStandards - 1) {
            break;
        }
        // For each word, loop through all the words after it:
        for (int j = i + 1; j < nStandards; j++) {
            if (j >= countStandards) {
                break;
            }
            // If the words are the same:
            if ((strcmp(word1[i], word1[j]) == 0) && (strcmp(word2[i], word2[j]) == 0)) {
                distance[i] = max(distance[i], distance[j]);
                // Delete the word after it:
                for (int k = j; k < countStandards - 1; k++) {
                    strcpy(word1[k], word1[k + 1]);
                    strcpy(word2[k], word2[k + 1]);
                    distance[k] = distance[k + 1];
                }
                // Decrement j to continue checking if w1 and w2 at removed position i is a match too:
                j--;
                countStandards--;
            }
        }
    }
    return countStandards;
}

const int MAX_JEET_LENGTH = 280;
int determineMatchLevel(const int distance[],
                        const char word1[][MAX_WORD_LENGTH + 1],
                        const char word2[][MAX_WORD_LENGTH + 1],
                        int nStandards,
                        const char jeet[]) {
    // Treat negative nStandards as no match standards:
    if (nStandards < 0) {
        return 0;
    }
    // Create new 2d array to store each word of jeet into:
    char words[MAX_JEET_LENGTH + 1][MAX_JEET_LENGTH + 1];
    int wordIndex = 0;
    int letterIndex = 0;
    for (int i = 0; i < strlen(jeet); i++) {
        // If the jeet is a letter at position i, store it in words, move on to next char:
        if (isalpha(jeet[i])) {
            words[wordIndex][letterIndex] = tolower(jeet[i]);
            letterIndex++;
            // If there is a space:
        } else if (isblank(jeet[i])) {
            // If previous character(s) filled up letterIndex at 0, move on to next row of array:
            // Otherwise ignore if multiple spaces:
            if (letterIndex != 0) {
                wordIndex++;
            }
            // Start new word at position 0 if there is minimum of one space:
            letterIndex = 0;
        } else {
            // If not a letter:
            continue;
        }
    }
    // To be used in next loop:
    // Add one row to ensure last row of characters is not skipped:
    if (letterIndex != 0) {
        wordIndex++;
    }
    
    int result = 0;
    // Count matches in jeet:
    for (int i = 0; i < nStandards; i++) {
        // Repeatedly compare word1 word with all words in a jeet:
        for (int j = 0; j < wordIndex; j++) {
            bool matchFound = false;
            // If a word in word1 matches one in the jeet:
            if (strcmp(word1[i], words[j]) == 0) {
                // Repeatedly check if match is within k distances:
                    // For matching: maximum distance allowed is distance[i]
                for (int k = 1; k <= distance[i]; k++) {
                    // If match is not found because following words[] runs out:
                    if (j + k >= wordIndex) {
                        break;
                    }
                    // If word1 matches, and word2 is within k distances:
                    if (strcmp(word2[i], words[j + k]) == 0) {
                        result++;
                        matchFound = true;
                        break;
                    }
                }
                // Allow standard to contribute only once to match level:
                if (matchFound) {
                    break;
                }
            }
        }
    }

    return result;
}

int main() {
   /* const int TEST1_NSTANDARDS = 4;
    int test1distance[TEST1_NSTANDARDS] = {
    2, 4, 1, 3 };
    char test1word1[TEST1_NSTANDARDS][MAX_WORD_LENGTH+1] = { "eccentric", "eccentric", "ECCENTRIC", "eccentric"};
    char test1word2[TEST1_NSTANDARDS][MAX_WORD_LENGTH+1] = {
    "billionaire", "billionaire", "BILLIONAIRE", "BILLIONAIRE"};
    
    assert(editStandards(test1distance, test1word1, test1word2, TEST1_NSTANDARDS) == 1);
    cerr << test1word1[0] << " " << test1word2[0] << endl;
   */
    
    
    const int TEST1_NSTANDARDS = 4;
    int test1dist[TEST1_NSTANDARDS] = {
    2, 2, 1, 13 };
    char test1w1[TEST1_NSTANDARDS][MAX_WORD_LENGTH+1] = { "eccentric", "space", "electric", "were"
    };
    char test1w2[TEST1_NSTANDARDS][MAX_WORD_LENGTH+1] = { "billionaire", "capsule", "car", "eccentric" };

    assert(determineMatchLevel(test1dist, test1w1, test1w2, TEST1_NSTANDARDS, "The eccentric billionaire launched a space station cargo capsule.") == 1);
    
    assert(determineMatchLevel(test1dist, test1w1, test1w2, TEST1_NSTANDARDS, "eccentric billionaire eccentric billionaire eccentric billionaire") == 1);

    assert(determineMatchLevel(test1dist, test1w1, test1w2, TEST1_NSTANDARDS, "eccentri3c      b2illionaire ec2centric      bil2lionaire     ec2centric  bil!lionaire") == 1);
    
    assert(determineMatchLevel(test1dist, test1w1, test1w2, TEST1_NSTANDARDS, "eccentri3cb2illionaire ec2centricbil2lionaireec2centricbil!lionaire") == 0);
    
    cerr << "All tests succeeded" << endl;
}
