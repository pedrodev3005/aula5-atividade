# Aula 5- Atividade

## 📝 **Relatório – Atividade 5: Monitor de Exclusão com Priorização Condicional**

### 🎯 **Objetivo da Atividade**

A atividade teve como objetivo implementar um sistema multitarefa usando o FreeRTOS, com foco na contenção de recursos compartilhados (UART e LED) utilizando mutex. O principal aprendizado foi observar o comportamento de tarefas com diferentes perfis de uso do mutex e visualizar o impacto da retenção prolongada de recursos por meio de reações em tempo de execução.

---

### ⚙️ **Descrição da Implementação**

Foram criadas três tarefas com comportamentos distintos:

- **Tarefa 1:** Acessa o recurso de forma normal, realiza sua operação e libera o mutex imediatamente.
- **Tarefa 2:** Ao adquirir o mutex, executa três ciclos consecutivos com delays, mantendo o recurso por aproximadamente 3 segundos.
- **Tarefa 3:** Tenta acessar o recurso com timeout de 2 segundos. Caso não consiga, emite uma mensagem de alerta via UART e acende um LED como sinal de contenção.

Um único **mutex global (`MeuMutexHandle`)** foi utilizado para proteger os dois recursos compartilhados: **a UART e o LED**, conforme exigido.

Além disso, foi usada a função `osThreadGetId()` para registrar em tempo de execução qual tarefa está com a posse do mutex (`donoMutex`), permitindo rastreabilidade da propriedade do recurso.

---

### 🧪 **Comportamento Observado**

Durante os testes, o seguinte comportamento foi visualizado:

- A **Tarefa 2**, ao segurar o mutex por três ciclos com `osDelay(1000)`, impediu que outras tarefas acessassem os recursos por cerca de 3 segundos.
- A **Tarefa 3**, ao tentar adquirir o mutex, foi bloqueada por mais de 2 segundos e corretamente emitiu o alerta:
    
    ```
    Tarefa 3: Esperando Mutex por tempo excessivo
    ```
    
- A UART exibiu claramente as mensagens das tarefas e o **LED único** (LD3) foi controlado por todas as tarefas, com destaque para o uso de LED fixo na Tarefa 3 ao sinalizar contenção.

---

### 📌 **Critérios Atendidos**

| Critério | Status |
| --- | --- |
| Uso correto do `osMutexAcquire` e `osMutexRelease` | ✅ |
| Identificação da tarefa dona do mutex com `osThreadGetId()` | ✅ |
| Detecção de espera prolongada e alerta na Tarefa 3 | ✅ |
| Mensagens exibidas corretamente via UART | ✅ |
| LED único controlado corretamente pelas três tarefas | ✅ |

---

### 🎓 **Aprendizado**

Com esta atividade, foi possível:

- Entender o funcionamento prático do mecanismo de mutex em FreeRTOS.
- Visualizar o efeito de contenção prolongada de recursos, que compromete o desempenho do sistema.
- Implementar uma forma simples de monitoramento e reação a esse tipo de contenção (via alerta UART e LED).
- Praticar a coordenação segura de múltiplas tarefas sobre recursos únicos, um conceito essencial em sistemas embarcados com RTOS.

---

# Código das Tarefas:

```c
void Tarefa1Fun(void *argument)
{
  /* USER CODE BEGIN 5 */
  /* Infinite loop */
  for(;;)
  {

	if (osMutexAcquire(MeuMutexHandle, osWaitForever) == osOK) {
		donoMutex = osThreadGetId();

		char* msg = "Tarefa 1: Usando Recurso  \r\n";
		HAL_UART_Transmit(&huart3, (uint8_t*)msg, strlen(msg), HAL_MAX_DELAY);

		HAL_GPIO_TogglePin(GPIOB, LD2_Pin);
		osDelay(500);
		osMutexRelease(MeuMutexHandle);
	}
    osDelay(1000);
  }
  /* USER CODE END 5 */
}

/* USER CODE BEGIN Header_Tarefa2Fun */
/**
* @brief Function implementing the Tarefa2 thread.
* @param argument: Not used
* @retval None
*/
/* USER CODE END Header_Tarefa2Fun */
void Tarefa2Fun(void *argument)
{
  /* USER CODE BEGIN Tarefa2Fun */
  /* Infinite loop */
	uint8_t ciclos = 0;
  for(;;)
  {
	if (osMutexAcquire(MeuMutexHandle, osWaitForever) == osOK) {
		donoMutex = osThreadGetId();
		while(ciclos < 3){

			char msg[50];
			sprintf(msg, "Tarefa 2: Ciclo %d usando recurso\r\n", ciclos + 1);
			HAL_UART_Transmit(&huart3, (uint8_t*)msg, strlen(msg), HAL_MAX_DELAY);

			HAL_GPIO_TogglePin(GPIOB, LD2_Pin);
			osDelay(1000);
			ciclos++;
		}
		ciclos = 0;
		osMutexRelease(MeuMutexHandle);
	}
    osDelay(1500);
  }
  /* USER CODE END Tarefa2Fun */
}

/* USER CODE BEGIN Header_Tarefa3Fun */
/**
* @brief Function implementing the Tarefa3 thread.
* @param argument: Not used
* @retval None
*/
/* USER CODE END Header_Tarefa3Fun */
void Tarefa3Fun(void *argument)
{
  /* USER CODE BEGIN Tarefa3Fun */
  /* Infinite loop */
  for(;;)
  {
      if (osMutexAcquire(MeuMutexHandle, 2000) == osOK) {
		donoMutex = osThreadGetId();
		//printf("Tarefa 3: Conseguiu o recurso\n");
		char* msg = "Tarefa 3: Conseguiu o recurso \r\n";
		HAL_UART_Transmit(&huart3, (uint8_t*)msg, strlen(msg), HAL_MAX_DELAY);

		HAL_GPIO_TogglePin(GPIOB, LD2_Pin);
		osDelay(500);
		osMutexRelease(MeuMutexHandle);
	} else {
		//printf("Tarefa 3: Esperando Mutex por tempo excessivo.\n");

		char* msg = "Tarefa 3: Esperando Mutex por tempo excessivo \r\n";
		HAL_UART_Transmit(&huart3, (uint8_t*)msg, strlen(msg), HAL_MAX_DELAY);

		HAL_GPIO_WritePin(GPIOB, LD2_Pin, GPIO_PIN_SET);
		osDelay(500);
		HAL_GPIO_WritePin(GPIOB, LD2_Pin, GPIO_PIN_RESET);
	}
	osDelay(2000);
  }
  /* USER CODE END Tarefa3Fun */
} 
```
