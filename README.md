# rahul-os-k1654
file copy

#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <errno.h>
#include <fcntl.h> 

int main (int argc, char *argv[]) {
    int pipefile[2];
    char p_buf[100];
    char child_buf[100];
      
    if (argc < 2) {
      perror("Oops! Please enter the two file names. \n");
      exit(1);
    }
    char* srcFile = argv[1];
    char* dstFile = argv[2];

    if (pipe(pipefile) < 0) {
      printf("Something went wrong creating the pipe! %s\n", strerror(errno));
      exit(1);
    }
    
    if(fork() > 0){

	close(pipefile[0]);                                              
        int fileInDesc = open(srcFile, O_RDONLY);                       
        ssize_t num_bytes = read(fileInDesc, p_buf, sizeof(p_buf));   
        write(pipefile[1], p_buf, num_bytes);                           
        close(pipefile[1]); 
	}   
    if(fork() == 0){

	close(pipefile[1]);                                                       
        ssize_t num_bytes_child = read(pipefile[0], child_buf, sizeof(child_buf));  
        close(pipefile[0]);                                                        
        int targetDesc = open(dstFile, O_CREAT | O_WRONLY);                                  
        write(targetDesc, child_buf, num_bytes_child);  	
	}
    if(fork() == -1){

   	printf("Error forking child process. %s", strerror(errno));
        exit(1);
	}
    return 0;
}





multi thread


#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>

double average;        
int min;
int max;

typedef struct data
{
    int size;
    int * values;
}data;

void *avgFunction(void *ptr)
{
    data * copy;
    copy = (data *) ptr;   
    int sz = copy->size;
    int i;
    for(i = 0; i < sz; i++)
    {
        average += (copy->values[i]);    
    }                               
    average = (int)(average / sz);         
}

void *minFunction(void *ptr)
{
    data * copy;
    copy = (data *) ptr;   
    int sz = copy->size;
    int i;
    min = (copy->values[0]);
    for(i = 1; i < sz; i++)
    {
        if(min > (copy->values[i]))
        {
            min = (copy->values[i]);
        }
    }
}

void *maxFunction(void *ptr){
    
    data * copy;
    copy = (data *) ptr;
    int sz = copy->size;
    int i;   
    max = copy->values[0];
    for(i = 1; i < sz; i++)
    {
        if(max < copy->values[i])
        {
            max = copy->values[i];
        }
    }
}

int main(int argc, char *argv[])
{
    if(argc <=1)
    {
        printf("Oops! No input provided. Please provide some input.\n");
        exit(0);
	}
    int i = 0;
    int copy[argc-1];
    for(i; i < (argc -1); i++)
    {
        copy[i] = atoi(argv[i+1]);
    }
    pthread_t thread1, thread2, thread3;
    data ds = {argc - 1, copy};
    
    pthread_create(&thread1, NULL, (void *) avgFunction, (void *) &ds);   
    pthread_create(&thread2, NULL, (void *) minFunction, (void *) &ds);
    pthread_create(&thread3, NULL, (void *) maxFunction, (void *) &ds);
 
    pthread_join(thread1, NULL);                       
    pthread_join(thread2, NULL);
    pthread_join(thread3, NULL);

    printf("The average value is:  %g\n", average);
    printf("The minimum value is:  %d\n", min);
    printf("The maximum value is:  %d\n", max);
}

