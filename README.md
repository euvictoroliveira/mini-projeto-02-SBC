**Mini-Projeto 2 | Sistemas Baseados em Conhecimento UFPB ·
Controlador Fuzzy Mamdani implementado em `scikit-fuzzy`**

Período: **2026.1**
---
## Equipe

| #   | Nome completo             | Matrícula       |
| --- | ------------------------- | --------------- |
| 1   | *João Victor Oliveira*    | *(20240008468)* |
| 2   | *Kevin Gabriel Mangueira* | *(20240008000)* |
| 3   | *Luiz Henrique Santos*    | *(20240008261)* |
| 4   | *Victor Gabriel Menezes*  | *(20240008323)* |

---
## Descrição do Domínio

O **FuzzyFit** é um Sistema Baseado em Conhecimento que recebe duas variáveis linguísticas e decide a intensidade recomendada do treino em percentual da frequência cardíaca máxima (% FCmáx).

**Por que fuzzy é a abordagem certa?**  
Conceitos como *fadiga moderada* ou *atleta intermediário* não têm fronteiras nítidas no mundo real.
Um sistema crisp (`if-elif`) forçaria limiares arbitrários — `condicionamento > 7 → avançado` — criando saltos abruptos (*cliff edges*) na saída. 
O controlador fuzzy trata essa imprecisão linguística naturalmente, produzindo uma saída contínua e auditável por qualquer educador físico.

---

## Estrutura do Repositório

```
fuzzyfit/
├── FuzzyFit.ipynb      ← Notebook principal (Google Colab / Jupyter)
└── README.md           ← Este arquivo
```

---

## Variáveis Linguísticas

| Variável                | Tipo        | Universo           | Termos linguísticos                  | Função de pertinência   |
| ----------------------- | ----------- | ------------------ | ------------------------------------ | ----------------------- |
| `nivel_condicionamento` | Antecedente | 0 – 10             | iniciante · intermediário · avançado | trapmf · trimf · trapmf |
| `fadiga_atual`          | Antecedente | 0 – 10 (PSE¹)      | baixa · moderada · alta              | trapmf · trimf · trapmf |
| `intensidade_treino`    | Consequente | 50 – 100 (% FCmáx) | leve · moderada · intensa            | trapmf · trimf · trapmf |

> ¹ PSE = Percepção Subjetiva de Esforço (escala de Borg adaptada, 0–10).

### Funções de Pertinência — Parâmetros

**`nivel_condicionamento`**
```
iniciante     → trapmf [0, 0, 2, 4]
intermediario → trimf  [2, 5, 8]
avancado      → trapmf [6, 8, 10, 10]
```

**`fadiga_atual`**
```
baixa    → trapmf [0, 0, 2, 4]
moderada → trimf  [2, 5, 8]
alta     → trapmf [6, 8, 10, 10]
```

**`intensidade_treino`** (% FCmáx)
```
leve     → trapmf [50, 50, 58, 68]   # Zona 1–2: recuperação / aeróbico leve
moderada → trimf  [62, 74, 86]        # Zona 3–4: aeróbico moderado / limiar
intensa  → trapmf [80, 90, 100, 100]  # Zona 5:   anaeróbico / VO₂ máx
```

---

## Base de Regras Fuzzy (9 regras — cobertura total 3 × 3)

| #  | SE condicionamento É  | E fadiga É | ENTÃO intensidade É  |
|----|-----------------------|------------|----------------------|
| R1 | Iniciante             | Baixa      | **Leve**             |
| R2 | Iniciante             | Moderada   | **Leve**             |
| R3 | Iniciante             | Alta       | **Leve**             |
| R4 | Intermediário         | Baixa      | **Moderada**         |
| R5 | Intermediário         | Moderada   | **Moderada**         |
| R6 | Intermediário         | Alta       | **Leve**             |
| R7 | Avançado              | Baixa      | **Intensa**          |
| R8 | Avançado              | Moderada   | **Moderada**         |
| R9 | Avançado              | Alta       | **Leve**             |

### Matriz de Decisão

```
              fadiga:  BAIXA     MODERADA    ALTA
INICIANTE     →        LEVE      LEVE        LEVE
INTERMEDIÁRIO →        MODERADA  MODERADA    LEVE
AVANÇADO      →        INTENSA   MODERADA    LEVE
```

### Justificativa Biomecânica das Regras

- **Iniciante (R1–R3):** Independentemente da fadiga, limitar a intensidade previne overtraining durante a fase de adaptação inicial. O sistema locomotor ainda está construindo base aeróbica.
- **Intermediário (R4–R6):** Fadiga baixa ou moderada permite treino moderado (zona aeróbica/limiar). Fadiga alta exige redução para garantir recuperação adequada.
- **Avançado (R7–R9):** Atleta bem condicionado com fadiga baixa pode e deve treinar intensamente para manter adaptações de alto nível. Fadiga alta demanda respeito à recuperação mesmo em elite.

---

## Casos de Teste

| Teste | Condicionamento        | Fadiga          | Saída do sistema   | Zona fisiológica                        |
|-------|------------------------|-----------------|--------------------|-----------------------------------------|
| **A** | 9.0 (avançado)         | 1.5 (baixa)     | **92.2% FCmáx**    | Zona 5 — Anaeróbico / VO₂ Máx           |
| **B** | 5.0 (intermediário)    | 5.0 (moderada)  | **74.0% FCmáx**    | Zona 3–4 — Aeróbico Moderado            |
| **C** | 3.5 (iniciante)        | 7.5 (alta)      | **56.8% FCmáx**    | Zona 1–2 — Recuperação Ativa            |

---

## Como Executar

### Opção 1 — Google Colab (recomendado)

1. Faça upload do arquivo `FuzzyFit.ipynb` no [Google Colab](https://colab.research.google.com/).
2. Execute a **primeira célula** (`!pip install scikit-fuzzy`) para instalar a dependência.
3. Execute **todas as células** em ordem (`Runtime → Run All`).

### Opção 2 — Jupyter local

```bash
# 1. Instalar dependências
pip install scikit-fuzzy numpy matplotlib jupyter

# 2. Abrir o notebook
jupyter notebook FuzzyFit.ipynb
```

### Dependências

| Pacote           | Versão testada | Finalidade                                    |
|------------------|----------------|-----------------------------------------------|
| `scikit-fuzzy`   | 0.5.0          | Motor de inferência Mamdani                   |
| `numpy`          | ≥ 1.21         | Universos numéricos e operações               |
| `matplotlib`     | ≥ 3.4          | Visualização das MFs e superfície             |

---

## Comparativo: Regras Crisp (MP1) × Controlador Fuzzy (MP2)

| Aspecto                        | SBC Crisp (Mini-Projeto 1)                         | SBC Fuzzy (Mini-Projeto 2)                                |
|--------------------------------|----------------------------------------------------|-----------------------------------------------------------|
| **Regras**                     | `IF-THEN` com limiares fixos                       | `IF-THEN` com predicados linguísticos graduais            |
| **Fronteiras**                 | Abruptas (*cliff edges*)                           | Suaves — sobreposição de MFs                              |
| **Saída**                      | Rótulo simbólico (`"alto"`, `"leve"`)              | Valor contínuo (ex.: `74.0%`)                             |
| **Expressividade**             | Uma regra, um resultado                            | Múltiplas regras se combinam proporcionalmente            |
| **Manutenção**                 | Cada novo limiar exige nova regra                  | Ajustar uma MF afeta suavemente várias regiões            |
| **Auditabilidade**             | Alta — cada regra é independente                   | Alta — cada regra tem grau de ativação rastreável         |
| **Imprecisão linguística**     | Não tratada nativamente                             | Tratada nativamente via conjuntos fuzzy                   |

### Exemplo concreto de cliff edge evitado

```python
# Versão crisp (MP1 equivalente)
if condicionamento >= 7 and fadiga <= 3:
    intensidade = "intensa"
elif condicionamento >= 4 and fadiga <= 6:
    intensidade = "moderada"
else:
    intensidade = "leve"
# Atleta com cond=6.9 e fadiga=3 → "moderada"
# Atleta com cond=7.0 e fadiga=3 → "intensa"  ← salto abrupto!

# Versão fuzzy (MP2)
# cond=6.9 → avançado(0.45), intermediário(0.55)
# → saída proporcional: ~81% FCmáx (entre moderada e intensa)
# → SEM salto. Resposta gradual.
```

---

