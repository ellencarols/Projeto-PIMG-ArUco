# Processamento de Imagens ArUco: Detecção, Rotação e Remoção de Texto
   **Aluna: Ellen Caroline Soares dos Santos**
   **Link Youtube:https://youtu.be/9oN7H5wcumU**

Este projeto Python implementa um pipeline completo para processar imagens contendo marcadores ArUco.

Este projeto é capaz de:
1.  **Analisar a imagem inicial** para identificar regiões que podem ser o ArUco e possíveis áreas de texto circundante.
2.  **Identificar e Rotacionar** o marcador ArUco para uma orientação padrão, onde o texto associado fica sempre abaixo dele.
3.  **Remover** todo o texto ao redor do ArUco rotacionado.
4.  **Realizar uma Detecção Precisa** do ArUco, focando exclusivamente na figura 8x8 sem as bordas cinzas que poderiam estar presentes na imagem original.

## Funcionalidades Principais e Fluxo de Execução

O pipeline opera em etapas sequenciais para alcançar o objetivo final:

### Parte 1: Identificação, Orientação e Remoção Inicial de Texto (Função `process_aruco_complete`)

Esta primeira parte do pipeline se concentra em preparar a imagem, identificar a orientação correta do ArUco e fazer uma limpeza inicial do texto.

1.  **Carregamento e Pré-processamento:**
    *   Carrega a imagem de entrada.
    *   Converte a imagem para escala de cinza (`np.dot`).

2.  **Identificação do Núcleo do ArUco e Posição do Texto (para Orientação):**
    *   **Isolamento da Região Preta (`detect_black_region_boundaries`):** Cria uma máscara dos pixels *muito escuros* (black_threshold = 80). Esta etapa é crucial para focar na estrutura interna do ArUco, **ignorando as bordas cinzas** que possam estar presentes, para uma detecção de linhas e cantos mais precisa.
    *   **Detecção de Bordas (`sobel_edge_detection`):** Encontra as bordas na máscara preta gerada.
    *   **Detecção de Linhas (`hough_transform_lines`):** Identifica linhas retas a partir das bordas usando a Transformada de Hough.
    *   **Filtragem de Linhas Principais (`filter_main_lines`):** Seleciona as 2 linhas horizontais e 2 verticais mais proeminentes, que definem o perímetro do ArUco.
    *   **Identificação de Cantos (`find_aruco_corners`):** Calcula os pontos de interseção das linhas filtradas, organizando-os como os cantos do ArUco.
    *   **Detecção da Posição do Texto (`detect_text_position`):** Analisa as regiões adjacentes aos cantos do ArUco para determinar se há texto e em qual direção (acima, abaixo, esquerda, direita).

3.  **Correção da Orientação:**
    *   **Cálculo da Rotação (`calculate_rotation_from_text_position`):** Determina o ângulo necessário para que o texto fique abaixo do ArUco (0, 90, 180 ou -90 graus).
    *   **Rotação da Imagem (`rotate_image`):** Aplica a rotação calculada à imagem. Se a orientação já estiver correta, nenhuma rotação é aplicada.

4.  **Remoção de Texto e Limpeza de Bordas (`remove_text_preserve_aruco`):**
    *   Esta função opera na imagem *já rotacionada*. Internamente, ela utiliza `detect_aruco_region` para identificar a área geral do ArUco.
    *   Em seguida, cria uma nova imagem branca e **copia para ela apenas a região do ArUco**, efetivamente removendo todo o texto, eventuais bordas cinzas e outros elementos externos indesejados que não fazem parte do ArUco principal.

### Parte 2: Detecção Precisa do ArUco 8x8 sem Bordas (Função `detect_aruco_black_only`)

Após a imagem ter sido processada pela `process_aruco_complete` (resultando em `final_img`), a função `detect_aruco_black_only` pode ser aplicada para uma detecção mais refinada dos cantos do ArUco 8x8 puro, já sem o texto e sem as bordas cinzas, focando apenas no padrão interno.

## Como Usar

O pipeline completo é encapsulado na função `process_aruco_complete`.

### Função Principal

`process_aruco_complete(input_path, debug=True)`

*   `input_path`: Caminho para a imagem de entrada (por exemplo, `"Imagem4.jpg"`).
*   `debug`: (Padrão: `True`) Se `True`, imprime mensagens de depuração durante o processo.

### Exemplo de Uso do Pipeline Principal

```python
input_image = "Imagem4.jpg"

# Processar a imagem
rotated_img, final_img = process_aruco_complete(
    input_image,
    debug=True
)

# Exibir resultados (se o processamento foi bem-sucedido)
if rotated_img is not None and final_img is not None:
    import matplotlib.pyplot as plt

    plt.figure(figsize=(12, 6))

    plt.subplot(1, 2, 1)
    plt.imshow(rotated_img)
    plt.title("1 - ArUco Rotacionado")
    plt.axis("off")

    plt.subplot(1, 2, 2)
    plt.imshow(final_img)
    plt.title("2 - ArUco Final sem Texto e Bordas")
    plt.axis("off")

    plt.tight_layout()
    plt.show()
```

### Exemplo de Detecção Precisa do ArUco 8x8 (após o pipeline principal)

Após obter a `final_img` limpa da `process_aruco_complete`, podemos usar `detect_aruco_black_only` para identificar os cantos precisos da figura 8x8:

```python
# Usando a 'final_img' obtida do pipeline principal
corners, edges, black_mask = detect_aruco_black_only(final_img, debug=True)

if corners is not None:
    import matplotlib.pyplot as plt
    print("DETECÇÃO DO ArUco (REGIÃO PRETA)")
    print(f"Número de cantos detectados: {len(corners)}")

    plt.figure(figsize=(20, 5))

    plt.subplot(1, 4, 1)
    plt.imshow(edges, cmap="gray")
    plt.title("Bordas do ArUco 8x8")
    plt.axis("off")

    plt.subplot(1, 4, 2)
    plt.imshow(black_mask, cmap="gray")
    plt.title("Máscara do ArUco 8x8")
    plt.axis("off")

    plt.subplot(1, 4, 3)
    plt.imshow(black_mask, cmap="gray")
    plt.scatter(corners[:, 0], corners[:, 1], c="red", s=40)
    plt.title("Cantos do ArUco 8x8")
    plt.axis("off")

    resultado_final_visual = draw_detection_result(final_img, corners)
    plt.figure(figsize=(6, 6))
    plt.imshow(resultado_final_visual)
    plt.title("ArUco 8x8 Detectado – Cantos e Retas")
    plt.axis("off")
    plt.show()
    plt.tight_layout()
    plt.show()
else:
    print("Nenhum ArUco 8x8 foi detectado na imagem final.")
```

## Funções Auxiliares Detalhadas

*   `detect_black_region_boundaries(gray_array)`: Recebe uma imagem em escala de cinza e retorna uma máscara binária destacando pixels *muito escuros* (black_threshold = 80). É fundamental para focar apenas na região do ArUco.
*   `sobel_edge_detection(gray_array)`: Aplica o filtro de Sobel para calcular a magnitude e direção das bordas em uma imagem em escala de cinza.
*   `hough_transform_lines(edge_image, threshold)`: Detecta linhas retas em uma imagem de borda usando a Transformada de Hough, retornando as linhas mais proeminentes.
*   `filter_main_lines(lines, image_shape)`: Filtra as linhas detectadas, buscando as duas horizontais e duas verticais mais votadas que formam o contorno do ArUco.
*   `line_intersection(line1, line2)`: Calcula o ponto de interseção entre duas linhas dadas em formato polar (rho, theta).
*   `find_aruco_corners(h_lines, v_lines)`: Utiliza as linhas horizontais e verticais filtradas para encontrar e ordenar os quatro cantos do ArUco.
*   `detect_text_position(gray_array, corners)`: Analisa as regiões adjacentes aos cantos do ArUco para determinar se há texto e em qual direção (acima, abaixo, esquerda, direita).
*   `calculate_rotation_from_text_position(text_pos)`: Converte a posição detectada do texto em um ângulo de rotação (0°, 90°, 180°, -90°) para corrigir a orientação.
*   `rotate_image(img_array, angle_deg)`: Rotaciona uma imagem (RGB ou em escala de cinza) por um ângulo especificado em graus.
*   `detect_aruco_region(gray_array)`: Heuristicamente detecta a região retangular contendo o ArUco em uma imagem em escala de cinza. É usada internamente por `remove_text_preserve_aruco` para saber qual área do ArUco preservar.
*   `remove_text_preserve_aruco(img_array)`: Recebe uma imagem (já rotacionada) e retorna uma nova imagem onde apenas a região do ArUco (detectada por `detect_aruco_region`) é mantida, e o restante é preenchido com branco, garantindo a remoção de texto, bordas cinzas e outros elementos.
*   `binarize_image(gray_array, threshold)`: Converte uma imagem em escala de cinza em uma imagem binária (preto e branco) com base em um limiar.
*   `draw_detection_result(img_array, corners)`: Para fins de visualização, desenha as linhas e os cantos detectados do ArUco sobre uma cópia da imagem original.
*   `detect_aruco_black_only(img_array, debug)`: Um pipeline de detecção de ArUco que se concentra apenas na região preta do marcador, retornando os cantos detectados e as imagens intermediárias de borda e máscara. Ideal para ser usado em imagens *já limpas* para encontrar o ArUco 8x8 sem ruídos.

## Requisitos

*   `numpy`
*   `imageio` (especificamente `imageio.v3`)
*   `matplotlib`
