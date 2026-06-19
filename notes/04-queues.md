## Queues

As filas são objetos fundamentais no FreeRTOS, utilizadas para transferir dados de forma segura entre diferentes partes do sistema.

## Introcução e Caracteristicas

*   **Comunicação Intertarefas:** Filas são o método primário para enviar mensagens entre tarefas e entre interrupções e tarefas.
*   **Armazenamento de Dados:** Uma fila pode conter um número **finito** (*length*) e fixo de itens de dados de tamanho determinado no momento da criação.
*   **Método de Manipulação FIFO:** Por padrão, operam como buffers **First In First Out** (primeiro a entrar, primeiro a sair), onde os dados são gravados no final (*tail*) e lidos do início (*head*).

    <div align="center">
    <video controls src="../docs/imgs/queue_animation.mp4" title="Title"></video>
    </div>

*   **Cópia vs. Referência:** O FreeRTOS utiliza o método de **fila por cópia**. Isso significa que o dado é copiado byte a byte para dentro da fila, permitindo que a tarefa enviadora reutilize imediatamente a variável original.
*   **Bloqueio de Tarefas:** As tarefas podem especificar um "tempo de bloqueio" ao tentar ler de uma fila vazia ou escrever em uma fila cheia. Durante esse tempo, a tarefa permanece no estado **Blocked**, não consumindo tempo de processamento.

## API de Queues

As principais funções para manipular filas incluem:

*   **Criação:**
    *   **`xQueueCreate()`**: Cria a fila dinamicamente alocando RAM do heap.
    *   **`xQueueCreateStatic()`**: Cria a fila usando memória RAM pré-alocada pelo desenvolvedor.
*   **Envio de Dados:**
    *   **`xQueueSendToBack()`** (ou `xQueueSend()`): Envia dados para o final da fila.
    *   **`xQueueSendToFront()`**: Envia dados para o início da fila.
    *   **`xQueueOverwrite()`**: Sobrescreve o dado em filas de tamanho 1 (mailboxes), mesmo que já estejam cheias.
*   **Recebimento de Dados:**
    *   **`xQueueReceive()`**: Lê e remove um item da fila.
    *   **`xQueuePeek()`**: Lê um item da fila sem removê-lo; o item permanece disponível para a próxima leitura.
*   **Utilitarios e Status:**
    *   **`uxQueueMessagesWaiting()`**: Retorna o número de itens atualmente na fila.
    *   **`uxQueueSpacesAvailable()`**: Informa quantos espaços livres restam na fila.
    *   **`xQueueReset()`**: Restaura a fila ao seu estado vazio original.
    *   **`vQueueDelete()`**: Deleta a fila e libera os recursos.

## Conjuntos de Filas (Queue Sets)

Os **Queue Sets** permitem que uma tarefa bloqueie simultaneamente em múltiplas filas ou semáforos.

*   **Uso:** É útil quando uma tarefa deve processar eventos de diferentes fontes que não podem ser unificadas em uma única fila.
*   **Configuração:** Deve-se definir `configUSE_QUEUE_SETS` como 1 no `FreeRTOSConfig.h`.
*   **Fluxo de Operação:**
    1.  Cria-se o conjunto com **`xQueueCreateSet()`**.
    2.  Adiciona-se as filas ou semáforos (que devem estar vazios) via **`xQueueAddToSet()`**.
    3.  A tarefa chama **`xQueueSelectFromSet()`**, que retorna o *handle* da fila ou semáforo que contém dados prontos para leitura.

## ISRs e Funções `_FromISR()`

Muitas funções da API do FreeRTOS realizam ações não permitidas dentro de uma **Rotina de Serviço de Interrupção (ISR)**, como colocar uma tarefa no estado Bloqueado. Por isso, o FreeRTOS fornece versões específicas para uso em interrupções, identificadas pelo sufixo **`FromISR`**.

*   **Segurança em Interrupções:** Nunca chame uma função que não termine em `FromISR` de dentro de uma ISR.
*   **`pxHigherPriorityTaskWoken`:** Quase todas as funções `FromISR` possuem este parâmetro ponteiro. Se a chamada da função fizer com que uma tarefa de maior prioridade saia do estado Bloqueado, o kernel definirá esse valor como `pdTRUE`.
*   **Troca de Contexto Determinística:** Se o valor retornado for `pdTRUE`, o desenvolvedor deve solicitar uma troca de contexto antes de sair da ISR usando a macro **`portYIELD_FROM_ISR()`** (ou `portEND_SWITCHING_ISR()`). Isso garante que a interrupção retorne diretamente para a tarefa de maior prioridade agora pronta.
*   **Exemplos Comuns:** `xQueueSendToBackFromISR()`, `xQueueReceiveFromISR()` e `xQueueIsQueueEmptyFromISR()`.

## Exercícios do modulo:
| Aula | Exercícios |
|:--:|:--|
| 36 | [`s06_l1_g474re`](/projects/s06_l1_g474re/) |
| 37 | [`s06_l2_g474re`](/projects/s06_l2_g474re/) |
| 38 | [`s06_l3_g474re`](/projects/s06_l3_g474re/) e [`s06_l4_g474re`](/projects/s06_l4_g474re/)  |

## Referencias
- [Queue Management](https://freertos.org/Documentation/02-Kernel/04-API-references/06-Queues/00-QueueManagement)
- [Queue Set](https://freertos.org/Documentation/02-Kernel/04-API-references/07-Queue-sets/00-RTOS-queue-sets)