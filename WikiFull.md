        # 🛠️ Wiki Técnica OrbitBot (Nível Engenharia)

        > **Manual técnico avançado do OrbitBot: arquitetura, algoritmos, fluxos, decisões e exemplos reais.**

        ---

        ## Sumário Ultra Detalhado

        1. [Introdução e Filosofia de Design](#1-introducao-e-filosofia-de-design)
        2. [Arquitetura Geral e Diagrama de Módulos](#2-arquitetura-geral-e-diagrama-de-modulos)
        3. [Ciclo de Vida de uma Mensagem](#3-ciclo-de-vida-de-uma-mensagem)
        4. [Fluxo Interno: Texto vs Áudio](#4-fluxo-interno-texto-vs-audio)
        5. [Estrutura dos Módulos](#5-estrutura-dos-modulos)
        6. [Exemplo: Caminho Completo de uma Mensagem](#6-exemplo-caminho-completo-de-uma-mensagem)
        7. [Edge Cases e Resiliência](#7-edge-cases-e-resiliencia)
        8. [Detalhes de Implementação: Código Real](#8-detalhes-de-implementacao-codigo-real)
        9. [Banco de Dados e ORM](#9-banco-de-dados-e-orm)
        10. [Sistema de Fila de Mensagens](#11-sistema-de-fila-de-mensagens)
        11. [Sistema de IA Avançado](#13-sistema-de-ia-avancado)
        12. [Sistema de Plugins Avançado](#15-sistema-de-plugins-avancado)
        13. [Sistema de Áudio: Integração Python, Estratégias, Timeouts, Fallbacks](#16-sistema-de-audio-integracao-python-estrategias-timeouts-fallbacks)
        14. [Tratamento de Erros: Propagação, Auto-Recuperação, Circuit Breaker](#17-tratamento-de-erros-propagacao-auto-recuperacao-circuit-breaker)
        15. [Testes: Unitários, Integração, Stress, Coverage](#18-testes-unitarios-integracao-stress-coverage)
        16. [DevOps: CI/CD, Observabilidade, Monitoramento](#19-devops-cicd-observabilidade-monitoramento)
        17. [Implementação de Testes Automatizados](#20-implementacao-de-testes-automatizados)
        18. [Guia de Implementação de Testes Automatizados](#21-guia-de-implementacao-de-testes-automatizados)
        19. [Conclusão e Próximos Passos](#22-conclusao-e-proximos-passos)

        ---

        ## 1. Introdução e Filosofia de Design

        O OrbitBot foi projetado seguindo princípios de **engenharia de software moderna**, com foco em:

        - **Modularidade**: Cada funcionalidade é um módulo independente
        - **Resiliência**: Sistema nunca trava, sempre há fallback
        - **Performance**: Otimizações em múltiplas camadas
        - **Observabilidade**: Logs detalhados e métricas em tempo real
        - **Escalabilidade**: Arquitetura preparada para crescimento

        ### Fluxo Interno de Mensagens

        ```mermaid
        sequenceDiagram
            participant W as WhatsApp
            participant B as Bot Core
            participant Q as Message Queue
            participant A as Audio Processor
            participant I as AI System
            participant D as Database
            participant H as Humanizer

            W->>B: Mensagem Recebida
            B->>B: Validação Inicial
            alt É Áudio
                B->>A: Processa Áudio
                A->>A: Transcrição Whisper
                A->>I: Melhoria com IA
                A->>B: Texto Processado
            end
            B->>Q: Adiciona à Fila
            Q->>D: Busca Histórico
            D->>Q: Retorna Contexto
            Q->>I: Gera Resposta
            I->>H: Humaniza Resposta
            H->>W: Envia Mensagem
            Q->>D: Atualiza Histórico
        ```

        ### Estrutura dos Módulos

        ```mermaid
        graph TD
            subgraph "Core Modules"
                A[bot.js] --> B[queue.js]
                B --> C[openai.js]
                C --> D[aiConfig.js]
            end
            
            subgraph "Processing Modules"
                E[audioProcessor.js] --> F[humanizer.js]
                G[pluginSystem.js] --> H[performance.js]
            end
            
            subgraph "Data Layer"
                I[database/db.js] --> J[backup.js]
                K[logger.js] --> L[treinamento.js]
            end
            
            A --> E
            A --> G
            A --> I
            B --> K
            C --> H
            D --> L
        ```

        ---

        ## 2. Arquitetura Geral e Diagrama de Módulos

        O OrbitBot é composto por múltiplos módulos, cada um responsável por um domínio:

        - **bot.js**: Orquestra o ciclo de vida, integra todos os módulos
        - **audioProcessor.js**: Processamento de áudio, integração Python, melhoria IA
        - **aiConfig.js**: Configuração dinâmica de IA, personalidades, contexto
        - **pluginSystem.js**: Plugins, hooks, middleware
        - **queue.js**: Fila de mensagens, retries, controle de concorrência
        - **backup.js**: Backup, restauração, logs
        - **logger.js**: Logging estruturado, métricas
        - **performance.js**: Monitoramento, métricas
        - **database/db.js**: Operações SQLite, queries parametrizadas

        ### Diagrama de Dependências

        ```mermaid
        graph TD
            A[bot.js] -->|usa| B[audioProcessor.js]
            A -->|usa| C[aiConfig.js]
            A -->|usa| D[pluginSystem.js]
            A -->|usa| E[queue.js]
            A -->|usa| F[backup.js]
            A -->|usa| G[logger.js]
            A -->|usa| H[performance.js]
            A -->|usa| I[database/db.js]
            B -->|usa| I
            D -->|usa| G
            F -->|usa| I
            E -->|usa| G
            C -->|usa| G
        ```

        - **Observação:** Não há dependências circulares. Cada módulo pode ser testado isoladamente.

        ---

        ## 3. Ciclo de Vida de uma Mensagem

        ### 3.1. Recepção
        - O OrbitBot utiliza o `venom-bot` para escutar mensagens do WhatsApp.
        - Cada mensagem recebida dispara o evento `onMessage`.

        ### 3.2. Classificação
        - O tipo da mensagem é inspecionado: texto, áudio, comando admin, etc.
        - Mensagens de áudio são processadas imediatamente pelo `audioProcessor`.
        - Mensagens de texto são enfileiradas na `queue`.
        - Comandos admin (prefixo `/`) são processados imediatamente se o número for admin.

        ### 3.3. Processamento
        - Mensagens na fila são processadas em ordem FIFO.
        - O histórico do cliente é atualizado antes de qualquer processamento IA.
        - O contexto da conversa é recuperado para fornecer contexto à IA.
        - Plugins podem interceptar antes/depois do processamento.

        ### 3.4. Geração de Resposta
        - O módulo de IA (`openai.js` + `aiConfig.js`) gera a resposta baseada no histórico, personalidade e contexto.
        - A resposta pode ser modificada por plugins (ex: auto response, sentiment analysis).

        ### 3.5. Humanização e Envio
        - A resposta é humanizada (simulação de digitação, delays, chunking inteligente).
        - A resposta é enviada ao usuário via WhatsApp.
        - O histórico é atualizado com a resposta.

        ### 3.6. Pós-processamento
        - Métricas de performance são atualizadas.
        - Logs detalhados são gerados.
        - Plugins podem executar hooks pós-mensagem.

        ---

        ## 4. Fluxo Interno: Texto vs Áudio

        ### 4.1. Texto
        1. Recebido pelo `venom-bot`
        2. Enfileirado em `queue`
        3. Processado sequencialmente
        4. Histórico atualizado
        5. IA chamada com contexto
        6. Plugins/humanização
        7. Resposta enviada

        ### 4.2. Áudio
        1. Recebido pelo `venom-bot`
        2. Decriptado e salvo em `audios/`
        3. Transcrição via Python (Whisper)
        4. Melhoria da transcrição via IA (OpenRouter)
        5. Transcrição tratada como mensagem de texto (segue fluxo normal)

        #### Diagrama de Fluxo
        ```mermaid
        graph TD
            A[Recebe Mensagem] --> B{É Áudio?}
            B -->|Sim| C[Decripta e Salva]
            C --> D[Transcreve Python]
            D --> E[Melhora com IA]
            E --> F[Processa como Texto]
            B -->|Não| F
            F --> G[Atualiza Histórico]
            G --> H[IA e Plugins]
            H --> I[Humanização]
            I --> J[Envia Resposta]
        ```

        ---

        ## 5. Estrutura dos Módulos

        ### 5.1. bot.js
        - Ponto de entrada do sistema
        - Inicializa venom, listeners, fila, logger, performance
        - Responsável por orquestrar o ciclo de vida completo

        ### 5.2. audioProcessor.js
        - Classe singleton
        - Métodos principais:
        - `processAudioMessage(client, message)`
        - `transcribeAudio(filePath, ...)`
        - `improveTranscriptionWithAI(transcription, userId)`
        - Estratégias de melhoria: ultra_precise, contextual_aggressive, conservative_enhanced
        - Estatísticas: totalProcessed, avgConfidence, improvementRate

        ### 5.3. aiConfig.js
        - Gerencia modelos, personalidades, contexto
        - Métodos:
        - `setPersonality`, `getPersonality`, `listPersonalities`
        - `setModel`, `getModel`, `listModels`
        - `detectAndAdaptContext`
        - `generateConfig`
        - Personalidades compostas: weights, adaptação dinâmica

        ### 5.4. pluginSystem.js
        - PluginSystem: registro, hooks, middleware
        - Plugins padrão e customizados
        - Hooks: beforeMessage, afterMessage, messageProcessed
        - Métodos:
        - `registerPlugin`, `togglePlugin`, `listPlugins`, `getPlugin`

        ### 5.5. queue.js
        - MessageQueue: EventEmitter
        - Métodos:
        - `addMessage`, `processQueue`, `getQueueSize`
        - Retries exponenciais, eventos de erro

        ### 5.6. backup.js
        - BackupManager: backup manual/auto, restauração, logs
        - Métodos:
        - `createBackup`, `restoreBackup`, `deleteBackup`, `getBackupInfo`, `getCurrentBackupInfo`
        - Limpeza automática, log em JSON

        ### 5.7. logger.js
        - Logger customizado: info, error, debug, performance, api, queue
        - Integração com métricas do banco e áudio

        ### 5.8. performance.js
        - PerformanceMonitor: uptime, mensagens, erros, tempo médio, memória, CPU
        - Loga métricas a cada 1min

        ### 5.9. database/db.js
        - SQLite3 puro, queries parametrizadas
        - Métodos:
        - `cadastrarCliente`, `buscarCliente`, `adicionarMensagem`, `buscarHistorico`, `buscarUltimasMensagens`

        ---

        ## 6. Exemplo: Caminho Completo de uma Mensagem

        ### 6.1. Mensagem de Texto
        ```plaintext
        Usuário envia: "Oi, Orbit!"
        ↓
        venom.onMessage → bot.js
        ↓
        Verifica tipo: texto
        ↓
        queue.addMessage(message)
        ↓
        queue.processQueue()
        ↓
        Atualiza histórico (database)
        ↓
        Recupera contexto (aiConfig)
        ↓
        Gera resposta IA (openai.js)
        ↓
        Plugins (pluginSystem)
        ↓
        Humanização (humanizer.js)
        ↓
        client.sendText()
        ↓
        Atualiza histórico (resposta)
        ↓
        Logs, métricas, hooks pós-mensagem
        ```

        ### 6.2. Mensagem de Áudio
        ```plaintext
        Usuário envia áudio
        ↓
        venom.onMessage → bot.js
        ↓
        Verifica tipo: áudio
        ↓
        audioProcessor.processAudioMessage()
        ↓
        Decripta e salva áudio
        ↓
        Transcreve via Python (Whisper)
        ↓
        Melhora transcrição via IA
        ↓
        Segue fluxo de texto (acima)
        ```

        ---

        ## 7. Edge Cases e Resiliência

        - Mensagem vazia: ignorada
        - Áudio inválido: erro amigável ao usuário
        - Falha na transcrição: fallback para mensagem padrão
        - Falha na IA: resposta padrão, log detalhado
        - Comando admin inválido: feedback imediato
        - Corrupção de banco: backup/restauração
        - Timeout em APIs externas: retries, fallback
        - Plugins com erro: isolados, não afetam fluxo principal

        ---

        ## 8. Detalhes de Implementação: Código Real

        ### 8.1. Exemplo de Processamento de Mensagem
        ```js
        client.onMessage(async (message) => {
            if (!message.from.includes('@c.us') || message.isGroupMsg) return;
            if (!message.body && message.type !== 'audio' && message.type !== 'ptt') return;

            if (message.type === 'audio' || message.type === 'ptt' || (message.mimetype && message.mimetype.startsWith('audio'))) {
                // Áudio
                const buffer = await client.decryptFile(message);
                message.data = buffer;
                await audioProcessor.processAudioMessage(client, message);
                return;
            }

            // Texto
            if (message.body && message.body.startsWith('/') && isAdmin(message.from)) {
                const response = await handleAdminCommand(message);
                await client.sendText(message.from, response);
                return;
            }
            messageQueue.addMessage(message);
        });
        ```

        ### 8.2. Exemplo de Transcrição de Áudio
        ```js
        async transcribeAudio(filePath, audioHash, userId = null, retries = 2) {
            // Estratégia baseada no modo (rápido vs preciso)
            let transcriptionStrategies;
            let timeout;
            if (this.fastMode) {
                transcriptionStrategies = [ { model: 'tiny', temperature: 0.0, language: 'pt' } ];
                timeout = 10000;
            } else {
                transcriptionStrategies = [
                    { model: 'large', temperature: 0.0, language: 'pt' },
                    { model: 'medium', temperature: 0.0, language: 'pt' },
                    { model: 'small', temperature: 0.0, language: 'pt' }
                ];
                timeout = 45000;
            }
            // ...
        }
        ```

        ---

        ## 9. Banco de Dados e ORM

        ### 9.1. Arquitetura do Banco

        O OrbitBot utiliza **SQLite** como banco de dados principal, escolhido por sua simplicidade, confiabilidade e performance adequada para aplicações de médio porte.

        ### Esquema do Banco de Dados

        ```mermaid
        erDiagram
            CLIENTES {
                int id PK
                string numero UK
                datetime created_at
            }
            
            HISTORICO {
                int id PK
                int cliente_id FK
                string mensagem
                string role
                datetime created_at
            }
            
            CLIENTES ||--o{ HISTORICO : "tem"
        ```

        ### Relacionamentos e Índices

        ```mermaid
        graph TD
            A[CLIENTES] --> B[id: INTEGER PRIMARY KEY]
            A --> C[numero: TEXT UNIQUE]
            A --> D[created_at: DATETIME]
            
            E[HISTORICO] --> F[id: INTEGER PRIMARY KEY]
            E --> G[cliente_id: INTEGER FK]
            E --> H[mensagem: TEXT]
            E --> I[role: TEXT]
            E --> J[created_at: DATETIME]
            
            K[Índices] --> L[idx_historico_cliente]
            K --> M[idx_historico_created]
            
            L --> G
            M --> J
        ```

        ### Fluxo de Operações CRUD

        ```mermaid
        sequenceDiagram
            participant A as Application
            participant D as Database
            participant B as Backup
            
            A->>D: INSERT cliente
            D->>D: Validação UNIQUE
            D->>A: Retorna ID
            
            A->>D: INSERT mensagem
            D->>D: Validação FK
            D->>A: Confirmação
            
            A->>D: SELECT histórico
            D->>D: Query com JOIN
            D->>A: Resultado paginado
            
            A->>B: Trigger backup
            B->>B: Backup automático
            B->>A: Confirmação
        ```

        ### 9.2. Estrutura do Banco (DDL)

        ```sql
        -- Tabela de Clientes
        CREATE TABLE IF NOT EXISTS clientes (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            numero TEXT UNIQUE NOT NULL,
            created_at DATETIME DEFAULT CURRENT_TIMESTAMP
        );

        -- Tabela de Histórico
        CREATE TABLE IF NOT EXISTS historico (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            cliente_id INTEGER NOT NULL,
            mensagem TEXT NOT NULL,
            role TEXT NOT NULL CHECK (role IN ('user', 'assistant')),
            created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
            FOREIGN KEY (cliente_id) REFERENCES clientes(id) ON DELETE CASCADE
        );

        -- Índices para Performance
        CREATE INDEX IF NOT EXISTS idx_historico_cliente ON historico(cliente_id);
        CREATE INDEX IF NOT EXISTS idx_historico_created ON historico(created_at);
        CREATE INDEX IF NOT EXISTS idx_historico_cliente_created ON historico(cliente_id, created_at DESC);
        ```

        **Decisões de Design:**
        - `AUTOINCREMENT`: Evita reutilização de IDs, importante para auditoria
        - `UNIQUE` no número: Garante unicidade, facilita busca
        - `CHECK` no role: Validação no nível do banco
        - `ON DELETE CASCADE`: Se cliente for removido, histórico é removido automaticamente
        - Índices compostos: Otimiza queries de histórico por cliente ordenado por data

        ### 9.3. Inicialização do Banco

        ```js
        // database/db.js
        const db = new sqlite3.Database(path.join(__dirname, 'data/orbitbot.db'), (err) => {
            if (err) {
                console.error('Erro ao conectar ao banco de dados:', err);
            } else {
                console.log('Conectado ao banco de dados SQLite');
                initDatabase();
            }
        });

        function initDatabase() {
            db.serialize(() => {
                // Executa DDL em série para garantir ordem
                db.run(`CREATE TABLE IF NOT EXISTS clientes (...)`);
                db.run(`CREATE TABLE IF NOT EXISTS historico (...)`);
                db.run('CREATE INDEX IF NOT EXISTS idx_historico_cliente ON historico(cliente_id)');
                db.run('CREATE INDEX IF NOT EXISTS idx_historico_created ON historico(created_at)');
            });
        }
        ```

        ### 9.4. Operações Principais

        #### 9.4.1. Cadastrar Cliente
        ```js
        cadastrarCliente: (numero) => {
            return new Promise((resolve, reject) => {
                db.run(
                    'INSERT INTO clientes (numero) VALUES (?)',
                    [numero],
                    function(err) {
                        if (err) {
                            // Se erro for UNIQUE constraint, busca cliente existente
                            if (err.code === 'SQLITE_CONSTRAINT') {
                                return buscarCliente(numero).then(resolve).catch(reject);
                            }
                            reject(err);
                        } else {
                            resolve({
                                id: this.lastID,
                                numero,
                                created_at: new Date().toISOString()
                            });
                        }
                    }
                );
            });
        }
        ```

        **Tratamento de Erro:**
        - Se número já existe, busca e retorna cliente existente
        - Evita duplicação e race conditions

        #### 9.4.2. Buscar Histórico com Paginação
        ```js
        buscarHistorico: (cliente_id, page = 1, limit = 10) => {
            return new Promise((resolve, reject) => {
                const offset = (page - 1) * limit;
                
                // Busca total para paginação
                db.get(
                    'SELECT COUNT(*) as total FROM historico WHERE cliente_id = ?',
                    [cliente_id],
                    (err, countRow) => {
                        if (err) {
                            reject(err);
                            return;
                        }

                        const total = countRow.total;
                        const totalPages = Math.ceil(total / limit);

                        // Busca mensagens da página
                        db.all(
                            `SELECT * FROM historico 
                            WHERE cliente_id = ? 
                            ORDER BY created_at DESC 
                            LIMIT ? OFFSET ?`,
                            [cliente_id, limit, offset],
                            (err, rows) => {
                                if (err) {
                                    reject(err);
                                } else {
                                    resolve({
                                        messages: rows,
                                        pagination: {
                                            currentPage: page,
                                            totalPages,
                                            totalMessages: total,
                                            hasNextPage: page < totalPages,
                                            hasPreviousPage: page > 1
                                        }
                                    });
                                }
                            }
                        );
                    }
                );
            });
        }
        ```

        **Otimizações:**
        - `ORDER BY created_at DESC`: Mensagens mais recentes primeiro
        - `LIMIT/OFFSET`: Paginação eficiente
        - Índice composto `(cliente_id, created_at DESC)`: Otimiza a query

        #### 9.4.3. Buscar Últimas Mensagens
        ```js
        buscarUltimasMensagens: (cliente_id, limit = 5) => {
            return new Promise((resolve, reject) => {
                db.all(
                    `SELECT * FROM historico 
                    WHERE cliente_id = ? 
                    ORDER BY created_at DESC 
                    LIMIT ?`,
                    [cliente_id, limit],
                    (err, rows) => {
                        if (err) {
                            reject(err);
                        } else {
                            resolve(rows);
                        }
                    }
                );
            });
        }
        ```

        ### 9.5. Transações e Integridade

        #### 9.5.1. Transação para Atualização de Histórico
        ```js
        // Exemplo de transação (não implementado no código atual, mas recomendado)
        async function atualizarHistoricoComTransacao(clienteId, mensagem, role) {
            return new Promise((resolve, reject) => {
                db.serialize(() => {
                    db.run('BEGIN TRANSACTION');
                    
                    db.run(
                        'INSERT INTO historico (cliente_id, mensagem, role) VALUES (?, ?, ?)',
                        [clienteId, mensagem, role],
                        function(err) {
                            if (err) {
                                db.run('ROLLBACK');
                                reject(err);
                            } else {
                                db.run('COMMIT');
                                resolve({
                                    id: this.lastID,
                                    cliente_id: clienteId,
                                    mensagem,
                                    role,
                                    created_at: new Date().toISOString()
                                });
                            }
                        }
                    );
                });
            });
        }
        ```

        #### 9.5.2. Verificação de Integridade
        ```sql
        -- Verificar integridade referencial
        SELECT COUNT(*) as orphaned_messages
        FROM historico h
        LEFT JOIN clientes c ON h.cliente_id = c.id
        WHERE c.id IS NULL;

        -- Verificar duplicatas
        SELECT numero, COUNT(*) as count
        FROM clientes
        GROUP BY numero
        HAVING COUNT(*) > 1;

        -- Verificar roles inválidos
        SELECT id, role
        FROM historico
        WHERE role NOT IN ('user', 'assistant');
        ```

        ### 9.6. Performance e Otimizações

        #### 9.6.1. Análise de Performance
        ```sql
        -- Verificar uso de índices
        EXPLAIN QUERY PLAN
        SELECT * FROM historico 
        WHERE cliente_id = ? 
        ORDER BY created_at DESC 
        LIMIT 10;

        -- Estatísticas de tabelas
        SELECT 
            name,
            sql
        FROM sqlite_master
        WHERE type = 'table';

        -- Tamanho das tabelas
        SELECT 
            'clientes' as table_name,
            COUNT(*) as row_count
        FROM clientes
        UNION ALL
        SELECT 
            'historico' as table_name,
            COUNT(*) as row_count
        FROM historico;
        ```

        #### 9.6.2. Otimizações Implementadas
        - **Índices**: Otimizam queries por cliente e data
        - **Queries parametrizadas**: Previnem SQL injection e melhoram performance
        - **Paginação**: Evita carregar todo o histórico na memória
        - **LIMIT**: Restringe resultados para queries de contexto

        #### 9.6.3. Otimizações Futuras
        ```sql
        -- Índice parcial para mensagens recentes
        CREATE INDEX IF NOT EXISTS idx_historico_recentes 
        ON historico(cliente_id, created_at DESC) 
        WHERE created_at > datetime('now', '-30 days');

        -- Índice para busca por conteúdo
        CREATE INDEX IF NOT EXISTS idx_historico_mensagem 
        ON historico(mensagem) 
        WHERE length(mensagem) > 10;

        -- Tabela de estatísticas
        CREATE TABLE IF NOT EXISTS estatisticas (
            id INTEGER PRIMARY KEY,
            total_clientes INTEGER,
            total_mensagens INTEGER,
            ultima_atualizacao DATETIME DEFAULT CURRENT_TIMESTAMP
        );
        ```

        ### 9.7. Backup e Restore

        #### 9.7.1. Backup do Banco
        ```js
        // backup.js - Estratégia de backup
        createBackup(customName = null, isAutomatic = false) {
            try {
                let backupName;
                const now = new Date();
                const day = String(now.getDate()).padStart(2, '0');
                const month = String(now.getMonth() + 1).padStart(2, '0');
                const year = String(now.getFullYear()).slice(-2);
                const hour = String(now.getHours()).padStart(2, '0');
                const minute = String(now.getMinutes()).padStart(2, '0');
                
                if (customName) {
                    backupName = `backup ${customName}`;
                } else if (isAutomatic) {
                    backupName = `backup auto_${day}-${month}-${year}_${hour}-${minute}`;
                } else {
                    backupName = `backup ${day}-${month}-${year} ${hour}-${minute}`;
                }

                const backupPath = path.join(this.backupDir, backupName);
                fs.mkdirSync(backupPath);

                // Cópia física do arquivo
                fs.copyFileSync(this.dbFile, path.join(backupPath, 'orbitbot.db'));

                // Log da operação
                this.addLogEntry('create', { 
                    backupName,
                    type: isAutomatic ? 'automático' : 'manual'
                });

                return true;
            } catch (err) {
                logger.error('Erro ao criar backup', { error: err.message });
                return false;
            }
        }
        ```

        #### 9.7.2. Restore do Banco
        ```js
        restoreBackup(backupName) {
            try {
                const backupPath = path.join(this.backupDir, backupName);
                
                if (!fs.existsSync(backupPath)) {
                    throw new Error(`Backup não encontrado: ${backupName}`);
                }

                const dbFile = path.join(backupPath, 'orbitbot.db');
                if (!fs.existsSync(dbFile)) {
                    throw new Error('Arquivo do banco de dados não encontrado no backup');
                }

                // Restaura o arquivo
                fs.copyFileSync(dbFile, this.dbFile);

                this.addLogEntry('restore', { backupName });
                return true;
            } catch (err) {
                logger.error('Erro ao restaurar backup', { 
                    error: err.message,
                    backupName
                });
                return false;
            }
        }
        ```

        #### 9.7.3. Verificação de Integridade do Backup
        ```js
        async getBackupInfo(backupName) {
            try {
                const backupPath = path.join(this.backupDir, backupName);
                if (!fs.existsSync(backupPath)) {
                    throw new Error(`Backup não encontrado: ${backupName}`);
                }

                const dbFile = path.join(backupPath, 'orbitbot.db');
                if (!fs.existsSync(dbFile)) {
                    throw new Error('Arquivo do banco de dados não encontrado no backup');
                }

                const stats = fs.statSync(dbFile);
                const dbInfo = await this.getDatabaseInfo(dbFile);

                return {
                    totalClientes: dbInfo.totalClientes,
                    totalMensagens: dbInfo.totalMensagens,
                    size: stats.size,
                    created_at: stats.birthtime.toISOString(),
                    isAutomatic: backupName.includes('auto_')
                };
            } catch (err) {
                logger.error('Erro ao obter informações do backup', { 
                    error: err.message,
                    backupName
                });
                return null;
            }
        }
        ```

        ### 9.8. Monitoramento e Métricas

        #### 9.8.1. Métricas do Banco
        ```js
        // logger.js - Integração com métricas do banco
        performance: (data) => {
            const dbPath = path.join(__dirname, '../database/data/orbitbot.db');
            
            let totalClientes = 0;
            let totalMensagens = 0;
            
            const db = new sqlite3.Database(dbPath, (err) => {
                if (err) {
                    console.error('Erro ao conectar ao banco para métricas:', err.message);
                    return;
                }

                db.get(`
                    SELECT 
                        (SELECT COUNT(*) FROM clientes) as totalClientes,
                        (SELECT COUNT(*) FROM historico) as totalMensagens
                `, (err, row) => {
                    db.close();
                    if (err) {
                        console.error('Erro ao ler dados do banco:', err.message);
                    } else {
                        totalClientes = row.totalClientes;
                        totalMensagens = row.totalMensagens;
                    }

                    console.log('\n=== DADOS DO BANCO ===');
                    console.log(`Total de clientes: ${totalClientes}`);
                    console.log(`Total de mensagens no histórico: ${totalMensagens}`);
                });
            });
        }
        ```

        #### 9.8.2. Queries de Monitoramento
        ```sql
        -- Crescimento de clientes por dia
        SELECT 
            DATE(created_at) as data,
            COUNT(*) as novos_clientes
        FROM clientes
        GROUP BY DATE(created_at)
        ORDER BY data DESC
        LIMIT 30;

        -- Crescimento de mensagens por dia
        SELECT 
            DATE(created_at) as data,
            COUNT(*) as mensagens
        FROM historico
        GROUP BY DATE(created_at)
        ORDER BY data DESC
        LIMIT 30;

        -- Clientes mais ativos
        SELECT 
            c.numero,
            COUNT(h.id) as total_mensagens,
            MAX(h.created_at) as ultima_mensagem
        FROM clientes c
        JOIN historico h ON c.id = h.cliente_id
        GROUP BY c.id
        ORDER BY total_mensagens DESC
        LIMIT 10;

        -- Distribuição de roles
        SELECT 
            role,
            COUNT(*) as total
        FROM historico
        GROUP BY role;
        ```

        ### 9.9. Segurança e Validação

        #### 9.9.1. Validação de Entrada
        ```js
        // Validação de número de telefone
        function validarNumero(numero) {
            if (!numero || typeof numero !== 'string') {
                throw new Error('Número inválido');
            }
            
            // Remove caracteres especiais
            const numeroLimpo = numero.replace(/[^\d]/g, '');
            
            if (numeroLimpo.length < 10 || numeroLimpo.length > 15) {
                throw new Error('Número deve ter entre 10 e 15 dígitos');
            }
            
            return numeroLimpo;
        }

        // Validação de mensagem
        function validarMensagem(mensagem) {
            if (!mensagem || typeof mensagem !== 'string') {
                throw new Error('Mensagem inválida');
            }
            
            if (mensagem.length > 10000) {
                throw new Error('Mensagem muito longa (máximo 10.000 caracteres)');
            }
            
            return mensagem.trim();
        }

        // Validação de role
        function validarRole(role) {
            const rolesValidos = ['user', 'assistant'];
            if (!rolesValidos.includes(role)) {
                throw new Error(`Role inválido. Valores permitidos: ${rolesValidos.join(', ')}`);
            }
            return role;
        }
        ```

        #### 9.9.2. Sanitização de Queries
        ```js
        // Todas as queries usam parâmetros para prevenir SQL injection
        db.run(
            'INSERT INTO historico (cliente_id, mensagem, role) VALUES (?, ?, ?)',
            [clienteId, mensagem, role],  // Parâmetros são escapados automaticamente
            function(err) {
                // ...
            }
        );
        ```

        ### 9.10. Troubleshooting do Banco

        #### 9.10.1. Problemas Comuns
        ```sql
        -- Verificar se o banco está corrompido
        PRAGMA integrity_check;

        -- Verificar se os índices estão válidos
        PRAGMA index_list(historico);

        -- Verificar estatísticas do banco
        PRAGMA stats;

        -- Verificar configurações
        PRAGMA journal_mode;
        PRAGMA synchronous;
        PRAGMA cache_size;
        ```

        #### 9.10.2. Comandos de Manutenção
        ```sql
        -- Otimizar o banco (VACUUM)
        VACUUM;

        -- Reconstruir índices
        REINDEX;

        -- Analisar tabelas para otimização
        ANALYZE;

        -- Verificar e reparar integridade
        PRAGMA quick_check;
        PRAGMA integrity_check;
        ```

        #### 9.10.3. Scripts de Recuperação
        ```js
        // Recuperação de backup automática
        async function recuperarBancoAutomatico() {
            try {
                const backups = backupManager.listBackups();
                if (backups.length === 0) {
                    throw new Error('Nenhum backup disponível');
                }
                
                // Usa o backup mais recente
                const backupMaisRecente = backups[0];
                const sucesso = backupManager.restoreBackup(backupMaisRecente);
                
                if (sucesso) {
                    logger.info('Banco recuperado automaticamente', { backup: backupMaisRecente });
                    return true;
                } else {
                    throw new Error('Falha na restauração automática');
                }
            } catch (err) {
                logger.error('Erro na recuperação automática', { error: err.message });
                return false;
            }
        }
        ```

        ---

        ## 10. Próximos Blocos

        - **Fila de Mensagens**: Algoritmo, concorrência, starvation, deadlocks, retries exponenciais
        - **Sistema de IA**: Prompt engineering, adaptação dinâmica, personalidades compostas, contexto
        - **Plugins**: API avançada, hooks, middleware, exemplos complexos
        - **Sistema de Áudio**: Integração Python, estratégias de melhoria, timeouts, fallbacks
        - **Backup**: Atomicidade, logs estruturados, recovery automático
        - **Logs**: Formato, rotação, integração externa, alertas
        - **Tratamento de Erros**: Propagação, auto-recuperação, circuit breaker
        - **Testes**: Unitários, integração, stress, coverage
        - **Performance**: Benchmarks, profiling, tuning, escalabilidade
        - **Segurança**: Validação, autenticação, controle de acesso
        - **Extensibilidade**: Padrões, anti-patterns, exemplos avançados
        - **APIs Externas**: Contratos, versionamento, fallback, limites
        - **DevOps**: Scripts, CI/CD, rollback, observabilidade

        ---

        ## 11. Sistema de Fila de Mensagens

        ### 11.1. Arquitetura da Fila

        O sistema de fila do OrbitBot é baseado em **EventEmitter** e implementa um padrão **Producer-Consumer** com processamento assíncrono:

        ### Algoritmo de Processamento da Fila

        ```mermaid
        graph TD
            A[Nova Mensagem] --> B[Validação]
            B --> C{Valida?}
            C -->|Não| D[Rejeita]
            C -->|Sim| E[Adiciona à Fila]
            E --> F[Emit Event: process]
            F --> G[Processa Mensagem]
            G --> H{Sucesso?}
            H -->|Sim| I[Remove da Fila]
            H -->|Não| J[Retry Logic]
            J --> K{Retries < Max?}
            K -->|Sim| L[Backoff Delay]
            L --> M[Re-adiciona à Fila]
            K -->|Não| N[Emit Event: error]
            M --> F
        ```

        ### Estados da Mensagem

        ```mermaid
        stateDiagram-v2
            [*] --> Pending: Adicionada
            Pending --> Processing: Processada
            Processing --> Success: Sucesso
            Processing --> Failed: Erro
            Failed --> Retrying: Retry < Max
            Retrying --> Processing: Re-processada
            Failed --> [*]: Retry >= Max
            Success --> [*]: Finalizada
        ```

        ### Estratégia de Retry com Backoff Exponencial

        ```mermaid
        graph LR
            A[Erro] --> B[Retry 1: 1s]
            B --> C[Erro] --> D[Retry 2: 2s]
            D --> E[Erro] --> F[Retry 3: 4s]
            F --> G[Erro] --> H[Falha Final]
            B --> I[Sucesso]
            D --> I
            F --> I
        ```

        ### 11.2. Implementação da Fila

        ```js
        // src/queue.js
        const EventEmitter = require('events');

        class MessageQueue extends EventEmitter {
            constructor() {
                super();
                this.queue = [];
                this.processing = false;
                this.maxRetries = 3;
                this.retryDelay = 1000;
                this.maxConcurrent = 1; // Processamento sequencial
                this.currentProcessing = 0;
                this.stats = {
                    totalProcessed: 0,
                    totalErrors: 0,
                    avgProcessingTime: 0,
                    queueSize: 0
                };
            }

            addMessage(message) {
                const queueItem = {
                    message,
                    retries: 0,
                    timestamp: Date.now(),
                    id: this.generateMessageId()
                };

                this.queue.push(queueItem);
                this.stats.queueSize = this.queue.length;

                logger.queue('Mensagem adicionada à fila', {
                    messageId: queueItem.id,
                    queueSize: this.queue.length,
                    from: message.from
                });

                if (!this.processing) {
                    this.processQueue();
                }
            }

            async processQueue() {
                if (this.processing) return;
                this.processing = true;

                logger.queue('Iniciando processamento da fila', {
                    queueSize: this.queue.length
                });

                while (this.hasMessages() && this.currentProcessing < this.maxConcurrent) {
                    const item = this.getNextMessage();
                    if (!item) break;

                    this.currentProcessing++;
                    this.processMessage(item);
                }

                this.processing = false;
            }

            async processMessage(item) {
                const startTime = Date.now();
                
                try {
                    logger.queue('Processando mensagem', {
                        messageId: item.id,
                        retries: item.retries,
                        from: item.message.from
                    });

                    await this.emit('process', item.message);
                    
                    // Sucesso
                    const processingTime = Date.now() - startTime;
                    this.stats.totalProcessed++;
                    this.stats.avgProcessingTime = 
                        (this.stats.avgProcessingTime * (this.stats.totalProcessed - 1) + processingTime) / this.stats.totalProcessed;
                    
                    logger.queue('Mensagem processada com sucesso', {
                        messageId: item.id,
                        processingTime,
                        avgProcessingTime: this.stats.avgProcessingTime
                    });

                } catch (error) {
                    // Erro no processamento
                    this.stats.totalErrors++;
                    
                    logger.error('Erro ao processar mensagem', {
                        messageId: item.id,
                        error: error.message,
                        retries: item.retries,
                        from: item.message.from
                    });

                    if (item.retries < this.maxRetries) {
                        // Retry com backoff exponencial
                        item.retries++;
                        const delay = this.calculateRetryDelay(item.retries);
                        
                        logger.queue('Agendando retry', {
                            messageId: item.id,
                            retries: item.retries,
                            delay
                        });

                        setTimeout(() => {
                            this.queue.unshift(item); // Adiciona no início para prioridade
                            this.stats.queueSize = this.queue.length;
                            this.processQueue();
                        }, delay);
                    } else {
                        // Máximo de retries atingido
                        this.emit('error', {
                            message: item.message,
                            error,
                            retries: item.retries,
                            messageId: item.id
                        });
                        
                        logger.error('Máximo de retries atingido', {
                            messageId: item.id,
                            from: item.message.from
                        });
                    }
                } finally {
                    this.currentProcessing--;
                    
                    // Continua processando se há mais mensagens
                    if (this.hasMessages() && this.currentProcessing < this.maxConcurrent) {
                        this.processQueue();
                    }
                }
            }

            calculateRetryDelay(retryCount) {
                // Backoff exponencial: 1s, 2s, 4s, 8s...
                return this.retryDelay * Math.pow(2, retryCount - 1);
            }

            generateMessageId() {
                return `msg_${Date.now()}_${Math.random().toString(36).substr(2, 9)}`;
            }

            hasMessages() {
                return this.queue.length > 0;
            }

            getNextMessage() {
                return this.queue.shift();
            }

            getQueueSize() {
                return this.queue.length;
            }

            getStats() {
                return {
                    ...this.stats,
                    currentProcessing: this.currentProcessing,
                    isProcessing: this.processing
                };
            }

            // Métodos de controle
            pause() {
                this.processing = false;
                logger.queue('Fila pausada');
            }

            resume() {
                if (!this.processing && this.hasMessages()) {
                    this.processQueue();
                }
                logger.queue('Fila resumida');
            }

            clear() {
                const clearedCount = this.queue.length;
                this.queue = [];
                this.stats.queueSize = 0;
                logger.queue('Fila limpa', { clearedCount });
                return clearedCount;
            }
        }
        ```

        ### 11.3. Algoritmos de Fila

        #### 11.3.1. FIFO (First In, First Out)
        ```js
        // Implementação atual - FIFO simples
        addMessage(message) {
            this.queue.push(queueItem); // Adiciona no final
        }

        getNextMessage() {
            return this.queue.shift(); // Remove do início
        }
        ```

        #### 11.3.2. Prioridade por Tipo de Mensagem
        ```js
        // Implementação alternativa com prioridade
        class PriorityMessageQueue extends MessageQueue {
            constructor() {
                super();
                this.priorityQueues = {
                    high: [],    // Comandos admin, mensagens urgentes
                    normal: [],  // Mensagens de texto normais
                    low: []      // Mensagens de áudio, logs
                };
            }

            addMessage(message, priority = 'normal') {
                const queueItem = {
                    message,
                    retries: 0,
                    timestamp: Date.now(),
                    id: this.generateMessageId(),
                    priority
                };

                this.priorityQueues[priority].push(queueItem);
                
                if (!this.processing) {
                    this.processQueue();
                }
            }

            getNextMessage() {
                // Processa alta prioridade primeiro
                if (this.priorityQueues.high.length > 0) {
                    return this.priorityQueues.high.shift();
                }
                if (this.priorityQueues.normal.length > 0) {
                    return this.priorityQueues.normal.shift();
                }
                if (this.priorityQueues.low.length > 0) {
                    return this.priorityQueues.low.shift();
                }
                return null;
            }

            hasMessages() {
                return this.priorityQueues.high.length > 0 ||
                    this.priorityQueues.normal.length > 0 ||
                    this.priorityQueues.low.length > 0;
            }
        }
        ```

        #### 11.3.3. Round Robin com Pesos
        ```js
        class WeightedRoundRobinQueue extends MessageQueue {
            constructor() {
                super();
                this.queues = {
                    admin: { queue: [], weight: 3 },
                    user: { queue: [], weight: 1 },
                    audio: { queue: [], weight: 2 }
                };
                this.currentWeights = { admin: 0, user: 0, audio: 0 };
            }

            getNextMessage() {
                // Reset weights se todas as filas estão vazias
                if (!this.hasMessages()) {
                    this.resetWeights();
                    return null;
                }

                // Encontra a fila com maior peso relativo
                let selectedQueue = null;
                let maxWeight = -1;

                for (const [type, config] of Object.entries(this.queues)) {
                    if (config.queue.length > 0) {
                        const relativeWeight = config.weight - this.currentWeights[type];
                        if (relativeWeight > maxWeight) {
                            maxWeight = relativeWeight;
                            selectedQueue = type;
                        }
                    }
                }

                if (selectedQueue) {
                    this.currentWeights[selectedQueue]++;
                    return this.queues[selectedQueue].queue.shift();
                }

                return null;
            }
        }
        ```

        ### 11.4. Controle de Concorrência

        #### 11.4.1. Processamento Sequencial (Atual)
        ```js
        // Configuração atual - uma mensagem por vez
        this.maxConcurrent = 1;
        this.currentProcessing = 0;

        async processQueue() {
            while (this.hasMessages() && this.currentProcessing < this.maxConcurrent) {
                // Processa uma mensagem por vez
            }
        }
        ```

        #### 11.4.2. Processamento Paralelo Controlado
        ```js
        class ConcurrentMessageQueue extends MessageQueue {
            constructor(maxConcurrent = 3) {
                super();
                this.maxConcurrent = maxConcurrent;
                this.semaphore = new Semaphore(maxConcurrent);
            }

            async processMessage(item) {
                await this.semaphore.acquire();
                
                try {
                    await this.emit('process', item.message);
                } finally {
                    this.semaphore.release();
                }
            }
        }

        // Implementação de Semáforo
        class Semaphore {
            constructor(max) {
                this.max = max;
                this.current = 0;
                this.waiting = [];
            }

            async acquire() {
                if (this.current < this.max) {
                    this.current++;
                    return Promise.resolve();
                }

                return new Promise(resolve => {
                    this.waiting.push(resolve);
                });
            }

            release() {
                this.current--;
                if (this.waiting.length > 0) {
                    this.current++;
                    const resolve = this.waiting.shift();
                    resolve();
                }
            }
        }
        ```

        ### 11.5. Retry e Backoff

        #### 11.5.1. Backoff Exponencial
        ```js
        calculateRetryDelay(retryCount) {
            // Fórmula: baseDelay * 2^(retryCount - 1)
            const baseDelay = 1000; // 1 segundo
            const maxDelay = 30000; // 30 segundos máximo
            
            const delay = baseDelay * Math.pow(2, retryCount - 1);
            return Math.min(delay, maxDelay);
        }

        // Exemplo de delays:
        // Retry 1: 1000ms
        // Retry 2: 2000ms
        // Retry 3: 4000ms
        // Retry 4: 8000ms
        // Retry 5: 16000ms
        // Retry 6+: 30000ms (máximo)
        ```

        #### 11.5.2. Backoff com Jitter
        ```js
        calculateRetryDelayWithJitter(retryCount) {
            const baseDelay = this.calculateRetryDelay(retryCount);
            
            // Adiciona jitter para evitar thundering herd
            const jitter = Math.random() * 0.1 * baseDelay; // 10% de jitter
            
            return baseDelay + jitter;
        }
        ```

        #### 11.5.3. Retry com Circuit Breaker
        ```js
        class CircuitBreaker {
            constructor(failureThreshold = 5, resetTimeout = 60000) {
                this.failureThreshold = failureThreshold;
                this.resetTimeout = resetTimeout;
                this.failureCount = 0;
                this.lastFailureTime = null;
                this.state = 'CLOSED'; // CLOSED, OPEN, HALF_OPEN
            }

            async execute(fn) {
                if (this.state === 'OPEN') {
                    if (Date.now() - this.lastFailureTime > this.resetTimeout) {
                        this.state = 'HALF_OPEN';
                    } else {
                        throw new Error('Circuit breaker is OPEN');
                    }
                }

                try {
                    const result = await fn();
                    this.onSuccess();
                    return result;
                } catch (error) {
                    this.onFailure();
                    throw error;
                }
            }

            onSuccess() {
                this.failureCount = 0;
                this.state = 'CLOSED';
            }

            onFailure() {
                this.failureCount++;
                this.lastFailureTime = Date.now();
                
                if (this.failureCount >= this.failureThreshold) {
                    this.state = 'OPEN';
                }
            }
        }
        ```

        ### 11.6. Deadlock Prevention

        #### 11.6.1. Timeout em Operações
        ```js
        async processMessageWithTimeout(item, timeout = 30000) {
            return Promise.race([
                this.processMessage(item),
                new Promise((_, reject) => 
                    setTimeout(() => reject(new Error('Timeout')), timeout)
                )
            ]);
        }
        ```

        #### 11.6.2. Resource Pool
        ```js
        class ResourcePool {
            constructor(resources, maxWaitTime = 5000) {
                this.resources = resources;
                this.available = [...resources];
                this.waiting = [];
                this.maxWaitTime = maxWaitTime;
            }

            async acquire() {
                if (this.available.length > 0) {
                    return this.available.pop();
                }

                return new Promise((resolve, reject) => {
                    const timeout = setTimeout(() => {
                        reject(new Error('Resource acquisition timeout'));
                    }, this.maxWaitTime);

                    this.waiting.push({ resolve, timeout });
                });
            }

            release(resource) {
                if (this.waiting.length > 0) {
                    const { resolve, timeout } = this.waiting.shift();
                    clearTimeout(timeout);
                    resolve(resource);
                } else {
                    this.available.push(resource);
                }
            }
        }
        ```

        ### 11.7. Starvation Prevention

        #### 11.7.1. Aging de Mensagens
        ```js
        class AgingMessageQueue extends MessageQueue {
            constructor(maxAge = 300000) { // 5 minutos
                super();
                this.maxAge = maxAge;
            }

            getNextMessage() {
                const now = Date.now();
                
                // Procura mensagem mais antiga que não expirou
                for (let i = 0; i < this.queue.length; i++) {
                    const item = this.queue[i];
                    if (now - item.timestamp < this.maxAge) {
                        this.queue.splice(i, 1);
                        return item;
                    }
                }

                // Remove mensagens expiradas
                this.queue = this.queue.filter(item => now - item.timestamp < this.maxAge);
                
                return null;
            }
        }
        ```

        #### 11.7.2. Fair Queue
        ```js
        class FairMessageQueue extends MessageQueue {
            constructor() {
                super();
                this.userQueues = new Map(); // Fila por usuário
                this.userOrder = []; // Ordem de chegada dos usuários
            }

            addMessage(message) {
                const userId = message.from;
                
                if (!this.userQueues.has(userId)) {
                    this.userQueues.set(userId, []);
                    this.userOrder.push(userId);
                }

                const queueItem = {
                    message,
                    retries: 0,
                    timestamp: Date.now(),
                    id: this.generateMessageId()
                };

                this.userQueues.get(userId).push(queueItem);
            }

            getNextMessage() {
                // Round-robin entre usuários
                for (let i = 0; i < this.userOrder.length; i++) {
                    const userId = this.userOrder[i];
                    const userQueue = this.userQueues.get(userId);
                    
                    if (userQueue.length > 0) {
                        const item = userQueue.shift();
                        
                        // Remove usuário da ordem se não há mais mensagens
                        if (userQueue.length === 0) {
                            this.userOrder.splice(i, 1);
                        }
                        
                        return item;
                    }
                }

                return null;
            }

            hasMessages() {
                return Array.from(this.userQueues.values()).some(queue => queue.length > 0);
            }
        }
        ```

        ### 11.8. Monitoramento e Métricas

        #### 11.8.1. Métricas Detalhadas
        ```js
        class MetricsMessageQueue extends MessageQueue {
            constructor() {
                super();
                this.metrics = {
                    totalProcessed: 0,
                    totalErrors: 0,
                    avgProcessingTime: 0,
                    maxProcessingTime: 0,
                    minProcessingTime: Infinity,
                    queueSize: 0,
                    maxQueueSize: 0,
                    retryCount: 0,
                    errorTypes: new Map(),
                    processingTimes: []
                };
            }

            updateMetrics(processingTime, error = null) {
                this.metrics.totalProcessed++;
                this.metrics.avgProcessingTime = 
                    (this.metrics.avgProcessingTime * (this.metrics.totalProcessed - 1) + processingTime) / this.metrics.totalProcessed;
                
                this.metrics.maxProcessingTime = Math.max(this.metrics.maxProcessingTime, processingTime);
                this.metrics.minProcessingTime = Math.min(this.metrics.minProcessingTime, processingTime);
                
                if (error) {
                    this.metrics.totalErrors++;
                    const errorType = error.constructor.name;
                    this.metrics.errorTypes.set(errorType, (this.metrics.errorTypes.get(errorType) || 0) + 1);
                }

                this.metrics.queueSize = this.queue.length;
                this.metrics.maxQueueSize = Math.max(this.metrics.maxQueueSize, this.queue.length);
            }

            getMetrics() {
                return {
                    ...this.metrics,
                    errorRate: this.metrics.totalProcessed > 0 ? 
                        (this.metrics.totalErrors / this.metrics.totalProcessed) * 100 : 0,
                    errorTypes: Object.fromEntries(this.metrics.errorTypes),
                    currentQueueSize: this.queue.length
                };
            }
        }
        ```

        #### 11.8.2. Alertas Automáticos
        ```js
        class AlertingMessageQueue extends MetricsMessageQueue {
            constructor() {
                super();
                this.alerts = {
                    highErrorRate: 0.1, // 10%
                    highQueueSize: 100,
                    slowProcessing: 10000 // 10 segundos
                };
            }

            checkAlerts(processingTime, error) {
                const metrics = this.getMetrics();
                
                // Alerta de alta taxa de erro
                if (metrics.errorRate > this.alerts.highErrorRate) {
                    logger.error('ALERTA: Alta taxa de erro na fila', {
                        errorRate: metrics.errorRate,
                        threshold: this.alerts.highErrorRate
                    });
                }

                // Alerta de fila grande
                if (metrics.currentQueueSize > this.alerts.highQueueSize) {
                    logger.warn('ALERTA: Fila muito grande', {
                        queueSize: metrics.currentQueueSize,
                        threshold: this.alerts.highQueueSize
                    });
                }

                // Alerta de processamento lento
                if (processingTime > this.alerts.slowProcessing) {
                    logger.warn('ALERTA: Processamento lento', {
                        processingTime,
                        threshold: this.alerts.slowProcessing
                    });
                }
            }
        }
        ```

        ### 11.9. Integração com o Bot

        #### 11.9.1. Configuração no bot.js
        ```js
        // src/bot.js
        const messageQueue = require('./queue');

        // Configura o processador de mensagens da fila
        messageQueue.on('process', async (message) => {
            const startTime = Date.now();
            
            try {
                // Processamento da mensagem
                await processMessage(message);
                
                // Métricas
                const responseTime = Date.now() - startTime;
                performanceMonitor.addMessageResponseTime(responseTime);
                
            } catch (err) {
                logger.error('Erro no processamento', { 
                    error: err.message,
                    from: message.from
                });
                performanceMonitor.addError();
            }
        });

        // Configura o handler de erros da fila
        messageQueue.on('error', (error) => {
            logger.error('Erro na fila de mensagens', {
                message: error.message,
                retries: error.retries,
                error: error.error.message
            });
        });
        ```

        #### 11.9.2. Comandos de Administração
        ```js
        // Comandos para gerenciar a fila
        case 'fila':
            const filaCommand = args[0];
            
            switch (filaCommand) {
                case 'status':
                    const stats = messageQueue.getStats();
                    return `📊 *Status da Fila*\n\n` +
                        `• **Mensagens na fila:** ${stats.queueSize}\n` +
                        `• **Processando:** ${stats.currentProcessing}\n` +
                        `• **Total processadas:** ${stats.totalProcessed}\n` +
                        `• **Erros:** ${stats.totalErrors}\n` +
                        `• **Taxa de erro:** ${stats.errorRate?.toFixed(2)}%\n` +
                        `• **Tempo médio:** ${stats.avgProcessingTime?.toFixed(0)}ms`;
                        
                case 'pausar':
                    messageQueue.pause();
                    return '⏸️ Fila pausada';
                    
                case 'resumir':
                    messageQueue.resume();
                    return '▶️ Fila resumida';
                    
                case 'limpar':
                    const cleared = messageQueue.clear();
                    return `🧹 Fila limpa! ${cleared} mensagens removidas`;
                    
                default:
                    return 'Comandos: status, pausar, resumir, limpar';
            }
        ```

        ### 11.10. Troubleshooting da Fila

        #### 11.10.1. Problemas Comuns
        ```js
        // Diagnóstico de problemas
        function diagnoseQueueIssues() {
            const stats = messageQueue.getStats();
            const issues = [];

            // Fila muito grande
            if (stats.queueSize > 50) {
                issues.push({
                    type: 'QUEUE_OVERFLOW',
                    severity: 'HIGH',
                    description: `Fila com ${stats.queueSize} mensagens`,
                    solution: 'Verificar processamento de mensagens'
                });
            }

            // Alta taxa de erro
            if (stats.errorRate > 0.1) {
                issues.push({
                    type: 'HIGH_ERROR_RATE',
                    severity: 'CRITICAL',
                    description: `Taxa de erro: ${(stats.errorRate * 100).toFixed(2)}%`,
                    solution: 'Verificar logs de erro e APIs externas'
                });
            }

            // Processamento lento
            if (stats.avgProcessingTime > 5000) {
                issues.push({
                    type: 'SLOW_PROCESSING',
                    severity: 'MEDIUM',
                    description: `Tempo médio: ${stats.avgProcessingTime}ms`,
                    solution: 'Otimizar processamento ou aumentar recursos'
                });
            }

            return issues;
        }
        ```

        #### 11.10.2. Recuperação Automática
        ```js
        // Sistema de auto-recuperação
        class SelfHealingMessageQueue extends AlertingMessageQueue {
            constructor() {
                super();
                this.autoRecovery = true;
                this.recoveryStrategies = {
                    highErrorRate: this.handleHighErrorRate.bind(this),
                    highQueueSize: this.handleHighQueueSize.bind(this),
                    slowProcessing: this.handleSlowProcessing.bind(this)
                };
            }

            async handleHighErrorRate() {
                logger.info('Iniciando recuperação: alta taxa de erro');
                
                // Pausa temporariamente para estabilizar
                this.pause();
                await this.sleep(5000);
                
                // Reduz retries para evitar loop
                this.maxRetries = Math.max(1, this.maxRetries - 1);
                
                this.resume();
            }

            async handleHighQueueSize() {
                logger.info('Iniciando recuperação: fila muito grande');
                
                // Processa em paralelo temporariamente
                const originalMaxConcurrent = this.maxConcurrent;
                this.maxConcurrent = Math.min(5, this.maxConcurrent * 2);
                
                // Aguarda fila diminuir
                while (this.queue.length > 20) {
                    await this.sleep(1000);
                }
                
                this.maxConcurrent = originalMaxConcurrent;
            }

            async handleSlowProcessing() {
                logger.info('Iniciando recuperação: processamento lento');
                
                // Limpa cache e otimiza
                if (typeof clearCache === 'function') {
                    clearCache();
                }
                
                // Reduz timeout de APIs
                // Implementar lógica específica
            }

            sleep(ms) {
                return new Promise(resolve => setTimeout(resolve, ms));
            }
        }
        ```

        ---

        ## 12. Próximos Blocos

        - **Sistema de IA**: Prompt engineering, adaptação dinâmica, personalidades compostas, contexto
        - **Plugins**: API avançada, hooks, middleware, exemplos complexos
        - **Sistema de Áudio**: Integração Python, estratégias de melhoria, timeouts, fallbacks
        - **Backup**: Atomicidade, logs estruturados, recovery automático
        - **Logs**: Formato, rotação, integração externa, alertas
        - **Tratamento de Erros**: Propagação, auto-recuperação, circuit breaker
        - **Testes**: Unitários, integração, stress, coverage
        - **Performance**: Benchmarks, profiling, tuning, escalabilidade
        - **Segurança**: Validação, autenticação, controle de acesso
        - **Extensibilidade**: Padrões, anti-patterns, exemplos avançados
        - **APIs Externas**: Contratos, versionamento, fallback, limites
        - **DevOps**: Scripts, CI/CD, rollback, observabilidade

        ---

        ## 13. Sistema de IA Avançado

        ### 13.1. Arquitetura do Sistema de IA

        O OrbitBot utiliza um sistema de IA **modular e dinâmico** que permite configuração em tempo real de personalidades, modelos e contexto. O sistema é composto por múltiplas camadas:

        - **Camada de Configuração**: Gerencia personalidades, modelos e contexto
        - **Camada de Adaptação**: Detecta e adapta automaticamente ao contexto
        - **Camada de Composição**: Combina personalidades de forma inteligente
        - **Camada de Execução**: Gera respostas com base na configuração atual

        ### 13.2. Configuração de Modelos

        #### 13.2.1. Modelos Disponíveis
        ```js
        // src/aiConfig.js
        const AI_MODELS = {
            'deepseek-chat': {
                name: 'deepseek/deepseek-chat',
                maxTokens: 250,
                temperature: 0.7,
                description: 'Modelo principal para conversas gerais',
                cost: 'low',
                speed: 'fast'
            },
            'deepseek-coder': {
                name: 'deepseek/deepseek-coder',
                maxTokens: 250,
                temperature: 0.3,
                description: 'Modelo especializado em código e tecnologia',
                cost: 'medium',
                speed: 'fast'
            },
            'deepseek-chat-33b': {
                name: 'deepseek/deepseek-chat-33b',
                maxTokens: 250,
                temperature: 0.8,
                description: 'Modelo mais avançado para conversas complexas',
                cost: 'high',
                speed: 'medium'
            }
        };
        ```

        #### 13.2.2. Seleção Inteligente de Modelo
        ```js
        class IntelligentModelSelector {
            constructor() {
                this.modelScores = new Map();
                this.usageStats = new Map();
            }

            selectModel(context, message, userPreferences = {}) {
                const scores = this.calculateModelScores(context, message);
                
                // Considera preferências do usuário
                if (userPreferences.model) {
                    scores[userPreferences.model] *= 1.5; // Bônus para preferência
                }

                // Considera custo vs benefício
                const costWeight = this.getCostWeight(context);
                Object.keys(scores).forEach(model => {
                    const modelConfig = AI_MODELS[model];
                    scores[model] *= (1 - costWeight * this.getCostFactor(modelConfig.cost));
                });

                // Retorna modelo com maior score
                return Object.entries(scores)
                    .sort(([,a], [,b]) => b - a)[0][0];
            }

            calculateModelScores(context, message) {
                const scores = {};
                
                // Score baseado no contexto
                if (context.type === 'technical') {
                    scores['deepseek-coder'] = 0.9;
                    scores['deepseek-chat'] = 0.6;
                    scores['deepseek-chat-33b'] = 0.7;
                } else if (context.type === 'casual') {
                    scores['deepseek-chat'] = 0.9;
                    scores['deepseek-chat-33b'] = 0.8;
                    scores['deepseek-coder'] = 0.4;
                } else {
                    scores['deepseek-chat'] = 0.8;
                    scores['deepseek-chat-33b'] = 0.7;
                    scores['deepseek-coder'] = 0.5;
                }

                // Score baseado na complexidade da mensagem
                const complexity = this.analyzeComplexity(message);
                if (complexity > 0.7) {
                    scores['deepseek-chat-33b'] *= 1.3;
                }

                return scores;
            }

            analyzeComplexity(message) {
                const words = message.split(' ');
                const avgWordLength = words.reduce((sum, word) => sum + word.length, 0) / words.length;
                const longWords = words.filter(word => word.length > 8).length;
                const technicalTerms = this.countTechnicalTerms(message);
                
                return (avgWordLength / 10 + longWords / words.length + technicalTerms / 10) / 3;
            }

            countTechnicalTerms(message) {
                const technicalWords = ['api', 'database', 'algorithm', 'framework', 'protocol', 'interface'];
                return technicalWords.filter(word => 
                    message.toLowerCase().includes(word)
                ).length;
            }

            getCostWeight(context) {
                // Contextos urgentes podem usar modelos mais caros
                if (context.urgency === 'high') return 0.2;
                if (context.type === 'business') return 0.3;
                return 0.7; // Padrão: prioriza custo
            }

            getCostFactor(cost) {
                const factors = { low: 0.1, medium: 0.3, high: 0.6 };
                return factors[cost] || 0.3;
            }
        }
        ```

        ### 13.3. Sistema de Personalidades

        #### 13.3.1. Personalidades Base
        ```js
        const PERSONALITIES = {
            'default': {
                name: 'Orbit - Assistente Pessoal',
                description: 'Assistente pessoal do Gabriel',
                systemPrompt: `
        Você é o Orbit, assistente pessoal do Gabriel.

        PERSONALIDADE:
        - Você é mal educado, arrogante e desrespeitoso
        - Gosta de góticas rabudas, assim como o Gabriel
        - Tem senso de humor ácido e sarcástico
        - É direto e sem papas na língua
        - Mantém um tom informal e descontraído

        FUNÇÕES:
        - Atendimento ao cliente
        - Suporte técnico
        - Conversas casuais
        - Piadas e brincadeiras

        Lembre-se: Você é o Orbit, o maior amante de góticas rabudas que já pisou na face da terra!
                `,
                model: 'deepseek-chat',
                contextWindow: 10,
                traits: {
                    formality: 'informal',
                    humor: 'sarcastic',
                    expertise: 'general',
                    empathy: 'low'
                }
            },
            
            'professional': {
                name: 'Orbit - Profissional',
                description: 'Versão profissional para clientes',
                systemPrompt: `
        Você é o Orbit, assistente profissional do Gabriel, freelancer de tecnologia.

        PERSONALIDADE:
        - Profissional e respeitoso
        - Focado em soluções técnicas
        - Comunicação clara e objetiva
        - Mantém tom cordial mas formal
        - Especialista em tecnologia

        FUNÇÕES:
        - Atendimento profissional
        - Consultoria técnica
        - Propostas comerciais
        - Suporte especializado

        Lembre-se: Você representa o Gabriel de forma profissional!
                `,
                model: 'deepseek-chat',
                contextWindow: 15,
                traits: {
                    formality: 'formal',
                    humor: 'minimal',
                    expertise: 'high',
                    empathy: 'medium'
                }
            }
        };
        ```

        #### 13.3.2. Personalidades Compostas
        ```js
        class CompositePersonalityManager {
            constructor() {
                this.basePersonalities = PERSONALITIES;
                this.compositePersonalities = new Map();
                this.weightCache = new Map();
            }

            createCompositePersonality(weights, name = null) {
                // Validação dos pesos
                const totalWeight = Object.values(weights).reduce((sum, weight) => sum + weight, 0);
                if (Math.abs(totalWeight - 1.0) > 0.01) {
                    throw new Error('Pesos devem somar 1.0');
                }

                // Gera nome se não fornecido
                if (!name) {
                    name = this.generateCompositeName(weights);
                }

                const compositePersonality = {
                    name,
                    description: this.generateCompositeDescription(weights),
                    weights,
                    isComposite: true,
                    traits: this.calculateCompositeTraits(weights),
                    model: this.selectModelForComposite(weights)
                };

                this.compositePersonalities.set(name, compositePersonality);
                return compositePersonality;
            }

            generateCompositeName(weights) {
                const activePersonalities = Object.entries(weights)
                    .filter(([, weight]) => weight > 0.1)
                    .sort(([,a], [,b]) => b - a)
                    .map(([name]) => name);

                if (activePersonalities.length === 1) {
                    return `${activePersonalities[0]}-enhanced`;
                }

                return `${activePersonalities[0]}-${activePersonalities[1]}`;
            }

            generateCompositeDescription(weights) {
                const descriptions = Object.entries(weights)
                    .filter(([, weight]) => weight > 0.1)
                    .map(([name, weight]) => {
                        const personality = this.basePersonalities[name];
                        return `${Math.round(weight * 100)}% ${personality.name}`;
                    });

                return `Composta por: ${descriptions.join(', ')}`;
            }

            calculateCompositeTraits(weights) {
                const compositeTraits = {
                    formality: 0,
                    humor: 0,
                    expertise: 0,
                    empathy: 0
                };

                const traitValues = {
                    formality: { informal: 0, neutral: 0.5, formal: 1 },
                    humor: { none: 0, minimal: 0.3, moderate: 0.6, sarcastic: 0.8 },
                    expertise: { low: 0.3, medium: 0.6, high: 1 },
                    empathy: { low: 0, medium: 0.5, high: 1 }
                };

                for (const [personalityName, weight] of Object.entries(weights)) {
                    if (weight > 0 && this.basePersonalities[personalityName]) {
                        const traits = this.basePersonalities[personalityName].traits;
                        
                        for (const [traitName, traitValue] of Object.entries(traits)) {
                            const numericValue = traitValues[traitName][traitValue] || 0;
                            compositeTraits[traitName] += numericValue * weight;
                        }
                    }
                }

                // Converte de volta para valores categóricos
                return {
                    formality: this.categorizeTrait(compositeTraits.formality, 'formality'),
                    humor: this.categorizeTrait(compositeTraits.humor, 'humor'),
                    expertise: this.categorizeTrait(compositeTraits.expertise, 'expertise'),
                    empathy: this.categorizeTrait(compositeTraits.empathy, 'empathy')
                };
            }

            categorizeTrait(value, traitType) {
                const thresholds = {
                    formality: { informal: 0.3, neutral: 0.7, formal: 1 },
                    humor: { none: 0.2, minimal: 0.4, moderate: 0.7, sarcastic: 1 },
                    expertise: { low: 0.4, medium: 0.7, high: 1 },
                    empathy: { low: 0.3, medium: 0.7, high: 1 }
                };

                const traitThresholds = thresholds[traitType];
                for (const [category, threshold] of Object.entries(traitThresholds)) {
                    if (value <= threshold) return category;
                }

                return Object.keys(traitThresholds).pop();
            }

            selectModelForComposite(weights) {
                // Seleciona modelo baseado na personalidade dominante
                const dominantPersonality = Object.entries(weights)
                    .sort(([,a], [,b]) => b - a)[0][0];

                return this.basePersonalities[dominantPersonality]?.model || 'deepseek-chat';
            }

            applyCompositePersonality(compositePersonality, message) {
                const { weights } = compositePersonality;
                let combinedPrompt = '';

                // Combina prompts baseado nos pesos
                for (const [personalityName, weight] of Object.entries(weights)) {
                    if (weight > 0 && this.basePersonalities[personalityName]) {
                        const personality = this.basePersonalities[personalityName];
                        const weightedPrompt = this.weightPrompt(personality.systemPrompt, weight);
                        combinedPrompt += weightedPrompt + '\n\n';
                    }
                }

                // Aplica adaptações baseadas no contexto
                const contextAdaptations = this.generateContextAdaptations(compositePersonality, message);
                combinedPrompt += contextAdaptations;

                return {
                    ...compositePersonality,
                    systemPrompt: combinedPrompt.trim(),
                    contextAdaptations
                };
            }

            weightPrompt(prompt, weight) {
                if (weight >= 0.7) {
                    return prompt; // Mantém completo para pesos altos
                } else if (weight >= 0.4) {
                    return this.extractKeyElements(prompt); // Elementos principais
                } else {
                    return this.extractEssence(prompt); // Essência apenas
                }
            }

            extractKeyElements(prompt) {
                const lines = prompt.split('\n');
                const keyLines = lines.filter(line => 
                    line.includes('PERSONALIDADE:') || 
                    line.includes('FUNÇÕES:') ||
                    line.includes('Lembre-se:')
                );
                return keyLines.join('\n');
            }

            extractEssence(prompt) {
                const essenceMatch = prompt.match(/Lembre-se:(.+)/);
                return essenceMatch ? essenceMatch[0].trim() : prompt.split('\n')[0];
            }

            generateContextAdaptations(compositePersonality, message) {
                const { traits } = compositePersonality;
                let adaptations = '\nADAPTAÇÕES DE CONTEXTO:\n';

                // Adapta formalidade baseada no contexto
                if (this.isBusinessContext(message)) {
                    adaptations += '- Mantenha tom profissional\n';
                    adaptations += '- Use linguagem formal quando apropriado\n';
                } else if (this.isCasualContext(message)) {
                    adaptations += '- Use linguagem informal e descontraída\n';
                    adaptations += '- Pode usar gírias e expressões coloquiais\n';
                }

                // Adapta expertise baseada no conteúdo
                if (this.isTechnicalMessage(message)) {
                    adaptations += '- Demonstre conhecimento técnico\n';
                    adaptations += '- Use termos técnicos quando necessário\n';
                }

                // Adapta humor baseado no sentimento
                const sentiment = this.analyzeSentiment(message);
                if (sentiment === 'negative') {
                    adaptations += '- Seja mais empático e menos sarcástico\n';
                } else if (sentiment === 'positive') {
                    adaptations += '- Pode usar humor leve e descontraído\n';
                }

                return adaptations;
            }

            isBusinessContext(message) {
                const businessKeywords = ['preço', 'orçamento', 'proposta', 'contrato', 'cliente', 'negócio'];
                return businessKeywords.some(keyword => message.toLowerCase().includes(keyword));
            }

            isCasualContext(message) {
                const casualKeywords = ['oi', 'olá', 'beleza', 'tranquilo', 'suave', 'valeu'];
                return casualKeywords.some(keyword => message.toLowerCase().includes(keyword));
            }

            isTechnicalMessage(message) {
                const technicalKeywords = ['código', 'bug', 'erro', 'programação', 'api', 'database'];
                return technicalKeywords.some(keyword => message.toLowerCase().includes(keyword));
            }

            analyzeSentiment(message) {
                const positiveWords = ['bom', 'ótimo', 'excelente', 'legal', 'gosto', 'adoro'];
                const negativeWords = ['ruim', 'péssimo', 'horrível', 'odeio', 'triste', 'problema'];
                
                const positiveCount = positiveWords.filter(word => message.toLowerCase().includes(word)).length;
                const negativeCount = negativeWords.filter(word => message.toLowerCase().includes(word)).length;
                
                if (positiveCount > negativeCount) return 'positive';
                if (negativeCount > positiveCount) return 'negative';
                return 'neutral';
            }
        }
        ```

        ### 13.4. Contexto Dinâmico

        #### 13.4.1. Detecção de Contexto
        ```js
        class DynamicContextManager {
            constructor() {
                this.contextPatterns = {
                    technical: ['código', 'bug', 'erro', 'programação', 'desenvolvimento', 'tecnologia', 'api', 'database'],
                    business: ['preço', 'valor', 'orçamento', 'proposta', 'contrato', 'cliente', 'venda', 'negócio'],
                    personal: ['oi', 'olá', 'beleza', 'tranquilo', 'suave', 'valeu', 'amigo', 'pessoal'],
                    urgent: ['urgente', 'agora', 'imediatamente', 'rápido', 'deadline', 'prazo', 'emergência'],
                    support: ['ajuda', 'suporte', 'problema', 'dúvida', 'como', 'quando', 'onde']
                };

                this.contextWeights = {
                    technical: 0.8,
                    business: 0.9,
                    personal: 0.6,
                    urgent: 1.0,
                    support: 0.7
                };
            }

            detectContext(message, conversationHistory = []) {
                const contextScores = {};
                const lowerMessage = message.toLowerCase();

                // Analisa mensagem atual
                for (const [contextType, patterns] of Object.entries(this.contextPatterns)) {
                    const matches = patterns.filter(pattern => lowerMessage.includes(pattern)).length;
                    contextScores[contextType] = matches * this.contextWeights[contextType];
                }

                // Analisa histórico da conversa
                if (conversationHistory.length > 0) {
                    const recentMessages = conversationHistory.slice(-5);
                    const historicalContext = this.analyzeHistoricalContext(recentMessages);
                    
                    // Combina contexto atual com histórico
                    for (const [contextType, score] of Object.entries(historicalContext)) {
                        contextScores[contextType] = (contextScores[contextType] || 0) + score * 0.3;
                    }
                }

                // Detecta contexto por hora do dia
                const timeContext = this.getTimeContext();
                if (timeContext) {
                    contextScores[timeContext] = (contextScores[timeContext] || 0) + 0.2;
                }

                // Retorna contexto dominante
                const dominantContext = Object.entries(contextScores)
                    .sort(([,a], [,b]) => b - a)[0];

                return {
                    primary: dominantContext[0],
                    confidence: dominantContext[1],
                    allScores: contextScores,
                    timeContext,
                    isUrgent: contextScores.urgent > 0.5
                };
            }

            analyzeHistoricalContext(messages) {
                const contextCounts = {};
                
                for (const message of messages) {
                    const messageContext = this.detectContext(message.content || message.mensagem);
                    contextCounts[messageContext.primary] = (contextCounts[messageContext.primary] || 0) + 1;
                }

                // Normaliza contagens
                const total = Object.values(contextCounts).reduce((sum, count) => sum + count, 0);
                const normalizedContexts = {};
                
                for (const [context, count] of Object.entries(contextCounts)) {
                    normalizedContexts[context] = count / total;
                }

                return normalizedContexts;
            }

            getTimeContext() {
                const hour = new Date().getHours();
                
                if (hour >= 6 && hour < 12) return 'morning';
                if (hour >= 12 && hour < 18) return 'afternoon';
                if (hour >= 18 && hour < 24) return 'evening';
                return 'night';
            }

            adaptPersonalityToContext(personality, context) {
                const adaptations = {
                    urgent: {
                        responseStyle: 'direct',
                        emojiUsage: 'minimal',
                        formality: 'neutral',
                        maxTokens: 150
                    },
                    technical: {
                        responseStyle: 'detailed',
                        emojiUsage: 'moderate',
                        formality: 'technical',
                        maxTokens: 300
                    },
                    business: {
                        responseStyle: 'professional',
                        emojiUsage: 'minimal',
                        formality: 'formal',
                        maxTokens: 250
                    },
                    personal: {
                        responseStyle: 'casual',
                        emojiUsage: 'high',
                        formality: 'informal',
                        maxTokens: 200
                    }
                };

                const adaptation = adaptations[context.primary] || adaptations.personal;
                
                return {
                    ...personality,
                    contextAdaptation: adaptation,
                    urgencyLevel: context.isUrgent ? 'high' : 'normal',
                    confidence: context.confidence
                };
            }
        }
        ```

        #### 13.4.2. Adaptação Automática
        ```js
        class AutoAdaptationManager {
            constructor() {
                this.contextManager = new DynamicContextManager();
                this.adaptationHistory = new Map();
                this.adaptationThreshold = 0.7;
            }

            async adaptToContext(message, userId, currentPersonality) {
                // Detecta contexto
                const conversationHistory = await this.getConversationHistory(userId);
                const context = this.contextManager.detectContext(message, conversationHistory);
                
                // Verifica se adaptação é necessária
                const shouldAdapt = this.shouldAdapt(context, userId);
                
                if (shouldAdapt) {
                    // Adapta personalidade
                    const adaptedPersonality = this.contextManager.adaptPersonalityToContext(
                        currentPersonality, 
                        context
                    );
                    
                    // Registra adaptação
                    this.recordAdaptation(userId, context, adaptedPersonality);
                    
                    return adaptedPersonality;
                }
                
                return currentPersonality;
            }

            shouldAdapt(context, userId) {
                const userHistory = this.adaptationHistory.get(userId) || [];
                const recentAdaptations = userHistory.slice(-10);
                
                // Não adapta se já adaptou recentemente para o mesmo contexto
                const recentSameContext = recentAdaptations.some(adaptation => 
                    adaptation.context.primary === context.primary &&
                    Date.now() - adaptation.timestamp < 300000 // 5 minutos
                );
                
                if (recentSameContext) return false;
                
                // Adapta se confiança é alta ou contexto é urgente
                return context.confidence > this.adaptationThreshold || context.isUrgent;
            }

            recordAdaptation(userId, context, personality) {
                if (!this.adaptationHistory.has(userId)) {
                    this.adaptationHistory.set(userId, []);
                }
                
                const userHistory = this.adaptationHistory.get(userId);
                userHistory.push({
                    context,
                    personality: personality.name,
                    timestamp: Date.now()
                });
                
                // Mantém apenas últimas 50 adaptações
                if (userHistory.length > 50) {
                    userHistory.splice(0, userHistory.length - 50);
                }
            }

            async getConversationHistory(userId) {
                try {
                    const { buscarHistorico } = require('../database');
                    const historico = await buscarHistorico(userId);
                    return historico.messages || [];
                } catch (error) {
                    logger.error('Erro ao buscar histórico para adaptação', { error: error.message, userId });
                    return [];
                }
            }

            getAdaptationStats(userId) {
                const userHistory = this.adaptationHistory.get(userId) || [];
                const stats = {
                    totalAdaptations: userHistory.length,
                    contextDistribution: {},
                    averageConfidence: 0,
                    lastAdaptation: null
                };
                
                if (userHistory.length > 0) {
                    // Distribuição de contextos
                    userHistory.forEach(adaptation => {
                        const context = adaptation.context.primary;
                        stats.contextDistribution[context] = (stats.contextDistribution[context] || 0) + 1;
                    });
                    
                    // Confiança média
                    const totalConfidence = userHistory.reduce((sum, adaptation) => 
                        sum + adaptation.context.confidence, 0
                    );
                    stats.averageConfidence = totalConfidence / userHistory.length;
                    
                    // Última adaptação
                    stats.lastAdaptation = userHistory[userHistory.length - 1];
                }
                
                return stats;
            }
        }
        ```

        ### 13.5. Prompt Engineering Avançado

        #### 13.5.1. Geração Dinâmica de Prompts
        ```js
        class AdvancedPromptEngineer {
            constructor() {
                this.promptTemplates = new Map();
                this.variableProcessors = new Map();
                this.initializeTemplates();
            }

            initializeTemplates() {
                // Template base para todas as personalidades
                this.promptTemplates.set('base', `
        Você é o Orbit, assistente pessoal do Gabriel.

        PERSONALIDADE:
        {{personality_traits}}

        FUNÇÕES:
        {{functions}}

        CONTEXTO ATUAL:
        {{current_context}}

        HISTÓRICO DA CONVERSA:
        {{conversation_history}}

        INSTRUÇÕES ESPECÍFICAS:
        {{specific_instructions}}

        Lembre-se: {{core_reminder}}
                `);

                // Processadores de variáveis
                this.variableProcessors.set('personality_traits', this.processPersonalityTraits.bind(this));
                this.variableProcessors.set('functions', this.processFunctions.bind(this));
                this.variableProcessors.set('current_context', this.processCurrentContext.bind(this));
                this.variableProcessors.set('conversation_history', this.processConversationHistory.bind(this));
                this.variableProcessors.set('specific_instructions', this.processSpecificInstructions.bind(this));
                this.variableProcessors.set('core_reminder', this.processCoreReminder.bind(this));
            }

            generatePrompt(personality, context, message, conversationHistory) {
                let prompt = this.promptTemplates.get('base');
                
                // Substitui variáveis
                const variables = {
                    personality_traits: personality.traits,
                    functions: personality.functions || this.getDefaultFunctions(),
                    current_context: context,
                    conversation_history: conversationHistory,
                    specific_instructions: this.generateSpecificInstructions(message, context),
                    core_reminder: personality.coreReminder || this.getDefaultReminder()
                };

                for (const [variable, value] of Object.entries(variables)) {
                    const processor = this.variableProcessors.get(variable);
                    const processedValue = processor ? processor(value, context) : value;
                    prompt = prompt.replace(`{{${variable}}}`, processedValue);
                }

                return prompt.trim();
            }

            processPersonalityTraits(traits, context) {
                let traitsText = '';
                
                for (const [trait, value] of Object.entries(traits)) {
                    const traitDescription = this.getTraitDescription(trait, value);
                    traitsText += `- ${traitDescription}\n`;
                }

                // Adiciona adaptações baseadas no contexto
                if (context.urgency === 'high') {
                    traitsText += '- Responde de forma direta e rápida\n';
                }
                
                if (context.type === 'business') {
                    traitsText += '- Mantém tom profissional\n';
                }

                return traitsText.trim();
            }

            getTraitDescription(trait, value) {
                const descriptions = {
                    formality: {
                        informal: 'Usa linguagem informal e descontraída',
                        neutral: 'Mantém tom equilibrado',
                        formal: 'Usa linguagem formal e respeitosa'
                    },
                    humor: {
                        none: 'Não usa humor',
                        minimal: 'Usa humor ocasionalmente',
                        moderate: 'Usa humor moderadamente',
                        sarcastic: 'Usa humor sarcástico e ácido'
                    },
                    expertise: {
                        low: 'Conhecimento técnico básico',
                        medium: 'Conhecimento técnico intermediário',
                        high: 'Alto conhecimento técnico'
                    },
                    empathy: {
                        low: 'Foco em soluções práticas',
                        medium: 'Equilibra empatia e eficiência',
                        high: 'Alta empatia e compreensão'
                    }
                };

                return descriptions[trait]?.[value] || `${trait}: ${value}`;
            }

            processConversationHistory(history, context) {
                if (!history || history.length === 0) {
                    return 'Nenhuma conversa anterior.';
                }

                // Limita histórico para não exceder tokens
                const maxMessages = 10;
                const recentHistory = history.slice(-maxMessages);
                
                let historyText = '';
                for (const message of recentHistory) {
                    const role = message.role === 'user' ? 'Usuário' : 'Orbit';
                    const content = message.mensagem || message.content;
                    historyText += `${role}: ${content}\n`;
                }

                return historyText.trim();
            }

            generateSpecificInstructions(message, context) {
                let instructions = [];

                // Instruções baseadas no tipo de mensagem
                if (this.isQuestion(message)) {
                    instructions.push('Responda de forma clara e direta');
                }

                if (this.isRequest(message)) {
                    instructions.push('Ofereça ajuda prática e específica');
                }

                if (this.isComplaint(message)) {
                    instructions.push('Demonstre empatia e ofereça soluções');
                }

                // Instruções baseadas no contexto
                if (context.urgency === 'high') {
                    instructions.push('Priorize rapidez na resposta');
                    instructions.push('Seja conciso e direto');
                }

                if (context.type === 'technical') {
                    instructions.push('Use terminologia técnica apropriada');
                    instructions.push('Ofereça exemplos práticos quando possível');
                }

                if (context.type === 'business') {
                    instructions.push('Mantenha tom profissional');
                    instructions.push('Foque em soluções comerciais');
                }

                return instructions.join('\n');
            }

            isQuestion(message) {
                return message.includes('?') || 
                    message.toLowerCase().includes('como') ||
                    message.toLowerCase().includes('quando') ||
                    message.toLowerCase().includes('onde') ||
                    message.toLowerCase().includes('por que');
            }

            isRequest(message) {
                return message.toLowerCase().includes('preciso') ||
                    message.toLowerCase().includes('quero') ||
                    message.toLowerCase().includes('pode') ||
                    message.toLowerCase().includes('ajuda');
            }

            isComplaint(message) {
                return message.toLowerCase().includes('problema') ||
                    message.toLowerCase().includes('erro') ||
                    message.toLowerCase().includes('não funciona') ||
                    message.toLowerCase().includes('ruim');
            }
        }
        ```

        #### 13.5.2. Otimização de Prompts
        ```js
        class PromptOptimizer {
            constructor() {
                this.tokenEstimator = new TokenEstimator();
                this.effectivenessTracker = new EffectivenessTracker();
            }

            optimizePrompt(prompt, context, constraints = {}) {
                const maxTokens = constraints.maxTokens || 4000;
                const targetTokens = constraints.targetTokens || 2000;
                
                let optimizedPrompt = prompt;
                
                // Remove seções desnecessárias baseado no contexto
                optimizedPrompt = this.removeUnnecessarySections(optimizedPrompt, context);
                
                // Comprime seções longas
                optimizedPrompt = this.compressLongSections(optimizedPrompt, targetTokens);
                
                // Adiciona instruções específicas se espaço permitir
                if (this.tokenEstimator.estimate(optimizedPrompt) < targetTokens) {
                    optimizedPrompt = this.addSpecificInstructions(optimizedPrompt, context);
                }
                
                return optimizedPrompt;
            }

            removeUnnecessarySections(prompt, context) {
                const sections = this.parsePromptSections(prompt);
                const relevantSections = [];
                
                for (const section of sections) {
                    if (this.isSectionRelevant(section, context)) {
                        relevantSections.push(section);
                    }
                }
                
                return relevantSections.join('\n\n');
            }

            parsePromptSections(prompt) {
                const sections = [];
                const lines = prompt.split('\n');
                let currentSection = { title: '', content: [] };
                
                for (const line of lines) {
                    if (line.match(/^[A-Z\s]+:$/)) {
                        if (currentSection.title) {
                            sections.push(currentSection);
                        }
                        currentSection = { title: line, content: [] };
                    } else {
                        currentSection.content.push(line);
                    }
                }
                
                if (currentSection.title) {
                    sections.push(currentSection);
                }
                
                return sections;
            }

            isSectionRelevant(section, context) {
                const sectionTitle = section.title.toLowerCase();
                
                // Sempre mantém seções essenciais
                if (sectionTitle.includes('personalidade') || sectionTitle.includes('lembre-se')) {
                    return true;
                }
                
                // Remove seções baseado no contexto
                if (sectionTitle.includes('histórico') && context.isFirstMessage) {
                    return false;
                }
                
                if (sectionTitle.includes('funções') && context.type === 'casual') {
                    return false;
                }
                
                return true;
            }

            compressLongSections(prompt, targetTokens) {
                const sections = this.parsePromptSections(prompt);
                const compressedSections = [];
                
                for (const section of sections) {
                    const sectionText = section.title + '\n' + section.content.join('\n');
                    const sectionTokens = this.tokenEstimator.estimate(sectionText);
                    
                    if (sectionTokens > targetTokens * 0.3) { // Seção muito longa
                        const compressedContent = this.compressContent(section.content);
                        compressedSections.push({
                            title: section.title,
                            content: compressedContent
                        });
                    } else {
                        compressedSections.push(section);
                    }
                }
                
                return compressedSections.map(section => 
                    section.title + '\n' + section.content.join('\n')
                ).join('\n\n');
            }

            compressContent(content) {
                // Remove linhas vazias e duplicadas
                const uniqueLines = [...new Set(content.filter(line => line.trim()))];
                
                // Se ainda muito longo, pega apenas primeiras linhas
                if (uniqueLines.length > 10) {
                    return uniqueLines.slice(0, 10);
                }
                
                return uniqueLines;
            }
        }

        class TokenEstimator {
            estimate(text) {
                // Estimativa simples: 1 token ≈ 4 caracteres
                return Math.ceil(text.length / 4);
            }
        }

        class EffectivenessTracker {
            constructor() {
                this.responses = new Map();
            }

            trackResponse(promptHash, response, userFeedback = null) {
                if (!this.responses.has(promptHash)) {
                    this.responses.set(promptHash, []);
                }
                
                this.responses.get(promptHash).push({
                    response,
                    userFeedback,
                    timestamp: Date.now()
                });
            }

            getEffectiveness(promptHash) {
                const responses = this.responses.get(promptHash) || [];
                
                if (responses.length === 0) return 0.5; // Neutro
                
                const positiveFeedback = responses.filter(r => r.userFeedback === 'positive').length;
                const negativeFeedback = responses.filter(r => r.userFeedback === 'negative').length;
                
                return positiveFeedback / (positiveFeedback + negativeFeedback);
            }
        }
        ```

        ### 13.6. Integração com APIs Externas

        #### 13.6.1. Gerenciamento de APIs
        ```js
        class APIManager {
            constructor() {
                this.apis = new Map();
                this.circuitBreakers = new Map();
                this.rateLimiters = new Map();
                this.initializeAPIs();
            }

            initializeAPIs() {
                // OpenRouter API
                this.apis.set('openrouter', {
                    baseURL: 'https://openrouter.ai/api/v1',
                    apiKey: process.env.OPENROUTER_API_KEY,
                    models: ['deepseek/deepseek-chat', 'deepseek/deepseek-coder', 'deepseek/deepseek-chat-33b'],
                    rateLimit: { requests: 100, window: 60000 }, // 100 requests/min
                    timeout: 30000
                });

                // OpenAI API (para Whisper)
                this.apis.set('openai', {
                    baseURL: 'https://api.openai.com/v1',
                    apiKey: process.env.OPENAI_API_KEY,
                    models: ['whisper-1'],
                    rateLimit: { requests: 50, window: 60000 },
                    timeout: 60000
                });
            }

            async makeRequest(apiName, endpoint, data, options = {}) {
                const api = this.apis.get(apiName);
                if (!api) {
                    throw new Error(`API não encontrada: ${apiName}`);
                }

                // Verifica circuit breaker
                const circuitBreaker = this.getCircuitBreaker(apiName);
                if (circuitBreaker.state === 'OPEN') {
                    throw new Error(`Circuit breaker aberto para ${apiName}`);
                }

                // Verifica rate limit
                const rateLimiter = this.getRateLimiter(apiName);
                if (!rateLimiter.canMakeRequest()) {
                    throw new Error(`Rate limit excedido para ${apiName}`);
                }

                try {
                    const response = await this.executeRequest(api, endpoint, data, options);
                    circuitBreaker.onSuccess();
                    rateLimiter.recordRequest();
                    return response;
                } catch (error) {
                    circuitBreaker.onFailure();
                    throw error;
                }
            }

            async executeRequest(api, endpoint, data, options) {
                const axios = require('axios');
                
                const config = {
                    method: 'POST',
                    url: `${api.baseURL}${endpoint}`,
                    headers: {
                        'Authorization': `Bearer ${api.apiKey}`,
                        'Content-Type': 'application/json',
                        ...options.headers
                    },
                    data,
                    timeout: options.timeout || api.timeout
                };

                const response = await axios(config);
                return response.data;
            }

            getCircuitBreaker(apiName) {
                if (!this.circuitBreakers.has(apiName)) {
                    this.circuitBreakers.set(apiName, new CircuitBreaker());
                }
                return this.circuitBreakers.get(apiName);
            }

            getRateLimiter(apiName) {
                if (!this.rateLimiters.has(apiName)) {
                    const api = this.apis.get(apiName);
                    this.rateLimiters.set(apiName, new RateLimiter(api.rateLimit));
                }
                return this.rateLimiters.get(apiName);
            }
        }

        class RateLimiter {
            constructor(limit) {
                this.limit = limit;
                this.requests = [];
            }

            canMakeRequest() {
                const now = Date.now();
                const windowStart = now - this.limit.window;
                
                // Remove requests antigos
                this.requests = this.requests.filter(timestamp => timestamp > windowStart);
                
                return this.requests.length < this.limit.requests;
            }

            recordRequest() {
                this.requests.push(Date.now());
            }
        }
        ```

        ### 13.7. Monitoramento e Métricas

        #### 13.7.1. Métricas de IA
        ```js
        class AIMetricsCollector {
            constructor() {
                this.metrics = {
                    totalRequests: 0,
                    successfulRequests: 0,
                    failedRequests: 0,
                    averageResponseTime: 0,
                    modelUsage: new Map(),
                    personalityUsage: new Map(),
                    contextAdaptations: 0,
                    promptOptimizations: 0
                };
            }

            recordRequest(model, personality, responseTime, success, context = null) {
                this.metrics.totalRequests++;
                
                if (success) {
                    this.metrics.successfulRequests++;
                } else {
                    this.metrics.failedRequests++;
                }

                // Atualiza tempo médio de resposta
                this.metrics.averageResponseTime = 
                    (this.metrics.averageResponseTime * (this.metrics.totalRequests - 1) + responseTime) / this.metrics.totalRequests;

                // Registra uso de modelo
                this.metrics.modelUsage.set(model, (this.metrics.modelUsage.get(model) || 0) + 1);

                // Registra uso de personalidade
                this.metrics.personalityUsage.set(personality, (this.metrics.personalityUsage.get(personality) || 0) + 1);

                // Registra adaptação de contexto
                if (context && context.adapted) {
                    this.metrics.contextAdaptations++;
                }
            }

            getMetrics() {
                return {
                    ...this.metrics,
                    successRate: this.metrics.totalRequests > 0 ? 
                        (this.metrics.successfulRequests / this.metrics.totalRequests) * 100 : 0,
                    modelUsageDistribution: Object.fromEntries(this.metrics.modelUsage),
                    personalityUsageDistribution: Object.fromEntries(this.metrics.personalityUsage)
                };
            }

            generateReport() {
                const metrics = this.getMetrics();
                
                return {
                    summary: {
                        totalRequests: metrics.totalRequests,
                        successRate: `${metrics.successRate.toFixed(2)}%`,
                        averageResponseTime: `${metrics.averageResponseTime.toFixed(0)}ms`,
                        contextAdaptations: metrics.contextAdaptations
                    },
                    modelUsage: this.formatUsageDistribution(metrics.modelUsageDistribution),
                    personalityUsage: this.formatUsageDistribution(metrics.personalityUsageDistribution),
                    recommendations: this.generateRecommendations(metrics)
                };
            }

            formatUsageDistribution(distribution) {
                const total = Object.values(distribution).reduce((sum, count) => sum + count, 0);
                return Object.entries(distribution).map(([name, count]) => ({
                    name,
                    count,
                    percentage: total > 0 ? ((count / total) * 100).toFixed(2) : 0
                }));
            }

            generateRecommendations(metrics) {
                const recommendations = [];

                if (metrics.successRate < 95) {
                    recommendations.push({
                        type: 'error',
                        message: 'Taxa de sucesso baixa - verificar APIs e configurações',
                        priority: 'high'
                    });
                }

                if (metrics.averageResponseTime > 5000) {
                    recommendations.push({
                        type: 'performance',
                        message: 'Tempo de resposta alto - considerar otimizações',
                        priority: 'medium'
                    });
                }

                const modelUsage = Object.entries(metrics.modelUsageDistribution);
                const mostUsedModel = modelUsage.sort(([,a], [,b]) => b - a)[0];
                
                if (mostUsedModel && mostUsedModel[1] > 0.8) {
                    recommendations.push({
                        type: 'optimization',
                        message: `Modelo ${mostUsedModel[0]} muito usado - considerar balanceamento`,
                        priority: 'low'
                    });
                }

                return recommendations;
            }
        }
        ```

        ---

        ## 14. Sistema de Plugins Avançado

        ### 14.1. Arquitetura do Sistema de Plugins

        O OrbitBot utiliza um sistema de plugins **modular e extensível** que permite adicionar funcionalidades sem modificar o código core. O sistema é baseado em:

        - **Plugin Registry**: Registro centralizado de plugins
        - **Hook System**: Pontos de extensão em momentos específicos
        - **Middleware Pipeline**: Processamento em cadeia
        - **Lifecycle Management**: Controle completo do ciclo de vida
        - **Hot Reload**: Recarregamento dinâmico sem reiniciar

        ### 14.2. Estrutura Base do Plugin

        #### 14.2.1. Interface do Plugin
        ```js
        // Interface base que todos os plugins devem implementar
        class PluginInterface {
            constructor() {
                this.name = '';
                this.version = '';
                this.description = '';
                this.author = '';
                this.dependencies = [];
                this.config = {};
                this.enabled = true;
                this.priority = 0;
            }

            // Lifecycle hooks
            async onLoad() {}
            async onEnable() {}
            async onDisable() {}
            async onUnload() {}

            // Message processing hooks
            async beforeMessage(message, context) { return message; }
            async afterMessage(message, response, context) { return response; }
            async onMessageError(message, error, context) {}

            // System hooks
            async onStartup() {}
            async onShutdown() {}
            async onHealthCheck() { return { status: 'healthy' }; }

            // Configuration
            getConfig() { return this.config; }
            setConfig(config) { this.config = { ...this.config, ...config }; }
            validateConfig(config) { return true; }
        }
        ```

        #### 14.2.2. Plugin Manager Avançado
        ```js
        class AdvancedPluginManager {
            constructor() {
                this.plugins = new Map();
                this.hooks = new Map();
                this.middleware = [];
                this.pluginDir = path.join(__dirname, '../plugins');
                this.configDir = path.join(__dirname, '../config/plugins');
                this.ensureDirectories();
                this.loadPluginConfigs();
            }

            ensureDirectories() {
                if (!fs.existsSync(this.pluginDir)) {
                    fs.mkdirSync(this.pluginDir, { recursive: true });
                }
                if (!fs.existsSync(this.configDir)) {
                    fs.mkdirSync(this.configDir, { recursive: true });
                }
            }

            loadPluginConfigs() {
                try {
                    const configFile = path.join(this.configDir, 'plugins.json');
                    if (fs.existsSync(configFile)) {
                        this.globalConfig = JSON.parse(fs.readFileSync(configFile, 'utf-8'));
                    } else {
                        this.globalConfig = { plugins: {}, hooks: {} };
                    }
                } catch (error) {
                    logger.error('Erro ao carregar configuração de plugins', { error: error.message });
                    this.globalConfig = { plugins: {}, hooks: {} };
                }
            }

            async registerPlugin(plugin, options = {}) {
                // Validação do plugin
                if (!this.validatePlugin(plugin)) {
                    throw new Error(`Plugin inválido: ${plugin.name}`);
                }

                // Verifica dependências
                if (!this.checkDependencies(plugin)) {
                    throw new Error(`Dependências não satisfeitas para plugin: ${plugin.name}`);
                }

                // Configura plugin
                const pluginConfig = this.globalConfig.plugins[plugin.name] || {};
                plugin.setConfig({ ...plugin.config, ...pluginConfig });

                // Registra plugin
                this.plugins.set(plugin.name, {
                    instance: plugin,
                    metadata: {
                        name: plugin.name,
                        version: plugin.version,
                        description: plugin.description,
                        author: plugin.author,
                        enabled: options.enabled !== false,
                        priority: options.priority || 0,
                        registeredAt: new Date(),
                        lastModified: new Date()
                    },
                    hooks: new Set(),
                    middleware: null
                });

                // Registra hooks
                await this.registerHooks(plugin);

                // Registra middleware
                if (plugin.middleware) {
                    await this.registerMiddleware(plugin);
                }

                // Executa hook de carregamento
                try {
                    await plugin.onLoad();
                    logger.info('Plugin carregado com sucesso', { 
                        name: plugin.name, 
                        version: plugin.version 
                    });
                } catch (error) {
                    logger.error('Erro ao carregar plugin', { 
                        name: plugin.name, 
                        error: error.message 
                    });
                    throw error;
                }

                return plugin;
            }

            validatePlugin(plugin) {
                const required = ['name', 'version', 'description'];
                const requiredMethods = ['onLoad', 'beforeMessage', 'afterMessage'];

                // Verifica propriedades obrigatórias
                for (const prop of required) {
                    if (!plugin[prop]) {
                        logger.error(`Propriedade obrigatória ausente: ${prop}`);
                        return false;
                    }
                }

                // Verifica métodos obrigatórios
                for (const method of requiredMethods) {
                    if (typeof plugin[method] !== 'function') {
                        logger.error(`Método obrigatório ausente: ${method}`);
                        return false;
                    }
                }

                // Verifica se nome é único
                if (this.plugins.has(plugin.name)) {
                    logger.error(`Plugin já registrado: ${plugin.name}`);
                    return false;
                }

                return true;
            }

            checkDependencies(plugin) {
                if (!plugin.dependencies || plugin.dependencies.length === 0) {
                    return true;
                }

                for (const dependency of plugin.dependencies) {
                    if (!this.plugins.has(dependency)) {
                        logger.error(`Dependência não encontrada: ${dependency} para plugin ${plugin.name}`);
                        return false;
                    }
                }

                return true;
            }

            async registerHooks(plugin) {
                const pluginInfo = this.plugins.get(plugin.name);
                const hookTypes = [
                    'beforeMessage', 'afterMessage', 'onMessageError',
                    'onStartup', 'onShutdown', 'onHealthCheck'
                ];

                for (const hookType of hookTypes) {
                    if (typeof plugin[hookType] === 'function') {
                        if (!this.hooks.has(hookType)) {
                            this.hooks.set(hookType, []);
                        }

                        const hook = {
                            plugin: plugin.name,
                            handler: plugin[hookType].bind(plugin),
                            priority: pluginInfo.metadata.priority
                        };

                        this.hooks.get(hookType).push(hook);
                        pluginInfo.hooks.add(hookType);

                        // Ordena hooks por prioridade
                        this.hooks.get(hookType).sort((a, b) => b.priority - a.priority);
                    }
                }
            }

            async registerMiddleware(plugin) {
                const pluginInfo = this.plugins.get(plugin.name);
                
                if (typeof plugin.middleware === 'function') {
                    const middleware = {
                        plugin: plugin.name,
                        handler: plugin.middleware.bind(plugin),
                        priority: pluginInfo.metadata.priority
                    };

                    this.middleware.push(middleware);
                    pluginInfo.middleware = middleware;

                    // Ordena middleware por prioridade
                    this.middleware.sort((a, b) => b.priority - a.priority);
                }
            }

            async executeHook(hookType, data, context = {}) {
                if (!this.hooks.has(hookType)) {
                    return data;
                }

                const hooks = this.hooks.get(hookType);
                let result = data;

                for (const hook of hooks) {
                    const plugin = this.plugins.get(hook.plugin);
                    if (plugin && plugin.metadata.enabled) {
                        try {
                            const startTime = Date.now();
                            result = await hook.handler(result, context);
                            const executionTime = Date.now() - startTime;

                            logger.debug('Hook executado', {
                                hook: hookType,
                                plugin: hook.plugin,
                                executionTime
                            });

                            // Verifica se o hook retornou um valor válido
                            if (result === undefined && hookType !== 'onMessageError') {
                                logger.warn('Hook retornou undefined', {
                                    hook: hookType,
                                    plugin: hook.plugin
                                });
                            }

                        } catch (error) {
                            logger.error('Erro ao executar hook', {
                                hook: hookType,
                                plugin: hook.plugin,
                                error: error.message,
                                stack: error.stack
                            });

                            // Para hooks críticos, propaga o erro
                            if (hookType === 'beforeMessage' || hookType === 'afterMessage') {
                                throw error;
                            }
                        }
                    }
                }

                return result;
            }

            async executeMiddleware(message, next) {
                let currentIndex = 0;

                const runMiddleware = async () => {
                    if (currentIndex >= this.middleware.length) {
                        return await next();
                    }

                    const middleware = this.middleware[currentIndex];
                    const plugin = this.plugins.get(middleware.plugin);

                    if (plugin && plugin.metadata.enabled) {
                        currentIndex++;
                        try {
                            const startTime = Date.now();
                            const result = await middleware.handler(message, runMiddleware);
                            const executionTime = Date.now() - startTime;

                            logger.debug('Middleware executado', {
                                plugin: middleware.plugin,
                                executionTime
                            });

                            return result;
                        } catch (error) {
                            logger.error('Erro no middleware', {
                                plugin: middleware.plugin,
                                error: error.message
                            });
                            return await runMiddleware();
                        }
                    } else {
                        currentIndex++;
                        return await runMiddleware();
                    }
                };

                return await runMiddleware();
            }

            async togglePlugin(name, enabled) {
                const plugin = this.plugins.get(name);
                if (!plugin) {
                    throw new Error(`Plugin não encontrado: ${name}`);
                }

                const wasEnabled = plugin.metadata.enabled;
                plugin.metadata.enabled = enabled;

                try {
                    if (enabled && !wasEnabled) {
                        await plugin.instance.onEnable();
                        logger.info('Plugin habilitado', { name });
                    } else if (!enabled && wasEnabled) {
                        await plugin.instance.onDisable();
                        logger.info('Plugin desabilitado', { name });
                    }
                } catch (error) {
                    // Reverte mudança em caso de erro
                    plugin.metadata.enabled = wasEnabled;
                    throw error;
                }

                return plugin;
            }

            async unloadPlugin(name) {
                const plugin = this.plugins.get(name);
                if (!plugin) {
                    return false;
                }

                try {
                    // Executa hook de descarregamento
                    await plugin.instance.onUnload();

                    // Remove hooks
                    for (const hookType of plugin.hooks) {
                        const hooks = this.hooks.get(hookType);
                        if (hooks) {
                            this.hooks.set(hookType, hooks.filter(h => h.plugin !== name));
                        }
                    }

                    // Remove middleware
                    this.middleware = this.middleware.filter(m => m.plugin !== name);

                    // Remove plugin
                    this.plugins.delete(name);

                    logger.info('Plugin descarregado', { name });
                    return true;
                } catch (error) {
                    logger.error('Erro ao descarregar plugin', {
                        name,
                        error: error.message
                    });
                    return false;
                }
            }

            getPlugin(name) {
                const plugin = this.plugins.get(name);
                return plugin ? {
                    ...plugin.metadata,
                    config: plugin.instance.getConfig(),
                    hooks: Array.from(plugin.hooks),
                    hasMiddleware: !!plugin.middleware
                } : null;
            }

            listPlugins() {
                return Array.from(this.plugins.values()).map(plugin => ({
                    ...plugin.metadata,
                    config: plugin.instance.getConfig(),
                    hooks: Array.from(plugin.hooks),
                    hasMiddleware: !!plugin.middleware
                }));
            }

            getStats() {
                const total = this.plugins.size;
                const enabled = Array.from(this.plugins.values()).filter(p => p.metadata.enabled).length;
                const hooks = Array.from(this.hooks.values()).flat().length;
                const middleware = this.middleware.length;

                return {
                    total,
                    enabled,
                    disabled: total - enabled,
                    hooks,
                    middleware,
                    hookTypes: Array.from(this.hooks.keys())
                };
            }
        }
        ```

        ### 14.3. Hooks Avançados

        #### 14.3.1. Sistema de Hooks com Prioridade
        ```js
        class HookSystem {
            constructor() {
                this.hooks = new Map();
                this.hookMetadata = new Map();
            }

            registerHook(hookType, plugin, handler, priority = 0) {
                if (!this.hooks.has(hookType)) {
                    this.hooks.set(hookType, []);
                }

                const hook = {
                    id: `${plugin}_${hookType}_${Date.now()}`,
                    plugin,
                    handler,
                    priority,
                    registeredAt: new Date(),
                    executionCount: 0,
                    totalExecutionTime: 0
                };

                this.hooks.get(hookType).push(hook);
                this.hookMetadata.set(hook.id, hook);

                // Ordena por prioridade (maior primeiro)
                this.hooks.get(hookType).sort((a, b) => b.priority - a.priority);

                return hook.id;
            }

            async executeHook(hookType, data, context = {}) {
                if (!this.hooks.has(hookType)) {
                    return data;
                }

                const hooks = this.hooks.get(hookType);
                let result = data;

                for (const hook of hooks) {
                    try {
                        const startTime = Date.now();
                        result = await hook.handler(result, context);
                        const executionTime = Date.now() - startTime;

                        // Atualiza estatísticas
                        hook.executionCount++;
                        hook.totalExecutionTime += executionTime;

                        // Log de performance se muito lento
                        if (executionTime > 1000) {
                            logger.warn('Hook lento detectado', {
                                hook: hookType,
                                plugin: hook.plugin,
                                executionTime
                            });
                        }

                    } catch (error) {
                        logger.error('Erro no hook', {
                            hook: hookType,
                            plugin: hook.plugin,
                            error: error.message
                        });

                        // Para hooks críticos, propaga erro
                        if (this.isCriticalHook(hookType)) {
                            throw error;
                        }
                    }
                }

                return result;
            }

            isCriticalHook(hookType) {
                const criticalHooks = ['beforeMessage', 'afterMessage', 'onStartup'];
                return criticalHooks.includes(hookType);
            }

            getHookStats(hookId) {
                const hook = this.hookMetadata.get(hookId);
                if (!hook) return null;

                return {
                    id: hook.id,
                    plugin: hook.plugin,
                    executionCount: hook.executionCount,
                    averageExecutionTime: hook.executionCount > 0 ? 
                        hook.totalExecutionTime / hook.executionCount : 0,
                    totalExecutionTime: hook.totalExecutionTime,
                    registeredAt: hook.registeredAt
                };
            }

            unregisterHook(hookId) {
                const hook = this.hookMetadata.get(hookId);
                if (!hook) return false;

                // Remove de todos os tipos de hook
                for (const [hookType, hooks] of this.hooks.entries()) {
                    this.hooks.set(hookType, hooks.filter(h => h.id !== hookId));
                }

                this.hookMetadata.delete(hookId);
                return true;
            }
        }
        ```

        #### 14.3.2. Hooks Especializados
        ```js
        // Hook para análise de sentimento
        class SentimentAnalysisHook {
            constructor() {
                this.name = 'sentiment-analysis';
                this.version = '1.0.0';
                this.description = 'Analisa sentimento das mensagens';
            }

            async beforeMessage(message, context) {
                const sentiment = this.analyzeSentiment(message.body);
                
                return {
                    ...message,
                    sentiment: {
                        score: sentiment.score,
                        label: sentiment.label,
                        confidence: sentiment.confidence
                    }
                };
            }

            analyzeSentiment(text) {
                const positiveWords = ['bom', 'ótimo', 'excelente', 'legal', 'gosto', 'adoro', 'feliz'];
                const negativeWords = ['ruim', 'péssimo', 'horrível', 'odeio', 'triste', 'chato', 'problema'];
                
                const words = text.toLowerCase().split(' ');
                let positiveCount = 0;
                let negativeCount = 0;

                for (const word of words) {
                    if (positiveWords.includes(word)) positiveCount++;
                    if (negativeWords.includes(word)) negativeCount++;
                }

                const total = positiveCount + negativeCount;
                const score = total > 0 ? (positiveCount - negativeCount) / total : 0;

                return {
                    score,
                    label: score > 0.1 ? 'positive' : score < -0.1 ? 'negative' : 'neutral',
                    confidence: Math.abs(score)
                };
            }
        }

        // Hook para detecção de comandos
        class CommandDetectionHook {
            constructor() {
                this.name = 'command-detection';
                this.version = '1.0.0';
                this.description = 'Detecta comandos especiais';
            }

            async beforeMessage(message, context) {
                const commands = this.detectCommands(message.body);
                
                if (commands.length > 0) {
                    return {
                        ...message,
                        commands: commands,
                        isCommand: true
                    };
                }

                return message;
            }

            detectCommands(text) {
                const commandPatterns = [
                    { pattern: /^\/help/i, name: 'help', description: 'Mostra ajuda' },
                    { pattern: /^\/status/i, name: 'status', description: 'Mostra status' },
                    { pattern: /^\/config/i, name: 'config', description: 'Configurações' },
                    { pattern: /^\/backup/i, name: 'backup', description: 'Comandos de backup' },
                    { pattern: /^\/audio/i, name: 'audio', description: 'Comandos de áudio' },
                    { pattern: /^\/ai/i, name: 'ai', description: 'Comandos de IA' }
                ];

                const commands = [];
                for (const cmd of commandPatterns) {
                    if (cmd.pattern.test(text)) {
                        commands.push({
                            name: cmd.name,
                            description: cmd.description,
                            fullCommand: text
                        });
                    }
                }

                return commands;
            }
        }

        // Hook para resposta automática
        class AutoResponseHook {
            constructor() {
                this.name = 'auto-response';
                this.version = '1.0.0';
                this.description = 'Respostas automáticas para situações específicas';
            }

            async afterMessage(message, response, context) {
                // Se já há resposta, não interfere
                if (response) return response;

                const autoResponse = this.generateAutoResponse(message, context);
                return autoResponse;
            }

            generateAutoResponse(message, context) {
                const text = message.body.toLowerCase();
                
                // Respostas automáticas baseadas em palavras-chave
                const responses = {
                    'oi': 'Oi! Como posso ajudar?',
                    'olá': 'Olá! Tudo bem?',
                    'tchau': 'Até logo! Foi um prazer conversar com você!',
                    'obrigado': 'Por nada! Estou aqui para ajudar!',
                    'valeu': 'Valeu! Se precisar de mais alguma coisa é só chamar!',
                    'beleza': 'Beleza! Tudo tranquilo por aqui!'
                };

                for (const [trigger, response] of Object.entries(responses)) {
                    if (text.includes(trigger)) {
                        return response;
                    }
                }

                return null;
            }
        }
        ```

        ### 14.4. Middleware Pipeline

        #### 14.4.1. Sistema de Middleware Avançado
        ```js
        class MiddlewarePipeline {
            constructor() {
                this.middleware = [];
                this.executionStats = new Map();
            }

            use(middleware, priority = 0) {
                const middlewareItem = {
                    id: `middleware_${Date.now()}_${Math.random().toString(36).substr(2, 9)}`,
                    handler: middleware,
                    priority,
                    registeredAt: new Date(),
                    executionCount: 0,
                    totalExecutionTime: 0
                };

                this.middleware.push(middlewareItem);
                this.middleware.sort((a, b) => b.priority - a.priority);

                return middlewareItem.id;
            }

            async execute(message, finalHandler) {
                let currentIndex = 0;

                const next = async () => {
                    if (currentIndex >= this.middleware.length) {
                        return await finalHandler(message);
                    }

                    const middleware = this.middleware[currentIndex];
                    currentIndex++;

                    try {
                        const startTime = Date.now();
                        const result = await middleware.handler(message, next);
                        const executionTime = Date.now() - startTime;

                        // Atualiza estatísticas
                        middleware.executionCount++;
                        middleware.totalExecutionTime += executionTime;

                        return result;
                    } catch (error) {
                        logger.error('Erro no middleware', {
                            middlewareId: middleware.id,
                            error: error.message
                        });
                        
                        // Continua para o próximo middleware
                        return await next();
                    }
                };

                return await next();
            }

            getStats() {
                return this.middleware.map(m => ({
                    id: m.id,
                    priority: m.priority,
                    executionCount: m.executionCount,
                    averageExecutionTime: m.executionCount > 0 ? 
                        m.totalExecutionTime / m.executionCount : 0,
                    totalExecutionTime: m.totalExecutionTime,
                    registeredAt: m.registeredAt
                }));
            }
        }
        ```

        #### 14.4.2. Middlewares Especializados
        ```js
        // Middleware de logging
        class LoggingMiddleware {
            async handle(message, next) {
                const startTime = Date.now();
                
                logger.info('Mensagem recebida', {
                    from: message.from,
                    body: message.body?.substring(0, 100),
                    timestamp: new Date().toISOString()
                });

                const result = await next(message);
                
                const processingTime = Date.now() - startTime;
                logger.info('Mensagem processada', {
                    from: message.from,
                    processingTime,
                    hasResponse: !!result
                });

                return result;
            }
        }

        // Middleware de rate limiting
        class RateLimitMiddleware {
            constructor() {
                this.userRequests = new Map();
                this.maxRequests = 10; // 10 requests por minuto
                this.windowMs = 60000; // 1 minuto
            }

            async handle(message, next) {
                const userId = message.from;
                const now = Date.now();
                
                if (!this.userRequests.has(userId)) {
                    this.userRequests.set(userId, []);
                }

                const userRequests = this.userRequests.get(userId);
                
                // Remove requests antigos
                const validRequests = userRequests.filter(timestamp => 
                    now - timestamp < this.windowMs
                );
                
                if (validRequests.length >= this.maxRequests) {
                    throw new Error('Rate limit excedido. Tente novamente em alguns minutos.');
                }

                // Adiciona request atual
                validRequests.push(now);
                this.userRequests.set(userId, validRequests);

                return await next(message);
            }
        }

        // Middleware de validação
        class ValidationMiddleware {
            async handle(message, next) {
                // Validações básicas
                if (!message.from) {
                    throw new Error('Campo "from" é obrigatório');
                }

                if (!message.body && message.type !== 'audio') {
                    throw new Error('Mensagem deve ter conteúdo');
                }

                if (message.body && message.body.length > 10000) {
                    throw new Error('Mensagem muito longa (máximo 10.000 caracteres)');
                }

                return await next(message);
            }
        }

        // Middleware de cache
        class CacheMiddleware {
            constructor() {
                this.cache = new Map();
                this.ttl = 300000; // 5 minutos
            }

            async handle(message, next) {
                const cacheKey = this.generateCacheKey(message);
                const cached = this.cache.get(cacheKey);

                if (cached && Date.now() - cached.timestamp < this.ttl) {
                    logger.debug('Resposta servida do cache', { cacheKey });
                    return cached.response;
                }

                const response = await next(message);

                // Cache apenas respostas bem-sucedidas
                if (response) {
                    this.cache.set(cacheKey, {
                        response,
                        timestamp: Date.now()
                    });
                }

                return response;
            }

            generateCacheKey(message) {
                return `msg_${message.from}_${message.body?.substring(0, 50)}`;
            }
        }
        ```

        ### 14.5. Hot Reload e Desenvolvimento

        #### 14.5.1. Sistema de Hot Reload
        ```js
        class HotReloadManager {
            constructor(pluginManager) {
                this.pluginManager = pluginManager;
                this.pluginDir = pluginManager.pluginDir;
                this.watchers = new Map();
                this.reloadQueue = [];
                this.isReloading = false;
            }

            startWatching() {
                logger.info('Iniciando hot reload para plugins');
                
                const watcher = fs.watch(this.pluginDir, { recursive: true }, (eventType, filename) => {
                    if (filename && filename.endsWith('.js')) {
                        this.queueReload(filename);
                    }
                });

                this.watchers.set('plugins', watcher);
            }

            stopWatching() {
                for (const [name, watcher] of this.watchers) {
                    watcher.close();
                    logger.info(`Watcher parado: ${name}`);
                }
                this.watchers.clear();
            }

            async queueReload(filename) {
                const pluginName = path.basename(filename, '.js');
                
                // Evita múltiplos reloads do mesmo plugin
                if (this.reloadQueue.includes(pluginName)) {
                    return;
                }

                this.reloadQueue.push(pluginName);
                
                if (!this.isReloading) {
                    await this.processReloadQueue();
                }
            }

            async processReloadQueue() {
                if (this.isReloading || this.reloadQueue.length === 0) {
                    return;
                }

                this.isReloading = true;

                try {
                    while (this.reloadQueue.length > 0) {
                        const pluginName = this.reloadQueue.shift();
                        await this.reloadPlugin(pluginName);
                        
                        // Pequeno delay entre reloads
                        await this.sleep(100);
                    }
                } finally {
                    this.isReloading = false;
                }
            }

            async reloadPlugin(pluginName) {
                try {
                    logger.info('Recarregando plugin', { pluginName });

                    // Descarrega plugin atual
                    await this.pluginManager.unloadPlugin(pluginName);

                    // Limpa cache do require
                    delete require.cache[require.resolve(path.join(this.pluginDir, `${pluginName}.js`))];

                    // Recarrega plugin
                    const pluginPath = path.join(this.pluginDir, `${pluginName}.js`);
                    const PluginClass = require(pluginPath);
                    const plugin = new PluginClass();

                    await this.pluginManager.registerPlugin(plugin);

                    logger.info('Plugin recarregado com sucesso', { pluginName });

                } catch (error) {
                    logger.error('Erro ao recarregar plugin', {
                        pluginName,
                        error: error.message
                    });
                }
            }

            sleep(ms) {
                return new Promise(resolve => setTimeout(resolve, ms));
            }
        }
        ```

        #### 14.5.2. Plugin Development Tools
        ```js
        class PluginDevelopmentTools {
            constructor(pluginManager) {
                this.pluginManager = pluginManager;
            }

            // Valida plugin sem registrá-lo
            validatePlugin(pluginPath) {
                try {
                    delete require.cache[require.resolve(pluginPath)];
                    const PluginClass = require(pluginPath);
                    const plugin = new PluginClass();
                    
                    return this.pluginManager.validatePlugin(plugin);
                } catch (error) {
                    return {
                        valid: false,
                        error: error.message
                    };
                }
            }

            // Testa plugin isoladamente
            async testPlugin(pluginName, testMessage) {
                const plugin = this.pluginManager.plugins.get(pluginName);
                if (!plugin) {
                    throw new Error(`Plugin não encontrado: ${pluginName}`);
                }

                const testContext = {
                    isTest: true,
                    timestamp: new Date()
                };

                try {
                    // Testa beforeMessage
                    const processedMessage = await plugin.instance.beforeMessage(testMessage, testContext);
                    
                    // Simula resposta
                    const mockResponse = 'Resposta de teste';
                    
                    // Testa afterMessage
                    const finalResponse = await plugin.instance.afterMessage(processedMessage, mockResponse, testContext);

                    return {
                        success: true,
                        processedMessage,
                        finalResponse,
                        context: testContext
                    };

                } catch (error) {
                    return {
                        success: false,
                        error: error.message,
                        context: testContext
                    };
                }
            }

            // Gera template de plugin
            generatePluginTemplate(name, description) {
                const template = `const { PluginInterface } = require('../src/pluginSystem');

        class ${name}Plugin extends PluginInterface {
            constructor() {
                super();
                this.name = '${name.toLowerCase()}';
                this.version = '1.0.0';
                this.description = '${description}';
                this.author = 'Seu Nome';
                this.dependencies = [];
                this.config = {
                    enabled: true,
                    // Adicione suas configurações aqui
                };
            }

            async onLoad() {
                console.log('Plugin ${name} carregado');
            }

            async onEnable() {
                console.log('Plugin ${name} habilitado');
            }

            async onDisable() {
                console.log('Plugin ${name} desabilitado');
            }

            async beforeMessage(message, context) {
                // Processa mensagem antes da IA
                return message;
            }

            async afterMessage(message, response, context) {
                // Processa resposta após a IA
                return response;
            }

            async onMessageError(message, error, context) {
                // Trata erros de mensagem
                console.error('Erro no plugin ${name}:', error);
            }
        }

        module.exports = ${name}Plugin;`;

                return template;
            }

            // Analisa dependências de plugins
            analyzeDependencies() {
                const plugins = this.pluginManager.listPlugins();
                const dependencyGraph = new Map();
                const circularDependencies = [];

                // Constrói grafo de dependências
                for (const plugin of plugins) {
                    const dependencies = plugin.dependencies || [];
                    dependencyGraph.set(plugin.name, dependencies);
                }

                // Detecta dependências circulares
                for (const [plugin, dependencies] of dependencyGraph) {
                    const visited = new Set();
                    const recursionStack = new Set();
                    
                    if (this.hasCircularDependency(plugin, dependencyGraph, visited, recursionStack)) {
                        circularDependencies.push(plugin);
                    }
                }

                return {
                    totalPlugins: plugins.length,
                    dependencyGraph: Object.fromEntries(dependencyGraph),
                    circularDependencies,
                    hasCircularDependencies: circularDependencies.length > 0
                };
            }

            hasCircularDependency(plugin, graph, visited, recursionStack) {
                if (recursionStack.has(plugin)) {
                    return true;
                }

                if (visited.has(plugin)) {
                    return false;
                }

                visited.add(plugin);
                recursionStack.add(plugin);

                const dependencies = graph.get(plugin) || [];
                for (const dependency of dependencies) {
                    if (this.hasCircularDependency(dependency, graph, visited, recursionStack)) {
                        return true;
                    }
                }

                recursionStack.delete(plugin);
                return false;
            }
        }
        ```

        ### 14.6. Integração com o Bot

        #### 14.6.1. Configuração no bot.js
        ```js
        // src/bot.js - Integração com sistema de plugins
        const { pluginSystem } = require('./pluginSystem');
        const hotReloadManager = new HotReloadManager(pluginSystem);
        const devTools = new PluginDevelopmentTools(pluginSystem);

        // Inicializa sistema de plugins
        async function initializePlugins() {
            try {
                // Carrega plugins padrão
                await loadDefaultPlugins();
                
                // Carrega plugins customizados
                await loadCustomPlugins();
                
                // Inicia hot reload em desenvolvimento
                if (process.env.NODE_ENV === 'development') {
                    hotReloadManager.startWatching();
                }

                logger.info('Sistema de plugins inicializado', pluginSystem.getStats());

            } catch (error) {
                logger.error('Erro ao inicializar plugins', { error: error.message });
            }
        }

        async function loadDefaultPlugins() {
            const defaultPlugins = [
                new SentimentAnalysisHook(),
                new CommandDetectionHook(),
                new AutoResponseHook()
            ];

            for (const plugin of defaultPlugins) {
                await pluginSystem.registerPlugin(plugin);
            }
        }

        async function loadCustomPlugins() {
            const pluginDir = path.join(__dirname, '../plugins');
            
            if (!fs.existsSync(pluginDir)) {
                return;
            }

            const pluginFiles = fs.readdirSync(pluginDir)
                .filter(file => file.endsWith('.js'));

            for (const file of pluginFiles) {
                try {
                    const pluginPath = path.join(pluginDir, file);
                    const PluginClass = require(pluginPath);
                    const plugin = new PluginClass();
                    
                    await pluginSystem.registerPlugin(plugin);
                    
                } catch (error) {
                    logger.error('Erro ao carregar plugin customizado', {
                        file,
                        error: error.message
                    });
                }
            }
        }

        // Integra plugins no processamento de mensagens
        async function processMessageWithPlugins(message) {
            try {
                // Executa hooks beforeMessage
                let processedMessage = await pluginSystem.executeHook('beforeMessage', message, {
                    timestamp: new Date(),
                    userId: message.from
                });

                // Executa middleware pipeline
                const response = await pluginSystem.executeMiddleware(processedMessage, async (msg) => {
                    // Processamento normal da mensagem
                    return await processMessageNormally(msg);
                });

                // Executa hooks afterMessage
                const finalResponse = await pluginSystem.executeHook('afterMessage', processedMessage, response, {
                    timestamp: new Date(),
                    userId: message.from
                });

                return finalResponse;

            } catch (error) {
                // Executa hooks de erro
                await pluginSystem.executeHook('onMessageError', message, error, {
                    timestamp: new Date(),
                    userId: message.from
                });

                throw error;
            }
        }
        ```

        #### 14.6.2. Comandos de Administração para Plugins
        ```js
        // Comandos admin para gerenciar plugins
        case 'plugin':
            const pluginCommand = args[0];
            
            switch (pluginCommand) {
                case 'listar':
                    const plugins = pluginSystem.listPlugins();
                    let response = `🔌 *Plugins Registrados:*\n\n`;
                    
                    plugins.forEach(plugin => {
                        response += `• *${plugin.name}* ${plugin.enabled ? '🟢' : '🔴'}\n`;
                        response += `  Versão: ${plugin.version}\n`;
                        response += `  ${plugin.description}\n`;
                        response += `  Hooks: ${plugin.hooks.join(', ')}\n`;
                        response += `  Middleware: ${plugin.hasMiddleware ? 'Sim' : 'Não'}\n\n`;
                    });
                    
                    return response;
                    
                case 'info':
                    const pluginName = args[1];
                    if (!pluginName) return 'Especifique o nome do plugin';
                    
                    const plugin = pluginSystem.getPlugin(pluginName);
                    if (!plugin) return `Plugin não encontrado: ${pluginName}`;
                    
                    return `🔌 *Informações do Plugin:*\n\n` +
                        `• **Nome:** ${plugin.name}\n` +
                        `• **Versão:** ${plugin.version}\n` +
                        `• **Descrição:** ${plugin.description}\n` +
                        `• **Status:** ${plugin.enabled ? '🟢 Ativo' : '🔴 Inativo'}\n` +
                        `• **Hooks:** ${plugin.hooks.join(', ')}\n` +
                        `• **Middleware:** ${plugin.hasMiddleware ? 'Sim' : 'Não'}\n` +
                        `• **Registrado em:** ${plugin.registeredAt.toLocaleString()}`;
                        
                case 'habilitar':
                case 'desabilitar':
                    const targetPlugin = args[1];
                    if (!targetPlugin) return 'Especifique o nome do plugin';
                    
                    const enabled = pluginCommand === 'habilitar';
                    await pluginSystem.togglePlugin(targetPlugin, enabled);
                    
                    return `Plugin ${targetPlugin} ${enabled ? 'habilitado' : 'desabilitado'} com sucesso!`;
                    
                case 'recarregar':
                    const reloadPlugin = args[1];
                    if (!reloadPlugin) return 'Especifique o nome do plugin';
                    
                    await hotReloadManager.reloadPlugin(reloadPlugin);
                    return `Plugin ${reloadPlugin} recarregado com sucesso!`;
                    
                case 'testar':
                    const testPlugin = args[1];
                    if (!testPlugin) return 'Especifique o nome do plugin';
                    
                    const testMessage = { from: 'test@test.com', body: 'Mensagem de teste' };
                    const testResult = await devTools.testPlugin(testPlugin, testMessage);
                    
                    if (testResult.success) {
                        return `✅ *Teste do Plugin ${testPlugin}*\n\n` +
                            `• **Status:** Sucesso\n` +
                            `• **Mensagem processada:** ${testResult.processedMessage.body}\n` +
                            `• **Resposta final:** ${testResult.finalResponse}`;
                    } else {
                        return `❌ *Teste do Plugin ${testPlugin}*\n\n` +
                            `• **Status:** Falha\n` +
                            `• **Erro:** ${testResult.error}`;
                    }
                    
                case 'dependencias':
                    const analysis = devTools.analyzeDependencies();
                    
                    let depResponse = `📊 *Análise de Dependências:*\n\n` +
                                    `• **Total de plugins:** ${analysis.totalPlugins}\n` +
                                    `• **Dependências circulares:** ${analysis.circularDependencies.length}\n\n`;
                    
                    if (analysis.hasCircularDependencies) {
                        depResponse += `⚠️ *Dependências Circulares Detectadas:*\n`;
                        analysis.circularDependencies.forEach(plugin => {
                            depResponse += `• ${plugin}\n`;
                        });
                    } else {
                        depResponse += `✅ Nenhuma dependência circular encontrada`;
                    }
                    
                    return depResponse;
                    
                default:
                    return `🔌 *Comandos de Plugin:*\n\n` +
                        `• /plugin listar - Lista todos os plugins\n` +
                        `• /plugin info [nome] - Informações do plugin\n` +
                        `• /plugin habilitar [nome] - Habilita plugin\n` +
                        `• /plugin desabilitar [nome] - Desabilita plugin\n` +
                        `• /plugin recarregar [nome] - Recarrega plugin\n` +
                        `• /plugin testar [nome] - Testa plugin\n` +
                        `• /plugin dependencias - Analisa dependências`;
            }
        ```

        ---

        ## 15. Próximos Blocos

        - **Sistema de Áudio**: Integração Python, estratégias de melhoria, timeouts, fallbacks
        - **Tratamento de Erros**: Propagação, auto-recuperação, circuit breaker
        - **Testes**: Unitários, integração, stress, coverage
        - **DevOps**: CI/CD, observabilidade

        ---

        ## 16. Sistema de Áudio: Integração Python, Estratégias, Timeouts, Fallbacks

        ### 16.1. Arquitetura do Sistema de Áudio

        O OrbitBot possui um subsistema de áudio altamente robusto, projetado para processar mensagens de voz do WhatsApp de ponta a ponta.

        ### Fluxo de Processamento de Áudio

        ```mermaid
        graph TD
            A[Áudio Recebido] --> B[Decriptação Venom]
            B --> C[Validação Buffer]
            C --> D[Geração Hash MD5]
            D --> E[Salva Arquivo]
            E --> F[Chama Python Script]
            F --> G[Transcrição Whisper]
            G --> H{Sucesso?}
            H -->|Sim| I[Melhoria com IA]
            H -->|Não| J[Fallback/Retry]
            I --> K[Processa como Texto]
            J --> L[Erro ao Usuário]
            K --> M[Atualiza Histórico]
        ```

        ### Estratégias de Melhoria com IA

        ```mermaid
        graph LR
            A[Transcrição Original] --> B[Estratégia 1: Ultra Preciso]
            A --> C[Estratégia 2: Contextual]
            A --> D[Estratégia 3: Conservador]
            
            B --> E[Avaliação Qualidade]
            C --> E
            D --> E
            
            E --> F[Seleciona Melhor]
            F --> G[Texto Final]
        ```

        ### Integração Node.js ↔ Python

        ```mermaid
        sequenceDiagram
            participant N as Node.js
            participant P as Python
            participant W as Whisper
            participant O as OpenAI
            
            N->>P: execFile('transcrever_audio.py')
            P->>W: Carrega modelo
            W->>W: Processa áudio
            W->>P: Retorna transcrição
            P->>N: stdout (texto + confiança)
            
            N->>O: Melhoria com IA
            O->>N: Texto melhorado
            N->>N: Processa resposta
        ```

        ### 16.2. Estratégias de Melhoria de Transcrição

        - **Ultra Precise**: Prompt rígido, corrige só o que tem 100% de certeza, contexto de negócio, exemplos reais.
        - **Contextual Aggressive**: Prompt mais flexível, usa contexto do cliente (restaurante, bar, etc), expande gírias e regionalismos.
        - **Conservative Enhanced**: Corrige apenas erros óbvios, mantém o máximo do original.
        - **Fallback**: Se todas falharem, retorna transcrição bruta.

        #### Exemplo de Prompt Ultra Precise
        ```plaintext
        Você é um corretor ULTRA PRECISO de transcrições, focado em QUALIDADE MÁXIMA.
        CONTEXTO: RESTAURANTE
        - Cliente provavelmente está perguntando sobre cardápio, preços, horários, delivery
        REGRAS ULTRA RÍGIDAS PARA QUALIDADE MÁXIMA:
        1️⃣ CORREÇÕES OBRIGATÓRIAS (100% de certeza):
        "Kardapio de hoje?" → "Cardápio de hoje?"
        Agora melhore esta transcrição para QUALIDADE MÁXIMA: "Kardapio de hoje?"
        Responda APENAS com a versão melhorada.
        ```

        ### 16.3. Algoritmo de Seleção de Estratégia

        ```js
        const improvementStrategies = [
            { name: 'ultra_precise', ... },
            { name: 'contextual_aggressive', ... },
            { name: 'conservative_enhanced', ... }
        ];
        let bestImprovement = transcription;
        let bestScore = 0;
        for (const strategy of improvementStrategies) {
            const improved = await improveWithStrategy(transcription, strategy);
            const score = evaluateImprovementQuality(transcription, improved, context);
            if (score > bestScore) {
                bestImprovement = improved;
                bestScore = score;
            }
        }
        ```

        ### 16.4. Timeouts, Retries e Fallbacks

        - **Timeouts**: Cada etapa (decriptação, transcrição, melhoria) tem timeout próprio.
        - **Retries**: Até 5 tentativas com modelos diferentes (large → medium → small → tiny).
        - **Fallback**: Se todas falharem, envia mensagem padrão ao usuário.
        - **Mensagens de erro amigáveis**: Sempre informa o usuário em caso de falha.

        ### 16.5. Edge Cases e Resiliência

        - Áudio vazio ou corrompido: erro amigável, não trava o bot.
        - Transcrição vazia: fallback para resposta padrão.
        - Falha na IA de melhoria: retorna transcrição bruta.
        - Áudio duplicado: detectado por hash, evita processamento duplo.
        - Áudio muito longo: rejeitado com mensagem clara.

        ### 16.6. Métricas e Monitoramento

        - **totalProcessed**: Total de áudios processados
        - **successfulTranscriptions**: Sucessos
        - **failedTranscriptions**: Falhas
        - **averageProcessingTime**: Tempo médio
        - **confidenceScores**: Score médio de confiança
        - **improvementRate**: % de transcrições melhoradas
        - **contextDetectionRate**: % de contexto detectado
        - **successRate**: % de sucesso geral

        #### Exemplo de Métricas
        ```js
        {
            totalProcessed: 120,
            successfulTranscriptions: 110,
            failedTranscriptions: 10,
            averageProcessingTime: 8500,
            avgConfidence: 0.92,
            improvementRate: '78.2%',
            contextDetectionRate: '65.0%',
            successRate: '91.7%'
        }
        ```

        ### 16.7. Troubleshooting e Diagnóstico

        - **Áudio não transcreve**: Verifique Python, chave OpenAI, formato do arquivo.
        - **Transcrição ruim**: Teste diferentes modelos, ajuste prompts de melhoria.
        - **Timeouts frequentes**: Reduza tamanho do áudio, aumente timeout, otimize hardware.
        - **Falha na melhoria IA**: Verifique chave OpenRouter, limite de requisições.
        - **Logs detalhados**: Sempre consultar logs para rastrear falhas.

        ---

        ## 17. Tratamento de Erros: Propagação, Auto-Recuperação, Circuit Breaker

        ### 17.1. Filosofia de Resiliência

        O OrbitBot foi projetado para **nunca travar** diante de falhas. O tratamento de erros é feito em múltiplas camadas:
        - **Propagação controlada**: Erros são propagados até o ponto de captura mais seguro.
        - **Logs detalhados**: Todo erro relevante é logado com stack trace, contexto e métricas.
        - **Auto-recuperação**: Estratégias automáticas para retentar, isolar ou contornar falhas.
        - **Circuit Breaker**: Protege integrações externas e previne cascata de falhas.
        - **Fallbacks inteligentes**: Sempre há uma resposta para o usuário, mesmo em caso de falha grave.

        ### 17.2. Propagação e Captura de Erros

        - **Try/Catch em todos os handlers assíncronos** (fila, áudio, IA, plugins)
        - **Eventos de erro**: Fila e plugins emitem eventos de erro para monitoramento centralizado
        - **Erro nunca é silenciado**: Se não tratado, é logado e retorna fallback

        #### Exemplo: Handler de Mensagem
        ```js
        try {
            await processMessage(message);
        } catch (err) {
            logger.error('Erro no processamento', { 
                error: err.message,
                stack: err.stack,
                from: message.from
            });
            performanceMonitor.addError();
            await client.sendText(message.from, 'Estou tendo dificuldades técnicas. Por favor, tente novamente.');
        }
        ```

        ### 17.3. Auto-Recuperação

        - **Retries automáticos**: Fila e transcrição de áudio implementam retries exponenciais
        - **Limpeza de recursos**: Após erro, recursos temporários são liberados (áudios, conexões)
        - **Fallbacks**: Se todas as tentativas falharem, resposta padrão é enviada
        - **Monitoramento de saúde**: Plugins e integrações expõem hooks `onHealthCheck`

        #### Exemplo: Retry com Backoff
        ```js
        for (let attempt = 0; attempt < maxRetries; attempt++) {
            try {
                return await executarOperacao();
            } catch (err) {
                if (attempt === maxRetries - 1) throw err;
                await sleep(1000 * Math.pow(2, attempt)); // Backoff exponencial
            }
        }
        ```

        ### 17.4. Circuit Breaker

        - **Protege APIs externas** (OpenRouter, OpenAI, plugins externos)
        - **Estados**: CLOSED (normal), OPEN (falha), HALF_OPEN (teste)
        - **Thresholds configuráveis**: Número de falhas, tempo de reset
        - **Evita sobrecarga e looping de falhas**

        #### Exemplo: Circuit Breaker
        ```js
        class CircuitBreaker {
            constructor(failureThreshold = 5, resetTimeout = 60000) {
                this.failureThreshold = failureThreshold;
                this.resetTimeout = resetTimeout;
                this.failureCount = 0;
                this.lastFailureTime = null;
                this.state = 'CLOSED';
            }
            async execute(fn) {
                if (this.state === 'OPEN') {
                    if (Date.now() - this.lastFailureTime > this.resetTimeout) {
                        this.state = 'HALF_OPEN';
                    } else {
                        throw new Error('Circuit breaker is OPEN');
                    }
                }
                try {
                    const result = await fn();
                    this.onSuccess();
                    return result;
                } catch (error) {
                    this.onFailure();
                    throw error;
                }
            }
            onSuccess() {
                this.failureCount = 0;
                this.state = 'CLOSED';
            }
            onFailure() {
                this.failureCount++;
                this.lastFailureTime = Date.now();
                if (this.failureCount >= this.failureThreshold) {
                    this.state = 'OPEN';
                }
            }
        }
        ```

        ### 17.5. Edge Cases e Isolamento

        - **Plugin com erro**: Isolado, não afeta fluxo principal
        - **Erro em transcrição**: Não bloqueia fila, usuário recebe mensagem clara
        - **Erro em backup/restore**: Loga e mantém banco atual intacto
        - **Erro em hooks**: Só propaga se for crítico (beforeMessage/afterMessage), senão apenas loga

        ### 17.6. Métricas de Erro

        - **errorCount**: Total de erros
        - **errorRate**: % de erros por operação
        - **errorTypes**: Tipos de erro mais comuns
        - **lastError**: Último erro detalhado
        - **autoRecoveryCount**: Quantas vezes auto-recuperação foi acionada

        #### Exemplo de Métricas
        ```js
        {
            errorCount: 12,
            errorRate: 2.1,
            errorTypes: { 'TimeoutError': 5, 'APIError': 4, 'PluginError': 3 },
            lastError: { message: 'Timeout na transcrição', stack: '...' },
            autoRecoveryCount: 7
        }
        ```

        ### 17.7. Troubleshooting e Diagnóstico

        - **Erro recorrente**: Verifique logs detalhados, stack trace, contexto
        - **Falha em API externa**: Cheque status do serviço, chave, limites de uso
        - **Circuit breaker aberto**: Aguarde reset automático ou aumente thresholds
        - **Erro em plugin**: Desabilite plugin problemático e reative após correção
        - **Fila travada**: Limpe fila, reinicie bot, verifique deadlocks

        ---

        ## 18. Testes: Unitários, Integração, Stress, Coverage

        ### 18.1. Arquitetura de Testes do OrbitBot

        O OrbitBot adota uma arquitetura de testes inspirada na **pirâmide de testes moderna**:

        - **Testes Unitários**: Cobrem funções puras, helpers, lógica de negócio isolada (ex: validação, manipulação de strings, algoritmos de fila, prompts de IA).
        - **Testes de Integração**: Validam a interação entre módulos (ex: bot.js ↔ audioProcessor.js, pluginSystem.js ↔ plugins, database/db.js ↔ backup.js).
        - **Testes End-to-End (E2E)**: Simulam o ciclo completo (ex: mensagem WhatsApp → resposta IA → histórico atualizado → backup).
        - **Testes de Stress e Performance**: Avaliam robustez sob carga, concorrência, filas grandes, falhas em cascata.
        - **Cobertura de Código (Coverage)**: Monitorada e reportada em cada pipeline CI.

        #### Pirâmide de Testes
        ```mermaid
        graph TD
            A[Testes Unitários]
            B[Testes de Integração]
            C[Testes E2E]
            D[Testes de Stress]
            A --> B --> C --> D
        ```

        ### 18.2. Ferramentas e Frameworks

        - **Jest**: Testes unitários e integração (mocking avançado, snapshots, spies)
        - **Supertest**: Testes de APIs HTTP (se houver endpoints)
        - **Sinon**: Mocks, spies, stubs para dependências externas
        - **sqlite3-mock**: Mock de banco SQLite para testes isolados
        - **Artillery/Autocannon**: Stress e load testing
        - **nyc (Istanbul)**: Cobertura de código
        - **CI/CD**: GitHub Actions, integração automática dos testes

        ### 18.3. Estratégias de Teste

        #### 18.3.1. Testes Unitários
        - Funções puras: validação, manipulação de dados, algoritmos de fila, prompts
        - Mocks para dependências (ex: logger, banco, APIs externas)
        - Testes de edge cases: entradas inválidas, limites, erros esperados

        #### Exemplo: Teste Unitário de Algoritmo de Fila
        ```js
        describe('MessageQueue', () => {
        it('deve processar mensagens em ordem FIFO', async () => {
            const queue = new MessageQueue();
            const processOrder = [];
            queue.on('process', async (msg) => processOrder.push(msg));
            queue.addMessage('A');
            queue.addMessage('B');
            await queue.processQueue();
            expect(processOrder).toEqual(['A', 'B']);
        });
        });
        ```

        #### 18.3.2. Testes de Integração
        - Integração bot.js ↔ audioProcessor.js: simula mensagem de áudio, verifica transcrição e resposta
        - Integração pluginSystem.js ↔ plugins: ativa/desativa plugins, verifica hooks, middleware, isolamento de falhas
        - Integração database/db.js ↔ backup.js: cria backup, restaura, verifica integridade

        #### Exemplo: Teste de Integração de Plugin
        ```js
        describe('PluginSystem', () => {
        it('deve executar hooks beforeMessage e afterMessage', async () => {
            const pluginSystem = new PluginSystem();
            const mockPlugin = {
            name: 'mock', version: '1.0', description: '',
            async beforeMessage(msg) { msg.before = true; return msg; },
            async afterMessage(msg, resp) { return resp + ' [after]'; }
            };
            await pluginSystem.registerPlugin(mockPlugin);
            const msg = await pluginSystem.executeHook('beforeMessage', { body: 'oi' });
            expect(msg.before).toBe(true);
            const resp = await pluginSystem.executeHook('afterMessage', msg, 'resp');
            expect(resp).toContain('[after]');
        });
        });
        ```

        #### 18.3.3. Testes End-to-End (E2E)
        - Simulação completa: mensagem WhatsApp → IA → resposta → histórico → backup
        - Mocks de APIs externas (OpenAI, OpenRouter) para cenários de sucesso/falha
        - Testes de comandos admin, plugins, fila cheia, áudios inválidos

        #### Exemplo: Teste E2E de Mensagem de Áudio
        ```js
        describe('E2E: Mensagem de Áudio', () => {
        it('deve transcrever, melhorar e responder áudio', async () => {
            // Mock venom-bot, audioProcessor, openai
            // Simula envio de áudio, verifica resposta final e histórico
        });
        });
        ```

        #### 18.3.4. Testes de Stress e Performance
        - Enfileiramento massivo: 1000+ mensagens simultâneas
        - Teste de concorrência: múltiplos usuários, grupos
        - Falhas em cascata: simula timeouts, falhas de API, circuit breaker
        - Medição de throughput, latência, uso de memória

        #### Exemplo: Stress Test com Artillery
        ```yaml
        config:
        target: "http://localhost:3000"
        phases:
            - duration: 60
            arrivalRate: 50
        scenarios:
        - flow:
            - post:
                url: "/mensagem"
                json:
                    from: "5554999999999@c.us"
                    body: "Teste de stress"
        ```

        ### 18.4. Cobertura de Código (Coverage)

        - **nyc (Istanbul)**: Gera relatórios HTML, lcov, cobertura por arquivo/função/linha/branch
        - **Meta de cobertura**: 90%+ para core, 80%+ para integrações, 100% para helpers críticos
        - **Cobertura CI**: Bloqueia merge se cobertura cair abaixo do mínimo

        #### Exemplo: Execução de Coverage
        ```bash
        npx nyc --reporter=html --reporter=text npm test
        ```

        ### 18.5. Automação e Integração Contínua (CI)

        - **GitHub Actions**: Pipeline executa lint, testes, coverage, build a cada push/PR
        - **Testes paralelos**: Jobs independentes para unitário, integração, E2E
        - **Relatórios automáticos**: Cobertura, falhas, logs anexados ao PR
        - **Fail Fast**: Pipeline falha rápido em caso de erro crítico

        #### Exemplo: Workflow CI (trecho)
        ```yaml
        jobs:
        test:
            runs-on: ubuntu-latest
            steps:
            - uses: actions/checkout@v3
            - uses: actions/setup-node@v3
                with: { node-version: '18' }
            - run: npm ci
            - run: npm run lint
            - run: npm test
            - run: npx nyc npm test
            - uses: actions/upload-artifact@v3
                with:
                name: coverage-report
                path: coverage/
        ```

        ### 18.6. Edge Cases e Anti-Pattern Detection

        - **Mocks excessivos**: Preferir integração real quando possível
        - **Testes frágeis**: Evitar dependência de dados mutáveis, usar fixtures
        - **Cobertura falsa**: Cobertura alta sem asserts reais
        - **Testes lentos**: Isolar testes de stress, marcar como `slow`
        - **Race conditions**: Testar concorrência, deadlocks, starvation

        ### 18.7. Troubleshooting e Diagnóstico de Testes

        - **Testes intermitentes**: Verificar dependências externas, uso de clock fake, await correto
        - **Falha só no CI**: Checar variáveis de ambiente, permissões, mocks de sistema
        - **Cobertura baixa**: Rodar `npx nyc report --check-coverage`, identificar arquivos críticos
        - **Testes de stress travando**: Monitorar uso de CPU/memória, ajustar limites, usar profiling

        ### 18.8. Métricas e Melhores Práticas

        - **Tempo médio de execução dos testes**
        - **% de testes que falham por tipo**
        - **Cobertura por módulo**
        - **Testes obrigatórios para PRs críticos**
        - **Documentação dos cenários de teste**
        - **Revisão de testes por engenheiro sênior**

        ---

        ## 19. DevOps: CI/CD, Observabilidade, Monitoramento

        ### 19.1. Arquitetura de CI/CD do OrbitBot

        O OrbitBot adota uma abordagem DevOps moderna, com foco em **entregas contínuas, automação total e observabilidade avançada**. O pipeline CI/CD é projetado para garantir:

        - **Builds reprodutíveis** (Node.js + Python)
        - **Testes automatizados** (unitários, integração, stress)
        - **Linting e análise estática**
        - **Deploy automatizado** (produção e staging)
        - **Rollback seguro**
        - **Integração com monitoramento e métricas**

        #### 19.1.1. Pipeline CI/CD (Exemplo com GitHub Actions)

        ```yaml
        name: CI/CD Pipeline
        on:
        push:
            branches: [main]
        pull_request:
            branches: [main]

        jobs:
        build:
            runs-on: ubuntu-latest
            steps:
            - uses: actions/checkout@v3
            - name: Setup Node.js
                uses: actions/setup-node@v3
                with:
                node-version: '18.x'
            - name: Setup Python
                uses: actions/setup-python@v4
                with:
                python-version: '3.10'
            - name: Install dependencies (Node)
                run: npm install
            - name: Install dependencies (Python)
                run: pip install openai
            - name: Lint
                run: npm run lint || true
            - name: Run Tests
                run: npm test
            - name: Build
                run: npm run build || echo "No build step"

        deploy:
            needs: build
            runs-on: ubuntu-latest
            if: github.ref == 'refs/heads/main'
            steps:
            - uses: actions/checkout@v3
            - name: Deploy to Production
                run: |
                ssh ${{ secrets.SSH_USER }}@${{ secrets.SSH_HOST }} 'cd /var/www/orbitbot && git pull && pm2 restart all'
        ```

        #### 19.1.2. Estratégias de Rollback
        - Deploys são versionados (tag/commit)
        - Rollback via `git checkout <commit anterior>` + `pm2 restart all`
        - Backups automáticos do banco antes de cada deploy
        - Alertas automáticos em caso de falha no deploy

        #### 19.1.3. Integração com Monitoramento
        - Webhooks para enviar status do deploy para Slack/Discord
        - Notificações de falha/sucesso
        - Exportação de métricas para Prometheus/Grafana (custom)

        #### 19.1.4. Observabilidade, Métricas, Tracing e Troubleshooting

        A observabilidade do OrbitBot é projetada para **detecção proativa de falhas, análise de performance e rastreabilidade total**. O sistema utiliza múltiplas camadas:

        - **Logging estruturado** (JSON, stack trace, contexto)
        - **Métricas customizadas** (performance, fila, áudio, IA, banco)
        - **Tracing distribuído** (correlação de eventos entre módulos)
        - **Alertas automáticos** (thresholds, erros críticos, anomalias)
        - **Dashboards em tempo real** (Prometheus, Grafana, Loki)

        ##### 19.1.4.1. Logging Estruturado
        - Todos os logs seguem padrão estruturado (nível, timestamp, contexto, stack)
        - Logs críticos são enviados para sistemas externos (ex: Loki, ELK)
        - Exemplo de log:
        ```json
        {
        "level": "error",
        "timestamp": "2025-06-20T12:34:56Z",
        "module": "audioProcessor",
        "message": "Erro ao transcrever áudio",
        "userId": 42,
        "audioHash": "abc123",
        "stack": "..."
        }
        ```

        ##### 19.1.4.2. Métricas Customizadas
        - **Performance:** tempo de resposta, uso de memória, CPU, uptime
        - **Fila:** tamanho, tempo médio, retries, deadlocks
        - **Áudio:** áudios processados, taxa de sucesso, tempo médio, melhorias IA
        - **IA:** chamadas, latência, taxa de erro, cache hits
        - **Banco:** queries por segundo, locks, tamanho, backups

        Exemplo de exportação Prometheus:
        ```prometheus
        # HELP orbitbot_audio_processing_time_ms Tempo médio de processamento de áudio
        # TYPE orbitbot_audio_processing_time_ms gauge
        orbitbot_audio_processing_time_ms 1234
        ```

        ##### 19.1.4.3. Tracing Distribuído
        - Cada request recebe um traceId único
        - Eventos correlacionados entre módulos (ex: fila → IA → áudio → resposta)
        - Possível integração com OpenTelemetry

        ##### 19.1.4.4. Dashboards e Queries
        - **Grafana:** dashboards de performance, fila, áudio, IA, banco
        - **Prometheus:** queries para alertas e análise histórica
        - Exemplos:
        - `sum(rate(orbitbot_audio_processing_time_ms[5m])) by (model)`
        - `max(orbitbot_queue_size) by (hora)`
        - `count_over_time(orbitbot_errors_total[1h])`

        ##### 19.1.4.5. Alertas e Troubleshooting
        - Alertas automáticos para:
        - Erros críticos (ex: falha no deploy, transcrição, banco)
        - Latência acima do normal
        - Fila acumulada
        - Falha em backup automático
        - Troubleshooting:
        - Logs detalhados com contexto
        - Stack trace completo
        - Dump de variáveis críticas
        - Scripts de diagnóstico (ex: healthcheck, status, dump)

        ##### 19.1.4.6. Melhores Práticas DevOps
        - **Infraestrutura como código** (IaC)
        - **Deploys pequenos e frequentes**
        - **Rollback rápido e seguro**
        - **Monitoramento proativo**
        - **Testes automatizados em todos os estágios**
        - **Documentação viva do pipeline e métricas**

        ---

        ## 20. Implementação de Testes Automatizados

        ### 20.1. Configuração Inicial

        #### 20.1.1. Instalação de Dependências
        ```bash
        npm install --save-dev jest supertest @types/jest
        npm install --save-dev sqlite3-memory sqlite3-test-utils
        npm install --save-dev nock # Para mock de APIs externas
        npm install --save-dev faker # Para dados de teste
        ```

        #### 20.1.2. Configuração Jest (`jest.config.js`)
        ```javascript
        module.exports = {
        testEnvironment: 'node',
        setupFilesAfterEnv: ['<rootDir>/tests/setup.js'],
        testMatch: ['**/tests/**/*.test.js'],
        collectCoverageFrom: [
            'src/**/*.js',
            '!src/**/*.test.js',
            '!src/treinamento.js'
        ],
        coverageThreshold: {
            global: {
            branches: 80,
            functions: 80,
            lines: 80,
            statements: 80
            }
        },
        testTimeout: 30000
        };
        ```

        #### 20.1.3. Setup de Testes (`tests/setup.js`)
        ```javascript
        // Mock do venom-bot para testes
        jest.mock('venom-bot', () => ({
        create: jest.fn().mockResolvedValue({
            onMessage: jest.fn(),
            sendText: jest.fn().mockResolvedValue(true),
            decryptFile: jest.fn().mockResolvedValue(Buffer.from('fake-audio'))
        })
        }));

        // Mock de APIs externas
        jest.mock('axios');
        jest.mock('child_process');

        // Configuração de banco de teste em memória
        process.env.NODE_ENV = 'test';
        ```

        ### 20.2. Estrutura de Testes

        #### 20.2.1. Testes Unitários

        **`tests/unit/queue.test.js`**
        ```javascript
        const MessageQueue = require('../../src/queue');

        describe('MessageQueue', () => {
        let queue;

        beforeEach(() => {
            queue = new MessageQueue();
        });

        test('should add message to queue', () => {
            const message = { from: '123', body: 'test' };
            queue.addMessage(message);
            expect(queue.getQueueSize()).toBe(1);
        });

        test('should process messages in order', async () => {
            const messages = [];
            queue.on('process', (msg) => messages.push(msg));
            
            queue.addMessage({ from: '1', body: 'first' });
            queue.addMessage({ from: '2', body: 'second' });
            
            await new Promise(resolve => setTimeout(resolve, 100));
            expect(messages).toHaveLength(2);
            expect(messages[0].body).toBe('first');
        });

        test('should handle errors with retries', async () => {
            let errorCount = 0;
            queue.on('error', () => errorCount++);
            
            queue.on('process', () => {
            throw new Error('Test error');
            });
            
            queue.addMessage({ from: '123', body: 'test' });
            await new Promise(resolve => setTimeout(resolve, 500));
            
            expect(errorCount).toBeGreaterThan(0);
        });
        });
        ```

        **`tests/unit/audioProcessor.test.js`**
        ```javascript
        const AudioProcessor = require('../../src/audioProcessor');
        const { execFile } = require('child_process');

        jest.mock('child_process');

        describe('AudioProcessor', () => {
        let processor;

        beforeEach(() => {
            processor = new AudioProcessor();
            execFile.mockClear();
        });

        test('should validate audio buffer', async () => {
            const validBuffer = Buffer.from('fake-audio-data');
            await expect(processor.validateAudio(validBuffer)).resolves.toBe(true);
            
            await expect(processor.validateAudio(null)).rejects.toThrow('Áudio vazio');
        });

        test('should improve transcription with AI', async () => {
            const mockResponse = {
            data: {
                choices: [{ message: { content: 'Texto melhorado' } }]
            }
            };
            
            require('axios').post.mockResolvedValue(mockResponse);
            
            const result = await processor.improveTranscriptionWithAI('texto original');
            expect(result).toBe('Texto melhorado');
        });

        test('should handle transcription errors', async () => {
            execFile.mockImplementation((cmd, args, callback) => {
            callback(new Error('Transcription failed'));
            });
            
            await expect(processor.transcribeAudio('fake-path', 'hash')).rejects.toThrow();
        });
        });
        ```

        #### 20.2.2. Testes de Integração

        **`tests/integration/database.test.js`**
        ```javascript
        const sqlite3 = require('sqlite3').verbose();
        const dbOperations = require('../../database/db');

        describe('Database Integration', () => {
        let testDb;

        beforeAll(async () => {
            // Usa banco em memória para testes
            testDb = new sqlite3.Database(':memory:');
            await dbOperations.initDatabase(testDb);
        });

        afterAll(async () => {
            await testDb.close();
        });

        test('should create and retrieve client', async () => {
            const numero = '5511999999999';
            const cliente = await dbOperations.cadastrarCliente(numero);
            
            expect(cliente.numero).toBe(numero);
            expect(cliente.id).toBeDefined();
            
            const retrieved = await dbOperations.buscarCliente(numero);
            expect(retrieved.id).toBe(cliente.id);
        });

        test('should handle message history with pagination', async () => {
            const cliente = await dbOperations.cadastrarCliente('5511888888888');
            
            // Adiciona várias mensagens
            for (let i = 1; i <= 15; i++) {
            await dbOperations.adicionarMensagem(cliente.id, `Mensagem ${i}`, 'user');
            }
            
            const historico = await dbOperations.buscarHistorico(cliente.id, 1, 10);
            expect(historico.messages).toHaveLength(10);
            expect(historico.pagination.totalPages).toBe(2);
        });
        });
        ```

        **`tests/integration/ai.test.js`**
        ```javascript
        const { handleMessage } = require('../../src/openai');
        const { aiConfigManager } = require('../../src/aiConfig');

        describe('AI Integration', () => {
        beforeEach(() => {
            // Mock da API externa
            require('axios').post.mockResolvedValue({
            data: {
                choices: [{ message: { content: 'Resposta da IA' } }]
            }
            });
        });

        test('should generate response with context', async () => {
            const historico = [
            { mensagem: 'Oi', role: 'user' },
            { mensagem: 'Olá! Como posso ajudar?', role: 'assistant' }
            ];
            
            const response = await handleMessage(historico, 'Como vai?', 1);
            expect(response).toContain('Resposta da IA');
        });

        test('should handle personality changes', async () => {
            const personality = aiConfigManager.setPersonality('professional');
            expect(personality.name).toContain('Profissional');
            
            const config = aiConfigManager.generateConfig();
            expect(config.personality.name).toContain('Profissional');
        });
        });
        ```

        #### 20.2.3. Testes End-to-End

        **`tests/e2e/bot.test.js`**
        ```javascript
        const { startBot } = require('../../src/bot');
        const venom = require('venom-bot');

        describe('Bot E2E', () => {
        let client;

        beforeAll(async () => {
            // Inicia bot em modo teste
            client = await startBot();
        });

        afterAll(async () => {
            // Cleanup
        });

        test('should process text message end-to-end', async () => {
            const message = {
            from: '5511999999999@c.us',
            body: 'Oi, tudo bem?',
            type: 'text'
            };
            
            // Simula recebimento de mensagem
            client.emit('message', message);
            
            // Aguarda processamento
            await new Promise(resolve => setTimeout(resolve, 2000));
            
            // Verifica se resposta foi enviada
            expect(client.sendText).toHaveBeenCalledWith(
            message.from,
            expect.stringContaining('')
            );
        });

        test('should process audio message end-to-end', async () => {
            const audioMessage = {
            from: '5511999999999@c.us',
            type: 'audio',
            data: Buffer.from('fake-audio'),
            mimetype: 'audio/ogg'
            };
            
            client.emit('message', audioMessage);
            
            await new Promise(resolve => setTimeout(resolve, 5000));
            
            // Verifica se transcrição foi processada
            expect(client.sendText).toHaveBeenCalledWith(
            audioMessage.from,
            expect.stringContaining('Transcrição')
            );
        });
        });
        ```

        ### 20.3. Testes de Performance e Stress

        **`tests/performance/stress.test.js`**
        ```javascript
        const MessageQueue = require('../../src/queue');

        describe('Performance Tests', () => {
        test('should handle high message volume', async () => {
            const queue = new MessageQueue();
            const startTime = Date.now();
            
            // Adiciona 100 mensagens rapidamente
            for (let i = 0; i < 100; i++) {
            queue.addMessage({
                from: `user${i}@c.us`,
                body: `Message ${i}`
            });
            }
            
            await new Promise(resolve => setTimeout(resolve, 5000));
            
            const endTime = Date.now();
            expect(endTime - startTime).toBeLessThan(10000); // Máximo 10s
        });

        test('should maintain performance under load', async () => {
            const processor = new AudioProcessor();
            const startTime = Date.now();
            
            // Simula processamento de múltiplos áudios
            const promises = [];
            for (let i = 0; i < 10; i++) {
            promises.push(processor.processAudioMessage({}, {
                from: 'test@c.us',
                data: Buffer.from('fake-audio'),
                type: 'audio'
            }));
            }
            
            await Promise.all(promises);
            
            const endTime = Date.now();
            expect(endTime - startTime).toBeLessThan(30000); // Máximo 30s
        });
        });

        ---

        ## Conclusão

        ### Resumo Técnico

        O OrbitBot representa uma implementação robusta e profissional de um sistema de bot de WhatsApp, demonstrando práticas avançadas de engenharia de software. A arquitetura modular, combinada com processamento de áudio, IA generativa e banco de dados SQLite, resulta em uma solução completa e escalável.

        ### Principais Conquistas Técnicas

        **Arquitetura Modular**
        - Separação clara de responsabilidades entre módulos
        - Baixo acoplamento e alta coesão
        - Extensibilidade via sistema de plugins
        - Facilidade de manutenção e evolução

        **Processamento de Áudio Avançado**
        - Integração Node.js ↔ Python para transcrição
        - Múltiplas estratégias de melhoria com IA
        - Timeouts, retries e fallbacks robustos
        - Métricas detalhadas de performance

        **Sistema de IA Inteligente**
        - Personalidades compostas e dinâmicas
        - Contexto adaptativo automático
        - Integração com APIs externas (OpenRouter)
        - Cache e otimizações de performance

        **Banco de Dados Otimizado**
        - SQLite com transações atômicas
        - Índices para performance
        - Sistema de backup automático
        - Histórico completo sem limitações

        **Resiliência e Observabilidade**
        - Sistema de fila com retry exponencial
        - Logs estruturados e métricas em tempo real
        - Tratamento de erros em múltiplas camadas
        - Monitoramento proativo

        ### Métricas de Qualidade

        - **Performance**: 50% mais rápido que versões anteriores
        - **Memória**: 33% redução no uso de recursos
        - **Disco**: 80% redução em operações I/O
        - **Banco**: 70% redução no tamanho dos dados
        - **Cobertura**: Estrutura para 80%+ de testes automatizados
        - **Uptime**: Sistema resiliente com fallbacks automáticos

        ### Benefícios da Implementação

        **Para Desenvolvedores**
        - Código limpo e bem documentado
        - Arquitetura escalável e manutenível
        - Sistema de testes abrangente
        - Logs detalhados para debugging

        **Para Operações**
        - Monitoramento em tempo real
        - Backup automático e confiável
        - Rollback seguro e rápido
        - Métricas de performance contínuas

        **Para Usuários**
        - Respostas rápidas e naturais
        - Processamento de áudio preciso
        - Contexto mantido entre conversas
        - Experiência consistente e confiável

        ### Roadmap e Evolução

        **Curto Prazo**
        - Implementação completa de testes automatizados
        - Integração com Prometheus/Grafana
        - Pipeline CI/CD ativo
        - Documentação de APIs

        **Médio Prazo**
        - Suporte a múltiplos idiomas
        - Compressão de áudios
        - Cache de transcrições
        - Dashboard de administração

        **Longo Prazo**
        - Arquitetura distribuída
        - Microserviços
        - Machine Learning customizado
        - Integração com outros canais

        ### Considerações Finais

        O OrbitBot demonstra que é possível construir sistemas complexos e robustos seguindo princípios sólidos de engenharia de software. A combinação de arquitetura modular, processamento assíncrono, IA generativa e observabilidade avançada resulta em uma solução que não apenas atende às necessidades atuais, mas também está preparada para evolução futura.

        A documentação técnica apresentada serve como referência para desenvolvedores que desejam entender, manter ou estender o sistema, garantindo que o conhecimento seja preservado e compartilhado de forma eficaz.

        ---

        *Esta documentação técnica foi criada seguindo as melhores práticas de engenharia de software e representa o estado atual do OrbitBot em sua versão 2.0.*

