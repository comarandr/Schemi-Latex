# processi cooperanti

## soluzioni software

### SOLUZIONI BUSY WAIT 

#### tentativo sbagliato

```c
while (TRUE)
    while (occupato =\= 0);
        sezione critica; (X) occupato :=1; 
    occupato := 0;
        sezione non critica
```
(X): punto critico, NON garantisce mutua esclusione

#### alternanza stretta

uso di una variabile **turn**. Se turn = i &rarr; $P_i$ può entrare in sezione critica

```c
while(TRUE){
    while (turn =\= i){};
        sezione critica;
    turn = j; (X)
        seione non critica;
};
```

(X): NON garantisce progresso, in quanto processo i se non entra in sez. critica blocca l'accesso agli altri  

#### Algoritmo di Peterson

strutture necessarie

```c
#define FALSE 0
#define TRUE 1
#define N 2

int turn;
int interested[N];
```

```c
void enter_region(int process){
    int other;
    other = 1 - process;
    interested[PROCESS] = TRUE;
    turn = process;
    while (turn == process && interested[other] == TRUE)
}

void leave_region(int process){
    interested[process] = FALSE
}
```

codice $P_i$

```c
while(TRUE){
    enter_region(process);
    sezione critica;
    leave_region(process);
}
```

#### Algoritmo del fornaio

#### istruzioni di Test&set

```c
function Test-and-set (var target: boolean) returns boolean;
    var temp: boolean;
    temp := target;
    target := TRUE;
    return temp;
```

```c
enter_region:
    TSL REGISTER, LOCK 
    CMP REGISTER, #0
    JNE enter_region
    RET

leave_region:
    MOVE LOCK, #0
    RET
```

### SOLUZIONI NON BUSY WAIT

#### PRODUTTORE CONSUMATORE SLEEP/WAKEUP

```c
#define N 100
int count = 0; //numero item nel buffer

void producer(void){
    int item;
    while(TRUE){
        item = produce_item();
        if (count == N) sleep();
        insert_item(item);
        count = count + 1;
        if (count == 1) wakeup(consumer);
    }
}

void consumer (void){
    int item;
    while (TRUE){
        if (count == 0) sleep();
        item = remove_item();
        count = count - 1;
        if (count == N - 1) wakeup (consumer):
        consume_item(item);
    }
}
```

vantaggi: risolve busy wait
criticità: race condition su count, signal perduti con rischio deadlock (necessità di salvare i segnali in attesa)  

### SEMAFORI

#### implementazione semafori

```c
while(TRUE){
    down(mutex);
        sezione critica
    u(mutex);
        sezione non critica
}
```

ESEMPIO DI SINCRONIZZAZIONE

con valore iniziale sync = 0

| Processo $P_1$ | Processo $P_2$ |
|----|----|
|$S_1$ | down(sync)|
|up(sync) | $S_2$ |

in questo modo finché non eseguo S1, S2 non può essere eseguito perché P2 è in waiting su sync

#### PRODUTTORE CONSUMATORE con SEMAFORO

```c
#define N 100
typedef int semaphore;
semaphore mutex = 1;
semaphore empty = N;
semaphore full = 0;

void producer(void){
    int item;
    while(TRUE){
        item = produce_item();
        down(&empty);
        down(&mutex);
        insert_item(item);
        up(&mutex);
        up(&full);
    }
}

void consumer(void){
    int item;
    while(TRUE){
        down(&full);
        down(&mutex);
        item=read_item();
        up(&mutex);
        up(&empty);
    }
}
```

#### implementazione semafori

```c
type semaphore = record
                    value: integer;
                    L: list of process;
                end;

down(S): s.value:=s.value-1
         if s.value < 0
             then begin
                 aggiungi this S.list
                 sleep();
             end;

up(S):  S.value:= s.value+1;
        if s.value <= 0;
            then begin
                togli un processo P da S.list
                wakeup(P);
            end
```

#### implementazione mutex
semafori con due valori, _bloccato_ e _non bloccato_
due primitive, _lock_ e _unlock_

```c
mutex_lock:
    TSL REGISTER, LOCK
    CMP REGISTER, #0 //lock = 0, unlocked
    JZE ok
    CALL thread_yield //cambia thread
    JMP mutex_lock
ok: RET //entra regione critica

mutex_unlock:
    MOVE MUTEX, #0 //lock = 0 unlock
    RET
```

#### esempio di deadlock con semafori

| Processo $P_0$ | Processo $P_1$ |
|----|----|
|down(S) | down(Q)|
|down(Q) | down(S)|
| ... | ... |
| up(Q) | up(S)|
| up(S) | up (Q)|

### Monitor

```
monitor example
    integer i;
    condition c;

    procedure producer();
    ...
    end

    procedure consumer();
    ...
    end

end monitor
```

si usano le variabili condition _c_, con le operazioni:

- wait(c)
- signal(c)

#### Produttore consumatore con monitor

```
monitor ProducerConsumer
    condition full, empty
    integer count;

    procedure insert(item:integer);
    begin
        if count = N then wait(full);
        insert_item(item);
        count := count + 1;
        if count = 1 then signal(empty)
    end;

    function remove:integer;
    begin
        if count = 0 then wait(empty);
        remove = remove_item;
        count := count - 1;
        if count = N - 1 then signal(full)
    end
    count := 0;
end monitor;
```

#### PRODUTTORE CONSUMATORE SCAMBIO MESSAGGI

```c
#define N 100

void producer(void){
    int item;
    message m;
    while(TRUE){
        item = produce_item();
        receive(consumer,&m);
        build_message(&m,item);
        send(consumer,&m);
    }
}

void consumer (void){
    int item;
    message m;
    for (i=0; i<N;i++) send(producer,&m);
    while(TRUE){
        receive(producer,&m);
        item=extract_item(&m);
        send(producer,&m);
        consume_item(item);
    }
}
```

#### I filosofi a cena - soluzione

```c
#define N 5

void philosopher(int i){
    while(TRUE){
        think();
        take_fork(i);
        take:fork((i+1)% N);
        eat();
        put_fork(i);
        put_fork((i+1)% N);
    }
}
```

```c
#define N 5
#define LEFT (i+N-1)%N
#define RIGHT (i+1)%N
#define THINKING 0
#define HUNGRY 1
#define EATING 2
typedef int semaphore;
int state[N]
semaphore mutex=1;
semaphore s[N];

void philosopher(int i){
    while (TRUE){
        think();
        take_forks(i);
        eat();
        put_forks(i);
    }
}

void take_forks(int i){
    down(&mutex);
    state[i]=HUNGRY;
    test(i);
    up(&mutex);
    down(&s[i]);
}

void put_forks(i){
    down(&mutex);
    state[i]=THINKING;
    test(LEFT);
    test(RIGHT);
    up(&mutex);
}

void test(i){
    if (state[i]==HUNGRY && state[LEFT]!=EATING && state[RIGHT]!=EATING){
        state[i]=EATING;
        up(&si);
    }
}
```

#### Lettori e scrittori

```c
typedef int semaphore;
semaphore mutex=1;
semaphore db=1;
int rc=0;

void reader(void){
    while(TRUE){
        down(&mutex);
        rc=rc+1;
        if (rc==1) down(&db);
        up(&mutex);
        read_data_base();
        down(&mutex);
        rc=rc-1;
        if (rc==0) up(&db);
        up(&mutex);
        use_data_read();
    }
}

void writer(void){
    while(TRUE){
        think_up_data();
        down(&db);
        write_data_base();
        up(&db);
    }
}
``` 

#### Il barbiere che dorme

```c
#define N 5
typedef int semaphore;
semaphore customers=0;
semaphore barbers=0;
semaphore mutex=1;
int waiting=0;

void barber(void){
    while(TRUE){
        down(&customers);
        down(&mutex);
        waiting=waiting-1;
        up(&barbers);
        up(&mutex);
        cut_hair();
    }
}

void customer(void){
    down(&mutex);
    if (waiting<N){
        waiting=waiting+1;
        up(&customers);
        up(&mutex);
        down(&barbers);
        get_haircut();
    } else {
        up(&mutex);
    }
}
```
