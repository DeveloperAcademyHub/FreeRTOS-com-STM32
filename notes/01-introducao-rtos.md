# Introdução aos Sistemas de Tempo Real

### O que é um Sistema Operacional e RTOS
Um **Sistema Operacional (SO)** atua como uma interface entre o hardware e o usuário, gerenciando recursos como processador, memória e periféricos. Em sistemas embarcados complexos, a abordagem tradicional de "superlaço" (foreground/background) torna-se difícil de manter, o que motiva o uso de um **RTOS (Real-Time Operating System)**. 

Um RTOS é projetado para sistemas onde o **tempo de resposta é fixo e determinístico**, priorizando o cumprimento de prazos em vez de apenas a velocidade de execução. Eles são classificados em:
*   **Hard Real-Time:** Onde a falha em cumprir um prazo resulta no colapso total do sistema, como em um airbag.
*   **Soft Real-Time:** Onde atrasos causam degradação de desempenho, mas não falha total, como em um teclado que parece "lento".

### O que é o FreeRTOS
O **FreeRTOS** é um kernel de tempo real líder de mercado, desenvolvido originalmente por Richard Barry por volta de 2003 e hoje sob gestão da AWS. Ele é distribuído sob a licença MIT e é especialmente adequado para microcontroladores e pequenos microprocessadores devido ao seu **tamanho reduzido** (tipicamente entre 4K e 9K bytes). O FreeRTOS fornece as funcionalidades fundamentais de escalonamento, comunicação entre tarefas e primitivas de sincronização.

### Tarefas (Tasks/Threads)
No FreeRTOS, as aplicações são organizadas como uma coleção de **tarefas (tasks)** independentes. Cada tarefa é um programa simples que executa como se tivesse o processador exclusivamente para si, possuindo sua própria **pilha (stack)** para armazenar variáveis locais e contexto. 

As tarefas podem estar em diferentes estados:
*   **Execução (Running):** Quando a tarefa está utilizando o processador.
*   **Pronta (Ready):** Pronta para executar, aguardando o escalonador.
*   **Bloqueada (Blocked):** Aguardando um evento externo ou temporal.
*   **Suspensa (Suspended) ou Inativa**.

### Escalonador (Scheduler) e Task Switching
O **Escalonador (Scheduler)** é a parte do kernel responsável por decidir qual tarefa no estado "Pronta" deve entrar no estado de "Execução", baseando-se na **prioridade** atribuída pelo desenvolvedor. 

O processo de alternância entre tarefas é chamado de **Troca de Contexto (Context Switching)**. Quando ocorre:
1.  O contexto da tarefa atual (registradores da CPU) é salvo em sua pilha.
2.  O ponteiro de pilha do processador é alterado para a pilha da nova tarefa.
3.  O contexto da nova tarefa é restaurado para os registradores, incluindo o contador de programa (PC), permitindo que a execução continue exatamente de onde parou.

O FreeRTOS suporta escalonamento **preemptivo**, onde uma tarefa de maior prioridade pode interromper imediatamente uma de menor prioridade, e escalonamento **cooperativo**, onde as tarefas devem ceder voluntariamente o processador.