# 🚀 Sistema INsanely-Fast Whisper para OrbitBot

## ⚡ Performance Absurda

O OrbitBot agora usa o sistema **INsanely-Fast Whisper** baseado no [repositório oficial](https://github.com/Vaibhavs10/insanely-fast-whisper) que oferece:

- **150 minutos de áudio em 98 segundos** (com Flash Attention 2)
- **10x mais rápido** que o Whisper padrão
- **Qualidade superior** com Whisper Large v3
- **Flash Attention 2** para máxima aceleração
- **Batching inteligente** para otimização

## 🛠️ Instalação

### 1. Instalação Automática (Recomendado)

```bash
chmod +x install_insanely_fast.sh
./install_insanely_fast.sh
```

### 2. Instalação Manual

```bash
# Dependências básicas
pip3 install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu121

# Sistema insanely-fast
pip3 install transformers>=4.35.0
pip3 install optimum>=1.15.0
pip3 install accelerate>=0.24.0

# Flash Attention 2 (se tiver GPU NVIDIA)
pip3 install flash-attn>=2.3.0 --no-build-isolation

# Dependências de áudio
pip3 install librosa>=0.10.0 soundfile>=0.12.0 numpy>=1.24.0
```

## 🎯 Configuração Padrão

O sistema já vem configurado para máxima performance:

- ✅ **Sistema insanely-fast ativado por padrão**
- ✅ **Modelo:** `openai/whisper-large-v3`
- ✅ **Batch size:** 24
- ✅ **Flash Attention 2:** Ativo (se GPU disponível)

## 📋 Comandos Admin

### Sistema INsanely-Fast
```
/audio insanely on/off     - Habilita/desabilita sistema
/audio flash on/off        - Habilita/desabilita Flash Attention 2
/audio batch [1-100]       - Altera batch size
/audio modelo [nome]       - Altera modelo
/audio benchmark           - Testa performance
```

### Estatísticas
```
/audio stats               - Estatísticas completas
/audio status              - Status atual do sistema
```

### Modos Legados (Fallback)
```
/audio modo rapido         - Modo rápido (tiny model)
/audio modo preciso        - Modo preciso (medium model)
/audio modo alternar       - Alterna modos
/audio modo info           - Informações do modo
```

## 🚀 Modelos Disponíveis

### INsanely-Fast (Recomendados)
- `openai/whisper-large-v3` - **Melhor qualidade e velocidade**
- `openai/whisper-large-v2` - Alta qualidade
- `openai/whisper-medium` - Equilibrado
- `distil-whisper/large-v2` - Rápido e eficiente
- `distil-whisper/medium` - Velocidade máxima

### Legados (Fallback)
- `tiny`, `base`, `small`, `medium`, `large`

## ⚡ Configuração para Máxima Performance

```bash
# Ativa sistema insanely-fast
/audio insanely on

# Ativa Flash Attention 2 (se tiver GPU NVIDIA)
/audio flash on

# Define batch size otimizado
/audio batch 24

# Usa melhor modelo
/audio modelo openai/whisper-large-v3
```

## 📊 Benchmarks Esperados

| Configuração | Tempo (150min áudio) | Velocidade |
|--------------|---------------------|------------|
| **INsanely-Fast + Flash Attention 2** | **98 segundos** | **10x mais rápido** |
| INsanely-Fast sem Flash | ~3 minutos | 5x mais rápido |
| Whisper padrão | ~15 minutos | Base |

## 🔧 Troubleshooting

### Erro: "CUDA not available"
- Instale drivers NVIDIA atualizados
- Verifique se CUDA está instalado
- O sistema funciona sem GPU, mas mais lento

### Erro: "Flash Attention not available"
- Instale com: `pip3 install flash-attn>=2.3.0 --no-build-isolation`
- Verifique se tem GPU NVIDIA
- Sistema funciona sem Flash Attention

### Erro: "Out of Memory"
- Reduza batch size: `/audio batch 8`
- Use modelo menor: `/audio modelo openai/whisper-medium`
- Feche outros programas que usam GPU

### Fallback Automático
Se o sistema insanely-fast falhar, ele automaticamente usa o sistema legado como fallback.

## 🎉 Benefícios

1. **Velocidade Absurda**: 10x mais rápido que Whisper padrão
2. **Qualidade Superior**: Whisper Large v3 por padrão
3. **Flash Attention 2**: Aceleração máxima com GPU
4. **Batching Inteligente**: Otimização automática
5. **Fallback Robusto**: Sistema legado como backup
6. **Compatibilidade**: Funciona com e sem GPU

## 🔄 Migração

O sistema é **compatível com versões anteriores**:
- Sistema legado ainda funciona
- Fallback automático em caso de erro
- Configurações preservadas
- Sem quebra de funcionalidade

---

**🎯 Resultado**: Transcrições ULTRA RÁPIDAS com qualidade superior! ⚡ 