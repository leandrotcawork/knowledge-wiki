domain: embedded
confidence: low
sources: 4
last_updated: 2026-04-13

# LVGL (Light and Versatile Graphics Library)

O **LVGL** é uma biblioteca gráfica de código aberto (MIT) escrita em C, projetada para criar interfaces de usuário (GUIs) em sistemas embarcados com recursos limitados. Ele é agnóstico em relação a hardware e sistema operacional, exigindo apenas um buffer de memória e funções básicas de cópia de pixels para funcionar.

Com a evolução para a versão 9 (e posteriores), o LVGL consolidou-se como o padrão da indústria para displays em microcontroladores (MCUs) e microprocessadores (MPUs), suportando desde telas monocromáticas até displays de alta resolução com aceleração por GPU (DMA2D, PXP, VG-Lite) e renderização 3D via OpenGL.

---

## Arquitetura e Internals

O LVGL opera com uma arquitetura orientada a objetos implementada em C. A biblioteca não possui dependências externas e gerencia seu próprio estado, árvore de widgets, sistema de estilos e loop de eventos.

### O Modelo de Execução

O LVGL não é preemptivo e não possui threads internas. Ele depende de um modelo de execução cooperativa baseado em dois pilares:
1. **Tick Interface (`lv_tick_inc` ou `lv_tick_set_cb`)**: Informa ao LVGL a passagem do tempo (em milissegundos) para animações, transições e debouncing de entrada.
2. **Timer Handler (`lv_timer_handler`)**: A função principal que deve ser chamada periodicamente no loop principal ou em uma task dedicada. Ela processa eventos, atualiza animações e aciona o pipeline de renderização.

```text
+-------------------------------------------------------------------+
|                          Aplicação / UI                           |
|  (Criação de Widgets, Configuração de Estilos, Callbacks de App)  |
+-------------------------------------------------------------------+
|                              LVGL                                 |
|                                                                   |
|  +-------------+  +-------------+  +-------------+  +----------+  |
|  | Widget Tree |  | Style Engine|  | Event System|  | Anim/Tmr |  |
|  +-------------+  +-------------+  +-------------+  +----------+  |
|                                                                   |
|  +-------------------------------------------------------------+  |
|  |                      Draw Pipeline                          |  |
|  |  (Invalidation -> Masking -> Draw Descriptors -> Draw Units)|  |
|  +-------------------------------------------------------------+  |
+-------------------------------------------------------------------+
|                      Hardware Abstraction Layer (HAL)             |
|                                                                   |
|  +-------------------+  +-------------------+  +---------------+  |
|  | Display Interface |  | Input Interface   |  | Tick Interface|  |
|  | (lv_display_t)    |  | (lv_indev_t)      |  | (OS/SysTick)  |  |
|  +-------------------+  +-------------------+  +---------------+  |
+-------------------------------------------------------------------+
|                      Hardware / OS / Drivers                      |
|  (Framebuffers, DMA, Touch I2C, FreeRTOS, Zephyr, Linux DRM)      |
+-------------------------------------------------------------------+
```

### Pipeline de Renderização (Draw Pipeline)

O processo de desenho do LVGL é altamente otimizado para economizar RAM e ciclos de CPU:
1. **Invalidação**: Quando uma propriedade de um widget muda (ex: cor, tamanho), a área ocupada por ele é marcada como "suja" (invalidada).
2. **Agrupamento**: O LVGL agrupa áreas sujas adjacentes para minimizar o número de redesenhos.
3. **Renderização em Buffer**: O LVGL desenha os pixels das áreas sujas em um *draw buffer* na RAM. Ele usa *Draw Units* (Software, DMA2D, Arm-2D) para executar operações como preenchimento, blend, transformações e rasterização de fontes.
4. **Flush**: Quando o buffer está cheio ou a área suja foi totalmente renderizada, o LVGL chama a função `flush_cb` fornecida pelo usuário para transferir os pixels do buffer para o display físico (geralmente via SPI, I8080, RGB ou MIPI DSI).

---

## Estratégias de Buffering e Gerenciamento de Memória

A escolha da estratégia de buffering define o balanço entre uso de RAM, performance e ocorrência de *tearing* (corte na imagem durante a atualização). O LVGL v9 introduziu o `lv_display_set_buffers` com modos explícitos.

| Estratégia | Configuração de Buffers | Uso de RAM | Performance | Tearing | Caso de Uso Ideal |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Single Partial** | 1 buffer (ex: 1/10 da tela) | Muito Baixo | Baixa (bloqueia renderização durante o flush) | Possível | MCUs pequenos (ex: STM32F1, ESP8266), displays SPI lentos. |
| **Double Partial** | 2 buffers (ex: 1/10 da tela cada) | Baixo | Alta (Renderiza no B1 enquanto DMA transfere B2) | Possível | Padrão da indústria para MCUs com DMA (ex: ESP32, RP2040). |
| **Double Full** | 2 buffers do tamanho total da tela | Muito Alto | Máxima | Zero | MPUs (Linux), MCUs com PSRAM/SDRAM e displays RGB/MIPI. |

*Nota de Implementação:* Para usar **Double Partial** eficientemente, a função `flush_cb` deve iniciar a transferência DMA e retornar imediatamente. A interrupção de fim de transferência do DMA deve chamar `lv_display_flush_ready(disp)`.

---

## Conceitos Core

### 1. Widgets e Árvore de Objetos
Tudo no LVGL deriva de `lv_obj_t`. Os widgets formam uma árvore hierárquica onde as coordenadas dos filhos são relativas aos pais. Se um pai é movido ou ocultado, os filhos acompanham. Se um filho ultrapassa os limites do pai, ele é cortado (clipped) por padrão.

### 2. Estilos (Styles)
Inspirado no CSS, o sistema de estilos do LVGL permite separar a lógica da aparência.
* **Cascading**: Estilos podem ser aplicados localmente a um objeto ou globalmente.
* **States**: Estilos podem ser ativados baseados no estado do widget (`LV_STATE_DEFAULT`, `LV_STATE_PRESSED`, `LV_STATE_DISABLED`, `LV_STATE_FOCUSED`).
* **Parts**: Widgets complexos são divididos em partes. Por exemplo, um Slider possui `LV_PART_MAIN` (o fundo), `LV_PART_INDICATOR` (a barra de progresso) e `LV_PART_KNOB` (o botão deslizante).

### 3. Layouts (Flex e Grid)
O LVGL suporta motores de layout nativos que eliminam a necessidade de posicionamento absoluto (`x`, `y`):
* **Flexbox**: Alinha itens em linhas ou colunas, com suporte a quebra de linha (wrap) e espaçamento dinâmico (`space-evenly`, `space-between`).
* **Grid**: Posiciona itens em uma tabela 2D com larguras e alturas de colunas/linhas fixas ou baseadas em "frações" (fr).

### 4. Event Bubbling
Eventos (como cliques) propagam-se do filho para o pai (bubbling). Isso permite colocar um único *event listener* em um contêiner pai para gerenciar cliques em dezenas de botões filhos, verificando o alvo original via `lv_event_get_target(e)`.

---

## Integração e Portabilidade (HAL)

Para rodar o LVGL em um novo hardware, você precisa implementar três interfaces. Abaixo está o padrão de integração atualizado para a **API v9**.

### Exemplo de Porting (C)

```c
#include "lvgl/lvgl.h"

// 1. Função de Tick (Pode ser omitida se usar lv_tick_set_cb)
static uint32_t my_tick_get_cb(void) {
    return get_system_milliseconds(); // Substitua pela função do seu OS/Hardware
}

// 2. Callback de Flush do Display
static void my_flush_cb(lv_display_t * disp, const lv_area_t * area, uint8_t * px_map) {
    uint32_t w = lv_area_get_width(area);
    uint32_t h = lv_area_get_height(area);
    
    // Exemplo: Enviar px_map via SPI ou DMA para as coordenadas area->x1, area->y1
    my_lcd_send_pixels(area->x1, area->y1, w, h, px_map);
    
    // IMPORTANTE: Avisar o LVGL que o flush terminou
    lv_display_flush_ready(disp);
}

// 3. Callback de Leitura de Input (Touch)
static void my_touch_read_cb(lv_indev_t * indev, lv_indev_data_t * data) {
    int16_t x, y;
    bool is_pressed = my_touch_get_coords(&x, &y);
    
    if(is_pressed) {
        data->point.x = x;
        data->point.y = y;
        data->state = LV_INDEV_STATE_PRESSED;
    } else {
        data->state = LV_INDEV_STATE_RELEASED;
    }
}

void lvgl_port_init(void) {
    lv_init();
    lv_tick_set_cb(my_tick_get_cb);

    // Configuração do Display (API v9)
    lv_display_t * disp = lv_display_create(320, 240);
    
    // Alocação de buffer (Double Partial - 1/10 da tela)
    static uint8_t buf1[320 * 24 * 2]; // 2 bytes por pixel (RGB565)
    static uint8_t buf2[320 * 24 * 2];
    lv_display_set_buffers(disp, buf1, buf2, sizeof(buf1), LV_DISPLAY_RENDER_MODE_PARTIAL);
    lv_display_set_flush_cb(disp, my_flush_cb);

    // Configuração do Input Device
    lv_indev_t * indev = lv_indev_create();
    lv_indev_set_type(indev, LV_INDEV_TYPE_POINTER);
    lv_indev_set_read_cb(indev, my_touch_read_cb);
}
```

---

## Padrões de Implementação de UI

Embora o LVGL seja nativo em C, o ecossistema moderno suporta múltiplas formas de desenvolvimento.

### 1. C Nativo (Data Binding e Observers)
A partir da v9, o LVGL introduziu o conceito de `lv_subject_t` e `lv_observer_t` para implementar Data Binding reativo, separando a lógica de negócios da UI.

```c
// Variável de estado global (Subject)
static lv_subject_t temp_subject;

void ui_init(void) {
    // Inicializa o subject com valor 25
    lv_subject_init_int(&temp_subject, 25);

    lv_obj_t * slider = lv_slider_create(lv_screen_active());
    lv_obj_center(slider);
    
    // Faz o bind bidirecional do slider com o subject
    lv_slider_bind_value(slider, &temp_subject);

    lv_obj_t * label = lv_label_create(lv_screen_active());
    lv_obj_align(label, LV_ALIGN_CENTER, 0, -30);
    
    // Faz o bind unidirecional do texto com o subject (formatação automática)
    lv_label_bind_text(label, &temp_subject, "Temperatura: %d °C");
}

// Em outra parte do código (ex: task de sensor):
// lv_subject_set_int(&temp_subject, 28); -> Atualiza slider e label automaticamente!
```

### 2. MicroPython (Prototipagem Rápida)
O LVGL possui bindings gerados automaticamente para MicroPython, sendo a escolha preferida para prototipagem em ESP32 e RP2040.

```python
import lvgl as lv
import display_driver # Driver específico da placa

lv.init()

# Criação de UI em Python
scr = lv.screen_active()
scr.set_flex_flow(lv.FLEX_FLOW.COLUMN)
scr.set_flex_align(lv.FLEX_ALIGN.CENTER, lv.FLEX_ALIGN.CENTER, lv.FLEX_ALIGN.CENTER)

btn = lv.button(scr)
label = lv.label(btn)
label.set_text("Clique Aqui")

def on_click(e):
    print("Botão clicado!")
    label.set_text("Clicado!")

btn.add_event_cb(on_click, lv.EVENT.CLICKED, None)

# O loop principal é gerenciado pelo driver MicroPython em background
```

### 3. XML e LVGL Pro / SquareLine Studio
Para equipes de design, escrever C ou Python para UI é ineficiente. Ferramentas como **SquareLine Studio** e o novo **LVGL Pro Editor** permitem desenhar a UI visualmente e exportar:
1. Código C gerado automaticamente.
2. Arquivos XML que podem ser carregados em tempo de execução pelo LVGL (recurso introduzido no ecossistema Pro).

---

## Pitfalls e Boas Práticas (Senior Notes)

### 1. Thread Safety (O Erro Mais Comum)
**O LVGL NÃO é thread-safe.** Se você estiver usando um RTOS (FreeRTOS, Zephyr), você não pode chamar funções do LVGL de múltiplas tasks ou interrupções simultaneamente.
* **Solução**: Use um Mutex. Trave o mutex antes de chamar `lv_timer_handler()` e antes de qualquer atualização de UI feita por outras tasks.

```c
// Na Task do LVGL:
lv_mutex_lock(&lvgl_mutex);
lv_timer_handler();
lv_mutex_unlock(&lvgl_mutex);

// Na Task do Sensor:
lv_mutex_lock(&lvgl_mutex);
lv_label_set_text(label, "Novo Valor");
lv_mutex_unlock(&lvgl_mutex);
```

### 2. Gargalos de PSRAM no ESP32
Colocar os *draw buffers* do LVGL na PSRAM externa do ESP32 para economizar RAM interna frequentemente destrói a taxa de quadros (FPS cai de 30 para 5).
* **Solução**: Mantenha os *draw buffers* (parciais) na SRAM interna (que é rápida e acessível por DMA). Use a PSRAM apenas para armazenar assets estáticos (imagens decodificadas, fontes grandes) ou como framebuffer final se estiver usando um display RGB paralelo que exija buffer completo.

### 3. Vazamento de Memória com User Data
Quando você deleta um objeto (`lv_obj_delete`), o LVGL limpa automaticamente os filhos e os estilos locais. No entanto, se você alocou memória dinamicamente e a associou ao objeto (via `lv_obj_set_user_data`), essa memória vazará.
* **Solução**: Adicione um evento `LV_EVENT_DELETE` ao objeto e libere a memória lá.

```c
static void cleanup_event_cb(lv_event_t * e) {
    my_struct_t * data = lv_event_get_user_data(e);
    free(data);
}
// ...
lv_obj_add_event_cb(obj, cleanup_event_cb, LV_EVENT_DELETE, my_allocated_data);
```

### 4. Decodificação de Imagens em Tempo Real
Carregar PNGs ou JPEGs diretamente do SD Card em tempo real causa engasgos na UI, pois a decodificação bloqueia o loop principal.
* **Solução**: Use o conversor de imagens do LVGL para converter imagens para o formato binário nativo (C array ou `.bin`) com o formato de cor correto (ex: RGB565). Se precisar usar PNG/JPEG, utilize o sistema de **Image Caching** (`lv_image_cache`) introduzido nas versões recentes.

---

## See Also

* [[freertos]] - Integração de sistemas operacionais de tempo real com bibliotecas gráficas.
* [[esp32]] - Microcontrolador amplamente utilizado com LVGL (via ESP-IDF ou Arduino).
* [[dma-direct-memory-access]] - Crucial para transferências de framebuffer sem bloquear a CPU.
* [[framebuffer]] - Conceitos de gerenciamento de memória de vídeo.

## Sources

* raw/articles/lvgl-overview-docs-8.3.md
* raw/articles/lvgl-docs-master.md
* raw/articles/lvgl-github-readme.md
* raw/articles/lvgl-esp32-tutorial-elecrow.md