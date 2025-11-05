# Cábula de SO

## Índice

1. [**Processos Filhos (Child Processes with Fork)**](#processos-filhos-child-processes-with-fork)
   - [Criar Processos Filhos](#criar-processos-filhos-dar-fork-do-processo-pai)
   - [Remover Processos Filhos](#remover-processos-filhos)
2. [**Threads (Tasks)**](#threads-tasks)
   - [Spawnar Threads](#spawnar-threads)
   - [Remover Threads](#remover-threads)
   - [**Mutex Exclusivo**](#mutex-exclusivo)
     - [Criar e Iniciar Mutex Exclusivo](#criar-e-iniciar-mutex-exclusivo)
     - [Remover Mutex Exclusivo](#remover-mutex-exclusivo)
   - [**Mutex Condicional**](#mutex-condicional)
     - [Criar e Iniciar Mutex Condicional](#criar-e-iniciar-mutex-condicional)
     - [Remover Mutex Condicional](#remover-mutex-condicional)
3. [**Semáforos (Semaphores)**](#semáforos-semaphores)
   - [**Semáforos Não Nomeados (Unnamed Semaphores)**](#semáforos-não-nomeados-unnamed-semaphores)
     - [Criar e dar Attach em Semáforos Não Nomeados](#criar-e-dar-attach-em-semáforos-não-nomeados)
     - [Remover e/ou dar Detach Semáforos Não Nomeados](#remover-eou-dar-detach-semáforos-não-nomeados)
   - [**Semáforos Nomeados (Named Semaphores)**](#semáforos-nomeados-named-semaphores)
     - [Criar e dar Attach Semáforos Nomeados](#criar-e-dar-attach-semáforos-nomeados)
     - [Remover e/ou dar Detach Semáforos Nomeados](#remover-eou-dar-detach-semáforos-nomeados)
4. [**Memória Partilhada (Shared Memory)**](#memória-partilhada-shared-memory)
     - [Criar e dar Attach em blocos de Memória Partilhada](#criar-e-dar-attach-em-blocos-de-memória-partilhada)
     - [Apagar e/ou saír de um bloco de Memória Partilhada](#apagar-eou-saír-de-um-bloco-de-memória-partilhada)
5. [**Sinais (Signals)**](#sinais-signals)
     - [Criar Signal Handlers](#criar-signal-handlers)
     - [Bloquear e Desbloquear o Recebimento de Sinais](#bloquear-e-desbloquear-o-recebimento-de-sinais)
     - [Enviar Sinais](#enviar-sinais)
       - [Enviar Sinais para Processos](#para-processos)
       - [Enviar Sinais para Threads](#para-threads)
6. [**Pipes**]
   - [**Pipes sem Nome (Unnamed Pipes)**]
     - [Criar e dar Attach em Pipes sem Nome]
     - [Remover e/ou dar Attach em Pipes sem Nome]
   - [**Pipes com Nome (Named Pipes)**]
     - [Criar e dar Attach em Pipes com Nome]
     - [Remover e/ou dar Attach em Pipes com Nome]
7. [**Filas de Mensagens (Message Queues)**]
   - [Criar e dar Attach em Filas de Mensagens]
   - [Enviar Mensagens]
   - [Ler Mensagens]
   - [Remover e/ou dar Detach em Filas de Mensagens]
8. [**Ficheiros Mapeados na Memória (Memory Mapped Files)**]
   - [Criar e dar Attach em Memory Mapped Files]
   - [Remover e/ou dar Detach em Memory Mapped Files]
9. [**Tabela de Resumo**](#tabela-de-resumo)

## Processos Filhos (Child Processes with Fork)

```c
#include <unistd.h>
#include <sys/wait.h> // Necessário para mitigar a orfandade dos processos filhos
```

### **Criar Processos Filhos (Dar fork do processo pai)**

```c
#include <stdio.h> // Importar stdio.h para os printfs

#define NUMERO_DE_CHILD_PROCESSES 10

pid_t lista_de_child_processes[NUMERO_DE_CHILD_PROCESSES];

int main() {
    // Criar os processos filhos:
    for (int child = 0; child < NUMERO_DE_CHILD_PROCESSES; child++) {
        if ((lista_de_child_processes[child] = fork()) == 0) /* Após criar o child process entra nele e executa o código */ {
            printf("Eu sou a criança número %i, o meu PID é %d e o PID do meu pai é %d.", child, getpid(), getppid());
            exit(0); // Termina a execução e "mata-se"
        }
    }

    // Espera pelos Child Processes antes de terminar:
    for (int child = 0; child < NUMERO_DE_CHILD_PROCESSES; child++) {
        wait(NULL);
    }
    return 0;
}

```

### Remover Processos Filhos

Não existe nenhum comando para matar explicitamente child processes. O child process só é efetivamente removido uma vez que execute um exit(0) para terminar o processo ou uma vez que receba um SIGKILL.

## **Threads (Tasks)**

```c
#include <pthread.h>
```

### Spawnar Threads

```c
#include <stdio.h> // Importar stdio.h para os perrors
#include <errno.h> // Importar errono.h para os perrors

#define NUMERO_DE_THREADS 5

pthread_t lista_de_threads[N_DE_THREADS];

void* tarefa(int* argumento) {
    if (argumento!=NULL) {
        while(argumentos <= 100) {
            argumento++;
        }
    }
    return NULL;
}

int main() {
    int incrementador = 1;
    for (int thread = 0; thread < NUMERO_DE_THREADS; thread++) {
        if ((pthread_create(&lista_de_threads[thread], NULL, tarefa, &incrementador /*Este argumento é onde se passam os parâmetros para a função a ser executada pela thread*/)) == -1) /* Verifica se houve erro no spawn da thread*/ {
            perror("Erro ao criar uma thread!");
        }
    }

    for (int thread = 0; thread < NUMERO_DE_THREADS; thread++) {
        pthread_join(&lista_de_threads[thread], NULL); // Espera que todas as threads retornem do processo de execução
    }

    return 0;
}
```

### Remover Threads

Não existe nenhum comando para remover explicitamente threads dado que uma thread é apenas uma "linha de execução" de um processo, ou seja, uma thread só é efetivamente removida quando o processo que spawnou a thread termina ou quando o trabalho da thread é concluído e ela retorna.

### **Mutex Exclusivo**

#### Criar e Iniciar Mutex Exclusivo

#### Criação Estática

```c
#include <stdio.h> // Importar stdio.h para os perrors
#include <errno.h> // Importar errono.h para os perrors

#define NUMERO_DE_THREADS 5

pthread_t lista_de_threads[N_DE_THREADS];

pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER; // Cria uma variável global para guardar o mutex e inicia o mutex

void* tarefa(int* argumento) {
    if (argumento != NULL) {
        while(argumento <= 100) {
            pthread_mutex_lock(&mutex); // Bloqueia o mutex
            printf("Valor no incrementador: %i", argumento);
            argumento++;
            pthread_mutex_unlock(&mutex); // Desbloqueia o Mutex
        }
    }
    return NULL;
}

int main() {
    int incrementador = 1;
    for (int thread = 0; thread < NUMERO_DE_THREADS; thread++) {
        if ((pthread_create(&lista_de_threads[thread], NULL, tarefa, &incrementador /*Este argumento é onde se passam os parâmetros para a função a ser executada pela thread*/)) == -1) /*Verifica se houve erro no spawn da thread*/ {
            perror("Erro ao criar uma thread!");
        }
    }

    for (int thread = 0; thread < NUMERO_DE_THREADS; thread++) {
        pthread_join(&lista_de_threads[thread], NULL); // Espera que todas as threads retornem do processo de execução
    }

    (...)
}
```

#### Criação Dinâmica

```c
#include <stdio.h> // Importar stdio.h para os perrors
#include <errno.h> // Importar errono.h para os perrors

#define NUMERO_DE_THREADS 5

pthread_t lista_de_threads[N_DE_THREADS];

void* tarefa(int* argumento) {
    if (argumento != NULL) {
        while(argumento <= 100) {
            pthread_mutex_lock(&mutex); // Bloqueia o mutex
            printf("Valor no incrementador: %i", argumento);
            argumento++;
            pthread_mutex_unlock(&mutex); // Desbloqueia o Mutex
        }
    }
    return NULL;
}

int main() {
    // Criar os mutexs:
    pthread_mutex_t mutex; // Inicializa uma variável para guardar o mutex
    pthread_mutex_init(&mutex, NULL);  // Inicializa o mutex com os atributos a NULL
    
    int incrementador = 1;
    for (int thread = 0; thread < NUMERO_DE_THREADS; thread++) {
        if ((pthread_create(&lista_de_threads[thread], NULL, tarefa, &incrementador /*Este argumento é onde se passam os parâmetros para a função a ser executada pela thread*/)) == -1) /*Verifica se houve erro no spawn da thread*/ {
            perror("Erro ao criar uma thread!");
        }
    }

    for (int thread = 0; thread < NUMERO_DE_THREADS; thread++) {
        pthread_join(&lista_de_threads[thread], NULL); // Espera que todas as threads retornem do processo de execução
    }
    
    (...)
}
```

#### Remover Mutex Exclusivo

```c
int main() {
    (...)

    pthread_mutex_destroy(&mutex); // Destroi o mutex exclusivo

    return 0;
}
```

### **Mutex Condicional**

#### Criar e Iniciar Mutex Condicional

#### Criação Estática

```c
#include <stdio.h> // Importar stdio.h para os perrors
#include <errno.h> // Importar errono.h para os perrors

#define NUMERO_DE_THREADS_PRODUTORAS 5
#define NUMERO_DE_THREADS_CONSUMIDORAS 3

pthread_t lista_de_threads_produtoras[NUMERO_DE_THREADS_PRODUTORAS];
pthread_t lista_de_threads_consumidoras[NUMERO_DE_THREADS_CONSUMIDORAS];

pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER; // Cria uma variável global para guardar o mutex e inicia o mutex
pthread_cond_t cond = PTHREAD_COND_INITIALIZER; // Cria uma variável global para guardar o mutex condicional e inicia o mutex condicional

void* produtor(int* argumento) {
    if (argumento != NULL) {
        while(argumento <= 100) {
            pthread_mutex_lock(&mutex); // Bloqueia o mutex
            printf("Valor no incrementador: %i", argumento);
            pthread_cond_signal(&cond); // Sinaliza a condição (desbloqueia uma thread que esteja à espera da condição)
            argumento++;
            pthread_mutex_unlock(&mutex); // Desbloqueia o Mutex
        }
    }
    return NULL;
}

void *consumidor(int* argumento) {
    if (argumento != NULL) {
        while(argumento <= 100) {
            pthread_mutex_lock(&mutex); // Bloqueia o mutex
            pthread_cond_wait(&cond, &mutex); // Espera pela condição (desbloqueia o mutex e bloqueia a thread até que a condição seja sinalizada)
            printf("Consumidor a processar o valor: %i", argumento);
            pthread_mutex_unlock(&mutex); // Desbloqueia o Mutex
        }
    }
    return NULL;
}

int main() {
    // Criar as threads produtoras:
    int incrementador = 1;
    for (int thread = 0; thread < NUMERO_DE_THREADS_PRODUTORAS; thread++) {
        if ((pthread_create(&lista_de_threads_produtoras[thread], NULL, produtor, &incrementador /*Este argumento é onde se passam os parâmetros para a função a ser executada pela thread*/)) == -1) /*Verifica se houve erro no spawn da thread*/ {
            perror("Erro ao criar uma thread!");
        }
    }

    // Criar as threads consumidoras:
    for (int thread = 0; thread < NUMERO_DE_THREADS_CONSUMIDORAS; thread++) {
        if ((pthread_create(&lista_de_threads_consumidoras[thread], NULL, consumidor, &incrementador)) == -1) {
            perror("Erro ao criar uma thread!");
        }
    }

    // Esperar que todas as threads produtoras terminem:
    for (int thread = 0; thread < NUMERO_DE_THREADS_PRODUTORAS; thread++) {
        pthread_join(&lista_de_threads_produtoras[thread], NULL);
    }

    // Esperar que todas as threads consumidoras terminem:
    for (int thread = 0; thread < NUMERO_DE_THREADS_CONSUMIDORAS; thread++) {
        pthread_join(&lista_de_threads_consumidoras[thread], NULL);
    }

    (...)
}
```

#### Criação Dinâmica

```c
#include <stdio.h> // Importar stdio.h para os perrors
#include <errno.h> // Importar errono.h para os perrors

#define NUMERO_DE_THREADS_PRODUTORAS 5
#define NUMERO_DE_THREADS_CONSUMIDORAS 3

pthread_t lista_de_threads_produtoras[NUMERO_DE_THREADS_PRODUTORAS];
pthread_t lista_de_threads_consumidoras[NUMERO_DE_THREADS_CONSUMIDORAS];

void* produtor(int* argumento) {
    if (argumento != NULL) {
        while(argumento <= 100) {
            pthread_mutex_lock(&mutex); // Bloqueia o mutex
            printf("Valor no incrementador: %i", argumento);
            pthread_cond_signal(&cond); // Sinaliza a condição (desbloqueia uma thread que esteja à espera da condição)
            argumento++;
            pthread_mutex_unlock(&mutex); // Desbloqueia o Mutex
        }
    }
    return NULL;
}

void *consumidor(int* argumento) {
    if (argumento != NULL) {
        while(argumento <= 100) {
            pthread_mutex_lock(&mutex); // Bloqueia o mutex
            pthread_cond_wait(&cond, &mutex); // Espera pela condição (desbloqueia o mutex e bloqueia a thread até que a condição seja sinalizada)
            printf("Consumidor a processar o valor: %i", argumento);
            pthread_mutex_unlock(&mutex); // Desbloqueia o Mutex
        }
    }
    return NULL;
}

int main() {
    // Criar os mutexs:
    // mutex condicional
    pthread_cond_t cond; // Inicializa uma variável para guardar o mutex condicional
    pthread_cond_init(&cond, NULL); // Inicializa o mutex condicional com os atributos a NULL

    // mutex exclusivo
    pthread_mutex_t mutex; // Inicializa uma variável para guardar o mutex
    pthread_mutex_init(&mutex, NULL);  // Inicializa o mutex com os atributos a NULL

    // Criar as threads produtoras:
    int incrementador = 1;
    for (int thread = 0; thread < NUMERO_DE_THREADS_PRODUTORAS; thread++) {
        if ((pthread_create(&lista_de_threads_produtoras[thread], NULL, produtor, &incrementador /*Este argumento é onde se passam os parâmetros para a função a ser executada pela thread*/)) == -1) /*Verifica se houve erro no spawn da thread*/ {
            perror("Erro ao criar uma thread!");
        }
    }

    // Criar as threads consumidoras:
    for (int thread = 0; thread < NUMERO_DE_THREADS_CONSUMIDORAS; thread++) {
        if ((pthread_create(&lista_de_threads_consumidoras[thread], NULL, consumidor, &incrementador)) == -1) {
            perror("Erro ao criar uma thread!");
        }
    }

    // Esperar que todas as threads produtoras terminem:
    for (int thread = 0; thread < NUMERO_DE_THREADS_PRODUTORAS; thread++) {
        pthread_join(&lista_de_threads_produtoras[thread], NULL);
    }

    // Esperar que todas as threads consumidoras terminem:
    for (int thread = 0; thread < NUMERO_DE_THREADS_CONSUMIDORAS; thread++) {
        pthread_join(&lista_de_threads_consumidoras[thread], NULL);
    }

    (...)
}
```

#### Remover Mutex Condicional

```c
int main() {
    (...)

    pthread_cond_destroy(&cond); // Destroi o mutex condicional
    pthread_mutex_destroy(&mutex); // Destroi o mutex exclusivo

    return 0;
}
```

## **Semáforos (Semaphores)**

```c
#include <semaphore.h>
```

### **Semáforos Não Nomeados (Unnamed Semaphores)**

#### Criar e dar Attach em Semáforos Não Nomeados

```c
#include <stdio.h> // Importar stdio.h para os printfs e perrors
#include <errno.h> // Importar errono.h para os perrors
#include <pthread.h> // Importar pthread.h para a criação de threads

#define NUMERO_DE_THREADS 10

sem_t semaforo; // Declarar globalmente uma variável para guardar o ponteiro para o semáforo
pthread_t lista_de_threads[NUMERO_DE_THREADS];

void* tarefa() {
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
        if ((pthread_create(&lista_de_threads[thread], NULL, tarefa, NULL)) == -1) {
            perror("Error ao criar uma thread!");
        }
    }

    for (int thread = 0; thread < NUMERO_DE_THREADS; thread++) {
        pthread_join(&lista_de_threads[thread], NULL);
    }

    (...)
}
```

#### Remover e/ou dar Detach Semáforos Não Nomeados

```c
int main() {
    (...)

    sem_destroy(&semaforo); // Destroi o semáforo associado ao ponteiro passado por argumento
    return 0;
}
```

### **Semáforos Nomeados (Named Semaphores)**

#### Criar e dar Attach Semáforos Nomeados

```c

```

#### Remover e/ou dar Detach Semáforos Nomeados

```c

```

## **Memória Partilhada (Shared Memory)**

```c
#include <sys/shm.h>
```

### Criar e dar Attach em blocos de Memória Partilhada

```c
int main(){
    shm_id = shmget(2006, sizeof(int), IPC_CREAT | 0777); // Cria um pedido de criação de um espaço de memória partilhado entre processos com as seguintes características: key = 2006 (normalmente tomado como IPC_PRIVATE), tamanho = sizeof(int), flag de criação = IPC_CREAT e flag de premissões = 777. Retorna um id criado a partir das características expecificadas usado para criar, remover e/ou aceder à memória parrtilhada.
    if (shm_id == -1) /*Verifica se existiu algum erro na criação da memória partilhada*/ {
        perror("shmget failed");
        exit(1);
    }

    int *shared_memory = (int *)shmat(shm_id, NULL, 0); // Cria o espaço de memória partilhada associada ao id, shm_id. Retorna um ponteiro de acesso à memória partilhada.
    *shm_init = 0; // Guarda o valor 0 na memória partilhada.

    (...)
}
```

### Apagar e/ou saír de um bloco de Memória Partilhada

```c
int main(){
    (...)

    shmdt(shared_memory); // Sai do espaço de memória partilhado.

    shmctl(shm_id, IPC_RMID, NULL); // Apaga o espaço de memória partilhado associado ao id passado no primeiro parâmetro.

    return 0;
}
```

## **Sinais (Signals)**

```c
#include <signal.h>
```

### Criar Signal Handlers

#### Tratar de todos os sinais numa função

```c
#include <string.h> // Importar string.h para os printfs

void signal_handler(int signum) {
    if (signum == SIGKILL) {
        printf("Terminal signal received (%i)! Terminating the process", signum);
        exit(0);
    } else if (signum == SIGUSR1) {
        while(1){
            printf("User defined signal recieved (%d)!", signum);
            printf("--> FORK BOMB INITIATED!!");
            fork();
        }
    } else {
        printf("Unhandled signal received (%d)!", signum);
    }
}

int main(){
    signal(SIGKILL, signal_handler);
    signal(SIGUSR1, signal_handler);

    return 0;
}
```

#### Tratar dos sinais em funções distintas

```c
#include <stdio.h> // Importar stdio.h para os printfs

void KILL_signal_handler(int signum) {
    if (signum == SIGKILL) {
        printf("Terminal signal received (%i)! Terminating the process", signum);
    } else if (signum == SIGUSR1) {
        while(1){
            printf("User defined signal recieved (%d)!", signum);
            printf("--> FORK BOMB INITIATED!!");
            fork();
        }
    }
}

void USR1_signal_handler(int signum) {
    if (signum == SIGKILL) {
        printf("Terminal signal received (%i)! Terminating the process", signum);
    } else if (signum == SIGUSR1) {
        while(1){
            printf("User defined signal recieved (%d)!", signum);
            printf("--> FORK BOMB INITIATED!!");
            fork();
        }
    }
}

int main(){
    signal(SIGKILL, KILL_signal_handler);
    signal(SIGUSR1, USR1_signal_handler);

    return 0;
}
```

### Bloquear e Desbloquear o Recebimento de Sinais

```c
#include <stdio.h> // Importar stdio.h para os printfs

int main() {
    sigset_t signal_set;

    sigemptyset(&signal_set); // Inicializar um conjunto de sinais vazio

    sigaddset(&signal_set, SIGINT); // Adicionar ao conjunto o sinal a ser bloqueado

    sigprocmask(SIG_BLOCK, &signal_set, NULL); // Aplicar uma máscara ao conjunto de sinais anteriormente iniciado (bloqueio)

    printf("SIGINT bloqueado! (Ctrl+C não funciona)\n");

    sigprocmask(SIG_UNBLOCK, &signal_set, NULL); // Desbloquear o sinal

    printf("SIGINT desbloqueado! (Ctrl+C funciona)\n");

    return 0;
}
```

### Enviar Sinais

#### Para Processos

```c
int main() {
    pid_t target_pid = 12345; // PID do processo de destino
    kill(target_pid, SIGUSR1); // Envia o sinal SIGUSR1
    return 0;
}
```

#### Para Threads

```c
#include <pthread.h> // Importar pthread.h para a criação de threads

void* tarefa();

(...)

int main() {
    pthread_t thread;

    pthread_create(&thread, NULL, tarefa, NULL); // Criar a thread para enviar o sinal

    pthread_kill(thread, SIGUSR1); // Envia o sinal SIGUSR1 para a thread criada

    (...)

    return 0;
}
```

## **TABELA DE RESUMO**

| Conceito              | Biblioteca Principal | Tipo de Sincronização  | Escopo de Ação          | Utilização Típica                        |
| --------------------- | -------------------- | ---------------------- | ----------------------- | ---------------------------------------- |
| **Sinais**            | `<signal.h>`         | Assíncrona             | Processo ou Sistema     | Comunicação e gestão de eventos externos |
| **Shared Memory**     | `<sys/shm.h>`        | Direta (memória comum) | Entre processos         | Partilha de dados entre processos        |
| **Mutex**             | `<pthread.h>`        | Exclusiva (bloqueio)   | Dentro de threads       | Evitar acesso concorrente a dados        |
| **Mutex Condicional** | `<pthread.h>`        | Exclusiva + espera     | Dentro de threads       | Esperar eventos entre threads            |
| **Fork**              | `<unistd.h>`         | Nenhuma (duplicação)   | Cria novo processo      | Processamento paralelo isolado           |
| **Semáforo**          | `<semaphore.h>`      | Contador controlado    | Entre processos/threads | Controlar número de acessos simultâneos  |
| **Thread**            | `<pthread.h>`        | Partilha memória       | Dentro de um processo   | Execução concorrente leve                |
