# Documentacao tecnica

Ver README.md para visao geral. Este arquivo detalha justificativas.

## Pre-processamento
- Grayscale BT.601: separa objeto escuro do fundo claro.
- Contraste por percentis 2-98: compensa granulacao e tons similares entre chao e parede.
- Gaussiano 7x7 sigma 1.8: reduz ruido antes do Sobel sem apagar orelhas.

## Canny manual
- Sobel 3x3, supressao de nao-maximos, histerese com limiares por percentil.
- Dilatacao 3x3: fecha pequenas quebras no contorno externo.

## Caminho
- Maior componente conexo ignora fragmentos da coleira.
- Rastreamento Moore ordena pixels para polilinha continua.
- Douglas-Peucker reduz pontos para a tartaruga.
- Mapeamento inverte Y e escala para [0.5, 10.5].

## ROS 2
- cmd_vel proporcional; set_pen e teleport em saltos > 0.35.
