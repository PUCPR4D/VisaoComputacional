# Fluxograma de Decisão: Transfer Learning e Fine-tuning

## Contexto

Agora que você conhece várias arquiteturas (CNN, ResNet, ViT), como escolher a melhor para seu problema?

Este exercício é sobre **aprender a tomar decisões** em Deep Learning.

## Objetivo

Criar um **fluxograma de decisão** que orienta:

1. **Qual dataset usar para pré-training?** (ImageNet, COCO, etc.)
2. **Qual arquitetura escolher?** (ResNet, EfficientNet, ViT, etc.)
3. **Quais camadas congelar?** (feature extractor vs. classifier)
4. **Qual learning rate usar?** (discriminative learning rates)
5. **Quando parar de treinar?** (early stopping)

## O Que Você Vai Entregar

Arquivo `resposta.md` contendo:

1. **Fluxograma visual** (ASCII art ou descrição) com decisões principais
2. **Exemplos práticos**: "Se meu problema é X, então..." 
3. **Trade-offs documentados**: Speed vs. Accuracy, Memory vs. Performance
4. **Validação cruzada**: Qual é a melhor prática?

## Estrutura do `resposta.md`

```markdown
# Resposta: Fluxograma de Decisão TL e Fine-tuning

## 1. Contexto
[Resumo do que você aprendeu em módulos anteriores]

## 2. Fluxograma Principal
[Descrição do fluxograma: decisões chave]

## 3. Exemplos Práticos
[Cenários reais e como aplicar]

## 4. Trade-offs e Considerações
[Tabela ou discussão]

## 5. Validação em Produção
[Como verificar que seu modelo está funcionando]
```

---

# ✅ RESPOSTA: Fluxograma de Decisão TL e Fine-tuning

## 1. Fluxograma de Decisão Principal

```
┌─────────────────────────────────────────────────┐
│ COMECE AQUI: Tem dados suficientes?             │
│ (Regra: >10K imagens = muitos; <1K = poucos)   │
└──────────────────┬──────────────────────────────┘
                   │
        ┌──────────┴──────────┐
        │                     │
    Muitos Dados         Poucos Dados
    (>10K)              (<1K)
        │                     │
        ▼                     ▼
┌──────────────┐       ┌─────────────────┐
│ Use Transfer │       │ OBRIGATÓRIO:    │
│ Learning +   │       │ Transfer        │
│ Fine-tune    │       │ Learning!       │
└──────┬───────┘       │ Sem exceções    │
       │               └────────┬────────┘
       │                        │
       └────────────┬───────────┘
                    │
        ┌───────────▼──────────┐
        │ Qual é seu problema? │
        └───────────┬──────────┘
                    │
        ┌───────────┴──────────────────┬──────────┐
        │                              │          │
   Classificação              Detecção/    Segmentação
                            Segmentação    ou 3D
        │                         │            │
        ▼                         ▼            ▼
    ResNet50            Faster R-CNN      U-Net ou
    EfficientNet        Mask R-CNN        DeepLabv3
    ViT (muitos         YOLO              PointNet
    dados)              (rápido)          (3D)
        │                    │                │
        └────────────┬───────┴────────┬──────┘
                     │                │
         ┌───────────▼───┐   ┌────────▼────┐
         │ Congelar      │   │ Treinar     │
         │ backbone      │   │ tudo        │
         │ (1-3 epochs)  │   │ (5-20)      │
         └───────────────┘   └─────────────┘
```

## 2. Tabela de Decisão Rápida

| Cenário | Arquitetura | Dados | Congelar | Learning Rate | Epochs |
|---------|-------------|-------|----------|---------------|--------|
| **Classificação rápida** | ResNet50 | 1-10K | Sim (80%) | 0.001 | 3-5 |
| **Classificação alta-acurácia** | EfficientNet-b4 | 10K+ | Parcial | 0.0001 | 10-20 |
| **Detecção em tempo real** | YOLO-v5 | 1K+ | Sim | 0.001 | 5-10 |
| **Detecção alta-precisão** | Faster R-CNN | 5K+ | Não | 0.0005 | 15+ |
| **Segmentação semântica** | DeepLabv3 | 1K+ | Sim | 0.001 | 10-20 |
| **Segmentação instância** | Mask R-CNN | 5K+ | Parcial | 0.0005 | 20+ |
| **Visão 3D** | PointNet | 1K+ objetos | Parcial | 0.001 | 50-100 |

## 3. Exemplos Práticos

### Exemplo 1: Classificação de Plantas (200 imagens)
```
Q: Tenho 200 imagens de diferentes plantas. Como treino?

A:
✅ Dados: POUCOS (200 < 1K)
✅ Problema: Classificação
✅ Arquitetura: ResNet50 (pré-treinada ImageNet)
✅ Estratégia:
   - Congelar backbone inteiro (ImageNet features são genéricas)
   - Treinar apenas última camada (Linear(2048 → num_classes))
   - Learning rate: 0.001
   - Epochs: 3-5 (evite overfitting)
   - Data augmentation: AGRESSIVA (flip, rotate, color jitter)
   
✅ Resultado esperado: 85-95% accuracy com pouco dado
```

### Exemplo 2: Detecção de Defeitos em Fábrica (5K imagens)
```
Q: Tenho 5K imagens de peças com/sem defeitos. Quero detecção em tempo real.

A:
✅ Dados: MÉDIOS (5K)
✅ Problema: Detecção rápida
✅ Arquitetura: YOLO-v5 (rápido, pré-treinado COCO)
✅ Estratégia:
   - Fine-tune tudo (dados suficientes)
   - Learning rate schedule: 0.001 → 0.0001 ao longo epochs
   - Epochs: 50-100 (muitos dados)
   - Validação: Usar mAP@0.5, mAP@0.5:0.95
   
✅ Resultado esperado: 60-75 mAP com pouco tempo de treinamento
```

### Exemplo 3: Segmentação Médica (100 scans com anotações)
```
Q: Tenho 100 CT scans com tumores anotados. Como segmento?

A:
✅ Dados: POUCOS (100 amostras) MAS cada scan tem 100+ fatias
✅ Problema: Segmentação semântica (tumor vs. não-tumor)
✅ Arquitetura: U-Net (pré-treinada ImageNet backbone)
✅ Estratégia:
   - Congelar encoder (parte de extração de features)
   - Treinar decoder (parte de upsampling)
   - Data augmentation: 3D (rot, shear, elastic deformations)
   - Loss: Dice Loss + Cross Entropy (mais adequado para segmentação)
   - Learning rate: 0.0001 (conservador com dados médicos)
   
✅ Resultado esperado: 85-92% Dice Score
```

## 4. Estratégia de Congelamento (Freeze Strategy)

```
STRATEGY 1: Freeze Tudo (Poucos Dados)
┌─────────────────────────────────┐
│ backbone (frozen) ✓             │
│ └─ layer1 (frozen) ✓            │
│ └─ layer2 (frozen) ✓            │
│ └─ layer3 (frozen) ✓            │
│ └─ layer4 (frozen) ✓            │
├─────────────────────────────────┤
│ head (trainable) ✗ ← TREINE     │
│ └─ fc1 (trainable) ✗            │
│ └─ fc2 (trainable) ✗            │
└─────────────────────────────────┘

STRATEGY 2: Progressive Unfreezing (Médio)
┌─────────────────────────────────┐
│ backbone                        │
│ └─ layer1 (frozen) ✓            │
│ └─ layer2 (frozen) ✓            │
│ └─ layer3 (trainable) ✗ ← LIBERE│
│ └─ layer4 (trainable) ✗         │
├─────────────────────────────────┤
│ head (trainable) ✗              │
└─────────────────────────────────┘

STRATEGY 3: Discriminative LR (Muito Dado)
┌─────────────────────────────────┐
│ backbone                        │
│ └─ early layers: lr=0.00001 ✗   │
│ └─ mid layers: lr=0.0001 ✗      │
│ └─ late layers: lr=0.001 ✗      │
├─────────────────────────────────┤
│ head: lr=0.01 ✗                 │
└─────────────────────────────────┘
```

## 5. Learning Rate Selection

```
Sua regra de ouro:

1. Se congelou backbone:
   → Learning rate = 0.001 (ResNet, CNN)
   → Learning rate = 0.0005 (ViT, transformers)

2. Se fine-tuning tudo:
   → Learning rate = 0.0001 (conservador)
   → Learning rate = 0.00005 (risco de underfitting)

3. Se discriminative LR:
   → Camadas cedo: 0.00001 (mudar pouco)
   → Camadas tarde: 0.0001 (mudar mais)
   → Head: 0.001 (mudar bastante)

Técnica: Learning Rate Warmup
- Comece com lr=0
- Aumente gradualmente até lr_max nos primeiros epochs
- Isso evita instabilidade no início
```

## 6. Early Stopping e Validação

```python
# Pseudocódigo para boas práticas

best_val_loss = float('inf')
patience = 10  # Parar se não melhora em 10 epochs
patience_counter = 0

for epoch in range(max_epochs):
    train_loss = train_one_epoch()
    val_loss = validate()
    
    if val_loss < best_val_loss:
        best_val_loss = val_loss
        save_model()  # Salve o melhor modelo
        patience_counter = 0
    else:
        patience_counter += 1
    
    if patience_counter >= patience:
        print("Early stopping!")
        break  # Parou de melhorar
    
    # Learning rate scheduling
    if epoch == half_way:
        lr = lr * 0.1  # Reduza lr na metade do caminho
```

## 7. Métricas de Avaliação por Problema

| Problema | Métrica Primária | Métrica Secundária | Baseline |
|----------|------------------|-------------------|----------|
| Classificação | Accuracy | F1-score, Precision, Recall | Random: 1/num_classes |
| Detecção | mAP@0.5 | mAP@0.5:0.95 | 0 (nenhuma detecção) |
| Segmentação | Dice Score | IoU, Specificity | Random: 0.5 |
| 3D | Chamfer Distance | Earth Mover Distance | - |

## 8. Erros Comuns e Como Evitar

| Erro | Causa | Solução |
|------|-------|--------|
| **Overfitting com poucos dados** | Learning rate muito alta | Reduzir lr, aumentar regularização |
| **Modelo não converge** | Learning rate muito baixa | Aumentar lr de 1e-5 para 1e-3 |
| **Loss oscilante** | Instabilidade | Usar learning rate warmup + scheduler |
| **Congelou demais** | Backbone muito restritivo | Liberar últimas camadas |
| **Treinou tudo** | Sem pré-training benefit | Recongelar backbone e recomece |
| **Train/val gap grande** | Overfitting severo | Aumentar augmentation, reduzir modelo |

## 9. Checklist de Produção

Antes de deploy, verifique:

- [ ] Accuracy/mAP ≥ baseline aceitável
- [ ] Train/val gap < 5% (não está overfitting)
- [ ] Testou em dados out-of-distribution (novo domínio)?
- [ ] Model explainability: consegue entender predições?
- [ ] Inference time aceitável? (<100ms para time-critical)
- [ ] Tratou class imbalance? (weighted loss ou resampling)
- [ ] Validação cruzada k-fold? (não só train/val split)
- [ ] Documentou hiperparâmetros? (reprodutibilidade)

## 10. Conclusão

**Sua estratégia em 3 passos:**

1. **Diagnóstico:** Identifique quantidade de dados e problema
2. **Seleção:** Escolha arquitetura e estratégia de congelamento
3. **Iteração:** Valide, ajuste learning rate, repita até convergir

**Lembre-se:**
- Transfer Learning é OBRIGATÓRIO com <1K imagens
- Fine-tuning é opcional com >10K imagens
- Sempre valide em dados separados (test set)
- Documentacao é tão importante quanto código

---

## Erros Comuns

### ❌ "Sempre use ViT"
**Realidade:** ViT é ótimo com 10K+ imagens. Com poucos dados, ResNet é melhor.

### ❌ "Mais camadas = melhor"
**Realidade:** Mais camadas podem levar a overfitting com poucos dados.

### ❌ "Fine-tune tudo desde o início"
**Realidade:** Comece com camadas congeladas, depois desconele gradualmente.

## Extensões Sugeridas

- [ ] Pesquise papers de comparação de arquiteturas (e.g., EfficientNet paper)
- [ ] Implemente discriminative learning rates
- [ ] Teste múltiplos pré-trainings (ImageNet vs. CityScapes)

---

**Bom trabalho! 📚**
