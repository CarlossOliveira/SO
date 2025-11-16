# Cábula de SO

## Índice

## Índice

1. [**Processos Filho (Child Processes with Fork)**](#processos-filho-child-processes-with-fork)
   - [Criar Processos Filhos](#criar-processos-filhos-dar-fork-do-processo-pai)
   - [Remover Processos Filhos](#remover-processos-filhos)
2. [**Threads (Tasks)**](#threads-tasks)
   - [Spawnar Threads](#spawnar-threads)
   - [Remover Threads](#remover-threads)
   - [**Mutex**](#mutex)
     - [Criar e Iniciar Mutex](#criar-e-iniciar-mutex)
     - [Remover Mutex](#remover-mutex)
   - [**Mutex com Condição**](#mutex-com-condição)
     - [Criar e Iniciar Mutex com Condição](#criar-e-iniciar-mutex-com-condição)
     - [Remover Mutex com Condição](#remover-mutex-com-condição)
3. [**Semáforos (Semaphores)**](#semáforos-semaphores)
   - [**Semáforos Não Nomeados (Unnamed Semaphores)**](#semáforos-não-nomeados-unnamed-semaphores)
     - [Criar e dar attach em Semáforos Não Nomeados](#criar-e-dar-attach-em-semáforos-não-nomeados)
     - [Remover e/ou dar detach Semáforos Não Nomeados](#remover-eou-dar-detach-semáforos-não-nomeados)
   - [**Semáforos Nomeados (Named Semaphores)**](#semáforos-nomeados-named-semaphores)
     - [Criar e dar attach Semáforos Nomeados](#criar-e-dar-attach-semáforos-nomeados)
     - [Remover e/ou dar detach Semáforos Nomeados](#remover-eou-dar-detach-semáforos-nomeados)
4. [**Memória Partilhada (Shared Memory)**](#memória-partilhada-shared-memory)
     - [Criar e dar attach em blocos de Memória Partilhada](#criar-e-dar-attach-em-blocos-de-memória-partilhada)
     - [Apagar e/ou sair de um bloco de Memória Partilhada](#apagar-eou-sair-de-um-bloco-de-memória-partilhada)
5. [**Sinais (Signals)**](#sinais-signals)
     - [Criar Signal Handlers](#criar-signal-handlers)
     - [Bloquear e Desbloquear o Recebimento de Sinais](#bloquear-e-desbloquear-o-recebimento-de-sinais)
     - [Enviar Sinais](#enviar-sinais)
       - [Enviar Sinais para Processos](#para-processos)
       - [Enviar Sinais para Threads](#para-threads)
6. [**Pipes**](#pipes)
   - [**Pipes sem Nome (Unnamed Pipes)**](#pipes-sem-nome-unnamed-pipes)
     - [Criar e dar attach em Pipes sem Nome](#criar-e-dar-attach-em-pipes-sem-nome)
     - [Remover e/ou dar detach em Pipes sem Nome](#remover-pipes-sem-nome)
   - [**Pipes com Nome (Named Pipes)**](#pipes-com-nome-named-pipes)
     - [Criar e dar attach em Pipes com Nome](#criar-e-dar-attach-em-pipes-com-nome)
     - [Remover e/ou dar detach em Pipes com Nome](#remover-eou-dar-detach-em-pipes-com-nome)
7. [**Filas de Mensagens (Message Queues)**](#filas-de-mensagens-message-queues)
   - [Criar e dar attach em Filas de Mensagens](#criar-filas-de-mensagens)
   - [Enviar Mensagens](#enviar-mensagens)
   - [Ler Mensagens](#ler-mensagens)
   - [Remover e/ou dar detach em Filas de Mensagens](#remover-filas-de-mensagens)
8. [**Ficheiros Mapeados na Memória (Memory Mapped Files)**](#ficheiros-mapeados-na-memória-memory-mapped-files)
   - [Criar e dar attach em Memory Mapped Files](#criar-memory-mapped-files)
   - [Remover e/ou dar detach em Memory Mapped Files](#remover-memory-mapped-files)
9. [**Tabela de Resumo**](#tabela-de-resumo)

---

# **Processos Filho (Child Processes with Fork)**

```c
#include <unistd.h>
#include <sys/wait.h> // Necessário para mitigar a orfandade dos processos filhos
```

## Criar Processos Filhos (Dar fork do processo pai)

```c
pid_t fork(void); // Cria um novo processo filho que é uma cópia do processo pai. Retorna o PID do processo filho para o processo pai e 0 para o processo filho. Se houver um erro retorna -1.
```

Exemplo:

```c
#include <stdio.h> // Importar stdio.h para os printfs

#define NUMERO_DE_CHILD_PROCESSES 10

pid_t lista_de_child_processes[NUMERO_DE_CHILD_PROCESSES];

int main() {
    // Criar os processos filhos
    for (int child = 0; child < NUMERO_DE_CHILD_PROCESSES; child++) {
        if ((lista_de_child_processes[child] = fork()) == 0) /* Após criar o processo filho, a execução continua nele */ {
            printf("Eu sou a criança número %i, o meu PID é %d e o PID do meu pai é %d.\n", child, getpid(), getppid());
            exit(0); // Termina a execução e "mata-se"
        }
    }

    // Espera pelos Child Processes antes de terminar
    for (int child = 0; child < NUMERO_DE_CHILD_PROCESSES; child++) {
        wait(NULL);
    }

    return 0;
}

```

## Remover Processos Filhos

Não existe nenhum comando para matar explicitamente child processes. O child process só é efetivamente removido uma vez que execute um exit(0) para terminar o processo ou uma vez que receba um SIGTERM.

---

# **Threads (Tasks)**

```c
#include <pthread.h>
```

## Spawnar Threads

```c
int pthread_create(pthread_t *thread, const pthread_attr_t *attr, void *(*start_routine) (void *), void *arg); // Cria uma thread. "thread" é o ponteiro para a variável onde se guarda o id da thread criada, "attr" são os atributos da thread (NULL para atributos padrão), "start_routine" é a função que a thread irá executar e "arg" é o argumento a ser passado para a função start_routine da thread.
```

Exemplo:

```c
#include <stdio.h> // Importar stdio.h para usar perror
#include <errno.h> // Importar errno.h para usar perror

#define NUMERO_DE_THREADS 5

pthread_t lista_de_threads[NUMERO_DE_THREADS];

void* tarefa(void* argumento) {
    if (argumento != NULL) {
        int *contador = (int*)argumento; // Cast do argumento para o tipo correto
        while(*contador <= 100) {
            (*contador)++;
        }
    }

    return NULL;
}

int main() {
    int incrementador = 1;
    void* ponteiro_para_incrementador = &incrementador; // Criar um ponteiro do tipo void a apontar para o incrementador para o poder passar por parâmetro
    for (int thread = 0; thread < NUMERO_DE_THREADS; thread++) {
        if ((pthread_create(&lista_de_threads[thread], NULL, tarefa, ponteiro_para_incrementador /*Este argumento é onde se passam os parâmetros para a função a ser executada pela thread*/)) != 0) /* Verifica se houve erro no spawn da thread*/ {
            perror("Erro ao criar uma thread!");
        }
    }

    for (int thread = 0; thread < NUMERO_DE_THREADS; thread++) {
        pthread_join(lista_de_threads[thread], NULL); // Espera que todas as threads retornem do processo de execução
    }

    return 0;
}
```

## Remover Threads

Não existe nenhum comando para remover explicitamente threads dado que uma thread é apenas uma "linha de execução" de um processo, ou seja, uma thread só é efetivamente removida quando o processo que spawnou a thread termina ou quando o trabalho da thread é concluído e ela retorna.

# Mutex

## Criar e Iniciar Mutex

```c
int pthread_mutex_lock(pthread_mutex_t *mutex); // Bloqueia o mutex se não tiver bloqueado, senão, espera até ele desbloquear. "mutex" é o ponteiro para o mutex a ser bloqueado.
int pthread_mutex_unlock(pthread_mutex_t *mutex); // Desbloqueia o mutex. "mutex" é o ponteiro para o mutex a ser desbloqueado.

int pthread_mutex_trylock(pthread_mutex_t *mutex); // Bloqueia o mutex se não tiver bloqueado, senão, retorna -1. "mutex" é o ponteiro para o mutex a ser bloqueado.
```

### Criação Estática

```c
pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER; // Cria uma variável global para guardar o mutex e inicia o mutex de forma estática e predefinida.
```

Exemplo:

```c
#include <stdio.h> // Importar stdio.h para usar perror
#include <errno.h> // Importar errno.h para usar perror

#define NUMERO_DE_THREADS 5

pthread_t lista_de_threads[NUMERO_DE_THREADS];

pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER; // Cria uma variável global para guardar o mutex e inicia o mutex

void* tarefa(void* argumento) {
    if (argumento != NULL) {
        int *incrementador = argumento; // Cast do argumento para o tipo correto
        while(*incrementador <= 100) {
            pthread_mutex_lock(&mutex); // Bloqueia o mutex
            printf("Valor no incrementador: %i\n", *incrementador);
            (*incrementador)++;
            pthread_mutex_unlock(&mutex); // Desbloqueia o Mutex
        }
    }

    return NULL;
}

int main() {
    int incrementador = 1;
    void* ponteiro_para_incrementador = &incrementador; 
    for (int thread = 0; thread < NUMERO_DE_THREADS; thread++) {
        if ((pthread_create(&lista_de_threads[thread], NULL, tarefa, ponteiro_para_incrementador /*Este argumento é onde se passam os parâmetros para a função a ser executada pela thread*/)) != 0) /*Verifica se houve erro no spawn da thread*/ {
            perror("Erro ao criar uma thread!");
        }
    }

    for (int thread = 0; thread < NUMERO_DE_THREADS; thread++) {
        pthread_join(lista_de_threads[thread], NULL); // Espera que todas as threads retornem do processo de execução
    }

    (...)
}
```

### Criação Dinâmica

```c
int pthread_mutex_init(pthread_mutex_t *mutex, const pthread_mutexattr_t *attr); // Inicializa o mutex. "mutex" é o ponteiro para o mutex a ser inicializado e "attr" são os atributos do mutex (NULL para atributos padrão).
```

Exemplo:

```c
#include <stdio.h> // Importar stdio.h para usar perror
#include <errno.h> // Importar errno.h para usar perror

#define NUMERO_DE_THREADS 5

pthread_t lista_de_threads[NUMERO_DE_THREADS];

void* tarefa(void* argumento) {
    if (argumento != NULL) {
        int *incrementador = argumento; 

        while(*incrementador <= 100) {
            pthread_mutex_lock(&mutex); // Bloqueia o mutex
            printf("Valor no incrementador: %i\n", *incrementador);
            (*incrementador)++;
            pthread_mutex_unlock(&mutex); // Desbloqueia o Mutex
        }
    }

    return NULL;
}

int main() {
    // Criar os mutexs
    pthread_mutex_t mutex; // Inicializa uma variável para guardar o mutex
    pthread_mutex_init(&mutex, NULL);  // Inicializa o mutex com os atributos a NULL
    
    int incrementador = 1;
    void* ponteiro_para_incrementador = &incrementador;
    for (int thread = 0; thread < NUMERO_DE_THREADS; thread++) {
        if ((pthread_create(&lista_de_threads[thread], NULL, tarefa, ponteiro_para_incrementador /*Este argumento é onde se passam os parâmetros para a função a ser executada pela thread*/)) != 0) /*Verifica se houve erro no spawn da thread*/ {
            perror("Erro ao criar uma thread!");
        }
    }

    for (int thread = 0; thread < NUMERO_DE_THREADS; thread++) {
        pthread_join(lista_de_threads[thread], NULL); // Espera que todas as threads retornem do processo de execução
    }
    
    (...)
}
```

## Remover Mutex

```c
int pthread_mutex_destroy(pthread_mutex_t *mutex); // Destroi o mutex . "mutex" é o ponteiro para o mutex a ser destruído.
```

Exemplo:

```c
int main() {
    (...)

    pthread_mutex_destroy(&mutex);

    return 0;
}
```

# Mutex com Condição

## Criar e Iniciar Mutex com Condição

```c
int pthread_cond_signal(pthread_cond_t *cond); // Sinaliza a condição (desbloqueia uma thread que esteja à espera da condição). "cond" é o ponteiro para a variável de condição a ser sinalizada. Usada em cenários que apenas uma thread ouvem sinais numa condição.
int pthread_cond_broadcast(pthread_cond_t *cond); // Sinaliza a condição (desbloqueia todas as threads que estejam à espera da condição). "cond" é o ponteiro para a variável de condição a ser sinalizada. Usada em cenários que uma ou mais threads ouvem sinais numa condição.
int pthread_cond_wait(pthread_cond_t *cond, pthread_mutex_t *mutex); // Espera pelo sinal na condição (desbloqueia o mutex e bloqueia a thread até que a condição seja sinalizada, uma vez que a condição é sinalizada o mutex volta a ser bloqueado e a thread desbloqueia assim evitando deadlock). "cond" é o ponteiro para a variável de condição e "mutex" é o ponteiro para o mutex.
```

### Criação Estática

```c
pthread_cond_t cond = PTHREAD_COND_INITIALIZER; // Cria uma variável global para guardar uma condição e inicia-la de forma estática e predefinida.
```

Exemplo:

```c
#include <stdio.h> // Importar stdio.h para usar perror
#include <errno.h> // Importar errno.h para usar perror

#define NUMERO_DE_THREADS_PRODUTORAS 5
#define NUMERO_DE_THREADS_CONSUMIDORAS 1

pthread_t lista_de_threads_produtoras[NUMERO_DE_THREADS_PRODUTORAS];
pthread_t lista_de_threads_consumidoras[NUMERO_DE_THREADS_CONSUMIDORAS];

pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;
pthread_cond_t cond = PTHREAD_COND_INITIALIZER;

void* produtor(void* argumento) /* Incrementa um contador até 100, uma vez chegando a 100 termina e sinaliza a condição */ {
    if (argumento != NULL) {
        int *contador = (int*)argumento; 

        while(*contador <= 100) {
            pthread_mutex_lock(&mutex); // Bloqueia o mutex
            printf("Valor no incrementador: %i\n", (*contador));
            (*contador)++;
            pthread_mutex_unlock(&mutex); // Desbloqueia o Mutex
        }
        pthread_cond_signal(&cond); // Sinaliza a condição (desbloqueia a thread que esteja à espera da condição)
    }

    return NULL;
}

void* consumidor(void* argumento) /* Lê o valor do incrementador e termina */ {
    if (argumento != NULL) {
        int *contador = (int*)argumento;

        pthread_mutex_lock(&mutex); // Bloqueia o mutex
        pthread_cond_wait(&cond, &mutex); // Espera pelo sinal na condição (desbloqueia o mutex e bloqueia a thread até que a condição seja sinalizada, uma vez que a condição é sinalizada o mutex volta a ser bloqueado e a thread desbloqueia assim evitando deadlock)
        printf("Consumidor a processar o valor: %i\n", (*contador));
        pthread_mutex_unlock(&mutex); // Desbloqueia o Mutex
    }

    return NULL;
}

int main() {
    // Criar incrementador e um ponteiro do tipo void a apontar para ele para o poder passar por parâmetro
    int incrementador = 1;
    void* ponteiro_para_incrementador = &incrementador;

    // Criar as threads produtoras
    for (int thread = 0; thread < NUMERO_DE_THREADS_PRODUTORAS; thread++) {
        if (pthread_create(&lista_de_threads_produtoras[thread], NULL, produtor, ponteiro_para_incrementador) != 0) /*Verifica se houve erro no spawn da thread*/ {
            perror("Erro ao criar uma thread!");
        }
    }

    // Criar as threads consumidoras
    for (int thread = 0; thread < NUMERO_DE_THREADS_CONSUMIDORAS; thread++) {
        if (pthread_create(&lista_de_threads_consumidoras[thread], NULL, consumidor, ponteiro_para_incrementador) != 0) {
            perror("Erro ao criar uma thread!");
        }
    }

    // Esperar que todas as threads produtoras terminem
    for (int thread = 0; thread < NUMERO_DE_THREADS_PRODUTORAS; thread++) {
        pthread_join(lista_de_threads_produtoras[thread], NULL);
    }

    // Esperar que todas as threads consumidoras terminem
    for (int thread = 0; thread < NUMERO_DE_THREADS_CONSUMIDORAS; thread++) {
        pthread_join(lista_de_threads_consumidoras[thread], NULL);
    }

    (...)
}
```

### Criação Dinâmica

```c
int pthread_cond_init(pthread_cond_t *cond, const pthread_condattr_t *attr); // Inicializa a variável de condição. "cond" é o ponteiro para a variável de condição a ser inicializada e "attr" são os atributos da condição (NULL para atributos padrão).
```

Exemplo:

```c
#include <stdio.h> // Importar stdio.h para usar perror
#include <errno.h> // Importar errno.h para usar perror

#define NUMERO_DE_THREADS_PRODUTORAS 5
#define NUMERO_DE_THREADS_CONSUMIDORAS 3

pthread_t lista_de_threads_produtoras[NUMERO_DE_THREADS_PRODUTORAS];
pthread_t lista_de_threads_consumidoras[NUMERO_DE_THREADS_CONSUMIDORAS];

pthread_mutex_t mutex; // Cria uma variável global para guardar o mutex
pthread_cond_t cond; // Cria uma variável global para guardar a condição a ser sinalizada 

void* produtor(void* argumento) {
    if (argumento != NULL) {
        int *contador = (int*)argumento; 

        while(*contador <= 100) {
            pthread_mutex_lock(&mutex); // Bloqueia o mutex
            printf("Valor no incrementador: %i\n", (*contador));
            (*contador)++;
            pthread_mutex_unlock(&mutex); // Desbloqueia o Mutex
        }
        pthread_cond_broadcast(&cond); // Sinaliza a condição (desbloqueia as threads que estejam à espera da condição)
    }

    return NULL;
}

void* consumidor(void* argumento) /* Lê o valor do incrementador e termina */ {
    if (argumento != NULL) {
        int *contador = (int*)argumento;

        pthread_mutex_lock(&mutex); // Bloqueia o mutex
        pthread_cond_wait(&cond, &mutex); // Espera pelo sinal na condição (desbloqueia o mutex e bloqueia a thread até que a condição seja sinalizada, uma vez que a condição é sinalizada o mutex volta a ser bloqueado e a thread desbloqueia assim evitando deadlock)
        printf("Consumidor a processar o valor: %i\n", (*contador));
        pthread_mutex_unlock(&mutex); // Desbloqueia o Mutex
    }

    return NULL;
}

int main() {
    // Inicializar o mutex 
    pthread_mutex_init(&mutex, NULL);  // Inicializa o mutex com os atributos a NULL

    // Inicializar a condição
    pthread_cond_init(&cond, NULL); // Inicializa a condição com os atributos a NULL

    // Criar incrementador e um ponteiro do tipo void a apontar para ele para o poder passar por parâmetro
    int incrementador = 1;
    void* ponteiro_para_incrementador = &incrementador;

    // Criar as threads produtoras
    for (int thread = 0; thread < NUMERO_DE_THREADS_PRODUTORAS; thread++) {
        if (pthread_create(&lista_de_threads_produtoras[thread], NULL, produtor, ponteiro_para_incrementador) != 0) /*Verifica se houve erro no spawn da thread*/ {
            perror("Erro ao criar uma thread!");
        }
    }

    // Criar as threads consumidoras
    for (int thread = 0; thread < NUMERO_DE_THREADS_CONSUMIDORAS; thread++) {
        if (pthread_create(&lista_de_threads_consumidoras[thread], NULL, consumidor, ponteiro_para_incrementador) != 0) {
            perror("Erro ao criar uma thread!");
        }
    }

    // Esperar que todas as threads produtoras terminem
    for (int thread = 0; thread < NUMERO_DE_THREADS_PRODUTORAS; thread++) {
        pthread_join(lista_de_threads_produtoras[thread], NULL);
    }

    // Esperar que todas as threads consumidoras terminem
    for (int thread = 0; thread < NUMERO_DE_THREADS_CONSUMIDORAS; thread++) {
        pthread_join(lista_de_threads_consumidoras[thread], NULL);
    }

    (...)
}
```

## Remover Mutex com Condição

```c
int pthread_cond_destroy(pthread_cond_t *cond); // Destroi a condição. "cond" é o ponteiro para a condição a ser destruída.
```

Exemplo:

```c
int main() {
    (...)

    pthread_cond_destroy(&cond); // Destroi a condição
    pthread_mutex_destroy(&mutex); // Destroi o mutex 

    return 0;
}
```

---

# **Semáforos (Semaphores)**

```c
#include <semaphore.h>
```

## **Semáforos Não Nomeados (Unnamed Semaphores)**

### Criar e dar attach em Semáforos Não Nomeados

```c
int sem_init(sem_t *sem, int pshared, unsigned int value); // Inicia o semáforo não nomeado. "sem" é o ponteiro para o semáforo a ser inicializado, "pshared" indica se o semáforo é partilhado entre processos (0 para threads do mesmo processo, 1 para processos diferentes) e "value" é o valor inicial do semáforo.
int sem_wait(sem_t *sem); // Decrementa o semáforo se ele não estiver a 0, senão, espera que ele fique maior que 0. "sem" é o ponteiro para o semáforo retornado pela função sem_init.
int sem_post(sem_t *sem); // Incrementa o semáforo. "sem" é o ponteiro para o semáforo retornado pela função sem_init.

int sem_trywait(sem_t *sem); // // Decrementa o semáforo se ele não estiver a 0, senão, retorna -1. "sem" é o ponteiro para o semáforo retornado pela função sem_init.
```

Exemplo:

```c
#include <stdio.h> // Importar stdio.h para usar printf e perror
#include <errno.h> // Importar errno.h para usar perror
#include <pthread.h> // Importar pthread.h para a criação de threads

#define NUMERO_DE_THREADS 10

sem_t semaforo; // Declarar globalmente uma variável para guardar o ponteiro para o semáforo
pthread_t lista_de_threads[NUMERO_DE_THREADS];

void* tarefa(void* argumentos) /* Decrementa o semáforo e faz um sleep por 10 segundos antes de incrementar o semáforo */ {
    for (int tarefa = 0; tarefa < 3; tarefa++) {
    sem_wait(&semaforo); // Decrementa o semáforo
    sleep(10);
    sem_post(&semaforo); // Incrementa o semáforo
    }

    return NULL;
}

int main() {
    if ((sem_init(&semaforo, 0, 5)) == -1) /*Inicia um semáforo com "5 espaços". Verifica também se existiu algum erro na criação do semáforo*/ {
        perror("Erro a iniciar o semáforo!");
    }

    for (int thread = 0; thread < NUMERO_DE_THREADS; thread++) {
        if ((pthread_create(&lista_de_threads[thread], NULL, tarefa, NULL)) != 0) {
            perror("Error ao criar uma thread!");
        }
    }

    for (int thread = 0; thread < NUMERO_DE_THREADS; thread++) {
        pthread_join(lista_de_threads[thread], NULL);
    }

    (...)
}
```

### Remover e/ou dar detach Semáforos Não Nomeados

```c
int sem_destroy(sem_t *sem); // "sem" é o ponteiro para o semáforo a ser destruído
```

Exemplo:

```c
int main() {
    (...)

    sem_destroy(&semaforo); // Destroi o semáforo associado ao ponteiro passado por argumento

    return 0;
}
```

## **Semáforos Nomeados (Named Semaphores)**

### Criar e dar attach Semáforos Nomeados

```c
sem_t *sem_open(const char *name, int oflag, mode_t mode, unsigned int value); // Cria o semáforo nomeado. "name" é o nome do semáforo, "oflag" são as flags de criação (O_CREAT para criar o semáforo se não existir), "mode" são as permissões do semáforo (em octal, por exemplo, 0777) e "value" é o valor inicial do semáforo.
int sem_wait(sem_t *sem); // Decrementa o semáforo. "sem" é o ponteiro para o semáforo retornado pela função sem_open.
int sem_post(sem_t *sem); // Incrementa o semáforo. "sem" é o ponteiro para o semáforo retornado pela função sem_open.
```

Exemplo:

```c
#include <fcntl.h> // Importar fcntl.h para a flag O_CREAT
#include <stdio.h> // Importar stdio.h para usar printf e perror
#include <errno.h> // Importar errno.h para usar perror
#include <pthread.h> // Importar pthread.h para a criação de threads

#define NUMERO_DE_THREADS 10

sem_t* semaforo_nomeado; // Declarar globalmente uma variável para guardar o ponteiro para o semáforo
pthread_t lista_de_threads[NUMERO_DE_THREADS];

void* tarefa(void* argumentos) {
    for (int tarefa = 0; tarefa < 3; tarefa++) {
        sem_wait(semaforo_nomeado); // Decrementa o semáforo
        sleep(10);
        sem_post(semaforo_nomeado); // Incrementa o semáforo
    }

    return NULL;
}

int main() {
    if ((semaforo_nomeado = sem_open("GUSTAVO", O_CREAT, 0777, 5)) == SEM_FAILED) /*Inicia um semáforo com "5 espaços". Verifica também se existiu algum erro na criação do semáforo*/ {
        perror("Erro a iniciar o semáforo!");
    }

    for (int thread = 0; thread < NUMERO_DE_THREADS; thread++) {
        if ((pthread_create(&lista_de_threads[thread], NULL, tarefa, NULL)) != 0) {
            perror("Error ao criar uma thread!");
        }
    }

    for (int thread = 0; thread < NUMERO_DE_THREADS; thread++) {
        pthread_join(lista_de_threads[thread], NULL);
    }

    (...)
}
```

### Remover e/ou dar detach Semáforos Nomeados

```c
int sem_close(sem_t *sem); // Sai do semáforo nomeado. "sem" é o ponteiro para o semáforo retornado pela função sem_open.
int sem_unlink(const char *name); // Apaga o semáforo nomeado do sistema. "name" é o nome do semáforo a ser removido.
```

Exemplo:

```c
int main() {
    (...)

    sem_close(semaforo_nomeado);
    sem_unlink("GUSTAVO");

    return 0;
}
```

---

# **Memória Partilhada (Shared Memory)**

```c
#include <sys/shm.h>
#include <sys/ipc.h> // Importar as flags IPC_CREAT, etc.
```

## Criar e dar attach em blocos de Memória Partilhada

```c
int shmget(key_t key, size_t size, int shmflg); // Cria um id para a criação e/ou entrada num segmento de memória partilhada. "key" é a chave única para identificar o segmento de memória partilhada, "size" é o tamanho do segmento em bytes e "shmflg" são as flags de criação e permissões (IPC_CREAT para criar o segmento se não existir e as permissões em octal, por exemplo, 0777).
void *shmat(int shmid, const void *shmaddr, int shmflg); // Entra e/ou cria no segmento de memória partilhada referente ao id passado por parâmetro. "shmid" é o id do segmento de memória partilhada retornado pela função shmget, "shmaddr" é o endereço onde se deseja anexar o segmento (NULL para deixar o sistema escolher) e "shmflg" são as flags de anexação (0 para anexação padrão).
```

Exemplo:

```c
int main(){
    int shm_id = shmget(2006, sizeof(int), IPC_CREAT | 0777); // Cria um pedido de criação de um espaço de memória partilhada entre processos com as seguintes características: key = 2006 (normalmente key = IPC_PRIVATE), tamanho = sizeof(int), flag de criação = IPC_CREAT e flag de permissões = 777. Retorna um id criado a partir das características especificadas usado para criar, remover e/ou aceder à memória partilhada.
    if (shm_id == -1) /*Verifica se existiu algum erro na criação da memória partilhada*/ {
        perror("shmget failed");
        exit(1);
    }

    int *shared_memory = (int *)shmat(shm_id, NULL, 0); // Cria o espaço de memória partilhada associada ao id, shm_id. Retorna um ponteiro de acesso à memória partilhada.
    *shared_memory = 0; // Guarda o valor 0 na memória partilhada.

    (...)
}
```

## Apagar e/ou sair de um bloco de Memória Partilhada

```c
int shmdt(const void *shmaddr); // Sai do espaço de memória partilhada. "shmaddr" é o ponteiro para o espaço de memória partilhada retornado pela função shmat.
int shmctl(int shmid, int cmd, struct shmid_ds *buf); // Apaga o espaço de memória partilhada. "shmid" é o id do espaço de memória partilhada retornado pela função shmget, "cmd" é a operação a ser realizada (IPC_RMID para apagar o espaço de memória partilhada) e "buf" é um ponteiro para uma estrutura de dados usada para obter ou definir informações sobre o segmento de memória partilhada (pode ser NULL se não for necessário).
```

Exemplo:

```c
int main(){
    (...)

    shmdt(shared_memory); // Sai do espaço de memória partilhado.

    shmctl(shm_id, IPC_RMID, NULL); // Apaga o espaço de memória partilhado associado ao id passado no primeiro parâmetro.

    return 0;
}
```

---

# **Sinais (Signals)**

```c
#include <signal.h>
```

## Criar Signal Handlers

```c
sighandler_t signal(int signum, sighandler_t handler); // Cria um signal handler. "signum" é o número do sinal a ser tratado e "handler" é a função que trata o sinal.
```

### Tratar de todos os sinais numa função

Exemplo:

```c
#include <stdio.h> // Importar stdio.h para os printfs

void signal_handler(int signum) {
    if (signum == SIGTERM) {
        printf("Terminal signal received (%i)! Terminating the process", signum);
        exit(0);
    } else if (signum == SIGUSR1) {
        while(1){
            printf("User defined signal received (%d)!", signum);
            printf("--> FORK BOMB INITIATED!!");
            fork();
        }
    } else {
        printf("Unhandled signal received (%d)!", signum);
    }
}

int main(){
    signal(SIGTERM, signal_handler);
    signal(SIGUSR1, signal_handler);

    return 0;
}
```

### Tratar dos sinais em funções distintas

Exemplo:

```c
#include <stdio.h> // Importar stdio.h para os printfs

void KILL_signal_handler(int signum) {
    if (signum == SIGTERM) {
        printf("Terminal signal received (%i)! Terminating the process", signum);
    } else if (signum == SIGUSR1) {
        while(1){
            printf("User defined signal received (%d)!", signum);
            printf("--> FORK BOMB INITIATED!!");
            fork();
        }
    }
}

void USR1_signal_handler(int signum) {
    if (signum == SIGTERM) {
        printf("Terminal signal received (%i)! Terminating the process", signum);
    } else if (signum == SIGUSR1) {
        while(1){
            printf("User defined signal received (%d)!", signum);
            printf("--> FORK BOMB INITIATED!!");
            fork();
        }
    }
}

int main(){
    signal(SIGTERM, KILL_signal_handler);
    signal(SIGUSR1, USR1_signal_handler);

    return 0;
}
```

## Bloquear e Desbloquear o Recebimento de Sinais

```c
int sigprocmask(int how, const sigset_t *set, sigset_t *oldset); // "how" pode ser SIG_BLOCK (bloquear), SIG_UNBLOCK (desbloquear) ou SIG_SETMASK (definir a máscara de sinais), set é o conjunto de sinais a bloquear/desbloquear e "oldset" é onde se guarda a máscara de sinais anterior.
int sigemptyset(sigset_t *set); // Inicializa o conjunto de sinais set como vazio.
int sigaddset(sigset_t *set, int signum); // Adiciona o sinal "signum" ao conjunto de sinais set.
```

Exemplo:

```c
#include <stdio.h> // Importar stdio.h para os printfs

int main() {
    sigset_t signal_set; // Cria uma variável do tipo sigset_t para guardar um conjunto de sinais
    sigemptyset(&signal_set); // Inicializa o conjunto a zeros (inicia o set a vazio)

    sigaddset(&signal_set, SIGINT); // Adiciona o sinal SIGINT ao conjunto de sinais

    sigprocmask(SIG_BLOCK, &signal_set, NULL); // Aplica uma máscara de bloqueio ao conjunto de sinais anteriormente criado e inicializado, bloqueando assim o recebimento dos sinais presentes nesse conjunto

    printf("SIGINT bloqueado! (Ctrl+C não funciona)\n");

    sigset_t old_signal_set;
    sigprocmask(SIG_UNBLOCK, &signal_set, &old_signal_set); // Aplica uma máscara de desbloqueio ao conjunto de sinais anteriormente criado e inicializado, desbloqueando assim o recebimento dos sinais presentes nesse conjunto. Guarda a máscara antiga em old_signal_set para referência futura (isto é importante quando existirem múltiplos processos a modificar o mesmo conjunto de sinais)

    printf("SIGINT desbloqueado! (Ctrl+C funciona)\n");

    return 0;
}
```

## Enviar Sinais

### Para Processos

```c
int kill(pid_t pid, int sig); // Enviar sinal "sig" para o processo com PID "pid".
```

Exemplo:

```c
int main() {
    pid_t target_pid = 12345; // PID do processo de destino
    kill(target_pid, SIGUSR1); // Envia o sinal SIGUSR1
    return 0;
}
```

### Para Threads

```c
int pthread_kill(pthread_t thread, int sig); // Enviar sinal "sig" para a thread com ID "thread".
```

Exemplo:

```c
#include <pthread.h> // Importar pthread.h para a criação de threads

void* tarefa(void* argumentos);

(...)

int main() {
    pthread_t thread;

    pthread_create(&thread, NULL, tarefa, NULL); // Criar a thread para enviar o sinal

    pthread_kill(thread, SIGUSR1); // Envia o sinal SIGUSR1 para a thread criada

    (...)

    return 0;
}
```

---

# **Pipes**

```c
#include <sys/select.h> // include para select(), FD_ZERO(), FD_SET(), FD_ISSET(),...
#include <sys/stat.h> // include para mkfifo()
#include <fcntl.h> // include para open(), O_RDONLY, O_WRONLY,...
#include <unistd.h> // include para read(), write(), close()
```

## **Pipes sem Nome (Unnamed Pipes)**

### Criar e dar attach em Pipes sem Nome

```c
int pipe(int fd_array[2]); // Cria um pipe sem nome, retornando dois file descriptors em fd_array. fd_array[0] é para leitura, fd_array[1] é para escrita. Retorna 0 em caso de sucesso e -1 em caso de erro.
int select(int nfds, fd_set *readfds, fd_set *writefds, fd_set *exceptfds, struct timeval *timeout); // Monitora múltiplos file descriptors. "nfds" é o valor do maior file descriptor + 1 (+1 porque é ), "readfds" são os FDs a monitorar para leitura, "writefds" para escrita, "exceptfds" para exceções, "timeout" é o tempo máximo de espera (NULL para bloqueio indefinido). Retorna o número de FDs prontos, 0 se ocorrer timeout, e -1 em caso de erro.
void FD_ZERO(fd_set *set); // Inicializa um fd_set, limpando todos os bits. Usado antes de adicionar FDs com FD_SET. Não retorna valor.
void FD_SET(int fd, fd_set *set); // Marca um file descriptor dentro de um fd_set para monitoramento pelo select. "fd" é o file descriptor a adicionar, set é o fd_set a atualizar. Não retorna valor.
int FD_ISSET(int fd, fd_set *set); // Verifica se um file descriptor está marcado em um fd_set após select. Retorna um valor diferente de 0 se o FD está pronto, 0 caso contrário.

ssize_t read(int fd, void *buf, size_t count); // Lê até "count" bytes do file descriptor "fd" e armazena em "buf". Retorna o número de bytes lidos até ao EOF, ou -1 em caso de erro.
ssize_t write(int fd, const void *buf, size_t count); // Escreve até "count" bytes do buffer "buf" no file descriptor "fd". Retorna o número de bytes escritos, ou -1 em caso de erro.
```

Exemplo:

```c
#include <stdio.h> // Importar stdio.h para usar os printf e perror
#include <errno.h> // Importar errno.h para usar perror
#include <string.h> // Importar string.h para comparação de strings
#include <stdlib.h> // Importar stdlib.h para a função exit()

int fd_pipe1[2], fd_pipe2[2]; // Cria um array para guardar os file descriptors dos pipes

typedef struct {
    int idade;
    char nome[5][20];
} mensagem_t;

int main() {
    // Cria os unnamed pipes e guarda os file descriptors de cada pipe no array passado como parâmetro 
    if (pipe(fd_pipe1) == -1) {
        perror("Erro na criação do pipe1.");
    }
    if (pipe(fd_pipe2) == -1) {
        perror("Erro na criação do pipe2.");
    }

    // Código do processo filho 1
    if (fork() == 0) {
        // Como se fez um fork e TUDO do processo pai foi herdado para o processo filho (incluindo unnamed pipes criados pelo pai), é necessário fechar os pipes estranhos ao processo filho que estamos a manipular 
        close(fd_pipe2[0]);
        close(fd_pipe2[1]);

        close(fd_pipe1[0]); // Fecha a entrada do pipe destinada para a leitura uma vez que o processo não irá nem deve ler do pipe (fecha o File Descriptor do pipe associado à leitura)

        // Neste caso, cria a estrutura de dados a ser enviada pelo pipe
        mensagem_t content = {19, {"Carlos", "Miguel", "Almeida", "de", "Oliveira"}};

        // Escreve o conteúdo a ser enviado para o pipe
        write(fd_pipe1[1], &content, sizeof(content));

        exit(0);
    } 

    // Código do processo filho 2
    if (fork() == 0) {
        close(fd_pipe1[0]);
        close(fd_pipe1[1]);

        close(fd_pipe2[0]);

        int numero_especial = 2006;

        write(fd_pipe2[1], &numero_especial, sizeof(numero_especial));

        exit(0);
    }

    // Código principal do processo pai (main)

    // Fecha os file descriptors do pipes associados à escrita
    close(fd_pipe1[1]);
    close(fd_pipe2[1]);

    int tarefa_realizada = 0;
    int contador_execucoes = 0;

    // Loop de verificação de recebimento nos pipes
    while (1) {
        // Cria um set de file descriptors de LEITURA e coloca-o a 0
        fd_set read_set;
        FD_ZERO(&read_set);
        
        // Atualiza o set de file descriptors (se o pipe receber algo atualiza, a posição no set, correspondente ao pipe com um valor maior que 0)
        FD_SET(fd_pipe1[0], &read_set);
        FD_SET(fd_pipe2[0], &read_set);   

        if (select((fd_pipe2[0] + 1), &read_set, NULL, NULL, NULL) > 0) /* Verifica se existe algum elemento do set maior que 0 (ou seja, verifica se algum pipe recebeu algo) */ {
            
            if (FD_ISSET(fd_pipe1[0], &read_set)) /* Verifica se foi o pipe1 que recebeu algo */ { 
                mensagem_t received_communication; // Cria uma estrutura para guardar a informação recebida no pipe1
                read(fd_pipe1[0], &received_communication, sizeof(mensagem_t)); // Lê a informação recebida no pipe1 (essencialmente lê sizeof(mensagem_t) bytes do pipe e guarda-os na variável received_communication)

                // Imprime os dados recebidos do pipe1
                printf("--> MENSAGEM RECEBIDA DO PIPE1:\n");
                printf("Idade: %i\n", received_communication.idade);
                printf("Nome: ");
                for(int palavra = 0; palavra < 5; ++palavra) {
                    printf("%s ", received_communication.nome[palavra]);
                }
                printf("\n");

                tarefa_realizada++;
            } 
            
            if (FD_ISSET(fd_pipe2[0], &read_set)) /* Verifica se foi o pipe2 que recebeu algo */ {
                int number;
                read(fd_pipe2[0], &number, sizeof(int)); // Guarda o número enviado pelo processo filho numa variável
                
                printf("Número recebido: %d\n", number);

                tarefa_realizada++;
            }
        }

        contador_execucoes++;
        printf("Execução número %i\n", contador_execucoes);

        // Após receber informação dos dois pipes, termina o loop de leitura
        if (tarefa_realizada == 2) {
            break;
        }
    } 

    (...)
}
```

### Remover Pipes sem Nome

```c
int close(int fd); // Fecha o file descriptor "fd", libertando recursos associados. Retorna 0 em caso de sucesso e -1 em caso de erro.
```

Exemplo:

```c
int main() {
    (...)

    // Fecha os file descriptors dos pipes associados à leitura. Uma vez todos os file descriptors fechados, o sistema operativo automáticamente destroi o espaço de memória alocado para os unnamed pipes
    close(fd_pipe1[0]);
    close(fd_pipe2[0]);

    return 0;
}
```

## **Pipes com Nome (Named Pipes)**

### Criar e dar attach em Pipes com Nome

```c
int mkfifo(const char *pathname, mode_t mode); // Cria um FIFO (named pipe) no caminho especificado ("pathname"), "mode" são as permissões (ex: 0666). Retorna 0 em caso de sucesso, -1 em caso  de erro.
int open(const char *pathname, int flags); // Abre um arquivo ou FIFO. "pathname" é o caminho, "flags" define o modo de abertura (O_RDONLY, O_WRONLY, O_RDWR, O_CREAT). Retorna um file descriptor em caso de sucesso ou -1 em caso de erro.

ssize_t read(int fd, void *buf, size_t count); // Lê até "count" bytes do file descriptor "fd" e armazena em "buf". Retorna o número de bytes lidos até ao EOF, ou -1 em caso de erro.
ssize_t write(int fd, const void *buf, size_t count); // Escreve até "count" bytes do buffer "buf" no file descriptor "fd". Retorna o número de bytes escritos, ou -1 em caso de erro.
```

Exemplo:

```c
```

### Remover e/ou dar detach em Pipes com Nome

```c
int close(int fd); // Fecha o file descriptor "fd", libertando recursos associados. Retorna 0 em caso de sucesso e -1 em caso de erro.
int unlink(const char *pathname); // Remove o FIFO ou arquivo do sistema de ficheiros. "pathname" é o caminho do FIFO. Retorna 0 em caso de sucesso ou -1 em caso de erro.
```

Exemplo:

```c
```

---

# **Filas de Mensagens (Message Queues)**

```c
#include <sys/msg.h>
#include <sys/ipc.h> // Importar as flags IPC_CREAT, IPC_RMID,...
```

## Criar Filas de Mensagens

```c
int msgget(key_t key, int flags); // Cria uma fila de mensagens não persistente (durante a execução do programa) com chave única "key" e flags de criação "flags" (IPC_CREAT para criar a fila se não existir e as permissões em octal, por exemplo, 0777). Retorna um id usado para criar, remover e/ou aceder à fila de mensagens.
```

Exemplo:

```c
#include <stdio.h> // Importar stdio.h para usar printf e perror
#include <errno.h> // Importar errno.h para usar perror

int main() {
    int message_queue_id = msgget(2006, IPC_CREAT | 0777);
    if  (message_queue_id == -1) /*Verifica se existiu algum erro na criação da fila de mensagens*/ {
        perror("message queue creation error!");
        exit(1);
    }

    (...)
}
```

## Enviar Mensagens

```c
int msgsnd(int msqid, const void* message, size_t length, int flags); // Envia a mensagem "message" para a fila de mensagens com id "msqid". "length" é o tamanho da mensagem a enviar e "flags" são as flags de envio.
```

Exemplo:

```c
#include <string.h> // Include usado para a manipulação e cópia de strings

// Payload da mensagem
typedef struct {
    char message[1024];
} payload_t;

// Estrutura da mensagem (estrutura do pacote de envio, contém um valor de prioridade e o conteúdo)
typedef struct {
    long PRIORITY;
    payload_t PAYLOAD;
} message_t;

int main() {
    (...)

    pid_t presidente, vice_presidente, secretario, Carlos; // PIDs dos processos filhos

    // O processo presidente manda uma mensagem para a fila
    if ((presidente = fork()) == 0) {
        message_t message_from_president;
        payload_t payload_from_president = {"Manda uma nuke tática na lua."};

        message_from_president.PRIORITY = 4;
        message_from_president.PAYLOAD = payload_from_president;

        msgsnd(message_queue_id, &message_from_president, sizeof(message_from_president) - sizeof(long), 0); // Insere a mensagem na fila de mensagens associada ao id message_queue_id. my_message é do tipo message_t e 0 significa que não há flags de entrada de mensagem especial

        printf("Mensagem enviada com sucesso!");

        exit(0);
    }

    // O processo vice_presidente manda uma mensagem para a fila
    if ((vice_presidente = fork()) == 0) {
        message_t message_from_vice_president;
        payload_t payload_from_vice_president;

        strcpy(payload_from_vice_president.message, "Despede o presidente dos SMTUC.");
        message_from_vice_president.PRIORITY = 3;
        message_from_vice_president.PAYLOAD = payload_from_vice_president;

        msgsnd(message_queue_id, &message_from_vice_president, sizeof(message_from_vice_president) - sizeof(long), 0);

        printf("Mensagem enviada com sucesso!");

        exit(0);
    }

    // O processo secretario manda uma mensagem para a fila
    if ((secretario = fork()) == 0) {
        message_t message_from_secretario;
        payload_t payload_from_secretario = {"Já está feito ca**lho!"};

        message_from_secretario.PRIORITY = 2;
        message_from_secretario.PAYLOAD = payload_from_secretario;

        msgsnd(message_queue_id, &message_from_secretario, sizeof(message_from_secretario) - sizeof(long), 0);

        printf("Mensagem enviada com sucesso!");

        exit(0);
    }

    // O processo Carlos manda uma mensagem para a fila
    if ((Carlos = fork()) == 0) {
        message_t message_from_Carlos;
        payload_t payload_from_Carlos;

        strcpy(payload_from_Carlos.message, "Manda vir um frango.");
        message_from_Carlos.PRIORITY = 1;
        message_from_Carlos.PAYLOAD = payload_from_Carlos;

        msgsnd(message_queue_id, &message_from_Carlos, sizeof(message_from_Carlos) - sizeof(long), 0);

        printf("Mensagem enviada com sucesso!");

        exit(0);
    }

    (...)
}
```

## Ler Mensagens

```c
int msgrcv(int msqid, void* message, size_t length, long msgtype, int flags); // Lê a mensagem da fila de mensagens com id "msqid" e guarda-a na estrutura "message". "length" é o tamanho da mensagem a ler e "msgtype" é o tipo de mensagem (o long da estrutura de dados da mensagem, message_t) a ler.
```

Exemplo:

```c
#include <unistd.h> // Include para os sleeps

int main() {
    (...)

    message_t received_message; // Estrutura para guardar a mensagem recebida

    // Processo que lê todas as mensagens com prioridade igual a 4 (lê todas as mensagens vindas do presidente)
    if (fork() == 0) {
        msgrcv(message_queue_id, &received_message, sizeof(received_message) - sizeof(long), 4, 0) {
            perror("Error receiving message!");
            exit(1);
        }
        
        printf("MENSAGEM RECEBIDA DO PRESIDENTE, PRIORIDADE %ld : %s", received_message.PRIORITY, received_message.PAYLOAD.message);

        sleep(10);
        exit(0);
    }

    // Processo que lê todas as mensagens com prioridade igual ou inferior a 3 (lê todas as mensagens vindas do vice_presidente, secretario e Carlos)
    if (fork() == 0) {
        if (msgrcv(message_queue_id, &received_message, sizeof(received_message) - sizeof(long), -3, 0) == -1) {
            perror("Error receiving message!");
            exit(1);
        }

        if (received_message.PRIORITY == 3) {
            printf("MENSAGEM RECEBIDA DO VICE-PRESIDENTE, PRIORIDADE %ld : %s", received_message.PRIORITY, received_message.PAYLOAD.message);
        } else if (received_message.PRIORITY == 2) {
            printf("MENSAGEM RECEBIDA DO SECRETARIO, PRIORIDADE %ld : %s", received_message.PRIORITY, received_message.PAYLOAD.message);
        } else if (received_message.PRIORITY == 1) {
            printf("MENSAGEM RECEBIDA DO CARLOS, PRIORIDADE %ld : %s", received_message.PRIORITY, received_message.PAYLOAD.message);
        }

        sleep(10);
        exit(0);
    }

    // Processo que lê todas as mensagens
    if (fork() == 0) {
        if (msgrcv(message_queue_id, &received_message, sizeof(received_message) - sizeof(long), 0, 0) == -1) /* Lê todas as mensagens enviadas para a fila de mensagens */ {
            perror("Error receiving message!");
            exit(1);
        }

        printf("MENSAGEM RECEBIDA PRIORIDADE %ld : %s", received_message.PRIORITY, received_message.PAYLOAD.message);

        sleep(10);
        exit(0);
    }

    (...)
}
```

## Remover Filas de Mensagens

```c
int msgctl(int msqid, int cmd, struct msqid_ds* buff); // Remove a fila de mensagens com id "msqid". "cmd" é a operação a realizar (IPC_RMID para remover a fila) e "buff" é um ponteiro para uma estrutura de dados usada para obter ou definir informações sobre a fila de mensagens (normalmente não é utilizado então atribui-se NULL).
```

Exemplo:

```c
int main() {
    (...)

    msgctl(message_queue_id, IPC_RMID, NULL); // Apaga a fila de mensagens associada ao id message_queue_id.

    return 0;
}
```

---

# **Ficheiros Mapeados na Memória (Memory Mapped Files)**

```c
#include <sys/mman.h>
```

## Criar Memory Mapped Files

```c
void *mmap(void *addr, size_t length, int prot, int flags, int fd, off_t offset); // Cria um ficheiro mapeado na memória. "addr" é o endereço de inicio do mapeamento (NULL para escolher o sistema), "length" é o tamanho a ser mapeado, "prot
```

Exemplo:

```c
int main() {

}
```

## Remover Memory Mapped Files

```c
void *mmap(void *addr, size_t length, int prot, int flags, int fd, off_t offset); // Remove um ficheiro mapeado na memória. "addr" é o endereço de inicio do mapeamento (NULL para escolher o sistema), "length" é o tamanho a ser mapeado, "prot
```

Exemplo:

```c
```

# **TABELA DE RESUMO**

| **Tema** | **Funções Principais** | **Bibliotecas Necessárias** | **Descrição / Observações** |
|-----------|------------------------|------------------------------|------------------------------|
| **Sinais (Signals)** | `signal()`, `kill()`, `sigaction()`, `pause()`, `raise()` | `<signal.h>` | Comunicação entre processos. `kill(pid, sig)` envia sinal. `signal(SIGINT, handler)` define tratador. |
| **Bloqueio de Sinais** | `sigemptyset()`, `sigaddset()`, `pthread_sigmask()` | `<signal.h>` | Cria máscara de sinais para bloquear/permitir certos sinais. |
| **Threads** | `pthread_create()`, `pthread_join()`, `pthread_exit()` | `<pthread.h>` | Cria threads num processo. `pthread_create(&tid, NULL, func, arg)` → cria nova thread. |
| **Mutexes** | `pthread_mutex_init()`, `pthread_mutex_lock()`, `pthread_mutex_unlock()`, `pthread_mutex_destroy()` | `<pthread.h>` | Exclusão mútua: protege regiões críticas. |
| **Variáveis Condicionais** | `pthread_cond_init()`, `pthread_cond_wait()`, `pthread_cond_signal()`, `pthread_cond_broadcast()`, `pthread_cond_destroy()` | `<pthread.h>` | Sincronizam threads com base em condições; usadas com mutexes. |
| **Semáforos** | `sem_init()`, `sem_wait()`, `sem_post()`, `sem_destroy()` | `<semaphore.h>` | Controlam acesso a recursos partilhados. `sem_init(&sem, 0, N)` cria semáforo com valor inicial `N`. |
| **Memória Partilhada (SHM)** | `shmget()`, `shmat()`, `shmdt()`, `shmctl()` | `<sys/ipc.h>`, `<sys/shm.h>` | Partilha memória entre processos. `shmget(key, size, IPC_CREAT \| 0666)` cria/obtém segmento. |
| **Fork (Processos)** | `fork()`, `wait()`, `waitpid()` | `<unistd.h>`, `<sys/wait.h>` | Cria novo processo. Retorna 0 ao filho e PID ao pai. |
| **Funções Auxiliares** | `getpid()`, `getppid()`, `sleep()`, `perror()` | `<unistd.h>`, `<stdio.h>` | Identificação e temporização de processos. |
