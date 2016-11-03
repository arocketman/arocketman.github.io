---
layout: post
title:  "Parallel programming: installing MPI on ubuntu and create a simple sum parallel program."
date:   2016-11-02 14:52:07 +0200
categories: main
icons: 
- icon-c
- fa-linux
---
Parallel programming is a very interesting topic, let's find out how we can use openMPI to create a simple program that adds N numbers using M processors. 

First things first, let's install openMPI. This is really simple we can open up our terminal and type the following:

{% highlight shell %}
sudo apt-get install mpich2
{% endhighlight %}

Once you're done installing just create a file and open it up with your favourite editor.
{% highlight shell %}
touch sumNumbers.c
gedit sumNumbers.c
{% endhighlight %}

Here's the code for the sum. I tried to comment it as much as I could to make it understandable. You can find more considerations at the end of the article:

{% highlight c %}

#include <stdio.h>
#include <stdlib.h>
//Include mpi.h library.
#include "mpi.h" 

#define N 100

int initializeMPI(int*,int*,int,char* []);

int main(int argc, char *argv[]){
	int numprocs, id;
	initializeMPI(&numprocs,&id , argc, argv);

	int elements_per_processor = N/numprocs;
	
	//The processor 0 will initialize the elements variable the sendcount variable and the displacement one and fill it.
	int * elements , *sendCount , *displacement;
	if(id==0){
		//elements array holds ALL the elements we want to sum.
		elements = (int*)malloc(N*sizeof(int));
		//The sendcount array contains in position i the number of elements to send to the i-th CPU.
		sendCount = (int*)malloc(numprocs*sizeof(int));
		//The displacement array contains in position i the displacement in the 'elements' array from where the sendCount[i] is going to start. 
		displacement = (int*)malloc(numprocs*sizeof(int));
		
		int i;
		//Fill the elements array with whatever values you'd like.
		for(i = 0; i < N; i++)
			elements[i] = i;

		//Assuming that each CPU will have N/numprocs elements:
		for(i = 0; i < numprocs; i++){
			sendCount[i] = elements_per_processor;
			displacement[i] = elements_per_processor*i;
		}
	}

	int * receivedElements;
	//For the sake of simplicity we'll assume that N will be a multiple of the number of processors.
	receivedElements = (int*)malloc(elements_per_processor*sizeof(int));

	//We are going to pass N/numprocs elements to each CPU. Each CPU will have in the receivedElements array such elements.
	//Notice how the CPU with ID 0 is the root processor.
	MPI_Scatterv(elements,sendCount,displacement,MPI_INT,receivedElements,elements_per_processor,MPI_INT,0,MPI_COMM_WORLD);

	int j , processorSum , totalSum; 
	for(j = 0; j < elements_per_processor; j++){
		processorSum += receivedElements[j];
	}

	printf("I am the cpu %d , my sum is: %d\n" ,id , processorSum);

	//Once every CPU has finished additioning all the numbers in their receivedElements array we are going to sum all the partial sums.
	//Such final addition is done by the CPU 0. (Root processor).
	MPI_Reduce(&processorSum,&totalSum,1,MPI_INT,MPI_SUM,0,MPI_COMM_WORLD);

	if(id == 0)
		printf("The total sum is %d\n",totalSum);

	MPI_Finalize();
	return 0;
}

/**
* This function initializes MPI and the numprocs,id variables.
*/
int initializeMPI(int *numprocs , int* id , int argc,char *argv[]){
	//We call the MPI_Init function that proceeds to take the parameters we pass through the terminal and initialize the MPI environment.
	MPI_Init(&argc,&argv);
	//We get the number of processors for the group COMM.
	MPI_Comm_size(MPI_COMM_WORLD,numprocs);
	//We get this specific processor id
	MPI_Comm_rank(MPI_COMM_WORLD,id);
}

{% endhighlight %}

After we are done writing the code, we can compile it and execute it with the following commands:

{% highlight bash %}
andrea@andrea-VirtualBox:~/Scrivania/CP/blog$ mpicc somma.c -o somma
andrea@andrea-VirtualBox:~/Scrivania/CP/blog$ mpirun -np 4 ./somma
I am the cpu 0 , my sum is: 300
I am the cpu 1 , my sum is: 925
I am the cpu 2 , my sum is: 1550
I am the cpu 3 , my sum is: 2175
The total sum is 4950
{% endhighlight %}

Here are some considerations to take into account:

* The order in which the printf are outputted could var* CPU 2 could finish before CPU 0 for all we know. If you want to sync the outputs you can use the MPI_Barrier function.
* You can use the MPI_Wtime function to observe performances.
* In the mpirun command you can specify the numbers of processors with the -np command. 
* If you do not want to make the assumption that N/numprocs is equal to an integer. You can take both the division and the remainder of such division and "spread" the remainder across the CPUs. For example, if we have 15 elements and 4 cpus. We'll have initially (int)15/4 elements for each CPU. That'd be 3 elements for each CPU. Then add the remainder equally across the CPU. So basically [3,3,3,3]->[4,4,4,3].

