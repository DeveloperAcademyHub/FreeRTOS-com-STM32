# Documentação e Padronização do FreeRTOS

O FreeRTOS oferece um conjunto robusto de materiais para auxiliar o desenvolvedor, divididos em diferentes formatos:

*   **Site Oficial ([FreeRTOS.org](https://www.freertos.org/)):** É a fonte primária de informações, contendo guias de início rápido, tutoriais detalhados e a documentação da API do kernel.
*   **Repositórios no GitHub (https://github.com/FreeRTOS):** O código-fonte, demos e até a documentação em markdown estão disponíveis para consulta e contribuição.


## Convenções do FreeRTOS

Para garantir que o código seja altamente portável entre diversos compiladores e arquiteturas (como o ARM Cortex-M do STM32), o FreeRTOS segue convenções rígidas de nomenclatura:

### Variáveis
Os nomes das variáveis possuem prefixos que indicam seu tipo:
*   **`c`**: `char`.
*   **`s`**: `int16_t` (short).
*   **`l`**: `int32_t` (long).
*   **`x`**: `BaseType_t` e outros tipos não padrão (estruturas, handles de tarefas ou filas).
*   **`u`**: Indica que o tipo é `unsigned` (ex: `uc` para `unsigned char`).
*   **`p`**: Indica um ponteiro (ex: `pc` para ponteiro de `char`).

### Funções
As funções são prefixadas com o seu tipo de retorno e o arquivo onde foram definidas:
*   **`vTaskPrioritySet()`**: Retorna `void` (`v`) e está definida em `tasks.c`.
*   **`xQueueReceive()`**: Retorna `BaseType_t` (`x`) e está definida em `queue.c`.
*   **`pvTimerGetTimerID()`**: Retorna um ponteiro para `void` (`pv`) e está definida em `timers.c`.
*   **`prv`**: Prefixo para funções de escopo privado (locais ao arquivo).

### Macros
A maioria das macros é escrita em letras maiúsculas e possui um prefixo em minúsculas indicando onde foi definida:
*   **`port`**: Definida em `portable.h` (ex: `portMAX_DELAY`).
*   **`task`**: Definida em `task.h` (ex: `taskENTER_CRITICAL()`).
*   **`pd`**: Definições comuns em `projdefs.h` (ex: `pdTRUE`, `pdPASS`).
*   **`config`**: Definida em `FreeRTOSConfig.h` (ex: `configUSE_PREEMPTION`).

## Composição do FreeRTOS

O FreeRTOS não é apenas um kernel, mas uma coleção de bibliotecas C modulares.

### Arquivos Principais (Core)
O kernel essencial reside em três arquivos principais localizados no diretório `FreeRTOS/Source`:
1.  **`tasks.c`**: Gerenciamento de tarefas e escalonamento.
2.  **`list.c`**: Implementação das listas encadeadas usadas internamente pelo kernel.
3.  **`queue.c`**: Gerenciamento de filas e semáforos.

### Camada Portátil (Portable Layer)
Contém o código específico para cada arquitetura e compilador. Para o STM32, os arquivos ficam em `FreeRTOS/Source/portable/[compiler]/[architecture]`. É aqui que o FreeRTOS lida com as particularidades do hardware.

### Gerenciamento de Memória (Heap)
O FreeRTOS oferece cinco esquemas de alocação de memória (`heap_1.c` a `heap_5.c`), permitindo desde alocações puramente estáticas até gerenciamentos que evitam fragmentação de memória.

### Bibliotecas Adicionais
Além do kernel, o ecossistema inclui bibliotecas para:
*   **Conectividade:** Pilha TCP/IP, MQTT, HTTP.
*   **Segurança:** Implementações de TLS e PKCS #11.
*   **Funcionalidades de Kernel:** Timers de software, buffers de mensagem e grupos de eventos.

### Configuração
O arquivo **`FreeRTOSConfig.h`** é vital. Ele é específico para cada aplicação e permite "ligar" ou "desligar" funcionalidades do kernel para economizar recursos de memória no microcontrolador.

