# Processamento de Imagens e Extração de Características

Este módulo aprofunda técnicas de pré-processamento e extração de características,
com foco em comparação experimental de pipelines.

---

## Comparando pipelines sob variação de iluminação

### Descrição geral

Neste desafio, você vai construir e comparar **duas ou mais pipelines de
pré-processamento** para segmentar um objeto sob **iluminação boa vs ruim**.
A ideia é ir além do exercício anterior e tratar isso como um pequeno
**experimento controlado**, avaliando qual pipeline é mais robusta.

Você vai testar combinações como:

- HSV direto, sem equalização.
- Equalização (CLAHE em Lab), depois HSV.
- (Opcional) Detecção de bordas (Canny) como apoio.

### Objetivos de aprendizagem

- Desenvolver raciocínio **experimental** em visão computacional:
  definir, testar e comparar pipelines.
- Entender o papel da **equalização local (CLAHE)** na robustez à iluminação.
- Reforçar a documentação de parâmetros e a análise crítica dos resultados.

### Cenário e dados sugeridos

- Reaproveite as mesmas **duas imagens do mesmo objeto**:
  - Uma bem iluminada.
  - Outra sob pouca luz.
- Mantenha o mesmo tamanho de imagem usado no desafio anterior para facilitar a
  comparação.

### 📦 Dataset disponível: ALOI (Amsterdam Library of Object Images)

Este desafio compartilha o mesmo dataset da Aula 01:

| Item | Descrição |
|------|-----------|
| **Localização** | `data/aloi_subset/` (link simbólico para Aula 01) |
| **Objetos** | 20 (pastas 1 a 20) |
| **Imagens por objeto** | 24 (8 direções × 3 intensidades) |
| **Total de imagens** | 480 |

#### Sugestão de pares para comparação

Para cada objeto, compare:
- **Boa iluminação:** `{obj}_l4c3.png` (luz frontal, alta intensidade)
- **Má iluminação:** `{obj}_l1c1.png` (luz lateral, baixa intensidade)

> 💡 Veja a documentação completa do dataset na Aula 01.

### Passo a passo sugerido

#### 1. Definir as pipelines

- **Pipeline 1 – HSV direto**
  - Converter de RGB para HSV.
  - Segmentar por limiar no canal H.

- **Pipeline 2 – CLAHE na luminância + HSV**
  - Converter RGB → Lab.
  - Aplicar **CLAHE** no canal L.
  - Converter de volta para RGB → HSV.
  - Segmentar por limiar no canal H.

- **(Opcional) Pipeline 3 – CLAHE + Canny**
  - Aplicar CLAHE na luminância.
  - Usar **Canny** para detectar bordas.
  - Opcionalmente combinar bordas com uma máscara de cor.

#### 2. Aplicar as pipelines nas duas imagens
- Execute cada pipeline nas imagens de boa e má iluminação.
- Gere:
  - Máscara final de segmentação.
  - (Opcional) mapa de bordas (Canny).

#### 3. Registrar parâmetros
- Para cada pipeline, registre:
  - Configurações de CLAHE (clipLimit, tileGridSize).
  - Kernel e sigma do Gaussian blur (se usado).
  - Faixas de H utilizadas na limiarização.
  - Parâmetros do Canny (limiar baixo/alto).

#### 4. Comparar os resultados
- Visualmente:
  - As máscaras se mantêm consistentes nas duas condições de luz?
  - Há mais artefatos em alguma pipeline?
- Se quiser, adicione um critério quantitativo simples:
  - Comparar área segmentada.
  - Contar pixels "desviantes" entre as máscaras.

#### 5. Analisar criticamente
- Qual pipeline é mais robusta à mudança de iluminação?
- Qual é mais sensível a ruído?
- Em um cenário industrial, qual delas você adotaria e por quê?

### Critérios de sucesso

- Pelo menos **duas pipelines** implementadas e documentadas.
- Comparação clara dos resultados (imagens + comentários).
- Conclusão explícita sobre qual pipeline é preferível em contexto real.

### Extensões possíveis

- Definir uma métrica simples (ex.: IoU com uma ROI "ideal" anotada à mão).
- Incluir uma **terceira condição de iluminação** (ex.: forte contra-luz).
- Experimentar outros espaços de cor (ex.: YCrCb) em vez de Lab.

---

## 🚀 Como Começar (Configuração do Ambiente)

Antes de começar a programar, você precisa configurar o ambiente Python.
Siga estes passos:

### 1️⃣ Abra o terminal na pasta do desafio

```bash
cd "01_Comparando_Pipelines_sob_Variacao_de_Iluminacao"
```

### 2️⃣ Crie um ambiente virtual

**Windows:**
```bash
python -m venv .venv
```

**Mac/Linux:**
```bash
python3 -m venv .venv
```

### 3️⃣ Ative o ambiente virtual

**Windows (Prompt de Comando):**
```bash
.venv\Scripts\activate
```

**Windows (PowerShell):**
```bash
.venv\Scripts\Activate.ps1
```

**Mac/Linux:**
```bash
source .venv/bin/activate
```

> ✅ Você verá `(.venv)` no início da linha do terminal quando estiver ativado.

### 4️⃣ Instale as bibliotecas

```bash
pip install -r requirements.txt
```

### 5️⃣ Abra o Jupyter Notebook

```bash
jupyter notebook
```

### 📦 Bibliotecas utilizadas neste desafio

| Biblioteca | Para que serve |
|------------|----------------|
| `opencv-python` | Processar imagens (CLAHE, blur, conversão de cores) |
| `numpy` | Operações matemáticas com arrays |
| `matplotlib` | Visualizar imagens e criar gráficos |
| `jupyter` | Ambiente interativo para escrever código |

---

## Estrutura de pastas

```
02_Processamento_de_Imagens_e_Extracao_de_Caracteristicas/
├── README.md
└── 01_Comparando_Pipelines_sob_Variacao_de_Iluminacao/
    ├── requirements.txt      ← Lista de bibliotecas
    ├── data/
    │   └── aloi_subset/      ← Dataset ALOI (link simbólico)
    ├── images/               ← Suas imagens (opcional)
    ├── .venv/                ← Ambiente virtual (criado por você)
    ├── experimento.ipynb     ← Seu notebook de experimentos
    └── resultados/           ← Máscaras e figuras geradas
```

---

## Referências úteis

- [OpenCV - CLAHE](https://docs.opencv.org/4.x/d5/daf/tutorial_py_histogram_equalization.html)
- [OpenCV - Color Spaces](https://docs.opencv.org/4.x/df/d9d/tutorial_py_colorspaces.html)
- [OpenCV - Gaussian Blur](https://docs.opencv.org/4.x/d4/d13/tutorial_py_filtering.html)
- [OpenCV - Canny Edge Detection](https://docs.opencv.org/4.x/da/d22/tutorial_py_canny.html)
