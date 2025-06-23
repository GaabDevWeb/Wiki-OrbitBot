# 🚀 Quick Start - Sistema INsanely-Fast Whisper

## ⚡ Instalação Rápida

### Windows
```cmd
install_insanely_fast.bat
```

### Linux/Mac
```bash
chmod +x install_insanely_fast.sh
./install_insanely_fast.sh
```

## 🎯 Configuração Automática

O sistema já vem configurado para máxima performance:
- ✅ Sistema insanely-fast **ATIVADO** por padrão
- ✅ Modelo: `openai/whisper-large-v3`
- ✅ Batch size: 24
- ✅ Flash Attention 2: Ativo (se GPU disponível)

## 📋 Comandos Essenciais

```bash
# Verificar status
/audio status

# Ver estatísticas
/audio stats

# Testar performance
/audio benchmark

# Configurar para máxima velocidade
/audio insanely on
/audio flash on
/audio batch 24
/audio modelo openai/whisper-large-v3
```

## ⚡ Performance Esperada

- **150 minutos de áudio em 98 segundos** (com Flash Attention 2)
- **10x mais rápido** que Whisper padrão
- **Qualidade superior** com Large v3

## 🔧 Se algo der errado

1. **Erro de CUDA**: Sistema funciona sem GPU, só mais lento
2. **Erro de Flash Attention**: Use `/audio flash off`
3. **Out of Memory**: Use `/audio batch 8`
4. **Fallback automático**: Sistema legado ativa automaticamente

## 🎉 Pronto!

Agora você tem transcrições **ULTRA RÁPIDAS**! ⚡

---

**💡 Dica**: Use `/audio benchmark` para ver a performance atual do seu sistema. 