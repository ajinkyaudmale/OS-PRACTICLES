# OS Practical Solutions — Simple C with Dynamic Input

---

## 1. Banker's Algorithm for Deadlock Avoidance

```c
#include <stdio.h>

int main() {
    int n, m;
    printf("Enter number of processes: ");
    scanf("%d", &n);
    printf("Enter number of resources: ");
    scanf("%d", &m);

    int allocation[10][10], max[10][10], need[10][10];
    int available[10], work[10], finish[10] = {0}, safeSeq[10];

    printf("Enter Allocation matrix:\n");
    for (int i = 0; i < n; i++)
        for (int j = 0; j < m; j++)
            scanf("%d", &allocation[i][j]);

    printf("Enter Max matrix:\n");
    for (int i = 0; i < n; i++)
        for (int j = 0; j < m; j++)
            scanf("%d", &max[i][j]);

    printf("Enter Available resources:\n");
    for (int j = 0; j < m; j++)
        scanf("%d", &available[j]);

    for (int i = 0; i < n; i++)
        for (int j = 0; j < m; j++)
            need[i][j] = max[i][j] - allocation[i][j];

    for (int j = 0; j < m; j++)
        work[j] = available[j];

    int count = 0;
    while (count < n) {
        int found = 0;
        for (int i = 0; i < n; i++) {
            if (!finish[i]) {
                int ok = 1;
                for (int j = 0; j < m; j++)
                    if (need[i][j] > work[j]) { ok = 0; break; }
                if (ok) {
                    for (int j = 0; j < m; j++)
                        work[j] += allocation[i][j];
                    safeSeq[count++] = i;
                    finish[i] = 1;
                    found = 1;
                }
            }
        }
        if (!found) { printf("System is NOT in safe state.\n"); return 1; }
    }

    printf("System is in SAFE state.\nSafe Sequence: ");
    for (int i = 0; i < n; i++)
        printf("P%d%s", safeSeq[i], i < n - 1 ? " -> " : "\n");

    return 0;
}
```

### Sample Input

```
Enter number of processes: 5
Enter number of resources: 3
Enter Allocation matrix:
0 1 0
2 0 0
3 0 2
2 1 1
0 0 2
Enter Max matrix:
7 5 3
3 2 2
9 0 2
2 2 2
4 3 3
Enter Available resources:
3 3 2
```

### Output

```
System is in SAFE state.
Safe Sequence: P1 -> P3 -> P4 -> P0 -> P2
```

---

## 2. Disk Scheduling — FCFS

```c
#include <stdio.h>
#include <stdlib.h>

int main() {
    int n, head, total = 0;

    printf("Enter number of disk requests: ");
    scanf("%d", &n);

    int req[n];
    printf("Enter disk requests:\n");
    for (int i = 0; i < n; i++)
        scanf("%d", &req[i]);

    printf("Enter initial head position: ");
    scanf("%d", &head);

    printf("\nFCFS Sequence: %d", head);
    for (int i = 0; i < n; i++) {
        total += abs(req[i] - head);
        head = req[i];
        printf(" -> %d", head);
    }

    printf("\nTotal Head Movement: %d\n", total);
    return 0;
}
```

### Sample Input

```
Enter number of disk requests: 8
Enter disk requests:
98 183 37 122 14 124 65 67
Enter initial head position: 53
```

### Output

```
FCFS Sequence: 53 -> 98 -> 183 -> 37 -> 122 -> 14 -> 124 -> 65 -> 67
Total Head Movement: 640
```

---

## 3. Disk Scheduling — SCAN

```c
#include <stdio.h>
#include <stdlib.h>

int cmp(const void *a, const void *b) { return *(int*)a - *(int*)b; }

int main() {
    int n, head, disk_size, total = 0;

    printf("Enter number of disk requests: ");
    scanf("%d", &n);

    int req[n];
    printf("Enter disk requests:\n");
    for (int i = 0; i < n; i++)
        scanf("%d", &req[i]);

    printf("Enter initial head position: ");
    scanf("%d", &head);
    printf("Enter disk size: ");
    scanf("%d", &disk_size);

    qsort(req, n, sizeof(int), cmp);

    int pos = 0;
    while (pos < n && req[pos] < head) pos++;

    int prev = head;
    printf("\nSCAN Sequence: %d", head);

    for (int i = pos; i < n; i++) {
        total += abs(req[i] - prev);
        prev = req[i];
        printf(" -> %d", prev);
    }
    total += abs((disk_size - 1) - prev);
    prev = disk_size - 1;
    printf(" -> %d", prev);

    for (int i = pos - 1; i >= 0; i--) {
        total += abs(req[i] - prev);
        prev = req[i];
        printf(" -> %d", prev);
    }

    printf("\nTotal Head Movement: %d\n", total);
    return 0;
}
```

### Sample Input

```
Enter number of disk requests: 8
Enter disk requests:
98 183 37 122 14 124 65 67
Enter initial head position: 53
Enter disk size: 200
```

### Output

```
SCAN Sequence: 53 -> 65 -> 67 -> 98 -> 122 -> 124 -> 183 -> 199 -> 37 -> 14
Total Head Movement: 331
```

---

## 4. Disk Scheduling — C-SCAN

```c
#include <stdio.h>
#include <stdlib.h>

int cmp(const void *a, const void *b) { return *(int*)a - *(int*)b; }

int main() {
    int n, head, disk_size, total = 0;

    printf("Enter number of disk requests: ");
    scanf("%d", &n);

    int req[n];
    printf("Enter disk requests:\n");
    for (int i = 0; i < n; i++)
        scanf("%d", &req[i]);

    printf("Enter initial head position: ");
    scanf("%d", &head);
    printf("Enter disk size: ");
    scanf("%d", &disk_size);

    qsort(req, n, sizeof(int), cmp);

    int pos = 0;
    while (pos < n && req[pos] < head) pos++;

    int prev = head;
    printf("\nC-SCAN Sequence: %d", head);

    for (int i = pos; i < n; i++) {
        total += abs(req[i] - prev);
        prev = req[i];
        printf(" -> %d", prev);
    }
    total += abs((disk_size - 1) - prev);
    prev = disk_size - 1;
    printf(" -> %d", prev);

    total += disk_size - 1;
    prev = 0;
    printf(" -> 0");

    for (int i = 0; i < pos; i++) {
        total += abs(req[i] - prev);
        prev = req[i];
        printf(" -> %d", prev);
    }

    printf("\nTotal Head Movement: %d\n", total);
    return 0;
}
```

### Sample Input

```
Enter number of disk requests: 8
Enter disk requests:
98 183 37 122 14 124 65 67
Enter initial head position: 53
Enter disk size: 200
```

### Output

```
C-SCAN Sequence: 53 -> 65 -> 67 -> 98 -> 122 -> 124 -> 183 -> 199 -> 0 -> 14 -> 37
Total Head Movement: 382
```

---

## 5. Disk Scheduling — LOOK

```c
#include <stdio.h>
#include <stdlib.h>

int cmp(const void *a, const void *b) { return *(int*)a - *(int*)b; }

int main() {
    int n, head, total = 0;

    printf("Enter number of disk requests: ");
    scanf("%d", &n);

    int req[n];
    printf("Enter disk requests:\n");
    for (int i = 0; i < n; i++)
        scanf("%d", &req[i]);

    printf("Enter initial head position: ");
    scanf("%d", &head);

    qsort(req, n, sizeof(int), cmp);

    int pos = 0;
    while (pos < n && req[pos] < head) pos++;

    int prev = head;
    printf("\nLOOK Sequence: %d", head);

    for (int i = pos; i < n; i++) {
        total += abs(req[i] - prev);
        prev = req[i];
        printf(" -> %d", prev);
    }
    for (int i = pos - 1; i >= 0; i--) {
        total += abs(req[i] - prev);
        prev = req[i];
        printf(" -> %d", prev);
    }

    printf("\nTotal Head Movement: %d\n", total);
    return 0;
}
```

### Sample Input

```
Enter number of disk requests: 8
Enter disk requests:
98 183 37 122 14 124 65 67
Enter initial head position: 53
```

### Output

```
LOOK Sequence: 53 -> 65 -> 67 -> 98 -> 122 -> 124 -> 183 -> 37 -> 14
Total Head Movement: 299
```

---

## 6. Disk Scheduling — C-LOOK

```c
#include <stdio.h>
#include <stdlib.h>

int cmp(const void *a, const void *b) { return *(int*)a - *(int*)b; }

int main() {
    int n, head, total = 0;

    printf("Enter number of disk requests: ");
    scanf("%d", &n);

    int req[n];
    printf("Enter disk requests:\n");
    for (int i = 0; i < n; i++)
        scanf("%d", &req[i]);

    printf("Enter initial head position: ");
    scanf("%d", &head);

    qsort(req, n, sizeof(int), cmp);

    int pos = 0;
    while (pos < n && req[pos] < head) pos++;

    int prev = head;
    printf("\nC-LOOK Sequence: %d", head);

    for (int i = pos; i < n; i++) {
        total += abs(req[i] - prev);
        prev = req[i];
        printf(" -> %d", prev);
    }
    if (pos > 0) {
        total += abs(req[0] - prev);
        prev = req[0];
        printf(" -> %d", prev);
        for (int i = 1; i < pos; i++) {
            total += abs(req[i] - prev);
            prev = req[i];
            printf(" -> %d", prev);
        }
    }

    printf("\nTotal Head Movement: %d\n", total);
    return 0;
}
```

### Sample Input

```
Enter number of disk requests: 8
Enter disk requests:
98 183 37 122 14 124 65 67
Enter initial head position: 53
```

### Output

```
C-LOOK Sequence: 53 -> 65 -> 67 -> 98 -> 122 -> 124 -> 183 -> 14 -> 37
Total Head Movement: 322
```

---

## 7. Disk Scheduling — SSTF

```c
#include <stdio.h>
#include <stdlib.h>

int main() {
    int n, head, total = 0;

    printf("Enter number of disk requests: ");
    scanf("%d", &n);

    int req[n], visited[n];
    printf("Enter disk requests:\n");
    for (int i = 0; i < n; i++) {
        scanf("%d", &req[i]);
        visited[i] = 0;
    }

    printf("Enter initial head position: ");
    scanf("%d", &head);

    printf("\nSSTF Sequence: %d", head);

    for (int c = 0; c < n; c++) {
        int minDist = 99999, idx = -1;
        for (int i = 0; i < n; i++) {
            if (!visited[i] && abs(req[i] - head) < minDist) {
                minDist = abs(req[i] - head);
                idx = i;
            }
        }
        visited[idx] = 1;
        total += minDist;
        head = req[idx];
        printf(" -> %d", head);
    }

    printf("\nTotal Head Movement: %d\n", total);
    return 0;
}
```

### Sample Input

```
Enter number of disk requests: 8
Enter disk requests:
98 183 37 122 14 124 65 67
Enter initial head position: 53
```

### Output

```
SSTF Sequence: 53 -> 65 -> 67 -> 37 -> 14 -> 98 -> 122 -> 124 -> 183
Total Head Movement: 236
```

---

## 8. Indexed File Allocation

```c
#include <stdio.h>

int main() {
    int n;
    printf("Enter number of blocks in index: ");
    scanf("%d", &n);

    int index[n];
    printf("Enter block numbers:\n");
    for (int i = 0; i < n; i++)
        scanf("%d", &index[i]);

    printf("\nIndexed File Allocation\n");
    printf("Index Block: ");
    for (int i = 0; i < n; i++)
        printf("%d ", index[i]);

    printf("\n\nSlot -> Data Block\n");
    for (int i = 0; i < n; i++)
        printf("  [%d]  ->  Block %d\n", i, index[i]);

    return 0;
}
```

### Sample Input

```
Enter number of blocks in index: 5
Enter block numbers:
4 7 2 9 1
```

### Output

```
Indexed File Allocation
Index Block: 4 7 2 9 1

Slot -> Data Block
  [0]  ->  Block 4
  [1]  ->  Block 7
  [2]  ->  Block 2
  [3]  ->  Block 9
  [4]  ->  Block 1
```

---

## 9. Sequential (Contiguous) File Allocation

```c
#include <stdio.h>

int main() {
    int disk_size, start, length;

    printf("Enter total disk size: ");
    scanf("%d", &disk_size);

    int disk[disk_size];
    for (int i = 0; i < disk_size; i++)
        disk[i] = 0;

    printf("Enter start block: ");
    scanf("%d", &start);
    printf("Enter file length (blocks): ");
    scanf("%d", &length);

    if (start + length > disk_size) {
        printf("Error: Not enough space!\n");
        return 1;
    }

    for (int i = start; i < start + length; i++)
        disk[i] = 1;

    printf("\nSequential File Allocation\n");
    printf("Disk Block Status:\n");
    for (int i = 0; i < disk_size; i++)
        printf("  Block %2d: %s\n", i, disk[i] ? "Allocated" : "Free");

    printf("\nBlocks allocated to file: ");
    for (int i = start; i < start + length; i++)
        printf("%d ", i);
    printf("\n");

    return 0;
}
```

### Sample Input

```
Enter total disk size: 10
Enter start block: 2
Enter file length (blocks): 4
```

### Output

```
Sequential File Allocation
Disk Block Status:
  Block  0: Free
  Block  1: Free
  Block  2: Allocated
  Block  3: Allocated
  Block  4: Allocated
  Block  5: Allocated
  Block  6: Free
  Block  7: Free
  Block  8: Free
  Block  9: Free

Blocks allocated to file: 2 3 4 5
```

---

## 10. Linked File Allocation

```c
#include <stdio.h>
#include <stdlib.h>

struct Block {
    int blockNum;
    struct Block *next;
};

int main() {
    int n;
    printf("Enter number of blocks for file: ");
    scanf("%d", &n);

    int blockNums[n];
    printf("Enter block numbers:\n");
    for (int i = 0; i < n; i++)
        scanf("%d", &blockNums[i]);

    struct Block *head = NULL, *tail = NULL;

    for (int i = 0; i < n; i++) {
        struct Block *newBlock = (struct Block*)malloc(sizeof(struct Block));
        newBlock->blockNum = blockNums[i];
        newBlock->next = NULL;
        if (!head) head = tail = newBlock;
        else { tail->next = newBlock; tail = newBlock; }
    }

    printf("\nLinked File Allocation\n");
    printf("File Block Chain:\n");
    struct Block *cur = head;
    int idx = 0;
    while (cur) {
        printf("  Block[%d] = %d -> %s\n", idx++, cur->blockNum,
               cur->next ? "Next Block" : "NULL (End)");
        cur = cur->next;
    }

    cur = head;
    while (cur) { struct Block *tmp = cur->next; free(cur); cur = tmp; }

    return 0;
}
```

### Sample Input

```
Enter number of blocks for file: 5
Enter block numbers:
4 7 2 9 5
```

### Output

```
Linked File Allocation
File Block Chain:
  Block[0] = 4 -> Next Block
  Block[1] = 7 -> Next Block
  Block[2] = 2 -> Next Block
  Block[3] = 9 -> Next Block
  Block[4] = 5 -> NULL (End)
```

---

## 11. Find Minimum from N Random Numbers

```c
#include <stdio.h>
#include <stdlib.h>
#include <time.h>

int main() {
    int n;
    printf("Enter count of random numbers: ");
    scanf("%d", &n);

    int arr[n];
    srand(time(0));

    printf("Generated Numbers: ");
    for (int i = 0; i < n; i++) {
        arr[i] = rand() % 10000;
        printf("%d ", arr[i]);
    }

    int min = arr[0];
    for (int i = 1; i < n; i++)
        if (arr[i] < min) min = arr[i];

    printf("\nMinimum value: %d\n", min);
    return 0;
}
```

### Sample Input

```
Enter count of random numbers: 10
```

### Output

```
Generated Numbers: 8324 1532 7841 294 6723 4891 9012 573 3847 2109
Minimum value: 294
```

---

## 12. Find Maximum from N Random Numbers

```c
#include <stdio.h>
#include <stdlib.h>
#include <time.h>

int main() {
    int n;
    printf("Enter count of random numbers: ");
    scanf("%d", &n);

    int arr[n];
    srand(time(0));

    printf("Generated Numbers: ");
    for (int i = 0; i < n; i++) {
        arr[i] = rand() % 10000;
        printf("%d ", arr[i]);
    }

    int max = arr[0];
    for (int i = 1; i < n; i++)
        if (arr[i] > max) max = arr[i];

    printf("\nMaximum value: %d\n", max);
    return 0;
}
```

### Sample Input

```
Enter count of random numbers: 10
```

### Output

```
Generated Numbers: 8324 1532 7841 294 6723 4891 9012 573 3847 2109
Maximum value: 9012
```

---

## 13. Sum of All Even Numbers from N Random Numbers

```c
#include <stdio.h>
#include <stdlib.h>
#include <time.h>

int main() {
    int n;
    printf("Enter count of random numbers: ");
    scanf("%d", &n);

    int arr[n];
    srand(time(0));

    printf("Generated Numbers: ");
    for (int i = 0; i < n; i++) {
        arr[i] = rand() % 10000;
        printf("%d ", arr[i]);
    }

    long long sum = 0;
    printf("\nEven Numbers: ");
    for (int i = 0; i < n; i++) {
        if (arr[i] % 2 == 0) {
            printf("%d ", arr[i]);
            sum += arr[i];
        }
    }

    printf("\nSum of Even Numbers: %lld\n", sum);
    return 0;
}
```

### Sample Input

```
Enter count of random numbers: 10
```

### Output

```
Generated Numbers: 8324 1532 7841 294 6723 4892 9011 572 3847 2108
Even Numbers: 8324 1532 294 4892 572 2108
Sum of Even Numbers: 17722
```

---

## 14. Sum of All Odd Numbers from N Random Numbers

```c
#include <stdio.h>
#include <stdlib.h>
#include <time.h>

int main() {
    int n;
    printf("Enter count of random numbers: ");
    scanf("%d", &n);

    int arr[n];
    srand(time(0));

    printf("Generated Numbers: ");
    for (int i = 0; i < n; i++) {
        arr[i] = rand() % 10000;
        printf("%d ", arr[i]);
    }

    long long sum = 0;
    printf("\nOdd Numbers: ");
    for (int i = 0; i < n; i++) {
        if (arr[i] % 2 != 0) {
            printf("%d ", arr[i]);
            sum += arr[i];
        }
    }

    printf("\nSum of Odd Numbers: %lld\n", sum);
    return 0;
}
```

### Sample Input

```
Enter count of random numbers: 10
```

### Output

```
Generated Numbers: 8324 1532 7841 294 6723 4892 9011 572 3847 2108
Odd Numbers: 7841 6723 9011 3847
Sum of Odd Numbers: 27422
```

---

## 15. Sum of All N Random Numbers

```c
#include <stdio.h>
#include <stdlib.h>
#include <time.h>

int main() {
    int n;
    printf("Enter count of random numbers: ");
    scanf("%d", &n);

    int arr[n];
    srand(time(0));

    printf("Generated Numbers: ");
    for (int i = 0; i < n; i++) {
        arr[i] = rand() % 10000;
        printf("%d ", arr[i]);
    }

    long long sum = 0;
    for (int i = 0; i < n; i++)
        sum += arr[i];

    printf("\nSum of all %d numbers: %lld\n", n, sum);
    return 0;
}
```

### Sample Input

```
Enter count of random numbers: 10
```

### Output

```
Generated Numbers: 8324 1532 7841 294 6723 4892 9011 572 3847 2108
Sum of all 10 numbers: 45144
```

---

> **Note:** Output for programs 11–15 will vary each run due to random number generation. For programs 1–7, use the same sample input to reproduce the exact output shown.
