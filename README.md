#include <stdio.h>
#include <unistd.h>
#include <sys/wait.h>

int main()
{
	int n;
	printf("Enter the range : ");
	scanf("%d",&n);
	
	pid_t pid = fork();
	
	if(pid == 0)
	{
		printf("You are in child process \n");
		for(int i=0;i<n;i+=2)
		{
			printf("%d",i);
		}
		printf("\n");
	}
	else if(pid > 0)
	{
		wait(NULL);
		printf("You are in parent process \n");
		for(int i=1;i<n;i+=2)
		{
			printf("%d",i);
		}
		printf("\n");
	}
	else
	{
		printf("Fork creation failed");
	}
	return(0);
}
_______________________________
#include <stdio.h>

int main() {
    int n, i;
    float avg_tat = 0, avg_wt = 0;

    printf("Enter number of processes: ");
    scanf("%d", &n);

    int pid[n], at[n], bt[n], ct[n], tat[n], wt[n];

    // Input details
    for(i = 0; i < n; i++) {
        pid[i] = i + 1;
        printf("\nEnter Arrival Time for Process %d: ", i+1);
        scanf("%d", &at[i]);
        printf("Enter Burst Time for Process %d: ", i+1);
        scanf("%d", &bt[i]);
    }

    // Sort processes by Arrival Time (FCFS)
    for(i = 0; i < n-1; i++) {
        for(int j = i+1; j < n; j++) {
            if(at[i] > at[j]) {
                int temp;
                temp = at[i]; at[i] = at[j]; at[j] = temp;
                temp = bt[i]; bt[i] = bt[j]; bt[j] = temp;
                temp = pid[i]; pid[i] = pid[j]; pid[j] = temp;
            }
        }
    }

    // Calculate Completion Time
    ct[0] = at[0] + bt[0];
    for(i = 1; i < n; i++) {
        if(at[i] > ct[i-1])
            ct[i] = at[i] + bt[i]; // CPU idle
        else
            ct[i] = ct[i-1] + bt[i];
    }

    // Calculate TAT and WT
    for(i = 0; i < n; i++) {
        tat[i] = ct[i] - at[i];
        wt[i] = tat[i] - bt[i];
        avg_tat += tat[i];
        avg_wt += wt[i];
    }

    avg_tat /= n;
    avg_wt /= n;

    // Display results
    printf("\nP\tAT\tBT\tCT\tTAT\tWT\n");
    for(i = 0; i < n; i++) {
        printf("P%d\t%d\t%d\t%d\t%d\t%d\n", pid[i], at[i], bt[i], ct[i], tat[i], wt[i]);
    }

    printf("\nAverage Turnaround Time: %.2f", avg_tat);
    printf("\nAverage Waiting Time: %.2f\n", avg_wt);

    return 0;
}
--------------------------
#include <stdio.h>

int main()
{
	int n;
	int avgwt = 0;
	int avgtat = 0;
	int t = 0;
	int completed = 0;
	
	printf("Enter the number of processes : ");
	scanf("%d",&n);
	
	int pid[n],at[n],bt[n],ct[n],tat[n],wt[n],rt[n];
	
	for(int i=0;i<n;i++)
	{
		pid[i] = i+1;
		printf("Enter the arrival time of process %d : ",i+1);
		scanf("%d",&at[i]);
		printf("Enter the burst time of process %d : ",i+1);
		scanf("%d",&bt[i]);
		rt[i] = bt[i];
	}
	
	for(int i=0;i<n;i++)
	{
		for(int j=i+1;j<n;j++)
		{
			if(at[i] > at[j])
			{
				int temp;
				
				temp = at[i];
				at[i] = at[j];
				at[j] = temp;
				
				temp = bt[i];
				bt[i] = bt[j];
				bt[j] = temp;
				
				temp = rt[i];
				rt[i] = rt[j];
				rt[j] = temp;
				
				temp = pid[i];
				pid[i] = pid[j];
				pid[j] = temp;
			}
		}
	}
	
	
	while(completed!=n)
	{
		int min = -1;
		for(int i=0;i<n;i++)
		{
			if(at[i] <=t && rt[i] > 0)
			{
				if(min == -1 || rt[i] < rt[min])
				{
					min = i;
				}
			}
		}
		if(min == -1)
		{
			t++;
			continue;
		}
		
		rt[min]--;
		t++;
		if(rt[min] == 0)
		{
			completed++;
			ct[min] = t;
		}
		
	}
	
	for(int i=0;i<n;i++)
	{
		tat[i] = ct[i]-at[i];
		wt[i] = tat[i] - bt[i];
		avgtat = avgtat + tat[i];
		avgwt = avgwt + wt[i];
	}
	
	avgtat = avgtat/n;
	avgwt = avgwt/n;
	
	
	printf("\nPID\tAT\tBT\tCT\tTAT\tWT\n");
	for(int i=0;i<n;i++)
	{
		printf("%d\t%d\t%d\t%d\t%d\t%d\t\n",pid[i],at[i],bt[i],ct[i],tat[i],wt[i]);
	}
	printf("Average waiting time is : %d", avgwt);
	printf("Average turn around time is : %d", avgtat);
	
	
	return(0);	
}
-----------------------------------

#include <stdio.h>
#include <unistd.h>
#include <string.h>

int main()
{
	int fd[2];
	char msg[] = "Hello from parent ";
	char buffer[50];
	
	pipe(fd);
	if(fork() == 0)
	{
		close(fd[1]);
		read(fd[0],buffer, sizeof(buffer));
		printf("Message from parent received : %s\n",buffer);
		close(fd[0]);
	}
	else
	{
		close(fd[0]);
		write(fd[1],msg,strlen(msg)+1);
		close(fd[1]);
	}
	return(0);
}
____________________________________________
#include <stdio.h>
#include <semaphore.h>
#include <pthread.h>

sem_t mutex, wrt;
int readcount = 0, data = 0;

void *reader(void *rno)
{
    int n = *(int *)rno;
    sem_wait(&mutex);
    readcount++;
    if (readcount == 1)
        sem_wait(&wrt);
    sem_post(&mutex);

    printf("Reader %d reads data = %d\n", n, data);

    sem_wait(&mutex);
    readcount--;
    if (readcount == 0)
        sem_post(&wrt);
    sem_post(&mutex);
}

void *writer(void *wno)
{
    int n = *(int *)wno;
    sem_wait(&wrt);
    data++;
    printf("Writer %d writes data = %d\n", n, data);
    sem_post(&wrt);
}

int main()
{
    pthread_t r[3], w[2];
    int rno[3] = {1, 2, 3}, wno[2] = {1, 2};

    sem_init(&mutex, 0, 1);
    sem_init(&wrt, 0, 1);

    pthread_create(&r[0], NULL, reader, &rno[0]);
    pthread_create(&w[0], NULL, writer, &wno[0]);
    pthread_create(&r[1], NULL, reader, &rno[1]);
    pthread_create(&r[2], NULL, reader, &rno[2]);
    pthread_create(&w[1], NULL, writer, &wno[1]);

    for (int i = 0; i < 3; i++)
        pthread_join(r[i], NULL);
    for (int i = 0; i < 2; i++)
        pthread_join(w[i], NULL);

    sem_destroy(&mutex);
    sem_destroy(&wrt);
    return 0;
}

_________________________________________________________________
#include <stdio.h>

int main() {
    int n, m, i, j, k;
    printf("Enter no. of processes and resources: ");
    scanf("%d %d", &n, &m);

    int alloc[n][m], max[n][m], avail[m], need[n][m], f[n], ans[n], ind = 0;

    printf("Enter allocation matrix:\n");
    for(i = 0; i < n; i++)
        for(j = 0; j < m; j++)
            scanf("%d", &alloc[i][j]);

    printf("Enter max matrix:\n");
    for(i = 0; i < n; i++)
        for(j = 0; j < m; j++)
            scanf("%d", &max[i][j]);

    printf("Enter available resources:\n");
    for(i = 0; i < m; i++)
        scanf("%d", &avail[i]);

    for(i = 0; i < n; i++) {
        f[i] = 0;
        for(j = 0; j < m; j++)
            need[i][j] = max[i][j] - alloc[i][j];
    }

    for(k = 0; k < n; k++) {
        for(i = 0; i < n; i++) {
            if(f[i] == 0) {
                int flag = 0;
                for(j = 0; j < m; j++) {
                    if(need[i][j] > avail[j]) {
                        flag = 1;
                        break;
                    }
                }
                if(flag == 0) {
                    ans[ind++] = i;
                    for(j = 0; j < m; j++)
                        avail[j] += alloc[i][j];
                    f[i] = 1;
                }
            }
        }
    }

    int flag = 1;
    for(i = 0; i < n; i++)
        if(f[i] == 0) { flag = 0; printf("System is not safe!\n"); break; }

    if(flag == 1) {
        printf("System is safe.\nSafe sequence: ");
        for(i = 0; i < n; i++)
            printf("P%d ", ans[i]);
    }
    return 0;
}
_______________________________________________
#include <stdio.h>

int main() {
    int pages[50], frame[10], n, f, i, j, k = 0, fault = 0;

    printf("Enter number of pages: ");
    scanf("%d", &n);
    printf("Enter page numbers: ");
    for (i = 0; i < n; i++)
        scanf("%d", &pages[i]);
    printf("Enter number of frames: ");
    scanf("%d", &f);

    for (i = 0; i < f; i++)
        frame[i] = -1;

    printf("\nPage\tFrames\n");

    for (i = 0; i < n; i++) {
        int found = 0;
        for (j = 0; j < f; j++)
            if (frame[j] == pages[i])
                found = 1;

        if (!found) {
            frame[k] = pages[i];
            k = (k + 1) % f;
            fault++;
        }

        printf("%d\t", pages[i]);
        for (j = 0; j < f; j++)
            if (frame[j] != -1) printf("%d ", frame[j]);
            else printf("- ");
        printf("\n");
    }

    printf("\nTotal Page Faults = %d\n", fault);
    return 0;
}

____________________________________________________________________

#include <stdio.h>

int main() {
    int pages[] = {7,0,1,2,0,3,0,4,2,3,0,3,2,1,2};
    int n = 15, frames = 4, f[10], count = 0;

    for (int i = 0; i < frames; i++) f[i] = -1;

    for (int i = 0; i < n; i++) {
        int page = pages[i], found = 0;
        for (int j = 0; j < frames; j++)
            if (f[j] == page) { found = 1; break; }

        if (!found) {
            int least = i, pos = 0;
            for (int j = 0; j < frames; j++) {
                int k, used = 0;
                for (k = i - 1; k >= 0; k--)
                    if (f[j] == pages[k]) { used = 1; if (k < least) { least = k; pos = j; } break; }
                if (!used) { pos = j; break; }
            }
            f[pos] = page;
            count++;
        }

        printf("Page %d -> ", page);
        for (int j = 0; j < frames; j++)
            if (f[j] != -1) printf("%d ", f[j]);
        printf("\n");
    }

    printf("\nTotal Page Faults = %d\n", count);
    return 0;
}
