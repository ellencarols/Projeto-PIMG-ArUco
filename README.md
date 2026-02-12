# Processamento de Imagens ArUco: Detecção, Rotação e Remoção de Texto

Este projeto Python implementa um pipeline completo para processar imagens contendo marcadores ArUco.

Este projeto é capaz de:
1.  **Detectar e Localizar** precisamente a região preta de um marcador ArUco em imagens.
2.  **Determinar a Orientação** do marcador analisando a posição relativa de textos próximos.
3.  **Corrigir a Rotação** da imagem para garantir que o texto associado ao ArUco esteja sempre alinhado abaixo dele.
4.  **Remover Seletivamente o Texto** ao redor do marcador, isolando e preservando apenas o ArUco na imagem final.

## Funcionalidades Principais e Fluxo de Execução

O pipeline opera em etapas sequenciais para alcançar o objetivo final:

1.  **Carregamento e Pré-processamento:**
    *   Carrega a imagem de entrada.
    *   Converte a imagem para escala de cinza (`np.dot`).

2.  **Detecção do Marcador ArUco (Orientação):**
    *   **Isolamento da Região Preta:** `detect_black_region_boundaries` cria uma máscara dos pixels mais escuros.
    *   **Detecção de Bordas:** `sobel_edge_detection` encontra as bordas na máscara preta.
    *   **Detecção de Linhas:** `hough_transform_lines` identifica linhas retas a partir das bordas.
    *   **Filtragem de Linhas Principais:** `filter_main_lines` seleciona as 2 linhas horizontais e 2 verticais mais proeminentes.
    *   **Identificação de Cantos:** `line_intersection` é usada para encontrar os pontos de interseção das linhas, e `find_aruco_corners` organiza esses pontos como os cantos do ArUco.

3.  **Análise e Correção de Orientação:**
    *   **Detecção da Posição do Texto:** `detect_text_position` analisa a distribuição de pixels escuros ao redor do ArUco para determinar a posição do texto.
    *   **Cálculo da Rotação:** `calculate_rotation_from_text_position` determina o ângulo necessário para que o texto fique abaixo do ArUco.
    *   **Rotação da Imagem:** `rotate_image` aplica a rotação calculada à imagem original ou à imagem rotacionada, se já estiver na orientação correta, nenhuma rotação é aplicada.

4.  **Remoção de Texto:**
    *   **Deteção da Região do ArUco para Preservação:** `detect_aruco_region` é usada internamente por `remove_text_preserve_aruco` para encontrar a área a ser mantida.
    *   **Remoção de Elementos Externos:** `remove_text_preserve_aruco` cria uma nova imagem branca e copia apenas a região do ArUco da imagem rotacionada para ela, efetivamente removendo todo o texto e elementos externos.

## Como Usar

O pipeline completo é encapsulado na função `process_aruco_complete`.

### Função Principal

`process_aruco_complete(input_path, debug=True)`

*   `input_path`: Caminho para a imagem de entrada (por exemplo, `"Imagem4.jpg"`).
*   `debug`: (Padrão: `True`) Se `True`, imprime mensagens de depuração durante o processo.

### Exemplo de Uso

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
    plt.title("2 - ArUco Final sem Texto")
    plt.axis("off")

    plt.tight_layout()
    plt.show()
```

Além disso, a função `detect_aruco_black_only` está disponível para detecção focada apenas na região preta do ArUco, sem considerar a rotação ou remoção de texto.

### Exemplo de Detecção Apenas da Região Preta

```python
# Assumindo 'final_img' é uma imagem já processada ou 'img_array' a original
corners, edges, black_mask = detect_aruco_black_only(final_img, debug=True)

if corners is not None:
    import matplotlib.pyplot as plt
    # ... (código de visualização como na célula de execução) ...
```

## Funções Auxiliares Detalhadas

*   `detect_black_region_boundaries(gray_array)`: Recebe uma imagem em escala de cinza e retorna uma máscara binária destacando pixels muito escuros (região preta do ArUco).
*   `sobel_edge_detection(gray_array)`: Aplica o filtro de Sobel para calcular a magnitude e direção das bordas em uma imagem em escala de cinza.
*   `hough_transform_lines(edge_image, threshold)`: Detecta linhas retas em uma imagem de borda usando a Transformada de Hough, retornando as linhas mais proeminentes.
*   `filter_main_lines(lines, image_shape)`: Filtra as linhas detectadas, buscando as duas horizontais e duas verticais mais votadas que formam o contorno do ArUco.
*   `line_intersection(line1, line2)`: Calcula o ponto de interseção entre duas linhas dadas em formato polar (rho, theta).
*   `find_aruco_corners(h_lines, v_lines)`: Utiliza as linhas horizontais e verticais filtradas para encontrar e ordenar os quatro cantos do ArUco.
*   `detect_text_position(gray_array, corners)`: Analisa as regiões adjacentes aos cantos do ArUco para determinar se há texto e em qual direção (acima, abaixo, esquerda, direita).
*   `calculate_rotation_from_text_position(text_pos)`: Converte a posição detectada do texto em um ângulo de rotação (0°, 90°, 180°, -90°) para corrigir a orientação.
*   `rotate_image(img_array, angle_deg)`: Rotaciona uma imagem (RGB ou em escala de cinza) por um ângulo especificado em graus.
*   `detect_aruco_region(gray_array)`: Heuristicamente detecta a região retangular contendo o ArUco em uma imagem em escala de cinza, usada para delimitar a área a ser preservada.
*   `remove_text_preserve_aruco(img_array)`: Recebe uma imagem e retorna uma nova imagem onde apenas a região do ArUco (detectada internamente) é mantida, e o restante é preenchido com branco.
*   `binarize_image(gray_array, threshold)`: Converte uma imagem em escala de cinza em uma imagem binária (preto e branco) com base em um limiar.
*   `draw_detection_result(img_array, corners)`: Para fins de visualização, desenha as linhas e os cantos detectados do ArUco sobre uma cópia da imagem original.
*   `detect_aruco_black_only(img_array, debug)`: Um pipeline de detecção de ArUco que se concentra apenas na região preta do marcador, retornando os cantos detectados e as imagens intermediárias de borda e máscara.

## Requisitos

*   `numpy`
*   `imageio` (especificamente `imageio.v3`)
*   `matplotlib`
