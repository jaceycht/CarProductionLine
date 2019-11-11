COMP3230 Principles of Operating Systems

# Background Story
Tesla Motors is no doubt the leading company of electric cars and is also the first company who made electric cars so popular around the world. Tesla not only put a lot of high technologies in their cars but also how they manufacture electric cars. Tesla factory almost automated the whole production process by using a lot of smart trainable robot workers. Knowing the development of the world most advanced technology is even more important than just write this simple simulation program. I’ve put some videos you may check to learn more about the simplicity of the design and the complexity of making it possible.

Your job in this assignment, as mentioned before, is to write a program to simulate the production process of Tesla electric car and make the production process as efficient as you can with limited resources.

# System overview
To simplify the process of manufacturing Tesla electric cars, 7 car parts need to be built before the final assembly of a car. These parts are skeleton, engine, chassis, body, window, battery. Each car should have 7 windows, 1 body, 4 tires and 1 battery pack. Each body should have 1 skeleton, 1 engine and 1 chassis. 

This system consists of: 
- definitions.h: Defines system variables like production time. You are not allowed to change those variables
- main.h and main.c: The main program, initiate factory status, manage and schedule workers, report results
- worker.h and worker.c: Contains the worker(thread) function
- job.h and job.c: Contains the manufacturing functions

The relationship among these files are rather simple:
- Main program creates workers and assign jobs to them
- Workers do their jobs

# Implementation details
Control of resources
- Control of resources including number of robot workers, number of storage space in the factory as well as each car parts is implemented with counting semaphore. Initial values of worker and space are predefined while the initial value for each car parts is set to 0. The initiation process is done by the function called initSem().

Job assignment
- Job is assigned to each robot worker(thread) via predefined struct work_pack. Each work pack contains worker ID, job ID (defined in definition.h), how many times should this worker do its job, and the resource package (semaphore package, struct resource_pack). You can find the above structures in work.h.

Manufacture process
- Since this is only a simulation of Tesla factory production line, there’s no need to implement how these car parts are built or how a car is assembled. Instead those functions just sleep a certain amount of time for each production job. You can find these build functions in job.c. Time lengths are defined in file definitions.h which you are not supposed to change to be fair. If a car part needs some other parts as its raw material like the car body, the worker must wait for the completion of building those parts. After a part of the car is built, the worker needs to put it in the storage space and notify its own semaphore that one piece has been made. Note that one storage space for on part only and the electric car doesn’t take any factory storage space as the car will be delivered to somewhere else.



# Q1 Complete the single threaded program
Your first task is to complete the single threaded version and get familiar with the program. Your tasks for this question:
1. In file job.c, you should complete 2 functions: makeBattery and makeCar. Functions for making other parts have been implemented. You may go through those functions and have an idea of how they work before you implement these 2 functions. 
2. Then you need to add lines to main.c to complete the rest of the program so that all parts will be made sequentially. To make it simple in this question, we just need to make 1 car. 
3. After you finish your code, you can compile your code by typing in command make in your Linux console (makefile has been provided). If there’s no error, you can run your program by execute ./tesla_factory.out. By default, the program will run in debug mode and print out some useful information of each step for debugging. If you don’t want these lines, you just set DEBUG variable in definition.h to 0 and recompile your program.
4. Include a screenshot of your program. 

Since we only consider the situation that use one robot worker to make one car with sufficient space, these corresponding parameters will be set in the main function. 

> It takes a pretty long time (i.e., 40 seconds) to make a car with only one thread.



# Q2 Implement a naïve multithreaded program
Now you have some basic ideas about how the program works. Your task here is to copy the completed code from q1 to q2 and make it a simple multithreaded version that multiple workers are working simultaneously to speed up the production process. Number of cars to be made, number of storage space and number of workers are passed to the main function as parameters for the ease of testing later.

To make life easier, you may start with creating only 8 threads to build 1 car and each thread corresponds to 1 job. Simply speaking, just assign 8 jobs from q1 to 8 threads. Note that you need to use sem_worker to keep track of how many workers are working at the same time and make sure that the number doesn’t exceed the limit of workers defined in variable num_workers. You also need to change the normal work function in worker.c to a thread start routine so that you can pass it as an argument to pthread_create(). You may consider 8 workers as a work group. They work together simultaneously producing Tesla electric cars.

Once your 8-thread version works fine, you need to extend your code so that it will work with multiples of 8 threads. Your program should have the ability to produce more than 1 car at this stage. Hint: you may consider it as adding more working groups.

Implement this simple scheduling scheme by yourself and write down your implementation details in your report. You need to also include a screenshot of you testing your code with different parameters in run.sh. 

> Performance is improved with multi-threading. For making 4 cars, doubling the number of workers reduces the production time by half. 8 workers take only 15 seconds to make a car. It is speeded up when compared to the 40 seconds in question 1. 



# Q3 Implement your own job assignment scheme 
You may find that the naïve implementation in question 2 has some problems:
1. Only works well for the multiple of 8 workers and will gain little speedup for other number of workers.
2. Naïve implementation will produce wasted parts that are not necessary.
3. Storage space is not considered as we just assign a large number of storage space making sure that it completes the whole process.
4. Deadlock is not handled.

Your task is to tackle these problems above with your own dynamic job assignment implementation. You should write down your detailed implementation introduction in your report and run benchmark experiments on your program.
Requirements:
1. Your program should accept different numbers of workers to produce different number of cars. You should test your program with many combinations of input parameters (number of cars, number of spaces, number of workers) and make sure your program is bug free. You need to draw a graph to show the scalability of your program with the number of workers and run time. Put the graph into your report. You should analyse your output and explain the results you have. Other graphs like showing the relationship of producing different number of cars with the given number of threads.
2. Your program should be able to handle deadlocks when given small number of storage space. You may include some screenshots to help you prove that without your deadlock handling code there’s deadlock and by adding your deadlock handling code there’s no deadlock. Here are some hints:
a. Deadlock detection: you don’t know whether you will encounter deadlock or not ahead. But once your program detects that certain threads runs for unreasonable amount of time, you know that deadlock happens. Then you figure out a way to break the deadlock so that the production process can move on. sem_trywait() or sem_timedwait() may be useful here.
b. Deadlock prevention: once the production goal is set, you know the number of each part to achieve the goal. You also know how many spaces you have. Your program may analyse these data and assign jobs to workers properly so that deadlock will not happen for sure.

> I realised that, in my main program in question 2, it has to wait for all workers to finish their tasks before moving to the next round (or building the next car). This is solved by moving the "thread-joining part" outside of other loops. 

> Besides, my program in question 2 produces wasted parts that are unnecessary. Modifications are made to not assign a thread to each worker. Instead, worker can start a new thread after the previous task is done. 

> Even with better tasks scheduling in the modified program, deadlock will be triggered if there is no storage space in the middle of production and the whole program halts. In my program, the first worker gets the first job, the second worker gets the second job, the next worker gets the next job... Deadlock occurs when storage spaces are occupied by other items made in previous jobs but no worker is assigned to get the items from the storage space yet. Improvements shall be done on this part. Say, there is only one storage space but more than two workers. To build a car, we can ask the first worker to do the first job and then ask the second worker to keep the item. Once the second worker gets all necessary items (i.e., skeleton, engine, and chassis), he/she can start making the body. The first worker can continue to make items like the window, tire, and battery pack. Once ready, the items are passed to the second worker again to make the actual car. Currently I just detect for deadlock and exit the program when necessasry. 

> Nevertheless, my program has a high scalability with the number of workers and production time. It is tested by making 10 cars with 40 storage spaces and different number of workers: (1) The production time decreases with the increasing number of workers. (2) The production time remains more or less the same despite the decrease in number of storage spaces. (3) Instead of doubling or tripling the amount of time required, the making of one extra car only needs roughly five more seconds.



