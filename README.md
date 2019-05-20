# os161_semaphore
tip> os161을 설치시 그리고 os161 설치후 sys161 kern 입력시 harvd-mips-머신이 없다는 오류 메시지가 나온다면<br>
os161을 설치한 디렉토리 내부 모든 디렉토리와 파일들의 권한을 7XX로 올려주자.<br>
sudo chmod -R 744 [os161설치 디렉토리 경로]<br>
권한때문에 문제가 생기는 것 같은데 어느 곳에서 나는지 모르니 모든 내부 디렉토리와 파일의 권한을 상승시켜보자<br><br>
os161운영체제에서 ~/os161/src/kern/test/synchtest.c 파일에 주석을 달아 보았다.<br>
synchtest.c 코드를 간단하게 보자면 
<pre><code>
int
semtest(int nargs, char **args)	//sys161 kernel 실행 이후 sy1을 입력하면 실행되는 함수
{
	int i, result;
	(void)nargs;
	(void)args;
	inititems();		//testsem = sem_create("testsem", 2);
	kprintf("Starting semaphore test...\n");
	kprintf("If this hangs, it's broken: ");
	P(testsem);		//testsem의 sem_count를 0으로 만드는 이유
	P(testsem);
	kprintf("ok\n");
	/*
	바로 밑에 for loop 에서 본격적으로 스레드를 생성하게 되는데 위에서 P(세마포어이름)를 안하면
	스레드 1생성 중에 스레드 0은 이미 실행되어 결국에는(인터럽트가 없다면)스레드 0 1 ... 31 순서로 만들어지게 된다.
	이 프로그램의 작성자의 의도는 이런것을 원하는 것이 아니라 동시실행 되어 스레드가 막 뒤섞이는 것을 원하므로
	스레드 생성 전에 스레드 실행을 막고(sem_count를 0으로 만들고)
	*/
	for (i=0; i<NTHREADS; i++) {
		result = thread_fork("carsem", NULL, car, NULL, i);
		if (result) {
			panic("semtest: thread_fork failed: %s\n",
			      strerror(result));
		}
	}
	kprintf("make sem done\n");
	/*
	스레드를 미리 전부 다 생성해 놓고 밑에 있는 for loop 안에 V(carsem)을 하여 실행 하게 된다. 이때 대기중이던 스레드중
	1개가 실행된다.(OS 스케줄러에 따라서 실행된다. FIFO일 수도 있고 다른것일 수도 있고) 
	*/
	for (i=0; i<NTHREADS; i++) {
		V(carsem);
		P(cardone);
	}
	/*
	위에서 P 함수로 잠가 놓았으니 testsem의 sem_count를 원상 복구 시킨다.(위에 init 함수에 2로 되어있다.)
	*/
	/* so we can run it again */
	V(testsem);
	V(testsem);
		
	kprintf("Semaphore test done.\n");
	return 0;
}
</code></pre>
