# Guia de Instalação: FreeRTOS no STM32

## Método 1: Modo Fácil (via STM32CubeIDE com CMSIS-RTOS)

Este método utiliza o ecossistema da STMicroelectronics para configurar o kernel automaticamente através de uma camada de abstração. No STM32Cube firmware, o FreeRTOS é utilizado por meio da camada de encapsulamento genérica **CMSIS-OS** fornecida pela Arm®.

<p align="center">
  <img src="../docs/imgs/cmsis.png" alt="Task States" width="250">
</p>

### Passo-a-passo:
1.  **Habilitação no Device Configuration Tool (.ioc):** No STM32CubeMX, abra o arquivo de configuração de hardware e procure pela seção de Middleware. Selecione **FREERTOS** e escolha a versão da API (ex: CMSIS_V2).

<p align="center">
  <img src="../docs/imgs/config_freertos_cubeide.png" alt="Conf" width="600">
</p>

2.  **Configuração Crítica do Timebase:** 
    *   Acesse **SYS Mode Configuration**.
    *   **Importante:** Altere o **Timebase Source** para um Timer diferente do **SysTick** (como o TIM1). O SysTick deve ser de uso exclusivo do FreeRTOS para o escalonamento.

  <p align="center">
  <img src="../docs/imgs/config_timebase_soucer.png" alt="Conf" width="400">
  </p>

3.  **Configurações Avançadas:** Nas "Advanced settings", certifique-se de que a opção **USE_NEWLIB_REENTRANT** esteja habilitada para garantir a segurança em bibliotecas padrão C.

  <p align="center">
  <img src="../docs/imgs/config_use_newlib_reentrant.png" alt="Conf" width="400">
  </p>
---

## Método 2: Modo Manual ("Hard Mode")

Este método é ideal para quem deseja controle total sobre a versão do kernel e a estrutura de diretórios, importando o código diretamente do repositório oficial.

### 1. Preparação do Projeto e Hardware
*   Crie um novo projeto no **STM32CubeIDE**.
*   Assim como no modo fácil, vá em **SYS Mode Configuration** e altere o **Timebase Source** para um timer que não seja o SysTick.

### 2. Download do FreeRTOS
*   Acesse o [GitHub oficial do FreeRTOS](https://github.com/FreeRTOS/FreeRTOS/releases/latest) e baixe ou clone o kernel.
*   Copie a pasta do kernel para dentro do diretório do seu projeto STM32.
*   **Dica de limpeza:** Para evitar erros de compilação, delete ou configure o compilador para ignorar as pastas `cmake_example` e `coverity` dentro do diretório `/examples`.

### 3. Ajuste do diretório `/portable`
O diretório `/portable` contém arquivos específicos para diversos compiladores e arquiteturas. Para o STM32 no CubeIDE, faça o seguinte:
*   **Mantenha apenas o compilador GCC**, removendo todos os outros (IAR, Keil, etc.).
*   **Arquitetura:** Como o STM32 (ex: NUCLEO-L476RG) utiliza ARM Cortex-M4 com FPU, mantenha apenas a pasta **ARM_CM4F** dentro de `/portable/GCC`.
*   **Gerenciamento de Memória:** Dentro de `/portable/MemMang`, mantenha apenas o arquivo **heap_4.c**, que é o gerenciador recomendado para a maioria das aplicações.

### 4. Configuração de Caminhos (Paths) no IDE
Para que o compilador encontre o FreeRTOS, acesse as **Properties** do projeto:
*   **Source Location:** Vá em `C/C++ General > Path and Symbols` e adicione o diretório onde o FreeRTOS foi baixado.

<p align="center">
  <img src="../docs/imgs/add_path_source.png" alt="Conf" width="600">
</p>

*   **Includes:** Na mesma tela, aba "Includes", adicione os caminhos para:
    *   `/include`
    *   `/portable/GCC/ARM_CM4F` (ou a pasta da sua arquitetura específica).

<p align="center">
  <img src="../docs/imgs/add_path_includes.png" alt="Conf" width="600">
</p>

### 5. Configuração do `FreeRTOSConfig.h`
Este arquivo é o cérebro da configuração do kernel.
*   Copie o arquivo template localizado em `/examples/template_configuration` para a pasta `/Core/Inc` do seu projeto.
*   **Ajustes necessários no arquivo:**
    *   Importe o header do seu microcontrolador (ex: `stm32l476xx.h`) para que o FreeRTOS tenha acesso à variável `SystemCoreClock`.
    *   Defina `configPRIO_BITS` buscando o valor em `__NVIC_PRIO_BITS` para configurar corretamente os níveis de prioridade de interrupção (no STM32L4, geralmente são 4 bits/16 níveis).

### 6. Mapeamento de Interrupções
O FreeRTOS precisa assumir o controle de três interrupções fundamentais do processador. No arquivo `stm32l4xx_it.c`, localize e chame os handlers do FreeRTOS dentro das funções de interrupção padrão:

*   **SVC_Handler:** Chame `vPortSVCHandler()`.
*   **PendSV_Handler:** Chame `xPortPendSVHandler()`.
*   **SysTick_Handler:** Chame `xPortSysTickHandler()`.

Esses protótipos de funções de exceção estão localizados no arquivo `port.c` dentro da pasta da sua arquitetura em `/portable/GCC/`.