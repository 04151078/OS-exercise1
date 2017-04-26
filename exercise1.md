

-------------
###一. 实验目的及实验环境 
####（一） 实验环境 
Linux 操作系统  
####（二）实验目的 
实验1 掌握Linux基本命令 和开发环境 
1. 掌握常用的Linux shell命令； 
2. 掌握编辑环境VIM； 
3. 掌握编译环境gcc及跟踪调试工具gdb。
 
实验2  进程 
 通过观察、分析实验现象，深入理解进程及进程在调度执行和内存空间等方面的特点，掌握在POSIX 规范中fork和kill系统调用的功能和使用。

实验3  线程 
通过观察、分析实验现象，深入理解线程及线程在调度执行和内存空间 等方面的特点，并掌握线程与进程的区别。掌握POSIX 规范中 pthread_create() 函数的功能和使用方法。  

 实验4  互斥 
通过观察、分析实验现象，深入理解理解互斥锁的原理及特点掌握在POSIX 规范中的互斥函数的功能及使用方法。

----------
###二.实验内容
#####实验2
1.你最初1认为运行结果会怎么样？

会持续输出0-9号进程，直到输入数字键+回车，则会杀死该进程，接下来的输出将不会有该进程号，当输入q+回车，则退出程序。

2.实际的结果什么样？有什么特点？试对产生该现象的原因进行分析。

实际的结果跟预期差不多，当输入20时，程序会自动判断，大于10就以10来创建进程。随机输出0~9号进程，sleep(SLEEP_INTERVAL)，循环输出，输入数字键，则会杀死该数字对应的进程，直到输入q退出循环，然后杀死本组所有进程。
分析：每创建一个子进程时，将其pid存储在pid[i]中，i存储在proc_number，然后调用死循环函数do_something()，输出该进程的代号proc_number； 当输入数字键时，主进程会执行kill(pid[ch-'0'],SIGTERM)，从而杀死(ch - ’0’)号进程。当输入q时循环退出，kill(0,SIGTERM)，杀死本组所有进程。程序退出。

3.proc_number这个全局变量在各个子进程里的值相同吗？为什么？

proc_number这个全局变量在各个子进程里的值相同，因为子进程相互独立资源互不影响。

4.kill 命令在程序中使用了几次？每次的作用是什么？执行后的现象是什么？

2次；第一次是杀死该进程号pid[ch-’0’]，执行后接下来的结果中不会有该进程号，打开另一个终端，使用命令ps aux | grep process查看进程状态，子进程先于父进程退出，则被杀死的进程为僵死状态，加了行代码wait(&pid[ch-'0'])，就会使该子进程真正结束。 
第二次是杀死本组所有进程。即主进程以及它创建的所有子进程。执行后程序退出，进程结束。

5.使用kill 命令可以在进程的外部杀死进程。进程怎样能主动退出？这两种退出方式哪种更好一些？

进程在main函数中return，或调用exit()函数都可以正常退出。 而使
用kill命令则是异常退出。当然是正常退出比较好；若在子进程退出前	使用kill命令杀死其父进程，系统会让init进程接管子进程。当用kill	命令使得子进程先于父进程退出时，而父进程又没有调用wait函数等待	子进程结束，子进程处于僵死状态，并且会一直保持下去，直到系统重	启。子进程处于僵死状态时，内核只保存该进程的必要信息以被父进程	所需，此时子进程始终占着资源，同时减少了系统可以创建的最大进程	数，当僵死的子进程占用资源过多时，很可能导致系统卡死。

#####实验3
1. 你最初认为前三列数会相等吗？最后一列斜杠两边的数字是相等，还是大于或者小于关系？ 

我认为前三列数不会相等，因为三个线程运行次数是随机的，结果不可预料，当然counter[i]一定不会相等。而我认为main_counter与sum值应该是相等的。因为都是三个线程的counter之和。

2.最后的结果如你所料吗？有什么特点？对原因进行分析。
 
实验结果是前三列数确实不相等。不过main_counter与sum的值也不相等，main_counter < sum。 
原因：因为三个线程在共同争取运行thread_worker()函数，比如main_counter初值为0，pthread_id[0]执行之后main_counter+1，此时还未来得及将值赋给main_counter，这时的main_counter还是0；pthread_id[1]也执行这个函数，main_counter+1，若此时在1号线程将main_counter+1的值还未赋给main_counter，即这时的main_counter还是0，pthread_id[2]也来执行这个函数，main_counter+1，此时三个线程才将加完之后的值赋给main_counter，则main_counter=0+1=1，而真正执行次sum=0+1+1+1=3。main_counter < sum。

3.thread 的CPU 占用率是多少？为什么会这样？ 

thread的CPU占用率在我的虚拟机中执行结果是101，因为三个线程是无限循环的运行，使得cpu占用率很高。

4. thread_worker()内是死循环，它是怎么退出的？你认为这样退出好吗？ 

thread_worker()函数内是死循环，因为主函数中设置的输入q时循环退出。输入q时主进程执行退出，return 退出程序，则子线程也强制退出。这样退出不好。
#####实验4
1.你预想deadlock.c 的运行结果会如何？

程序运行中出现终止现象，可能会资源互斥。

2.deadlock.c 的实际运行结果如何？多次运行每次的现象都一样吗？为什么会这样？

实际运行时程序会在运行期间中止，出现死锁现象。多次运行之后
现象都一样。原因是：主线程申请mutex1资源，而子线程申请mutex2资源，此时主线程继续申请mutex2资源，子线程来申请mutex1资源，而mutex2资源还未被子线程释放，主线程无法申请到，同样的，mutex1资源未被主线程释放则子线程也无法申请到，此时便处于无限循环等待，形成死锁。

--------
###四、测试数据及运行结果
######1、正常测试数据及运行结果
-------
######(1)


![](http://upload-images.jianshu.io/upload_images/5811683-98b034c7f2b458e5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



######(2)

![](http://upload-images.jianshu.io/upload_images/5811683-10de3818b4f32c24.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


######(3)


![](http://upload-images.jianshu.io/upload_images/5811683-1d426d35c3e5c7ab.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![
](http://upload-images.jianshu.io/upload_images/5811683-44e0218c7331f67d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


######2、非正常测试数据及结果

###五、总结

#####问题

 对于Linux系统的了解不够，以及对系统函数的操作有点不太熟悉，后来经过一步步的练习，我逐渐熟悉了系统的基本命令，对于进程的操作有了深刻的理解，在线程实验中我刚开始对于系统的创建线程的函数不是很了解，，由于一些参数的传递过程中，虽然按照了系统函数参数要求进行传递参数，传递过去的值，并不是我想要的值，后来，经过调试，已经掌握如何对其进行修改了。
 #####体会

在本次操作系统实验中，学会了在Linux虚拟机下如编写c程序，并且掌握了一些简单的操作，gcc , vim gdb ,ls 的简单命令的使用，我学会了很多关于线程进程的知识。用fork和kill创建和杀死进程，还详细了解了进程的状态，父进程子进程之间的关系。 
       int pthread_create(pthread_t *thread,pthread_attr_t *attr,void *(*start_routine)(void *),void *arg)函数的使用方法，可以创建线程，并熟悉了线程的并发，并且理解主线程与子线程之间执行的过程。
       并且大致掌握PTHREAD_MUTEX_NORMAL，PTHREAD_MUTEX_ERRORCHECK，PTHREAD_MUTEX_RECURSIVE和PTHREAD_MUTEXT_DEFAULT等线程互斥锁的类型。加深了我对互斥和同步的理解以及死锁及如何解除死锁等相关知识。使我对操作系统这门课程有了更深的认知，操作系统这门课不光是理论知识，实践是更快掌握这一门课的良好途径，因为在实践中学习，在Linux环境下，编写程序，并且执行这个过程，时期对其有了很好的兴趣，我现在很希望在Linux环境下，进行java编程，让我对跨平台的编程操作有一定的了解，并且掌握一些基础的知识，，熟悉操作系统中的生产者消费者问题，以及哲学家进餐问题，对于进程线程的操作有更加深入的了解吧，可以为学习Linux的操作系统的知识打下基础，受益匪浅。
###六、实验源代码

####1、进程
```
#include<stdio.h>
#include<sys/types.h>
#include<unistd.h>
#include<signal.h>
#include<ctype.h>

#define MAX_CHILD_NUMBER 10
#define SLEEP_INTERVAL 2 
```
```
int proc_number = 0;
```
```
void do_something(){
         for(;;){
                printf("This is process No.%d and its is %d\n",proc_number,getpid());
                sleep(SLEEP_INTERVAL);
         }
}
```
```
int main(int argc,char *argv[]){
        int child_proc_number=MAX_CHILD_NUMBER;
        int i,ch;
        pid_t child_pid;
        pid_t pid[10]={0};
        if(argc>1){
                child_proc_number = (child_proc_number > 10) ? 10 : child_proc_number;
        }

        for(i=0;i<child_proc_number;i++){
                child_pid=fork();
                if(child_pid == -1){
                        printf("process create error\n");
                }else if(child_pid == 0){
                        proc_number=i;
                        do_something();
                }else{
                        pid[i]=child_pid;
                }
        }

        while((ch=getchar())!='q'){
                if(isdigit(ch)){
                        kill(pid[ch-'0'],SIGTERM);
                }
        }

        for(i=0;i<=proc_number;i++){
                kill(pid[i],SIGTERM);
        }

        kill(0,SIGTERM);
        return 0;
}
```
 -----------                                                                                                                                                                           
####2、线程
```
#include <stdio.h>
#include <stdlib.h>
#include <sys/types.h>
#include <unistd.h>
#include <ctype.h>
#include <pthread.h>
#include<semaphore.h>
#define MAX_THREAD 3 /*线程的个数*/
```
```
pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;

unsigned long long main_counter=0,counter[MAX_THREAD]={0};
/*unsigned long long 是比long还长的整数*/
```
```
sem_t sl;
```
```
void* thread_worker(void* );
```
```
int main(int argc, char *argv[])
{
	int i, rtn, ch;
	sem_init(&sl,0,1);	

	pthread_t pthread_id[MAX_THREAD] = {0}; /*存放线程Id*/
	for(int i = 0; i < MAX_THREAD; i++){
		/*在这里填写代码，用pthread_create建一个普通的线程,线程id存入pthread_id[i],
			线程执行函数是thread_worker并i作为参数传给线程*/
		
		if(pthread_create(&pthread_id[i], NULL, thread_worker, (void *)i) !=0){
			printf("thread_create failed");
			exit(1);
		}
	}

	do{/*用户按一次回车执行下面的循环体一次。按q退出*/
		unsigned long long sum = 0;

		/*求所有线程的counter的和*/
        sem_wait(&sl);
		for(i = 0; i < MAX_THREAD; i++){/*求所有counter的和*/
			sum += counter[i];
			printf("the NO.%d thread %llu--- ",i+1, counter[i]);
		}
		printf(" %llu/%llu",main_counter, sum);
        sem_post(&sl);
	}while((ch = getchar()) != 'q');

	return 0;
}
```
```
void* thread_worker(void* p){
	int thread_num;
	/*在这里填写代码,把main中的i的值传递给thread_num*/
	thread_num = (int *)p;
	
	for(;;){	/*无限循环*/
		counter[thread_num]++;	/*本线程的counter加一*/
	//	pthread_mutex_lock(&mutex);
			
		sem_wait(&sl);
		main_counter++;			/*主counter加一*/
	//	pthread_mutex_unlock(&mutex);
	//	sleep(0.04);

		sem_post(&sl);
	}	
}
```
-----------
####3、互斥
```
#define LOOP_TIMES 10000
```
```
pthread_mutex_t mutex1 = PTHREAD_MUTEX_INITIALIZER;
pthread_mutex_t mutex2 = PTHREAD_MUTEX_INITIALIZER;
```
```
void * thread_worker(void *p);
void critical_section(int thread_num,int  i);
```
```
int main(void){

	int i,rtn;
	pthread_t pthread_id = 0;
	rtn = pthread_create(&pthread_id,NULL,thread_worker,NULL);
	if(rtn!=0){
		printf("Pthread_create ERROR!\n");
		return -1;
	}
	
	for(i=0;i<LOOP_TIMES;i++){
		pthread_mutex_lock(&mutex1);
		pthread_mutex_lock(&mutex2);

		critical_section(1,i);

		pthread_mutex_unlock(&mutex2);
		pthread_mutex_unlock(&mutex1);
	}

	pthread_mutex_destroy(&mutex2);
	pthread_mutex_destroy(&mutex1);

	return 0;
}
```

```
void* thread_worker(void *p){
	int i;
	for(i=0;i<LOOP_TIMES;i++){
		pthread_mutex_lock(&mutex1);
		pthread_mutex_lock(&mutex2);
		
		critical_section(2,i);
	
		pthread_mutex_unlock(&mutex1);
		pthread_mutex_unlock(&mutex2);
	}
	return NULL;
}
```
```
void critical_section(int thread_num,int i){
	printf("Thread %d : %d \n",thread_num,i);
}
```
