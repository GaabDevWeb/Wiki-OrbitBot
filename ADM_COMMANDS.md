# Comandos Administrativos do OrbitBot

Aqui estão sugestões de comandos administrativos para facilitar o gerenciamento do bot. Sinta-se livre para adaptar, remover ou adicionar conforme sua necessidade!

---

## 1. /restart
Reinicia o processo do bot.
- **Exemplo:** `/restart`
- Útil para aplicar atualizações rápidas ou resolver travamentos.

## 2. /shutdown
Desliga o bot de forma segura.
- **Exemplo:** `/shutdown`
- Pode pedir confirmação: `/shutdown confirmar`

## 3. /status
Mostra o status atual do bot (uptime, uso de memória, CPU, fila de mensagens, etc).
- **Exemplo:** `/status`

## 4. /logs [quantidade]
Exibe as últimas linhas dos logs do sistema.
- **Exemplo:** `/logs 20` (mostra as últimas 20 linhas)

## 5. /plugins [listar|ativar|desativar] [nome]
Gerencia plugins do sistema.
- **Exemplo:** `/plugins listar`, `/plugins ativar weather`, `/plugins desativar weather`

## 6. /config [chave] [valor]
Permite alterar configurações dinâmicas do bot sem reiniciar.
- **Exemplo:** `/config modo_manutencao on`

## 7. /users [listar|banir|desbanir] [número]
Gerencia usuários (banir/desbanir/listar usuários cadastrados).
- **Exemplo:** `/users listar`, `/users banir 5511999999999`

## 8. /metrics
Mostra métricas detalhadas de performance (mensagens processadas, erros, tempo médio de resposta, etc).
- **Exemplo:** `/metrics`

## 9. /broadcast [mensagem]
Envia uma mensagem para todos os usuários cadastrados.
- **Exemplo:** `/broadcast O bot passará por manutenção às 22h.`

## 10. /setadmin [número]
Adiciona ou remove um número da lista de administradores.
- **Exemplo:** `/setadmin 5511999999999`

## 11. /ban [número] [motivo]
Bane um usuário do bot, impedindo que ele envie mensagens.
- **Exemplo:** `/ban 5511999999999 spam`

## 12. /unban [número]
Remove o banimento de um usuário.
- **Exemplo:** `/unban 5511999999999`

## 13. /limparhistorico [número]
Limpa todo o histórico de mensagens de um usuário específico.
- **Exemplo:** `/limparhistorico 5511999999999`

## 14. /resetconfig
Restaura todas as configurações do bot para o padrão de fábrica.
- **Exemplo:** `/resetconfig confirmar`

## 15. /testarplugin [nome]
Executa um teste rápido em um plugin para verificar se está funcionando corretamente.
- **Exemplo:** `/testarplugin weather`

## 16. /debug [on/off]
Ativa ou desativa o modo debug, aumentando o nível de logs detalhados.
- **Exemplo:** `/debug on`

## 17. /set [chave] [valor]
Altera qualquer configuração dinâmica do bot (atalho para /config).
- **Exemplo:** `/set modo_manutencao off`

## 18. /info [número]
Mostra informações detalhadas sobre um usuário (data de cadastro, total de mensagens, status, etc).
- **Exemplo:** `/info 5511999999999`

## 19. /limparcache
Limpa todos os caches do bot (respostas, transcrições, etc).
- **Exemplo:** `/limparcache`

## 20. /forcarbackup [nome]
Força a criação de um backup manual imediatamente.
- **Exemplo:** `/forcarbackup nome_opcional`

## 21. /ajudaadm
Mostra todos os comandos administrativos disponíveis e exemplos de uso.
- **Exemplo:** `/ajudaadm`

---

**Dica:**
- Sempre peça confirmação para comandos destrutivos (ex: reset, shutdown, limpar histórico).
- Mantenha logs de todas as ações administrativas para auditoria. 