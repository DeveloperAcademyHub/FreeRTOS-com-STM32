# Semáforos e Mutex

Este módulo aborda os mecanismos de sincronização e exclusão mútua do FreeRTOS, essenciais para coordenar o acesso a recursos e a execução de tarefas.

## Introdução, Características e Tipos

Os **semáforos** são objetos utilizados para sincronização de atividades e gerenciamento de recursos em um RTOS. No FreeRTOS, eles são implementados como uma camada sobre a funcionalidade de filas.

### Tipos de Semáforos
*   **Semáforos Binários:** Possuem apenas dois valores (0 ou 1). São a melhor escolha para implementar **sincronização**, seja entre duas tarefas ou entre uma tarefa e uma interrupção. Conceitualmente, funcionam como uma fila de tamanho 1 que está sempre vazia ou cheia.
*   **Semáforos de Contagem (Counting):** Podem assumir uma faixa de valores definida na criação. São tipicamente usados para:
    1.  **Contagem de eventos:** O contador é incrementado quando um evento ocorre e decrementado quando é processado.
    2.  **Gerenciamento de recursos:** O valor indica a quantidade de recursos disponíveis.
*   **Mutexes (Mutual Exclusion):** É um tipo especial de semáforo binário otimizado para **exclusão mútua**. Ele funciona como um "token" que uma tarefa deve possuir para acessar um recurso compartilhado.
*   **Mutexes Recursivos:** Permitem que a tarefa que detém o mutex o obtenha múltiplas vezes sem causar um *deadlock*. Para liberar o mutex, a tarefa deve devolvê-lo o mesmo número de vezes que o obteve.

### Características e Diferenças
A principal diferença entre um mutex e um semáforo binário é o mecanismo de **herança de prioridade**. Se uma tarefa de alta prioridade tenta obter um mutex detido por uma tarefa de menor prioridade, o RTOS aumenta temporariamente a prioridade da tarefa que detém o mutex para evitar a inversão de prioridade ilimitada. Semáforos binários não possuem esse mecanismo.

## API de Semaphores

As funções para manipulação de semáforos binários e de contagem incluem:

*   **Criação:**
    *   **`xSemaphoreCreateBinary()`**: Cria um semáforo binário e o inicia no estado "vazio" (necessita ser dado antes de poder ser tomado).
    *   **`xSemaphoreCreateCounting()`**: Cria um semáforo de contagem especificando o valor máximo e o valor inicial.
*   **Uso Geral:**
    *   **`xSemaphoreTake()`**: Obtém (consome) o semáforo. Permite especificar um **tempo de bloqueio** caso o semáforo não esteja disponível.
    *   **`xSemaphoreGive()`**: Libera (devolve) o semáforo.
*   **Uso em Interrupções:**
    *   **`xSemaphoreGiveFromISR()`**: Versão para liberar o semáforo dentro de uma ISR, utilizando o parâmetro `pxHigherPriorityTaskWoken` para solicitar troca de contexto se necessário.
*   **Utilidade:**
    *   **`uxSemaphoreGetCount()`**: Retorna o valor atual do contador do semáforo.
    *   **`vSemaphoreDelete()`**: Deleta o semáforo e libera a memória associada.

* **Nota**: Para habilitar o uso dos semáforos contadores, em `FreeRTOSConfig.h` a macro `configUSE_COUNTING_SEMAPHORES` deve está definida como `1`.

## API de Mutexes

Os mutexes utilizam funções específicas para garantir a herança de prioridade e a recursividade:

*   **Criação:**
    *   **`xSemaphoreCreateMutex()`**: Cria um mutex padrão.
    *   **`xSemaphoreCreateRecursiveMutex()`**: Cria um mutex que pode ser tomado recursivamente pela mesma tarefa.
*   **Uso Padrão:** Mutexes comuns utilizam as mesmas funções `xSemaphoreTake()` e `xSemaphoreGive()` dos semáforos.
*   **Uso Recursivo:**
    *   **`xSemaphoreTakeRecursive()`**: Obtém o mutex recursivo. Deve ser usado exclusivamente com mutexes criados para este fim.
    *   **`xSemaphoreGiveRecursive()`**: Libera o mutex recursivo.
*   **Monitoramento:**
    *   **`xSemaphoreGetMutexHolder()`**: Retorna o *handle* da tarefa que detém o mutex no momento. Requer a definição de `INCLUDE_xSemaphoreGetMutexHolder` como 1 no `FreeRTOSConfig.h`.

**Dica de Segurança:** Mutexes **não devem ser usados** dentro de rotinas de serviço de interrupção (ISRs), pois eles envolvem mecanismos de herança de prioridade que só fazem sentido entre tarefas. Além disso, um mutex obtido para exclusão mútua **deve sempre ser devolvido** pela tarefa, caso contrário, o recurso ficará bloqueado para sempre.

## Exercícios do modulo:
| Aula | Exercícios |
|:--:|:--|
| 46 | [`s07_l1_g474re`](/projects/s07_l1_g474re/) |
| 49 | [`s07_l2_g474re`](/projects/s07_l2_g474re/) |

## Referencias
- [Semaphore and Mutexes](https://www.freertos.org/Documentation/02-Kernel/04-API-references/10-Semaphore-and-Mutexes/00-Semaphores)