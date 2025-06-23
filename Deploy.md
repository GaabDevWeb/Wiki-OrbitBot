# 🐳 ESTRATÉGIA DE CONTAINERIZAÇÃO + API - ORBITBOT

## 📋 **Visão Geral**

Este documento apresenta a **estratégia completa de containerização e arquitetura de microserviços** para transformar o OrbitBot em uma plataforma escalável e comercialmente viável.

---

## 🎯 **VANTAGENS DO DOCKER + API**

### **✅ Benefícios Técnicos:**
- **Deploy super fácil** - um comando e tá rodando
- **Escalabilidade** - pode rodar múltiplas instâncias
- **Isolamento** - cada cliente tem seu próprio container
- **API REST** - fácil de integrar com frontend
- **Microserviços** - cada funcionalidade separada

### **💰 Benefícios Comerciais:**
- **Deploy em 5 minutos** vs concorrentes que demoram dias
- **Isolamento total** entre clientes
- **Dashboard web** para configuração
- **API para integrações**
- **Escalabilidade infinita**

---

## 🏗️ **ARQUITETURA PROPOSTA**

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Frontend      │    │   API Gateway   │    │   WhatsApp Bot  │
│   (Dashboard)   │◄──►│   (Express)     │◄──►│   (Container)   │
└─────────────────┘    └─────────────────┘    └─────────────────┘
                                │
                                ▼
                       ┌─────────────────┐
                       │   Services      │
                       │   (Containers)  │
                       │   ├─ AI Service │
                       │   ├─ Audio API  │
                       │   ├─ Database   │
                       │   └─ Plugins    │
                       └─────────────────┘
```

### **🔧 Componentes da Arquitetura:**

#### **1. Frontend (Dashboard)**
- **React/Vue.js** para interface web
- **Configuração de personalidades**
- **Visualização de estatísticas**
- **Gestão de clientes**

#### **2. API Gateway**
- **Express.js** com JWT
- **Rate limiting** e autenticação
- **Roteamento** para microserviços
- **Load balancing**

#### **3. WhatsApp Bot**
- **Container isolado** por cliente
- **Venom-bot** para WhatsApp
- **Comunicação** via API Gateway

#### **4. Microserviços**
- **AI Service** - Processamento de IA
- **Audio Service** - Transcrição de áudio
- **Database Service** - PostgreSQL
- **Cache Service** - Redis

---

## 🐳 **DOCKER COMPOSE ESTRUTURA**

```yaml
# docker-compose.yml
version: '3.8'

services:
  # API Gateway
  api-gateway:
    build: ./api-gateway
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - JWT_SECRET=${JWT_SECRET}
      - DATABASE_URL=${DATABASE_URL}
    depends_on:
      - database
      - ai-service
      - audio-service
    networks:
      - orbitbot-network

  # Frontend Dashboard
  frontend:
    build: ./frontend
    ports:
      - "80:80"
    depends_on:
      - api-gateway
    networks:
      - orbitbot-network

  # WhatsApp Bot (um por cliente)
  whatsapp-bot:
    build: ./whatsapp-bot
    environment:
      - CLIENT_ID=${CLIENT_ID}
      - OPENROUTER_API_KEY=${OPENROUTER_API_KEY}
      - API_GATEWAY_URL=http://api-gateway:3000
    volumes:
      - ./sessions:/app/sessions
      - ./audios:/app/audios
    depends_on:
      - api-gateway
    networks:
      - orbitbot-network

  # AI Service (OpenAI/DeepSeek)
  ai-service:
    build: ./ai-service
    environment:
      - OPENROUTER_API_KEY=${OPENROUTER_API_KEY}
      - NODE_ENV=production
    ports:
      - "3001:3001"
    networks:
      - orbitbot-network

  # Audio Service (Whisper)
  audio-service:
    build: ./audio-service
    environment:
      - NODE_ENV=production
    ports:
      - "3002:3002"
    volumes:
      - ./audios:/app/audios
    networks:
      - orbitbot-network

  # Database (PostgreSQL)
  database:
    image: postgres:15
    environment:
      - POSTGRES_DB=orbitbot
      - POSTGRES_USER=orbitbot
      - POSTGRES_PASSWORD=${DB_PASSWORD}
    volumes:
      - postgres_data:/var/lib/postgresql/data
    ports:
      - "5432:5432"
    networks:
      - orbitbot-network

  # Redis (Cache)
  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    networks:
      - orbitbot-network

  # Nginx (Reverse Proxy)
  nginx:
    image: nginx:alpine
    ports:
      - "443:443"
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
      - ./ssl:/etc/nginx/ssl
    depends_on:
      - frontend
      - api-gateway
    networks:
      - orbitbot-network

volumes:
  postgres_data:

networks:
  orbitbot-network:
    driver: bridge
```

---

## 🔧 **REFATORAÇÃO PARA MICROSERVIÇOS**

### **1. API Gateway (`/api-gateway`)**

```javascript
// app.js
const express = require('express');
const jwt = require('jsonwebtoken');
const rateLimit = require('express-rate-limit');

const app = express();

// Rate limiting
const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutos
  max: 100 // limite por IP
});

app.use(limiter);
app.use(express.json());

// Autenticação JWT
const authenticateToken = (req, res, next) => {
  const token = req.headers['authorization']?.split(' ')[1];
  
  if (!token) {
    return res.sendStatus(401);
  }

  jwt.verify(token, process.env.JWT_SECRET, (err, user) => {
    if (err) return res.sendStatus(403);
    req.user = user;
    next();
  });
};

// Rotas da API
app.post('/api/clients/:clientId/messages', authenticateToken, async (req, res) => {
  try {
    const { clientId } = req.params;
    const { message, type } = req.body;
    
    // Roteia mensagem para o bot correto
    const response = await fetch(`${process.env.AI_SERVICE_URL}/api/chat`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ clientId, message, type })
    });
    
    const result = await response.json();
    res.json(result);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

app.get('/api/clients/:clientId/stats', authenticateToken, async (req, res) => {
  try {
    const { clientId } = req.params;
    
    // Busca estatísticas do cliente
    const stats = await getClientStats(clientId);
    res.json(stats);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

app.post('/api/clients/:clientId/configure', authenticateToken, async (req, res) => {
  try {
    const { clientId } = req.params;
    const { personality, settings } = req.body;
    
    // Configura cliente
    await updateClientConfig(clientId, personality, settings);
    res.json({ success: true });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
  console.log(`API Gateway rodando na porta ${PORT}`);
});
```

### **2. AI Service (`/ai-service`)**

```javascript
// app.js
const express = require('express');
const axios = require('axios');

const app = express();
app.use(express.json());

// Processa mensagens com IA
app.post('/api/chat', async (req, res) => {
  try {
    const { clientId, message, type } = req.body;
    
    // Busca configuração do cliente
    const config = await getClientConfig(clientId);
    
    // Processa com IA
    const response = await axios.post(
      'https://openrouter.ai/api/v1/chat/completions',
      {
        model: config.model,
        messages: [
          { role: 'system', content: config.personality },
          { role: 'user', content: message }
        ],
        temperature: config.temperature,
        max_tokens: config.maxTokens
      },
      {
        headers: {
          'Authorization': `Bearer ${process.env.OPENROUTER_API_KEY}`,
          'Content-Type': 'application/json'
        }
      }
    );
    
    const aiResponse = response.data.choices[0].message.content;
    
    // Salva no histórico
    await saveMessage(clientId, message, 'user');
    await saveMessage(clientId, aiResponse, 'assistant');
    
    res.json({ response: aiResponse });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// Melhora transcrições
app.post('/api/improve-transcription', async (req, res) => {
  try {
    const { transcription, context } = req.body;
    
    const improved = await improveTranscriptionWithAI(transcription, context);
    res.json({ improved });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

const PORT = process.env.PORT || 3001;
app.listen(PORT, () => {
  console.log(`AI Service rodando na porta ${PORT}`);
});
```

### **3. Audio Service (`/audio-service`)**

```javascript
// app.js
const express = require('express');
const multer = require('multer');
const { execFile } = require('child_process');
const path = require('path');

const app = express();

// Configuração do multer para upload
const storage = multer.diskStorage({
  destination: (req, file, cb) => {
    cb(null, '/app/audios');
  },
  filename: (req, file, cb) => {
    const uniqueSuffix = Date.now() + '-' + Math.round(Math.random() * 1E9);
    cb(null, `audio-${uniqueSuffix}.ogg`);
  }
});

const upload = multer({ storage });

// Transcreve áudio
app.post('/api/transcribe', upload.single('audio'), async (req, res) => {
  try {
    const { file } = req;
    const { model = 'medium', language = 'pt' } = req.body;
    
    if (!file) {
      return res.status(400).json({ error: 'Arquivo de áudio não fornecido' });
    }
    
    // Transcreve com Whisper
    const transcription = await transcribeAudio(file.path, model, language);
    
    // Remove arquivo temporário
    fs.unlinkSync(file.path);
    
    res.json({ transcription });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

async function transcribeAudio(filePath, model, language) {
  return new Promise((resolve, reject) => {
    execFile('python', [
      'transcrever_audio.py',
      filePath,
      model,
      '0.0',
      language
    ], (error, stdout, stderr) => {
      if (error) {
        reject(error);
        return;
      }
      
      const lines = stdout.split('\n').filter(Boolean);
      const transcription = lines[0];
      resolve(transcription);
    });
  });
}

const PORT = process.env.PORT || 3002;
app.listen(PORT, () => {
  console.log(`Audio Service rodando na porta ${PORT}`);
});
```

### **4. WhatsApp Bot (`/whatsapp-bot`)**

```javascript
// app.js
const venom = require('venom-bot');
const axios = require('axios');

class ClientBot {
  constructor(clientId) {
    this.clientId = clientId;
    this.apiUrl = process.env.API_GATEWAY_URL;
    this.sessionName = `session_${clientId}`;
  }
  
  async start() {
    try {
      this.client = await venom.create({
        session: this.sessionName,
        multidevice: true,
        headless: true
      });
      
      this.setupEventHandlers();
      console.log(`Bot iniciado para cliente ${this.clientId}`);
    } catch (error) {
      console.error(`Erro ao iniciar bot para cliente ${this.clientId}:`, error);
    }
  }
  
  setupEventHandlers() {
    this.client.onMessage(async (message) => {
      if (!message.from.includes('@c.us') || message.isGroupMsg) return;
      
      try {
        // Processa mensagem de texto
        if (message.body) {
          await this.handleTextMessage(message);
        }
        
        // Processa mensagem de áudio
        if (message.type === 'audio' || message.type === 'ptt') {
          await this.handleAudioMessage(message);
        }
      } catch (error) {
        console.error('Erro ao processar mensagem:', error);
      }
    });
  }
  
  async handleTextMessage(message) {
    try {
      // Envia para API Gateway
      const response = await axios.post(
        `${this.apiUrl}/api/clients/${this.clientId}/messages`,
        {
          message: message.body,
          type: 'text',
          from: message.from
        },
        {
          headers: {
            'Authorization': `Bearer ${process.env.API_TOKEN}`,
            'Content-Type': 'application/json'
          }
        }
      );
      
      const aiResponse = response.data.response;
      
      // Envia resposta
      await this.client.sendText(message.from, aiResponse);
      
    } catch (error) {
      console.error('Erro ao processar mensagem de texto:', error);
    }
  }
  
  async handleAudioMessage(message) {
    try {
      // Decripta áudio
      const buffer = await this.client.decryptFile(message);
      
      // Salva arquivo temporário
      const fileName = `audio_${Date.now()}.ogg`;
      const filePath = `/app/audios/${fileName}`;
      fs.writeFileSync(filePath, buffer);
      
      // Transcreve áudio
      const transcriptionResponse = await axios.post(
        `${process.env.AUDIO_SERVICE_URL}/api/transcribe`,
        {
          audio: fs.createReadStream(filePath),
          model: 'medium',
          language: 'pt'
        },
        {
          headers: {
            'Content-Type': 'multipart/form-data'
          }
        }
      );
      
      const transcription = transcriptionResponse.data.transcription;
      
      // Processa transcrição como mensagem normal
      await this.handleTextMessage({
        ...message,
        body: transcription
      });
      
      // Remove arquivo temporário
      fs.unlinkSync(filePath);
      
    } catch (error) {
      console.error('Erro ao processar mensagem de áudio:', error);
    }
  }
}

// Inicia bot para cliente específico
const clientId = process.env.CLIENT_ID;
if (!clientId) {
  console.error('CLIENT_ID não definido');
  process.exit(1);
}

const bot = new ClientBot(clientId);
bot.start();
```

---

## 🚀 **DEPLOY SIMPLIFICADO**

### **Script de Deploy:**

```bash
#!/bin/bash
# deploy_orbitbot.sh

set -e

echo "🚀 Deployando OrbitBot com Docker..."

# Verifica se Docker está instalado
if ! command -v docker &> /dev/null; then
    echo "❌ Docker não está instalado. Instalando..."
    curl -fsSL https://get.docker.com -o get-docker.sh
    sh get-docker.sh
    sudo usermod -aG docker $USER
    echo "✅ Docker instalado. Reinicie o terminal e execute novamente."
    exit 1
fi

# Verifica se Docker Compose está instalado
if ! command -v docker-compose &> /dev/null; then
    echo "❌ Docker Compose não está instalado. Instalando..."
    sudo curl -L "https://github.com/docker/compose/releases/download/v2.20.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
    sudo chmod +x /usr/local/bin/docker-compose
fi

# Cria diretório do projeto
PROJECT_DIR="/opt/orbitbot"
sudo mkdir -p $PROJECT_DIR
cd $PROJECT_DIR

# Clona repositório
if [ ! -d ".git" ]; then
    echo "📥 Clonando repositório..."
    git clone https://github.com/seu-usuario/OrbitBot.git .
fi

# Configura variáveis de ambiente
if [ ! -f ".env" ]; then
    echo "⚙️ Configurando variáveis de ambiente..."
    cp .env.example .env
    
    # Gera chaves secretas
    JWT_SECRET=$(openssl rand -hex 32)
    DB_PASSWORD=$(openssl rand -hex 16)
    
    # Atualiza .env
    sed -i "s/JWT_SECRET=.*/JWT_SECRET=$JWT_SECRET/" .env
    sed -i "s/DB_PASSWORD=.*/DB_PASSWORD=$DB_PASSWORD/" .env
    
    echo "🔑 Chaves geradas automaticamente"
fi

# Constrói e inicia containers
echo "🐳 Construindo containers..."
docker-compose build

echo "🚀 Iniciando serviços..."
docker-compose up -d

# Aguarda serviços ficarem prontos
echo "⏳ Aguardando serviços ficarem prontos..."
sleep 30

# Verifica status
echo "📊 Status dos serviços:"
docker-compose ps

echo ""
echo "✅ OrbitBot deployado com sucesso!"
echo "🌐 Dashboard: http://$(hostname -I | awk '{print $1}')"
echo "📱 API: http://$(hostname -I | awk '{print $1}'):3000"
echo ""
echo "📝 Próximos passos:"
echo "1. Acesse o dashboard"
echo "2. Configure seu primeiro cliente"
echo "3. Escaneie o QR Code do WhatsApp"
echo ""
echo "🔧 Comandos úteis:"
echo "  docker-compose logs -f    # Ver logs"
echo "  docker-compose down       # Parar serviços"
echo "  docker-compose restart    # Reiniciar serviços"
```

### **Comando único para deploy:**

```bash
curl -sSL https://raw.githubusercontent.com/seu-usuario/OrbitBot/main/deploy.sh | bash
```

---

## 💰 **VANTAGENS PARA VENDA**

### **1. Deploy Super Rápido:**
- **Cliente paga → Deploy automático em 5 minutos**
- **Cada cliente tem seu próprio container isolado**
- **Zero conflitos entre clientes**
- **Provisionamento automático**S

### **2. Escalabilidade:**
- **Pode ter 100+ clientes no mesmo servidor**
- **Cada cliente pode ter configurações diferentes**
- **Fácil de migrar para servidores maiores**
- **Load balancing automático**

### **3. Dashboard Web:**
- **Cliente acessa dashboard para configurar**
- **Vê estatísticas em tempo real**
- **Pode alterar personalidade via web**
- **Interface intuitiva e responsiva**

### **4. API para Integrações:**
- **Cliente pode integrar com outros sistemas**
- **Webhooks para notificações**
- **API para desenvolvedores**
- **Documentação completa**

---

## 🎯 **PRÓXIMOS PASSOS**

### **Fase 1: Refatoração (1-2 semanas)**
1. **Separar em microserviços**
   - API Gateway
   - AI Service
   - Audio Service
   - WhatsApp Bot

2. **Criar Dockerfiles**
   - Dockerfile para cada serviço
   - Multi-stage builds para otimização
   - Health checks

3. **Implementar API Gateway**
   - Autenticação JWT
   - Rate limiting
   - Roteamento inteligente

4. **Testar containerização**
   - Testes de integração
   - Testes de performance
   - Testes de escalabilidade

### **Fase 2: Dashboard (1 semana)**
1. **Frontend simples (React/Vue)**
   - Interface de configuração
   - Visualização de estatísticas
   - Gestão de clientes

2. **Configuração de personalidades**
   - Editor de prompts
   - Templates pré-definidos
   - Teste de personalidades

3. **Visualização de estatísticas**
   - Gráficos em tempo real
   - Métricas de performance
   - Relatórios exportáveis

4. **Gestão de clientes**
   - CRUD de clientes
   - Configurações por cliente
   - Billing e pagamentos

### **Fase 3: Automação (1 semana)**
1. **Script de deploy automático**
   - Deploy com um comando
   - Configuração automática
   - Rollback em caso de erro

2. **Provisionamento de novos clientes**
   - Criação automática de containers
   - Configuração inicial
   - Onboarding automatizado

3. **Backup automático**
   - Backup diário do banco
   - Backup das configurações
   - Restore automático

4. **Monitoramento**
   - Health checks
   - Alertas automáticos
   - Métricas de performance

### **🎯 SEMANA 5: Validação**
- [ ] Teste com 3-5 clientes reais
- [ ] Coletar feedback
- [ ] Ajustar baseado no uso
- [ ] Preparar para escala

---

## 🎉 **CONCLUSÃO**

### **✅ O que conseguimos:**
- **Arquitetura escalável** e moderna
- **Deploy simplificado** para clientes
- **Isolamento total** entre instâncias
- **API completa** para integrações
- **Dashboard web** para configuração

### **🚀 Próximos passos:**
1. **Executar refatoração** em 1-2 semanas
2. **Testar com clientes reais** na semana 3
3. **Validar mercado** na semana 4-5
4. **Escalar baseado no feedback**

### **💰 Potencial:**
- **Deploy em 5 minutos** = vantagem competitiva enorme
- **Isolamento total** = segurança e confiabilidade
- **API completa** = integrações infinitas
- **Dashboard web** = experiência superior

**Esta arquitetura vai transformar o OrbitBot em uma plataforma profissional e comercialmente viável!** 🚀

---

*Estratégia de Containerização criada em: 20/06/2025*  
*Status: 🚀 PRONTO PARA EXECUÇÃO*  
*Meta: Deploy automático em 5 minutos* 