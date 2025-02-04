# Controle de LEDs com Raspberry Pi Pico W

### Autor: Jabson Gama Santana Júnior

## Descrição
Este projeto implementa o controle de LEDs utilizando um Raspberry Pi Pico W. Os LEDs são ativados por meio de um botão e desligados em sequência com um temporizador.

## Componentes Utilizados
- Raspberry Pi Pico W
- 3 LEDs (Azul, Vermelho e Verde)
- 3 Resistores de 330Ω
- 1 Botão

## Funcionamento
- Ao pressionar o botão, os três LEDs acendem simultaneamente.
- Após 3 segundos, o LED azul apaga.
- Depois de mais 3 segundos, o LED vermelho apaga.
- Após mais 3 segundos, o LED verde apaga e o sistema fica pronto para nova ativação.
- O botão tem um debounce de 50ms para evitar acionamentos indesejados.

## Código
```c
#include <stdio.h>
#include "pico/stdlib.h"
#include "hardware/gpio.h"
#include "hardware/timer.h"

// Definição dos pinos do LED RGB (BitDogLab)
#define LED_BLUE   11
#define LED_RED    12
#define LED_GREEN  13
#define BUTTON     5

// Tempo de atraso entre mudanças de estado (3s = 3000ms)
#define LED_DELAY_MS 3000
// Tempo de debounce do botão (50ms)
#define DEBOUNCE_TIME 50

// Variável de estado do sistema
volatile bool sequence_active = false;

// Callback para desligar 1 LED (mantendo 2 ligados)
int64_t step2_callback(alarm_id_t id, void *user_data) {
    gpio_put(LED_BLUE, 0);  // Apaga o LED azul
    add_alarm_in_ms(LED_DELAY_MS, step3_callback, NULL, false);
    return 0;
}

// Callback para desligar 2 LEDs (mantendo 1 ligado)
int64_t step3_callback(alarm_id_t id, void *user_data) {
    gpio_put(LED_RED, 0);  // Apaga o LED vermelho
    add_alarm_in_ms(LED_DELAY_MS, step4_callback, NULL, false);
    return 0;
}

// Callback para desligar o último LED
int64_t step4_callback(alarm_id_t id, void *user_data) {
    gpio_put(LED_GREEN, 0);  // Apaga o LED verde
    sequence_active = false;  // Libera o botão para nova ativação
    return 0;
}

// Callback chamado quando o botão é pressionado
void button_callback(uint gpio, uint32_t events) {
    if (!sequence_active) {  // Só inicia a sequência se não estiver ativa
        sleep_ms(DEBOUNCE_TIME);  // Debounce do botão
        if (gpio_get(BUTTON) == 0) {  // Confirma se ainda está pressionado
            sequence_active = true;

            // Liga todos os LEDs
            gpio_put(LED_BLUE, 1);
            gpio_put(LED_RED, 1);
            gpio_put(LED_GREEN, 1);

            // Inicia a sequência de desligamento com temporizador
            add_alarm_in_ms(LED_DELAY_MS, step2_callback, NULL, false);
        }
    }
}

int main() {
    // Configuração dos LEDs como saída
    gpio_init(LED_BLUE);
    gpio_set_dir(LED_BLUE, GPIO_OUT);
    gpio_init(LED_RED);
    gpio_set_dir(LED_RED, GPIO_OUT);
    gpio_init(LED_GREEN);
    gpio_set_dir(LED_GREEN, GPIO_OUT);

    // Configuração do botão como entrada com pull-up
    gpio_init(BUTTON);
    gpio_set_dir(BUTTON, GPIO_IN);
    gpio_pull_up(BUTTON);

    // Configuração da interrupção no botão (falling edge)
    gpio_set_irq_enabled_with_callback(BUTTON, GPIO_IRQ_EDGE_FALL, true, &button_callback);

    // Loop principal
    while (true) {
        sleep_ms(1000);  // Evita alto consumo da CPU
    }

    return 0;
}
```

## Como Usar
1. Conecte os LEDs, resistores e botão ao Raspberry Pi Pico W seguindo o esquema de ligação.
2. Compile e carregue o código na placa.
3. Pressione o botão para ativar a sequência de LEDs.
4. Observe os LEDs apagando em sequência após serem acionados.

