# Tarefas

Este módulo foca no coração do FreeRTOS: como as tarefas são estruturadas, priorizadas e gerenciadas pelo kernel para permitir o multitarefa em microcontroladores.

## Tarefas (Tasks/Threads)
No FreeRTOS, uma aplicação é organizada como um conjunto de **tarefas independentes**, cada uma executando em seu próprio contexto sem dependência direta de outras. 
*   **Implementação:** As tarefas são funções C que **nunca devem retornar**; por isso, são normalmente implementadas como um laço infinito (`for(;;)` ou `while(1)`).
*   **Assinatura:** Devem seguir o protótipo `void vATaskFunction( void * pvParameters )`, retornando `void` e aceitando um ponteiro para `void` como parâmetro.
*   **Estados:** Uma tarefa pode estar em um de quatro estados principais: **Execução (Running)**, **Pronta (Ready)**, **Bloqueada (Blocked)** ou **Suspensa (Suspended)**. Quando uma tarefa é criada, ela entra inicialmente no estado "Pronta".

## Escalonador (Scheduler) e Task Switching
O **Escalonador** é o componente do kernel responsável por decidir qual tarefa no estado "Pronta" deve entrar no estado de "Execução".
*   **Política de Escalonamento:** Por padrão, o FreeRTOS utiliza um algoritmo **preemptivo por prioridade com compartilhamento de tempo (time slicing)**. Isso significa que a tarefa de maior prioridade sempre terá preferência, e tarefas de mesma prioridade dividirão o tempo de CPU.
*   **Troca de Contexto (Task Switching):** Quando o escalonador decide trocar a tarefa ativa, ocorre o "chaveamento de contexto". O contexto da tarefa atual (registradores da CPU) é salvo em sua própria pilha, e o contexto da nova tarefa é restaurado a partir da pilha dela, permitindo que a execução continue exatamente de onde parou.

## 3. API de Task
As principais funções para gerenciar o ciclo de vida das tarefas incluem:
*   **`xTaskCreate()`**: Cria uma nova tarefa com alocação dinâmica de memória para o TCB (Task Control Block) e para a pilha.
*   **`xTaskCreateStatic()`**: Cria uma tarefa usando blocos de memória RAM pré-alocados pelo desenvolvedor, útil em sistemas que proíbem alocação dinâmica.
*   **`vTaskDelete()`**: Remove uma instância de tarefa do kernel e encerra sua execução.
*   **`vTaskDelay()`**: Coloca a tarefa no estado "Bloqueado" por um número específico de *ticks*, liberando o processador para outras tarefas.

## Prioridades de Tarefas
Cada tarefa recebe uma prioridade no momento de sua criação, que pode ser alterada dinamicamente.
*   **Escala:** As prioridades variam de **0 (mínima)** até **`configMAX_PRIORITIES - 1` (máxima)**.
*   **Regra de Ouro:** O escalonador sempre garante que a tarefa de maior prioridade capaz de rodar seja aquela que recebe o tempo de processamento. Se uma tarefa de maior prioridade se torna "Pronta", ela **preempta** imediatamente uma tarefa de menor prioridade que esteja rodando.

## Idle Task e Hooks
A **Tarefa Ociosa (Idle Task)** é criada automaticamente pelo kernel quando o escalonador é iniciado.
*   **Função:** Garante que sempre haja pelo menos uma tarefa pronta para executar. Ela roda na prioridade mais baixa (0).
*   **Responsabilidade:** É encarregada de liberar a memória de tarefas que foram deletadas.
*   **Idle Hook:** É uma função de *callback* opcional (`vApplicationIdleHook`) que o desenvolvedor pode definir para ser executada em cada iteração da Idle Task. É ideal para colocar o processador em modo de **baixo consumo** ou executar processamentos de fundo muito leves.

## Stack Size
Cada tarefa possui sua própria pilha, usada para variáveis locais e armazenamento de contexto.
*   **Definição:** Ao criar uma tarefa, o parâmetro `usStackDepth` define o tamanho da pilha.
*   **Unidade:** O valor é especificado em **palavras (words)**, não em bytes. Por exemplo, em um STM32 (arquitetura de 32 bits), uma pilha de 100 palavras ocupará 400 bytes.
*   **`configMINIMAL_STACK_SIZE`**: Define o tamanho mínimo recomendado para qualquer tarefa naquela arquitetura específica.
*   **Monitoramento:** A função `uxTaskGetStackHighWaterMark()` pode ser usada para verificar o quão perto a tarefa chegou de estourar sua pilha, retornando o espaço mínimo restante desde o início da tarefa.
