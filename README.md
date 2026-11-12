# NEUIT_IPEN — Simulador de Neutrongrafia Policromática (IEA-R1 / IPEN) e Gerador de Dataset para U-Net

[![DOI](https://zenodo.org/badge/122098298.svg)](https://zenodo.org/badge/latestdoi/122098298)

Adaptação e extensão do **NEUIT** (*NEUtron Imaging Toolbox*) voltada à **simulação física realista de radiografia com nêutrons térmicos/frios** calibrada para a estação experimental **BH-14 do Reator Nuclear de Pesquisa IEA-R1 (IPEN-CNEN/SP)**, com geração automatizada de *datasets* sintéticos (imagens radiográficas em `float32` e máscaras semânticas em `uint8`) para o treinamento de redes neurais profundas de segmentação (**U-Net**).

---

## 🎯 Principais Funcionalidades

1. **Espectro Policromático Realista do Reator IEA-R1:**
   * Integração espectral ponderada pelo fluxo de nêutrons do feixe **BH-14** (`static/instrument_file/beam_flux_iear1.csv`).
   * Cálculo explícito do endurecimento de feixe (*beam hardening*) na transmissão policromática.

2. **Seções de Choque Macroscópicas &Sigma;(&lambda;) Dependentes do Comprimento de Onda:**
   * **Materiais Cristalinos (`Fe`, `Cu`, `Al`):** Modelagem das bordas de difração coerente (*Bragg-edges*) via `BraggEdgeModeling` (`diffpy.Structure` + `xscalc`).
   * **Materiais Amorfos, Ligas Metálicas, Madeiras, Polímeros e Gases:** Seções de choque avaliadas da biblioteca nuclear **ENDF/B-VIII** via `ImagingReso`.

3. **Calibração Experimental do Feixe e do Detector (BH-14 / IEA-R1):**
   * Malha de 512 &times; 512 pixels (FOV = 5,5 &times; 5,5 cm<sup>2</sup>, tamanho de pixel de 107,4 &mu;m), distância fonte-detector *L* = 220 cm (*L/D* = 110).
   * Perfil espacial do feixe (*Flat-Field*) calibrado diretamente a partir de **52 radiografias experimentais de campo aberto** (`flat_field/*.tif`) e perfis transversais em 16 bits (`perfil_256_x.txt` e `perfil_256_y.txt`), reproduzindo:
     * Descentralização real da janela circular do colimador: (*x*<sub>0</sub>, *y*<sub>0</sub>) = (&minus;0,108 cm, +0,010 cm) e raio efetivo *r*<sub>0</sub> = 2,453 cm;
     * Borda penumbral modelada via função erro complementar (`erfc`);
     * Assimetria espacial (gradiente horizontal/vertical do feixe) e fator de divergência geométrica (cos<sup>4</sup>&theta;);
     * Atenuação pelo caminho no ar atmosférico seco (N<sub>156</sub>O<sub>42</sub>Ar<sub>1</sub>) corrigido pela obliquidade dos raios (1/cos&theta;).
   * Resposta do detector cintilador + câmera científica:
     * *Point Spread Function* (PSF Gaussiana com &sigma;<sub>PSF</sub> = 1,2 px);
     * Corrente de escuro (*Dark Current*, *I*<sub>dark</sub>);
     * Ruído quântico de contagem governado pela **estatística de Poisson**.

4. **Gerador de Amostras 3D e Dataset Sintético para U-Net:**
   * Sorteio parametrizado de **3 famílias geométricas** com dimensões, posições e espessuras contínuas aleatórias:
     1. **Cunha de degraus (*Step-wedge*):** 1 a 5 degraus com espessuras independentes entre 0,1 cm e 2,0 cm;
     2. **Moeda / Disco visto de frente:** raio e espessura variáveis em posições aleatórias;
     3. **Cilindro vertical escalonado:** 1 a 3 seções cilíndricas empilhadas com perfil de espessura cordal $d(x) = 2\sqrt{R^2 - (x - x_c)^2}$.
   * Exportação pareada em formato **TIFF**:
     * `saida_tiff/img/<id>.tiff`: Radiografia sintética bruta em **32-bit float (`IEEE754 float32`)**;
     * `saida_tiff/mask/<id>.tiff`: Máscara de segmentação semântica em **8-bit unsigned int (`uint8`)** com os IDs das classes (`0` a `11`).

---

## 🧪 Classes de Materiais Suportadas (`labels`)

O simulador modela **11 classes de materiais** (além da classe `0 = fundo`), mapeadas no arquivo [`labels`](labels):

| ID (`uint8`) | Classe (`labels`) | Descrição | Fórmula / Estrutura | Densidade (g/cm<sup>3</sup>) | Modelo Físico |
| :---: | :--- | :--- | :--- | :---: | :--- |
| `0` | `fundo` | Feixe aberto (ar atmosférico) | N<sub>156</sub>O<sub>42</sub>Ar<sub>1</sub> | 0,001204 | ENDF/B-VIII (`ImagingReso`) |
| `1` | `bronze` | Liga de Bronze | Cu<sub>88</sub>Sn<sub>12</sub> | 8,80 | ENDF/B-VIII (`ImagingReso`) |
| `2` | `cedro` | Madeira Cedro (Celulose) | C<sub>6</sub>H<sub>10</sub>O<sub>5</sub> | 0,50 | ENDF/B-VIII (`ImagingReso`) |
| `3` | `chumbo` | Chumbo metálico | Pb | 11,34 | ENDF/B-VIII (`ImagingReso`) |
| `4` | `cobre` | Cobre cristalino (CFC / FCC) | Cu (*a* = 3,615 Å) | 8,96 | *Bragg-Edges* (`BraggEdgeModeling`) |
| `5` | `ferro` | Ferro cristalino (CCC / BCC) | Fe (*a* = 2,866 Å) | 7,87 | *Bragg-Edges* (`BraggEdgeModeling`) |
| `6` | `grafite` | Grafite | C | 2,20 | ENDF/B-VIII (`ImagingReso`) |
| `7` | `latao` | Liga de Latão | Cu<sub>65</sub>Zn<sub>35</sub> | 8,50 | ENDF/B-VIII (`ImagingReso`) |
| `8` | `lucite` | Acrílico / PMMA (Lucite) | C<sub>5</sub>O<sub>2</sub>H<sub>8</sub> | 1,19 | ENDF/B-VIII (`ImagingReso`) |
| `9` | `p_sabao` | Pedra-sabão (Talco / Esteatito) | Mg<sub>3</sub>Si<sub>4</sub>O<sub>12</sub>H<sub>2</sub> | 2,70 | ENDF/B-VIII (`ImagingReso`) |
| `10` | `prata` | Prata metálica | Ag | 10,49 | ENDF/B-VIII (`ImagingReso`) |
| `11` | `ouro` | Ouro metálico | Au | 19,30 | ENDF/B-VIII (`ImagingReso`) |

---

## 📂 Estrutura Principal do Projeto

```text
NEUIT_IPEN/
├── main.ipynb                                  # Notebook principal: física do feixe, calibração e gerador de dataset U-Net
├── labels                                      # Mapeamento das 11 classes de materiais (IDs 1..11)
├── perfil_256_x.txt                            # Perfil experimental horizontal (linha Y=256) em 16 bits
├── perfil_256_y.txt                            # Perfil experimental vertical (coluna X=256) em 16 bits
├── flat_field/                                 # 52 imagens experimentais de flat-field (.tif, 32 bits) do IEA-R1
├── static/instrument_file/beam_flux_iear1.csv  # Espectro de fluxo de nêutrons do Reator IEA-R1
├── saida_tiff/                                 # Dataset gerado (ignorado no Git):
│   ├── img/                                    #   Radiografias sintéticas 32-bit float (.tiff)
│   └── mask/                                   #   Máscaras de segmentação 8-bit uint8 (.tiff)
└── requirements.txt                            # Dependências Python do projeto
```

---

## 🚀 Como Executar

### 1. Configurar o ambiente Python
```bash
python3 -m venv .venv_img
source .venv_img/bin/activate
pip install -r requirements.txt
pip install tqdm tifffile pillow
```

### 2. Executar o Simulador e Gerador de Dataset
Abra o notebook [`main.ipynb`](main.ipynb) no VS Code / Antigravity ou Jupyter Lab e execute as células sequencialmente:
* **Partes 1 a 4:** Carregam o espectro do IEA-R1, calculam as curvas &Sigma;(&lambda;) com *Bragg-edges* e ENDF/B-VIII, e calibram a máscara espacial 2D do feixe.
* **Partes 5 a 7:** Definem as funções `gerador_amostra(X, Y)` e `radiografia(...)` e executam o loop com barra de progresso (`tqdm`) salvando os pares `(img, mask)` em `saida_tiff/`.

---

## 📚 Créditos e Referências

Este projeto utiliza como base de seções de choque e estrutura os pacotes científicos desenvolvidos no **Oak Ridge National Laboratory (ORNL)**:

1. Yuxuan Zhang, Jean Bilheux, Hassina Bilheux and Jiao Lin (2019), *"An interactive web-based tool to guide the preparation of neutron imaging experiments at Oak Ridge National Laboratory"*, Journal of Physics Communications, 3(10), 103003, [doi:10.1088/2399-6528/ab4ee6](https://iopscience.iop.org/article/10.1088/2399-6528/ab4ee6).
2. Yuxuan Zhang and Jean Bilheux (2017), *"ImagingReso: A Tool for Neutron Resonance Imaging"*, Journal of Open Source Software, 2(19), 407, [doi:10.21105/joss.00407](http://joss.theoj.org/papers/997d09281a9d76e95f4ec4d3279eeb8c).
