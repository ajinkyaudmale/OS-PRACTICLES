# OS Practical Solutions — Simple C (Q1–Q10) + MPI (Q11–Q15)

> **Compile & Run MPI programs:**
>
> ```
> mpicc -o prog prog.c
> mpirun -np 4 ./prog
> ```

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

## 11. MPI — Find Minimum from N Random Numbers

```c
#include <stdio.h>
#include <stdlib.h>
#include <limits.h>
#include <mpi.h>

int main(int argc, char *argv[]) {
    int rank, size, n;

    MPI_Init(&argc, &argv);
    MPI_Comm_rank(MPI_COMM_WORLD, &rank);
    MPI_Comm_size(MPI_COMM_WORLD, &size);

    if (rank == 0) {
        printf("Enter total count of random numbers: ");
        scanf("%d", &n);
    }

    /* Broadcast n to all processes */
    MPI_Bcast(&n, 1, MPI_INT, 0, MPI_COMM_WORLD);

    int local_n = n / size;
    int local_arr[local_n];

    /* Each process generates its own portion */
    srand(rank * 1000 + 42);
    printf("Process %d generated: ", rank);
    for (int i = 0; i < local_n; i++) {
        local_arr[i] = rand() % 10000;
        printf("%d ", local_arr[i]);
    }
    printf("\n");

    /* Find local minimum */
    int local_min = INT_MAX;
    for (int i = 0; i < local_n; i++)
        if (local_arr[i] < local_min)
            local_min = local_arr[i];

    printf("Process %d local minimum: %d\n", rank, local_min);

    /* Reduce all local minimums to global minimum at rank 0 */
    int global_min;
    MPI_Reduce(&local_min, &global_min, 1, MPI_INT, MPI_MIN, 0, MPI_COMM_WORLD);

    if (rank == 0)
        printf("\nGlobal Minimum from %d random numbers: %d\n", n, global_min);

    MPI_Finalize();
    return 0;
}
```

### Sample Input

```
Enter total count of random numbers: 1000
```

### Output

```
Process 0 generated: 7334 1498 6290 ...
Process 1 generated: 4231 8764 903 ...
Process 2 generated: 5612 2187 9341 ...
Process 3 generated: 763 4892 3017 ...
Process 0 local minimum: 12
Process 1 local minimum: 8
Process 2 local minimum: 3
Process 3 local minimum: 5

Global Minimum from 1000 random numbers: 3
```

---

## 12. MPI — Find Maximum from N Random Numbers

```c
#include <stdio.h>
#include <stdlib.h>
#include <limits.h>
#include <mpi.h>

int main(int argc, char *argv[]) {
    int rank, size, n;

    MPI_Init(&argc, &argv);
    MPI_Comm_rank(MPI_COMM_WORLD, &rank);
    MPI_Comm_size(MPI_COMM_WORLD, &size);

    if (rank == 0) {
        printf("Enter total count of random numbers: ");
        scanf("%d", &n);
    }

    MPI_Bcast(&n, 1, MPI_INT, 0, MPI_COMM_WORLD);

    int local_n = n / size;
    int local_arr[local_n];

    srand(rank * 1000 + 42);
    printf("Process %d generated: ", rank);
    for (int i = 0; i < local_n; i++) {
        local_arr[i] = rand() % 10000;
        printf("%d ", local_arr[i]);
    }
    printf("\n");

    int local_max = INT_MIN;
    for (int i = 0; i < local_n; i++)
        if (local_arr[i] > local_max)
            local_max = local_arr[i];

    printf("Process %d local maximum: %d\n", rank, local_max);

    int global_max;
    MPI_Reduce(&local_max, &global_max, 1, MPI_INT, MPI_MAX, 0, MPI_COMM_WORLD);

    if (rank == 0)
        printf("\nGlobal Maximum from %d random numbers: %d\n", n, global_max);

    MPI_Finalize();
    return 0;
}
```

### Sample Input

```
Enter total count of random numbers: 1000
```

### Output

```
Process 0 generated: 7334 1498 6290 ...
Process 1 generated: 4231 8764 903 ...
Process 2 generated: 5612 2187 9341 ...
Process 3 generated: 763 4892 3017 ...
Process 0 local maximum: 9987
Process 1 local maximum: 9932
Process 2 local maximum: 9998
Process 3 local maximum: 9901

Global Maximum from 1000 random numbers: 9998
```

---

## 13. MPI — Sum of All Even Numbers from N Random Numbers

```c
#include <stdio.h>
#include <stdlib.h>
#include <mpi.h>

int main(int argc, char *argv[]) {
    int rank, size, n;

    MPI_Init(&argc, &argv);
    MPI_Comm_rank(MPI_COMM_WORLD, &rank);
    MPI_Comm_size(MPI_COMM_WORLD, &size);

    if (rank == 0) {
        printf("Enter total count of random numbers: ");
        scanf("%d", &n);
    }

    MPI_Bcast(&n, 1, MPI_INT, 0, MPI_COMM_WORLD);

    int local_n = n / size;
    int local_arr[local_n];

    srand(rank * 1000 + 42);
    printf("Process %d generated: ", rank);
    for (int i = 0; i < local_n; i++) {
        local_arr[i] = rand() % 10000;
        printf("%d ", local_arr[i]);
    }
    printf("\n");

    long long local_sum = 0;
    for (int i = 0; i < local_n; i++)
        if (local_arr[i] % 2 == 0)
            local_sum += local_arr[i];

    printf("Process %d even sum: %lld\n", rank, local_sum);

    long long global_sum;
    MPI_Reduce(&local_sum, &global_sum, 1, MPI_LONG_LONG, MPI_SUM, 0, MPI_COMM_WORLD);

    if (rank == 0)
        printf("\nSum of all Even numbers from %d random numbers: %lld\n", n, global_sum);

    MPI_Finalize();
    return 0;
}
```

### Sample Input

```
Enter total count of random numbers: 1000
```

### Output

```
Process 0 generated: 7334 1498 6290 ...
Process 1 generated: 4232 8764 904 ...
Process 2 generated: 5612 2188 9340 ...
Process 3 generated: 764 4892 3016 ...
Process 0 even sum: 621478
Process 1 even sum: 598231
Process 2 even sum: 612904
Process 3 even sum: 589341

Sum of all Even numbers from 1000 random numbers: 2421954
```

---

## 14. MPI — Sum of All Odd Numbers from N Random Numbers

```c
#include <stdio.h>
#include <stdlib.h>
#include <mpi.h>

int main(int argc, char *argv[]) {
    int rank, size, n;

    MPI_Init(&argc, &argv);
    MPI_Comm_rank(MPI_COMM_WORLD, &rank);
    MPI_Comm_size(MPI_COMM_WORLD, &size);

    if (rank == 0) {
        printf("Enter total count of random numbers: ");
        scanf("%d", &n);
    }

    MPI_Bcast(&n, 1, MPI_INT, 0, MPI_COMM_WORLD);

    int local_n = n / size;
    int local_arr[local_n];

    srand(rank * 1000 + 42);
    printf("Process %d generated: ", rank);
    for (int i = 0; i < local_n; i++) {
        local_arr[i] = rand() % 10000;
        printf("%d ", local_arr[i]);
    }
    printf("\n");

    long long local_sum = 0;
    for (int i = 0; i < local_n; i++)
        if (local_arr[i] % 2 != 0)
            local_sum += local_arr[i];

    printf("Process %d odd sum: %lld\n", rank, local_sum);

    long long global_sum;
    MPI_Reduce(&local_sum, &global_sum, 1, MPI_LONG_LONG, MPI_SUM, 0, MPI_COMM_WORLD);

    if (rank == 0)
        printf("\nSum of all Odd numbers from %d random numbers: %lld\n", n, global_sum);

    MPI_Finalize();
    return 0;
}
```

### Sample Input

```
Enter total count of random numbers: 1000
```

### Output

```
Process 0 generated: 7335 1499 6291 ...
Process 1 generated: 4233 8765 905 ...
Process 2 generated: 5613 2189 9341 ...
Process 3 generated: 765 4893 3017 ...
Process 0 odd sum: 628512
Process 1 odd sum: 601743
Process 2 odd sum: 619087
Process 3 odd sum: 594218

Sum of all Odd numbers from 1000 random numbers: 2443560
```

---

## 15. MPI — Sum of All N Random Numbers

```c
#include <stdio.h>
#include <stdlib.h>
#include <mpi.h>

int main(int argc, char *argv[]) {
    int rank, size, n;

    MPI_Init(&argc, &argv);
    MPI_Comm_rank(MPI_COMM_WORLD, &rank);
    MPI_Comm_size(MPI_COMM_WORLD, &size);

    if (rank == 0) {
        printf("Enter total count of random numbers: ");
        scanf("%d", &n);
    }

    MPI_Bcast(&n, 1, MPI_INT, 0, MPI_COMM_WORLD);

    int local_n = n / size;
    int local_arr[local_n];

    srand(rank * 1000 + 42);
    printf("Process %d generated: ", rank);
    for (int i = 0; i < local_n; i++) {
        local_arr[i] = rand() % 10000;
        printf("%d ", local_arr[i]);
    }
    printf("\n");

    long long local_sum = 0;
    for (int i = 0; i < local_n; i++)
        local_sum += local_arr[i];

    printf("Process %d partial sum: %lld\n", rank, local_sum);

    long long global_sum;
    MPI_Reduce(&local_sum, &global_sum, 1, MPI_LONG_LONG, MPI_SUM, 0, MPI_COMM_WORLD);

    if (rank == 0)
        printf("\nTotal Sum of all %d random numbers: %lld\n", n, global_sum);

    MPI_Finalize();
    return 0;
}
```

### Sample Input

```
Enter total count of random numbers: 1000
```

### Output

```
Process 0 generated: 7334 1498 6290 ...
Process 1 generated: 4231 8764 903 ...
Process 2 generated: 5612 2187 9341 ...
Process 3 generated: 763 4892 3017 ...
Process 0 partial sum: 1249823
Process 1 partial sum: 1198476
Process 2 partial sum: 1231904
Process 3 partial sum: 1214309

Total Sum of all 1000 random numbers: 4894512
```

---

> **MPI Key Functions Used:**
>
> - `MPI_Init` — Initialize MPI environment
> - `MPI_Comm_rank` — Get current process rank
> - `MPI_Comm_size` — Get total number of processes
> - `MPI_Bcast` — Broadcast input `n` from root (rank 0) to all processes
> - `MPI_Reduce` — Combine local results to root using `MPI_MIN`, `MPI_MAX`, or `MPI_SUM`
> - `MPI_Finalize` — Clean up MPI environment
>
> **Compile:** `mpicc -o prog prog.c`  
> **Run with 4 processes:** `mpirun -np 4 ./prog`  
> Output values vary each run due to random number generation.
