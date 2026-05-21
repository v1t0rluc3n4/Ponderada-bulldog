# Bulldog Drawer — Contorno no Turtlesim (ROS 2)

Projeto que **lê uma imagem**, **extrai o contorno principal** com visão computacional implementada do zero (apenas **NumPy** para operações matriciais; **OpenCV só para `imread`**) e **comanda a tartaruga do `turtlesim`** para desenhar o contorno na tela.

## Estrutura do repositório

```
ros2_bulldog_turtlesim/
├── README.md
├── requirements.txt
├── scripts/
│   └── visualize_pipeline.py      # Visualização Matplotlib das etapas
└── src/bulldog_drawer/
    ├── images/bulldog.png           # Coloque sua foto aqui
    ├── launch/draw_contour.launch.py
    └── bulldog_drawer/
        ├── contour_drawer.py        # Nó ROS 2
        └── vision/
            ├── preprocess.py        # (01) Pré-processamento
            ├── edges.py             # (02) Detecção de bordas (Canny)
            ├── contours.py          # Rastreamento e simplificação
            ├── mapping.py           # (03) Mapeamento turtlesim
            └── pipeline.py          # Orquestração
```

## Requisitos

| Componente | Versão sugerida |
|------------|-----------------|
| Ubuntu + ROS 2 | Humble ou Jazzy |
| Python | 3.10+ |
| Pacotes ROS | `rclpy`, `turtlesim`, `geometry_msgs` |
| Pip | `numpy`, `opencv-python-headless`, `matplotlib` |

```bash
pip install -r requirements.txt
```

## Preparação da imagem

Salve a foto do bulldog (fornecida no enunciado) como:

`src/bulldog_drawer/images/bulldog.png`

## Build e execução ROS 2

**No Windows**, use o **WSL Ubuntu** (ROS 2 não funciona no Git Bash):

```bash
cd /mnt/c/Users/vitor/Downloads/ros2_bulldog_turtlesim/ros2_bulldog_turtlesim
source /opt/ros/jazzy/setup.bash   # ou humble
colcon build --packages-select bulldog_drawer
source install/setup.bash

# Terminal 1 — simulador
ros2 run turtlesim turtlesim_node

# Terminal 2 — desenho (após ~2 s)
ros2 run bulldog_drawer draw_contour
```

Ou tudo de uma vez (imagens + turtlesim):

```bash
bash scripts/run_all.sh
```

### Parâmetros úteis

```bash
ros2 run bulldog_drawer draw_contour --ros-args \
  -p image_path:=/caminho/absoluto/bulldog.png \
  -p linear_speed:=1.5 \
  -p goal_tolerance:=0.06
```

### Launch file (recomendado — abre turtlesim + desenho juntos)

```bash
source install/setup.bash
ros2 launch bulldog_drawer draw_contour.launch.py
```

Se usar `ros2 run bulldog_drawer draw_contour` sozinho, **antes** abra o turtlesim em outro terminal:

```bash
ros2 run turtlesim turtlesim_node
```

## Visualização offline (sem ROS)

```bash
python scripts/visualize_pipeline.py src/bulldog_drawer/images/bulldog.png --output-dir output/images
```

Sem argumentos, abre duas janelas como no enunciado: grade **Original / Grayscale / Sobel / Threshold** e **Contornos Detectados**.

Gera em `output/images/`:
`01_original`, `02_grayscale`, `03_sobel`, `04_threshold`, `05_contornos_detectados`, `06_pipeline_grid`, `07_contour_overlay`, `08_turtlesim_path`.

## Pipeline de visão (resumo)

| Etapa | Módulo | Método | Justificativa |
|-------|--------|--------|---------------|
| **01** | `preprocess.py` | Grayscale BT.601, esticamento de contraste (percentis 2–98), blur gaussiano 7×7 σ=1.8 | A foto tem **ruído granular**; o blur reduz falsas bordas sem apagar o contorno externo do cão |
| **02** | `edges.py` | Sobel → supressão de não-máximos → limiar duplo + histerese → dilatação 3×3 | **Canny** produz bordas finas e conectadas; limiares por percentil adaptam-se ao contraste da cena; dilatação fecha pequenas rupturas |
| **03** | `contours.py` + `mapping.py` | Maior componente conexo, rastreamento Moore, Douglas–Peucker, escala para [0.5, 10.5]² com inversão de Y | Ignora detalhes da coleira; reduz pontos para a tartaruga; alinha coordenadas de imagem (topo-esquerda) com turtlesim (origem embaixo) |
| **04** | `contour_drawer.py` | `cmd_vel` proporcional + `set_pen` / `teleport_absolute` | Desenha segmentos contínuos; em saltos grandes levanta a caneta e teleporta |

Documentação detalhada: [docs/DOCUMENTACAO.md](docs/DOCUMENTACAO.md)

## Restrições atendidas

- OpenCV **apenas** em `cv2.imread`
- Sem Pillow, scikit-image, scipy para CV
- NumPy para matrizes; Matplotlib só para visualização

## Ajuste fino para a foto do bulldog

Se o desenho ficar muito fragmentado ou com ruído:

1. Aumente `sigma` em `gaussian_blur` (ex.: 2.2)
2. Suba `high_ratio` em `ContourPipeline` (ex.: 0.32)
3. Aumente `simplify_epsilon` (ex.: 3.5) para menos pontos

## Licença

MIT
